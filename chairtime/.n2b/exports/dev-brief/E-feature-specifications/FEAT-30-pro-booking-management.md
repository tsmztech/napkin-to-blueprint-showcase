# FEAT-30 — Pro Booking Management

This chapter covers Pro Booking Management (FEAT-30), a Core-tier feature. It carries 13 specifications carrying 190 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-30.SPEC-001 | Cancel Booking (Pro-Initiated) | screen | 16 |
| FEAT-30.SPEC-002 | Reschedule Booking (Pro-Initiated) | screen | 14 |
| FEAT-30.SPEC-003 | Goodwill Deposit Refund | screen | 15 |
| FEAT-30.SPEC-004 | Book Client In | screen | 16 |
| FEAT-30.SPEC-005 | Cancel Several Bookings at Once | screen | 15 |
| FEAT-30.SPEC-006 | Pro Booking Action Rules | logic-rule | 19 |
| FEAT-30.SPEC-007 | Pro Cancel/Reschedule Commit | automation | 16 |
| FEAT-30.SPEC-008 | Bulk Cancellation Commit | automation | 13 |
| FEAT-30.SPEC-009 | Goodwill Refund Commit | automation | 12 |
| FEAT-30.SPEC-010 | Pro-Created Booking & Deposit Request Hold | automation | 14 |
| FEAT-30.SPEC-011 | Goodwill & Bulk-Cancellation Refund Execution | integration | 14 |
| FEAT-30.SPEC-012 | Pro Action Client Notice | notification | 13 |
| FEAT-30.SPEC-013 | Deposit Request & Expiry Notice | notification | 13 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Pro Booking Management

## Summary

**Feature:** Pro Booking Management
**ID:** FEAT-30
**Description:** The Pro can cancel, reschedule, or refund any of their own bookings, and book a client in themselves (for example, rebooking a regular at the chair), with the deposit handled the same fair way every time and the client told what happened.
**Priority:** Core
**Phase:** MVP
**Type:** User-Facing
**Rationale:** BRIEF.md's Target Users & Roles states the Pro "can mark no-shows, reschedule, cancel and refund within policy." The draft gave the Pro a feature for marking no-shows (FEAT-11) but only referenced the other three actions in passing inside the client-side feature, and never defined what happens to a client's deposit when the Pro is the one who cancels. Core because it is load-bearing for the brief's correctness bar: a Pro who falls ill with a full day booked must be able to unwind every booking without losing a client's money or trust. The in-person rebooking path keeps BRIEF.md's Success Criterion -- "the link is the only way to book them" -- true even at the chair, because the client still pays their deposit through a link. MVP: pros face cancellations and rebookings from their first week. [AUDIT-ADDED: 1 -- Core: counterpart-symmetry walk found no feature owning Pro-caused cancellations, Pro reschedules, or goodwill refunds, all of which BRIEF.md lists as Pro capabilities; without it the Pro has no correct way to cancel on a client, which directly threatens the "never lose a deposit" success criterion]

**Key Capabilities:**
- Cancel a client's booking -- the client's deposit is refunded in full automatically, whatever the timing, and the client is told
- Reschedule a booking to another genuinely free time -- the deposit carries over and the client receives the new time with a fresh manage link
- Refund a deposit in full as goodwill -- for an inside-window cancellation or instead of marking a no-show
- Book a client in on their behalf -- choose the service and time and an existing or new client; the client receives a deposit request link (or scans it from the Pro's screen at the chair) and the slot is held until they pay
- Cancel several bookings at once -- when blocking off a day that already has bookings (FEAT-17)

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-30.SPEC-001 | Cancel Booking (Pro-Initiated) | Screen | The Pro | Pro views a single booking and confirms cancelling it, seeing plainly that the client's deposit will be refunded in full whatever the timing; also offers an outbound link to FEAT-21.SPEC-010 (recurring series management) |
| FEAT-30.SPEC-002 | Reschedule Booking (Pro-Initiated) | Screen | The Pro | Pro picks a new genuinely free time for a client's booking (inside the Pro's own notice/horizon exception), seeing that the deposit carries over and a fresh manage link will go to the client |
| FEAT-30.SPEC-003 | Goodwill Deposit Refund | Screen | The Pro | Pro confirms a full goodwill refund on a booking's deposit, reached from a no-show prompt, a dispute timeline, or a booking row, available any time before the booking completes |
| FEAT-30.SPEC-004 | Book Client In | Screen | The Pro | Pro chooses a service, a time, and an existing or new client to book the client in directly, then issues a held deposit request (link or on-screen scan code) instead of collecting payment inside this screen |
| FEAT-30.SPEC-005 | Cancel Several Bookings at Once | Screen | The Pro | Pro reviews the set of bookings a new time block conflicts with and confirms cancelling them together, seeing the full-refund outcome for each before confirming |
| FEAT-30.SPEC-006 | Pro Booking Action Rules | Logic/Rule | The Pro, The Client | The validation and eligibility rules shared across every Pro-initiated action: the Pro's notice/horizon exception, the deposit-request hold window, the once-only and until-completion refund limits, and the completed/no-show cancellation cutoff |
| FEAT-30.SPEC-007 | Pro Cancel/Reschedule Commit | Automation | The Pro, The Client | Commits a Pro-initiated single cancellation or reschedule to the Booking, coordinating the deposit outcome, calendar mirroring, activity logging, freed-slot handoff, the client notice this triggers, and (on a reschedule) a fresh manage link via FEAT-08.SPEC-010 |
| FEAT-30.SPEC-008 | Bulk Cancellation Commit | Automation | The Pro, The Client | Commits a Pro-initiated cancellation of several bookings at once, reporting a per-booking outcome and coordinating each booking's refund, calendar removal, and client notice |
| FEAT-30.SPEC-009 | Goodwill Refund Commit | Automation | The Pro, The Client | Processes a confirmed goodwill refund against a booking's deposit, independent of the cancellation window and available until the booking completes |
| FEAT-30.SPEC-010 | Pro-Created Booking & Deposit Request Hold | Automation | The Pro, The Client | Creates the pending booking from a Pro-entered service/time/client, triggers the slot hold (up to 24 hours or until 2 hours before the appointment, via FEAT-03.SPEC-007), confirms the booking on payment, and relies on FEAT-03.SPEC-007 (sole writer of Expired (unpaid)) if it is not paid in time |
| FEAT-30.SPEC-011 | Goodwill & Bulk-Cancellation Refund Execution | Integration | The Pro, The Client | Requests each goodwill or bulk-cancellation refund from the payment-processing capability, drawing on the Pro's payout account, and reports back any refund that cannot complete immediately for retry |
| FEAT-30.SPEC-012 | Pro Action Client Notice | Notification | The Client | Trigger-and-audience contract for the client's notice that the Pro cancelled (with refund status), rescheduled (with the new time and a fresh manage link), or issued a goodwill refund; content is owned by FEAT-08.SPEC-004, and the Pro-side counterpart by FEAT-08.SPEC-005 |
| FEAT-30.SPEC-013 | Deposit Request & Expiry Notice | Notification | The Client, The Pro | Delivers a Pro-created deposit request to the client (by text, by email, or as an on-screen code to scan) and stops pending delivery when the request expires unpaid; the Pro's expiry notice is triggered by FEAT-03.SPEC-007 and its content owned by FEAT-08.SPEC-006 |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Cancel a client's booking -- the client's deposit is refunded in full automatically, whatever the timing, and the client is told | FEAT-30.SPEC-001, FEAT-30.SPEC-007, FEAT-30.SPEC-012 | SPEC-001 is the confirmation screen; SPEC-007 commits the cancellation and hands the full-refund determination to FEAT-09's outcome evaluation (XBR-09: any Pro cancellation refunds in full); SPEC-012 tells the client | Phase 2 (Explicit) |
| Reschedule a booking to another genuinely free time -- the deposit carries over and the client receives the new time with a fresh manage link | FEAT-30.SPEC-002, FEAT-30.SPEC-007, FEAT-30.SPEC-012 | SPEC-002 is the new-time selection screen, re-using FEAT-03's live availability with the Pro's notice/horizon exception (SPEC-006); SPEC-007 commits the reschedule (deposit carries over automatically, per XBR-09, since a Pro-made reschedule never exposes the client to the window); SPEC-007 requests the fresh manage link from FEAT-08.SPEC-010; SPEC-012 triggers the client notice (content owned by FEAT-08.SPEC-004) carrying the new time and that link | Phase 2 (Explicit) |
| Refund a deposit in full as goodwill -- for an inside-window cancellation or instead of marking a no-show | FEAT-30.SPEC-003, FEAT-30.SPEC-009, FEAT-30.SPEC-011, FEAT-30.SPEC-012 | SPEC-003 is the confirmation screen; SPEC-009 processes the refund (governed by SPEC-006's once-only, until-completion limits); SPEC-011 executes it through the payment-processing capability; SPEC-012 tells the client | Phase 2 (Explicit) |
| Book a client in on their behalf -- choose the service and time and an existing or new client; the client receives a deposit request link (or scans it from the Pro's screen at the chair) and the slot is held until they pay | FEAT-30.SPEC-004, FEAT-30.SPEC-010, FEAT-30.SPEC-013 | SPEC-004 is the selection screen (service, time, client); SPEC-010 creates the pending booking, holds the slot, and resolves it on payment or expiry; SPEC-013 delivers the deposit request by the right channel or on-screen code (the Pro's expiry notice is FEAT-03.SPEC-007 / FEAT-08.SPEC-006) | Phase 2 (Explicit) |
| Cancel several bookings at once -- when blocking off a day that already has bookings (FEAT-17) | FEAT-30.SPEC-005, FEAT-30.SPEC-008, FEAT-30.SPEC-011, FEAT-30.SPEC-012 | SPEC-005 is the review-and-confirm screen for the bookings a new time block conflicts with; SPEC-008 commits each cancellation and reports a per-booking outcome; SPEC-011 executes the resulting refunds; SPEC-012 notifies each affected client | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-30.SPEC-006 | Pro Booking Action Rules | Phase 5 (Rule Discovery) | The Validation & Limits field states five distinct rules (notice/horizon exception, 24-hour/2-hour hold window, once-only refund, until-completion goodwill availability, no cancelling a completed/auto-completed booking) that govern all five screens and four automations alike -- past the inline threshold and requiring one shared spec so every other spec references, rather than restates, the same limits |
| FEAT-30.SPEC-007 | Pro Cancel/Reschedule Commit | Phase 4 (Trigger-Response) | Confirming a Pro-initiated cancel or reschedule triggers cross-entity, cross-feature processing (deposit-outcome handoff to FEAT-09, calendar mirroring, activity logging, freed-slot/waitlist handoff, client notice) that exceeds a simple inline data write -- mirroring the Booking entity's High-contention, reject-with-refresh resolution recorded in the dependency map |
| FEAT-30.SPEC-008 | Bulk Cancellation Commit | Phase 4 (Trigger-Response) / Phase 6 (Failure Analysis) | The States field's requirement that "a multi-booking cancellation reports the outcome for each booking" and the Sick Day journey's Failure/Recovery Variant (one refund cannot complete yet) describe per-item outcome tracking across several Bookings at once -- a distinct processing shape from the single-booking commit in SPEC-007 |
| FEAT-30.SPEC-009 | Goodwill Refund Commit | Phase 4 (Trigger-Response) | A goodwill refund is not a Pro cancellation and is not evaluated by FEAT-09's outcome-evaluation automation (which only fires on cancellation, reschedule, or no-show); it is a standalone Pro override that needs its own processing spec, distinct from SPEC-007's cancellation-outcome handoff |
| FEAT-30.SPEC-010 | Pro-Created Booking & Deposit Request Hold | Phase 3 (Entity-Lifecycle) / Phase 6 (Failure Analysis) | The Booking entity's missing "Created by FEAT-30" path (Phase 3) and the Validation & Limits field's time-limited hold (24 hours or 2 hours before the appointment) together imply a time-based expiry path with its own failure handling (the Alternate flow: "the held slot is released, the pending booking is marked expired, and the Pro is notified"); in the revised specs this feature triggers the hold and relies on FEAT-03.SPEC-007 as sole writer of Expired (unpaid), with the Pro's notice owned by FEAT-08.SPEC-006 |
| FEAT-30.SPEC-011 | Goodwill & Bulk-Cancellation Refund Execution | Phase 4 (External Dependencies lens) | The Dependencies section of assumptions-constraints.md (ASMP-31) names the payment-processing capability; the External Touchpoints row for payment processing explicitly assigns "Pro-initiated and goodwill refund coverage" to this feature's analysis batch. A single Pro cancellation's refund is already executed by FEAT-09.SPEC-005 once FEAT-09.SPEC-004 evaluates it (per FEAT-09's own Brief); this feature's own Integration need is the refund calls that FEAT-09 never evaluates or batches: goodwill refunds (no FEAT-09 trigger exists for them) and the per-booking refund set a bulk cancellation produces |
| FEAT-30.SPEC-012 | Pro Action Client Notice | Phase 4 (Notification surfacing) | The Communications field names three distinct, content-bearing client messages (cancellation with refund, reschedule with new time, goodwill refund) -- each with real audience rules; the spec is a trigger-and-audience contract, with content owned by FEAT-08.SPEC-004/005 |
| FEAT-30.SPEC-013 | Deposit Request & Expiry Notice | Phase 4 (Notification surfacing) | The Communications field names a deposit-request message with explicit channel rules (text if consented, else email, or an on-screen code) and a Pro-facing expiry notification that this feature references but does not define (FEAT-03.SPEC-007 triggers, FEAT-08.SPEC-006 owns content); the deposit-request delivery is a Notification spec distinct from SPEC-012's outcome-notice contract |

## Entity-Lifecycle Coverage Matrix

**Entity: Booking**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-30.SPEC-004, FEAT-30.SPEC-010 | SPEC-004 captures the service, time, and client; SPEC-010 creates the Booking record in Pending Payment state with source = "Pro booked-in" | Bookings are also created by FEAT-05 and FEAT-21 -- out of this feature's scope |
| Read (single) | FEAT-30.SPEC-001, FEAT-30.SPEC-002, FEAT-30.SPEC-003, FEAT-30.SPEC-004, FEAT-30.SPEC-005 | Each screen loads the one booking (or set of bookings, for SPEC-005) the Pro is acting on | -- |
| Read (list) | N/A | This feature has no list of its own (States field: "N/A -- this feature acts on an existing booking or on a new one the Pro is creating; it has no list of its own") | The Pro reaches every action from FEAT-12's schedule list or FEAT-17's conflict handoff; browsing bookings is not this feature's responsibility |
| Update | FEAT-30.SPEC-007, FEAT-30.SPEC-008, FEAT-30.SPEC-010 | SPEC-007 writes the cancellation/reschedule state and timestamp for a single booking; SPEC-008 writes it for each booking in a bulk cancellation; SPEC-010 writes Pending Payment -> Confirmed (on payment) only; the -> Expired (unpaid) transition is written solely by FEAT-03.SPEC-007 and only read here | Goodwill refund (SPEC-009) never changes Booking state -- only the Deposit Transaction |
| Delete/Archive | N/A | Bookings are never deleted -- kept for the life of the account per SC-22; a cancelled or expired booking remains as history | Recorded as an explicit non-goal below rather than a silent gap |
| State Transition | FEAT-30.SPEC-007, FEAT-30.SPEC-008, FEAT-30.SPEC-010 | Confirmed/Awaiting Outcome -> Cancelled by Pro (SPEC-007 single, SPEC-008 bulk); the reschedule's in-place time update (SPEC-007); Pending Payment -> Confirmed (SPEC-010, on payment); Pending Payment -> Expired (unpaid) is written solely by FEAT-03.SPEC-007, not this feature | **Flagged discrepancy, not resolved by this Analyst:** as with FEAT-10's own Brief, whether a Pro-initiated reschedule updates start_time in place on the same record or transitions it to a terminal "Rescheduled" state that depends on a Creator elsewhere producing the new-time record is undefined in Stage 2 documents; carried forward for the Spec Writer and Stage 4 to resolve mechanically, consistent with the dependency map's Interactions being flagged rather than resolved by the Feature Analyst |

**Entity: Deposit Transaction (narrow slice -- Pro-initiated refund outcomes only, not fully managed by this feature)**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A -- owned by FEAT-07 | Per the dependency map's Deposit Transaction lifecycle line, "Created by FEAT-07" at the moment a deposit is captured, including the deposit this feature's own SPEC-010 requests from a client | This feature never creates a Deposit Transaction, only updates the outcome of an existing one |
| Read (single) | FEAT-30.SPEC-001, FEAT-30.SPEC-003, FEAT-30.SPEC-005, FEAT-30.SPEC-007, FEAT-30.SPEC-008, FEAT-30.SPEC-009 | Every action that previews or commits an outcome reads the transaction's current status first, so a deposit already refunded, forfeited, or disputed is never acted on twice | Consistent with the entity's own once-per-deposit resolution rule in the dependency map |
| Read (list) | N/A | This feature produces no list view of Deposit Transactions | The money list is FEAT-28's screen; the activity record is FEAT-16's |
| Update | FEAT-30.SPEC-009 (direct), FEAT-30.SPEC-011 (execution) | SPEC-009 determines the goodwill outcome is due and requests it; SPEC-011 executes the refund request through the payment-processing capability and writes status Refunded or Refund in Progress | A Pro-cancellation's refund is evaluated by FEAT-09.SPEC-004 and executed by FEAT-09.SPEC-005 once SPEC-007 hands off the cancellation event -- this feature triggers that outcome but does not write the field itself for the single-cancellation case, only for the goodwill and bulk-cancellation cases (SPEC-011) |
| Delete/Archive | N/A | No delete/archive path exists for a financial record | Per SC-22, retained for the life of the account and de-identified only after client deletion or account closure -- recorded as an explicit non-goal below |
| State Transition | FEAT-30.SPEC-009, FEAT-30.SPEC-011 | Captured/Applied -> Refunded (direct) or -> Refund in Progress -> Refunded (retried, per SPEC-011's shared idempotency behavior with FEAT-09.SPEC-006) | A single Pro-cancellation's Captured -> Refunded transition is FEAT-09's (SPEC-004/SPEC-005), triggered by this feature's SPEC-007 |

**Entity: Client**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-30.SPEC-004 | Pro enters a new client's name and phone (and email, if texting is declined) when booking them in for the first time | Also created by FEAT-05 on a client's own first booking |
| Read (single) | FEAT-30.SPEC-004 | Pro looks up an existing client by phone-number match to attach the new booking to their record | -- |
| Read (list) | N/A -- partial inline lookup only | SPEC-004 offers a quick existing-client lookup as part of booking-in, not a browsable list | The full searchable client list is FEAT-13's Client List Search & Filter screen; this feature never duplicates it |
| Update | N/A | This feature never edits an existing client's contact details or notes | Owned by FEAT-13 (Pro edits) and FEAT-06 (client's own email/consent updates) |
| Delete/Archive | N/A | This feature has no delete or archive path for Client | Owned by FEAT-13; per XBR-19, a Client with an upcoming booking cannot be deleted until that booking is resolved -- if the Pro cancels it here, that clears the way for a deletion FEAT-13 later performs |
| State Transition | N/A | Client carries no internal state machine, per its own dependency-map entry | -- |

**Entity: Balance Payment (v1 -- narrow slice, refund-on-cancellation only)**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A -- owned by FEAT-22 (v1) | Per the dependency map's Balance Payment lifecycle line, "Created by FEAT-22 (v1)" | Balance Payment does not exist at MVP; this row documents the v1 dependency named in Connected Entities |
| Read (single) | FEAT-30.SPEC-007, FEAT-30.SPEC-008 | Before cancelling, checks whether a Balance Payment already Succeeded for the booking, per XBR-23 | v1 behavior; at MVP this read always finds no record |
| Read (list) | N/A | This feature produces no list view of Balance Payments | -- |
| Update | FEAT-30.SPEC-007, FEAT-30.SPEC-008 | If a balance (and any tip) was already paid, it is refunded in full when either party cancels, per XBR-23 -- never forfeited | v1 behavior, layered onto the same cancellation commit that handles the deposit |
| Delete/Archive | N/A | No delete/archive path exists for a financial record; retained per SC-22 | Recorded as an explicit non-goal below |
| State Transition | FEAT-30.SPEC-007, FEAT-30.SPEC-008 | Succeeded -> Refunded, on either a single or bulk Pro-initiated cancellation | v1 behavior |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Cancellation Policy | FEAT-30.SPEC-006 | Read only to confirm that a Pro-initiated cancellation or reschedule never applies the window to the client (XBR-09, XBR-08) -- the policy's window value itself is never evaluated against a Pro action |
| Payout Account | FEAT-30.SPEC-011 | Refunds are drawn against the Pro's connected payout account balance; the Integration spec checks it is Active before requesting a refund, mirroring FEAT-09.SPEC-005 |
| Time Block | FEAT-30.SPEC-005 | Reads the set of confirmed Bookings a new Time Block conflicts with, handed over by FEAT-17, to populate the bulk-cancellation review screen |
| Access Link | N/A -- not read directly | A Pro-initiated reschedule (SPEC-007) triggers FEAT-08 to issue the client a fresh manage link (XBR-18); this feature never creates, reads, or invalidates an Access Link itself |
| Messaging Consent | N/A -- not read directly | The channel decision for every client notice this feature triggers (text vs. email) is made inside FEAT-08's delivery mechanism (XBR-15), not by this feature |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Pro opens Cancel Booking, Reschedule, Goodwill Refund, or Cancel Several screens | Re-check the booking's (or bookings') current eligibility (not already Completed, Auto-Completed, or otherwise terminal) and the Pro's own notice/horizon exception | Standalone Logic/Rule | FEAT-30.SPEC-006 |
| Pro confirms a single cancellation | Commit the Cancelled by Pro state; hand the full-refund determination to FEAT-09's outcome evaluation | Standalone Automation | FEAT-30.SPEC-007 |
| Pro confirms a reschedule | Commit the new time; deposit carries over automatically (no window applies to a Pro-made reschedule, XBR-09); trigger FEAT-08.SPEC-010 to issue a fresh client manage link (XBR-18) | Standalone Automation | FEAT-30.SPEC-007 |
| Pro Cancel/Reschedule Commit succeeds | Apply the deposit outcome for the single cancellation (always full refund) | Cross-feature -- logged in touchpoints | FEAT-09 responsibility (XBR-09, XBR-10) |
| Pro Cancel/Reschedule Commit succeeds | Move or remove the entry on the Pro's personal calendar | Cross-feature -- logged in touchpoints | FEAT-04 responsibility (XBR-13) |
| Pro Cancel/Reschedule Commit succeeds | Write an append-only activity event | Cross-feature -- logged in touchpoints | FEAT-16 responsibility (XBR-21) |
| Pro Cancel/Reschedule Commit succeeds (cancellation only) | Freed slot becomes publicly bookable immediately; matching waitlisted clients notified first with a 30-minute priority window | Cross-feature -- logged in touchpoints | FEAT-20 responsibility (XBR-28) |
| Pro Cancel/Reschedule Commit succeeds | Trigger the client's cancellation-with-refund or reschedule-with-new-time notice (content owned by FEAT-08.SPEC-004) | Standalone Notification (trigger-and-audience contract) | FEAT-30.SPEC-012 |
| A client-side action (FEAT-10) commits a conflicting transition first | Reject-with-refresh: the Pro is shown the booking's current state and must re-decide; the two transitions are never merged | Standalone Automation (failure handling) | FEAT-30.SPEC-007 |
| Pro chooses "cancel all" on the bulk-cancellation review screen | Commit Cancelled by Pro to every selected booking; hand each booking's full-refund determination to FEAT-09; report success or failure per booking | Standalone Automation | FEAT-30.SPEC-008 |
| One booking in a bulk cancellation fails to commit (e.g., it was already completed) | Report that booking's outcome as failed while the others still succeed; Pro sees which one needs a separate look | Standalone Automation (failure handling) | FEAT-30.SPEC-008 |
| Bulk Cancellation Commit succeeds, per booking | Same calendar removal, activity logging, freed-slot handoff, and client notice as a single cancellation, applied once per affected booking | Cross-feature / Standalone Notification | FEAT-04, FEAT-16, FEAT-20 responsibility; FEAT-30.SPEC-012 for the notice |
| Pro confirms a goodwill refund | Determine the refund is due (governed by SPEC-006's until-completion, once-only limits, independent of the window); request it | Standalone Automation | FEAT-30.SPEC-009 |
| Goodwill or bulk-cancellation refund is requested | Request the refund from the payment-processing capability, drawing on the Pro's payout account | Standalone Integration | FEAT-30.SPEC-011 |
| A requested refund cannot complete immediately (e.g., the Pro's payout balance cannot cover it yet) | Retry automatically without user action; flag the outcome clearly on the Pro's dashboard; keep the transaction at Refund in Progress until it resolves; show the client the refund as "in progress," never dropped | Standalone Integration (failure handling), cross-feature dashboard flag | FEAT-30.SPEC-011; FEAT-12 responsibility for the dashboard flag (XBR-10) |
| Goodwill or bulk-cancellation refund completes | Trigger the client's goodwill-refund or cancellation-with-refund notice | Standalone Notification | FEAT-30.SPEC-012 |
| Pro saves a new Pro-created booking (service, time, client) | Create the Booking in Pending Payment; trigger the slot hold via FEAT-03.SPEC-007; issue the deposit request | Standalone Automation | FEAT-30.SPEC-010 |
| Pro-created booking's slot conflicts with something that changed since selection (e.g., a client-side booking landed first) | Show a plain "just taken" message; Pro re-selects a time | Inline in triggering screen | FEAT-30.SPEC-004 |
| Deposit request is issued | Deliver it by text (if the client has active consent), by email otherwise, or render an on-screen code for the client to scan at the chair | Standalone Notification | FEAT-30.SPEC-013 |
| Client pays a Pro-created deposit request | Confirm the booking (Pending Payment -> Confirmed) exactly as a client-initiated booking would confirm | Standalone Automation | FEAT-30.SPEC-010 |
| Pro-created deposit request is not paid within its hold window (24 hours or 2 hours before the appointment, whichever comes first) | FEAT-03.SPEC-007 releases the held slot and is the sole writer of Booking -> Expired (unpaid), and triggers the Pro's expiry notice (content owned by FEAT-08.SPEC-006); this feature reads the resulting state and stops pending request delivery | Cross-feature (FEAT-03.SPEC-007, FEAT-08.SPEC-006); Standalone Notification for stopping delivery | FEAT-30.SPEC-010 (reads state); FEAT-30.SPEC-013 (stops delivery) |
| A booking already Completed or Auto-Completed | Block any cancel, reschedule, or new-goodwill-refund action; Pro sees a plain ineligibility message | Standalone Logic/Rule | FEAT-30.SPEC-006 |
| Any Pro-initiated action attempted while offline or connectivity drops | Plain message that connectivity is required; nothing is submitted; the most recently loaded schedule stays viewable read-only | Inline in triggering screen | FEAT-30.SPEC-001 / SPEC-002 / SPEC-003 / SPEC-004 / SPEC-005 |

## Shared Context

**Shared Entities:**
- Booking -- read by SPEC-001 through SPEC-005; created by SPEC-004/SPEC-010; updated by SPEC-007, SPEC-008, SPEC-010 (Pending Payment -> Confirmed only; Expired (unpaid) is written by FEAT-03.SPEC-007). Fields in scope here: state, start_time, source ("Pro booked-in"), cancellation/reschedule timestamps and optional private Pro reason, policy_version (read-only).
- Deposit Transaction (outcome slice only) -- read by every action screen and commit automation; updated directly only by SPEC-009/SPEC-011 (goodwill and bulk-cancellation refunds). A single Pro-cancellation's outcome write belongs to FEAT-09, triggered by this feature's SPEC-007.
- Client -- created by SPEC-004 for a new in-person booking; read (single lookup only) by SPEC-004.
- Balance Payment (v1, read-only field slice) -- read and refunded by SPEC-007/SPEC-008 whenever a cancellation touches a booking that already has one.

**Shared UI Patterns:**
- "See the outcome before confirming" pattern -- SPEC-001 (cancel), SPEC-002 (reschedule), SPEC-003 (goodwill refund), and SPEC-005 (bulk cancel) all show the deposit consequence plainly before the Pro commits, with an explicit confirm step; nothing changes if the Pro backs out. Spec Writers for all four should keep this ordering (outcome shown, then confirm) consistent, matching FEAT-10's equivalent client-side pattern.
- Live slot list reuse -- SPEC-002 and SPEC-004 both present the same real-time slot list mechanism a fresh booking uses (FEAT-03), with the Pro's own notice/horizon exception (SPEC-006) applied on top and the same "just taken" recovery message as any other booking path.
- Deposit-request delivery choice -- SPEC-004 and SPEC-010 both hand off to SPEC-013 for the same three-way delivery pattern (text, email, or on-screen code), so a Pro-created deposit request always looks and behaves the same whether issued from the daily schedule or the booking-in flow.

**Shared Validation:**
- FEAT-30.SPEC-006 (Pro Booking Action Rules) is referenced, not duplicated, by every screen (eligibility gates) and every automation (limits enforcement) in this feature.
- SPEC-011's refund-idempotency behavior mirrors, and is described consistently with, FEAT-09.SPEC-006's exactly-once-refund guarantee for the routine cancellation case -- both express the same XBR-10 rule for their respective refund paths.

## Internal Dependency Map

```
SPEC-001 (Cancel Booking) -> [checks eligibility using] -> SPEC-006 (Pro Booking Action Rules)
SPEC-001 (Cancel Booking) -> [Pro confirms] -> SPEC-007 (Pro Cancel/Reschedule Commit)
SPEC-001 (Cancel Booking) -> [Pro taps Recurring series link] -> FEAT-21.SPEC-010 (Pro Recurring Series Management, cross-feature)
SPEC-002 (Reschedule Booking) -> [checks eligibility and notice/horizon exception using] -> SPEC-006 (Pro Booking Action Rules)
SPEC-002 (Reschedule Booking) -> [Pro confirms new time] -> SPEC-007 (Pro Cancel/Reschedule Commit)
SPEC-003 (Goodwill Deposit Refund) -> [checks until-completion/once-only limits using] -> SPEC-006 (Pro Booking Action Rules)
SPEC-003 (Goodwill Deposit Refund) -> [Pro confirms] -> SPEC-009 (Goodwill Refund Commit)
SPEC-004 (Book Client In) -> [checks notice/horizon exception using] -> SPEC-006 (Pro Booking Action Rules)
SPEC-004 (Book Client In) -> [Pro saves service/time/client] -> SPEC-010 (Pro-Created Booking & Deposit Request Hold)
SPEC-005 (Cancel Several Bookings at Once) -> [checks eligibility for each booking using] -> SPEC-006 (Pro Booking Action Rules)
SPEC-005 (Cancel Several Bookings at Once) -> [Pro confirms] -> SPEC-008 (Bulk Cancellation Commit)
SPEC-007 (Pro Cancel/Reschedule Commit) -> [cancellation, refund due] -> FEAT-09 (external outcome evaluation and refund)
SPEC-007 (Pro Cancel/Reschedule Commit) -> [reschedule commits] -> FEAT-08.SPEC-010 (fresh manage link, cross-feature)
SPEC-007 (Pro Cancel/Reschedule Commit) -> [succeeds] -> SPEC-012 (Pro Action Client Notice)
SPEC-007 (Pro Cancel/Reschedule Commit) -> [a conflicting client-side transition wins] -> SPEC-001 / SPEC-002 [current state re-shown]
SPEC-008 (Bulk Cancellation Commit) -> [each booking, refund due] -> FEAT-09 (external outcome evaluation and refund)
SPEC-008 (Bulk Cancellation Commit) -> [each booking succeeds] -> SPEC-012 (Pro Action Client Notice)
SPEC-009 (Goodwill Refund Commit) -> [refund due] -> SPEC-011 (Goodwill & Bulk-Cancellation Refund Execution)
SPEC-011 (Goodwill & Bulk-Cancellation Refund Execution) -> [also executes] -> SPEC-008 (Bulk Cancellation Commit) [per-booking refunds]
SPEC-011 (Goodwill & Bulk-Cancellation Refund Execution) -> [cannot complete immediately] -> [retries automatically] -> SPEC-011
SPEC-011 (Goodwill & Bulk-Cancellation Refund Execution) -> [completes] -> SPEC-012 (Pro Action Client Notice)
SPEC-010 (Pro-Created Booking & Deposit Request Hold) -> [issues request] -> SPEC-013 (Deposit Request & Expiry Notice)
SPEC-010 (Pro-Created Booking & Deposit Request Hold) -> [places hold] -> FEAT-03.SPEC-007 (cross-feature; sole writer of Expired (unpaid))
FEAT-03.SPEC-007 -> [request expires unpaid] -> SPEC-013 (Deposit Request & Expiry Notice) [stops pending delivery] and FEAT-08.SPEC-006 (Pro expiry notice, content owner)
```

**Default Entry:** This feature has no single default landing screen -- the Pro arrives already viewing one booking or a set of bookings from FEAT-12's daily schedule (routing to SPEC-001, SPEC-002, or SPEC-003), from FEAT-17's time-block conflict handoff (routing to SPEC-005), from FEAT-11's no-show prompt (routing to SPEC-003), or opens Book Client In (SPEC-004) directly from the schedule to start a new in-person booking, per the Navigation connections in the dependency map.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-30.SPEC-001 / SPEC-002 / SPEC-003 | Inbound | FEAT-12 (Pro Daily Schedule Dashboard) | Pro taps a booking row and chooses cancel, reschedule, or refund | Pro taps a booking action |
| FEAT-30.SPEC-003 | Inbound | FEAT-11 (No-Show Marking & Deposit Forfeiture) | Pro chooses goodwill refund instead of marking a no-show | Pro is on the no-show prompt |
| FEAT-30.SPEC-003 | Inbound | FEAT-16 (Booking & Payment Activity Record) | Pro decides, from a dispute's timeline, to refund as goodwill | Pro reviews a no-show dispute timeline |
| FEAT-30.SPEC-005 | Inbound | FEAT-17 (Manual Time Blocking) | Pro's new time block conflicts with existing bookings and the Pro chooses to cancel the affected ones | Pro confirms a time block over existing bookings |
| FEAT-30.SPEC-007 | Inbound | FEAT-13 (Client Record Management) | Pro confirms deleting a client with an upcoming booking, which requires cancelling it with a full refund first | Pro confirms client deletion |
| FEAT-30.SPEC-008 | Inbound | FEAT-29 (Pro Sign-In & Account Lifecycle) | Pro requests account closure with upcoming bookings, which requires a bulk cancellation with full refunds first | Pro requests account closure |
| FEAT-30.SPEC-002 / SPEC-004 | Outbound | FEAT-03 (Real-Time Slot Availability Engine) | Reschedule and Book Client In both re-validate against the live slot list, with the Pro's own notice/horizon exception applied on top | Pro opens either screen or confirms a slot |
| FEAT-30.SPEC-007 / SPEC-008 / SPEC-009 | Outbound | FEAT-09 (Cancellation & No-Show Policy Engine) | Hands off the full-refund-always determination for Pro-initiated cancellations (single and bulk); goodwill refunds are this feature's own path, not FEAT-09's | Booking cancelled or rescheduled by the Pro |
| FEAT-30.SPEC-007 / SPEC-008 | Outbound | FEAT-04 (Two-Way Calendar Sync) | Commit causes the Pro's personal calendar entry to move or be removed | Booking cancelled or rescheduled by the Pro |
| FEAT-30.SPEC-007 / SPEC-008 | Outbound | FEAT-16 (Booking & Payment Activity Record) | Commit writes an append-only activity event | Booking cancelled or rescheduled by the Pro |
| FEAT-30.SPEC-007 / SPEC-008 | Outbound | FEAT-20 (Waitlist for Cancelled Slots) | A Pro-side cancellation frees the slot and triggers waitlist priority notification before general availability | Booking cancelled by the Pro |
| FEAT-30.SPEC-007 | Outbound | FEAT-08 (Automated Booking Messaging), FEAT-08.SPEC-010 | A Pro-made reschedule triggers a fresh booking-specific manage link for the client (XBR-18) | Reschedule commits |
| FEAT-30.SPEC-001 | Outbound | FEAT-21 (Recurring/Standing Appointments), FEAT-21.SPEC-010 | Link out to Pro Recurring Series Management; recurring management itself stays outside this feature | Pro taps the Recurring series link |
| FEAT-30.SPEC-010 | Outbound | FEAT-03 (Real-Time Slot Availability Engine), FEAT-03.SPEC-007 | Triggers the Pro-created deposit request hold; FEAT-03.SPEC-007 is the sole writer of Booking -> Expired (unpaid) and the trigger for the Pro's expiry notice | Pro saves a Pro-created booking / hold lapses unpaid |
| FEAT-30.SPEC-013 | Inbound | FEAT-08 (Automated Booking Messaging), FEAT-08.SPEC-006 | The Pro's expiry notice is content-owned by FEAT-08.SPEC-006; this feature only stops pending request delivery | Deposit request expires unpaid |
| FEAT-30.SPEC-011 | Outbound | FEAT-28 (Payout Account Connection & Payout Visibility) | Refunds are drawn against the Pro's connected payout account balance | A goodwill or bulk-cancellation refund is requested |
| FEAT-30.SPEC-011 | Outbound | FEAT-12 (Pro Daily Schedule Dashboard) | A refund that cannot complete immediately is flagged clearly on the Pro's dashboard until it resolves | Refund request fails to complete immediately |
| FEAT-30.SPEC-012 / SPEC-013 | Outbound | FEAT-08 (Automated Booking Messaging) | SPEC-012 is a trigger-and-audience contract for FEAT-08.SPEC-004/005 (which own content); SPEC-013 owns the deposit-request delivery paths; delivery (text/email channel choice per consent) is executed through FEAT-08's transactional messaging capability | Cancellation, reschedule, goodwill refund, or deposit request occurs |
| FEAT-30.SPEC-004 | Outbound | FEAT-07 (Deposit Payment at Booking) | The deposit request a Pro-created booking issues is paid through the same standard deposit-payment mechanism as any other booking | Client pays the deposit request |
| FEAT-30.SPEC-004 | Outbound | FEAT-13 (Client Record Management) | A quick existing-client lookup during booking-in; the full searchable client list belongs to FEAT-13 | Pro searches for an existing client while booking one in |

## Non-Functional Notes

**Data volumes / growth:** Pro-initiated actions are a subset of overall booking traffic for a solo Pro at 20-40 bookings a week (scope-boundaries SC-19); the Pro Change Correctness success metric implies this path carries meaningful weekly volume (rebooking regulars at the chair, occasional cancellations), not an edge case, and a bulk cancellation can affect a full day's bookings at once (the Sick Day journey: four bookings in one action).

**Responsiveness:** A Pro can cancel or reschedule a booking in under 30 seconds from the dashboard (Pro Change Correctness success metric); a full Pro-created booking-in flow follows the same one-second slot-appearance and under-one-minute completion benchmarks as any booking (ASMP-21). Outcome evaluation itself is instantaneous; only a refund that cannot complete immediately (SPEC-011) surfaces a visible "in progress" state rather than appearing instant, consistent with FEAT-09's own responsiveness note.

**Data sensitivity / privacy:** The booking, client, and deposit-outcome data this feature acts on is personal and financial data linked to an identifiable client, visible only to the Pro (Full) and, view-only, to Platform Operator (Support) for the resulting outcomes -- never card data (Access field; Access Matrix, Booking & Payment / Cancellation & No-Show Handling rows; SC-11). A new client's details entered here (name, phone, email) carry the same sensitivity as any Client record and are never visible to any other Pro or client (SC-03).

**Compliance flags:** N/A -- this feature applies no compliance regime of its own; the texting-consent rule governing deposit-request delivery is owned by FEAT-14 (XBR-15), and the money movement this feature triggers (refund or a fresh deposit charge) is a category-level payment-processing capability (ASMP-31) whose contract is documented by FEAT-09.SPEC-005 (routine cancellation refunds) and this feature's own FEAT-30.SPEC-011 (goodwill and bulk-cancellation refunds).

**Signals:** This feature emits booking_cancelled_by_pro and booking_rescheduled_by_pro on Pro Cancel/Reschedule Commit (SPEC-007, and per-booking on SPEC-008's bulk commit), goodwill_refund_issued on Goodwill Refund Commit (SPEC-009), pro_booking_created on Pro-Created Booking & Deposit Request Hold (SPEC-010) when the Pro saves the booking, and deposit_request_paid / deposit_request_expired on that same automation's payment or expiry outcome -- these six signals are the analytics basis for the Pro Change Correctness success metric (30-second action time, 100% refund-and-notify on Pro cancellations, 70%+ deposit-request payment-before-expiry rate).

## Non-Goals

- **Partial refunds or tiered cancellation schedules** -- Excluded per scope-boundaries SC-18: BRIEF.md's Business Context defines a binary deposit rule; every refund this feature triggers or executes (cancellation, reschedule, goodwill) is always full, never a percentage or tiered amount, per the Validation & Limits field ("refunds are always full in v1").
- **Charging a client's card later, or keeping one on file, for anything this feature does** -- Excluded per scope-boundaries SC-13: this feature's entire refund mechanism unwinds an already-captured deposit; the fresh deposit a Pro-created booking collects goes through the same standard deposit-payment mechanism (FEAT-07) as any other booking, never a stored-card charge made after the fact.
- **Chairtime adjudicating whether a goodwill refund is warranted** -- Excluded per scope-boundaries SC-17: the product never rules on a dispute; the goodwill decision is entirely the Pro's own judgment call, exercised through SPEC-003, with no automated "genuine emergency" detection.
- **Automatic purge or deletion of cancelled, rescheduled, or expired booking history** -- Intentional lifecycle decision surfaced by the CRUD matrix: bookings are retained for the life of the account per SC-22, so no cancelled, rescheduled, or expired booking this feature produces is ever deleted, only left as history with an updated state.
- **Support acting on a Pro's or client's behalf to cancel, reschedule, or refund** -- Excluded per scope-boundaries SC-05 and the Access Matrix: Platform Operator (Support) has view-only access to the outcomes and never performs any of this feature's actions itself, even to help resolve a support request.
- **Recurring series set-up and management** -- Not part of this feature: owned by FEAT-21.SPEC-010 (Pro Recurring Series Management). SPEC-001 only offers an outbound link to it; a Pro can also rebook a regular one visit at a time through Book Client In (SPEC-004).



# Screen Spec: Cancel Booking (Pro-Initiated)

## Overview

**Name:** Cancel Booking (Pro-Initiated)
**ID:** FEAT-30.SPEC-001
**Type:** Screen
**Purpose:** Talia views a single booking and confirms cancelling it, seeing plainly that the client's deposit will be refunded in full whatever the timing.
**Parent Feature:** FEAT-30 -- Pro Booking Management

## Scope and Non-Goals

**In Scope:**
- Showing the booking's details and the always-full-refund outcome before Talia confirms
- Capturing an optional private cancellation reason
- Triggering the commit and reflecting its success, rejection, or failure

**Non-Goals:**
- Determining or executing the deposit refund itself -- owned by FEAT-09, triggered through FEAT-30.SPEC-007; this screen only shows the outcome that XBR-09 guarantees
- Rescheduling the booking instead of cancelling it -- owned by FEAT-30.SPEC-002 (Reschedule Booking, Pro-Initiated), a distinct screen and action
- Cancelling more than one booking at a time -- owned by FEAT-30.SPEC-005 (Cancel Several Bookings at Once), a distinct review-and-confirm flow for a conflict set
- Setting up, viewing, or cancelling a client's recurring series or an occurrence of one -- recurring management is a Non-Goal of this feature and is owned by FEAT-21.SPEC-010 (Pro Recurring Series Management); this screen only offers an outbound link to it

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-12 (Pro Daily Schedule Dashboard) | Talia taps a booking row and chooses "Cancel" | Booking reference |
| FEAT-13 (Client Record Management) | Talia confirms deleting a client with an upcoming booking | Booking reference, with a note that this cancellation is required before the deletion can proceed |
| FEAT-29 (Pro Sign-In & Account Lifecycle) | Talia requests account closure with upcoming bookings, and this booking is one of a small set handled individually rather than in bulk | Booking reference |
| FEAT-30.SPEC-005 (Cancel Several Bookings at Once) | Talia taps "View" on a booking that failed in a bulk cancellation | The failed Booking reference |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen, for bookings she owns only | Confirm the cancellation, or back out with nothing changed | -- |
| The Client (Riley) | No | No | No control on any Client-facing surface reaches this screen; the Client's Booking & Payment access is Own-only, exercised through FEAT-10 |
| Platform Operator (Support) | Full screen, read-only, reached only through FEAT-19's account view | View only | Cancel confirm control and the Recurring series link are not shown, consistent with SC-05 |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29); no booking detail is shown |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- no unsaved input exists on this screen to preserve, since no cancellation has been confirmed yet |

## Layout and Content

**Header:** Back arrow (returns to FEAT-12) with the title "Cancel booking."

**Body:** A summary of the booking being cancelled -- client name, service, date and time, deposit amount -- followed by a plainly worded outcome statement: "{client_name}'s {deposit_amount} deposit will be refunded in full." An optional single-line text field labeled "Reason (private, not shared with {client_name})" for Talia's own note. Below that, a "Cancel booking" confirm button and a "Never mind" link to back out. Beneath the booking summary sits a "Recurring series" link, labeled "Repeat this booking" when the booking is not part of a series and "Manage recurring series" when it is, which opens FEAT-21.SPEC-010 (Pro Recurring Series Management) for this booking.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described, full width.
- **Medium size class and above:** Content remains single-column, capped at a consistent platform-wide form width and horizontally centered; no structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-12 (Pro Daily Schedule Dashboard) | Screen closes | Standard backward transition |
| Reason field | Type | Captures optional private text | Field shows entered text | Standard input focus state |
| "Cancel booking" button | Tap | Triggers FEAT-30.SPEC-006 eligibility re-check, then FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) | Button shows loading state during commit | Success: confirmation shown, then navigate to FEAT-12. Rejection: exact denial message from FEAT-30.SPEC-006 shown inline, no navigation. Failure: retry prompt shown, booking unchanged |
| "Cancel booking" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |
| "Recurring series" link ("Repeat this booking" / "Manage recurring series") | Tap | Navigate to FEAT-21.SPEC-010 (Pro Recurring Series Management) carrying the Booking reference; nothing on this screen is changed or committed | Screen closes without cancelling | Standard forward transition |
| "Never mind" link | Tap | Navigate to FEAT-12 with nothing changed | Screen closes | Standard backward transition |

### Accessibility Notes

- **Focus order:** Back arrow -> booking summary -> Recurring series link -> outcome statement -> Reason field -> Cancel booking button -> Never mind link.
- **Announcements:** The outcome statement and any denial message are announced to assistive technology when the screen loads or when a denial occurs.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Loading | A neutral loading placeholder in place of the booking summary and outcome statement | Screen opens | Booking data loads successfully or a load error occurs |
| Ready | Booking summary, outcome statement, and confirm/back-out controls shown | Booking data loads successfully and the eligibility check passes | Talia confirms or backs out |
| Ineligible | The exact denial message from FEAT-30.SPEC-006 shown in place of the confirm control (e.g., "This booking is already completed and can no longer be changed.") | Booking data loads successfully but the eligibility check fails | Talia backs out (only path forward, since no action is available) |
| Committing | Confirm button shows a loading state, all inputs disabled | Talia taps "Cancel booking" | Commit completes (success, rejection, or failure) |
| Committed | Confirmation shown ("Booking cancelled. {client_name}'s deposit is being refunded.") before returning to FEAT-12 | FEAT-30.SPEC-007 reports success | Talia is navigated to FEAT-12 |
| Rejected | Inline message showing the booking's current state (e.g., "This booking was already cancelled by {client_name}.") | FEAT-30.SPEC-007 reports a conflicting transition already won | Talia backs out; no retry of the same action is offered since the booking has moved on |
| Error | Inline error message "Couldn't load this booking. Try again." or "Couldn't cancel this booking. Try again." with a Retry action | The booking's details fail to load, or FEAT-30.SPEC-007 reports a processing failure | Talia taps Retry and it succeeds, or she backs out |
| Offline/Degraded | Banner "Check your connection and try again." replaces the confirm control; the booking summary remains viewable read-only | Connectivity is lost while the screen is open | Connectivity is restored and Talia can confirm |

## Validation Rules

Validation and eligibility governed by FEAT-30.SPEC-006 (Pro Booking Action Rules). See that spec for the exact conditions and denial messages. This screen applies the eligibility check on entry and again on confirm.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|-------------------------------------|
| Back arrow tap | FEAT-12 (Pro Daily Schedule Dashboard) | -- |
| "Never mind" tap | FEAT-12 (Pro Daily Schedule Dashboard) | -- |
| "Repeat this booking" / "Manage recurring series" tap | FEAT-21.SPEC-010 (Pro Recurring Series Management) | FEAT-21 (Recurring/Standing Appointments) |
| Successful cancellation | FEAT-12 (Pro Daily Schedule Dashboard) | -- |

## Data Model

**Reads:** Booking -- state, start_time, service, client, deposit_amount (via Deposit Transaction); Deposit Transaction -- status (eligibility precondition).
**Creates:** None directly -- the cancellation itself is written by FEAT-30.SPEC-007.
**Updates:** None directly -- delegated entirely to FEAT-30.SPEC-007 on confirm.
**Deletes:** None.

## Business Rules

- XBR-09: any Pro-initiated cancellation refunds the client's deposit in full, whatever the timing -- this screen states that outcome plainly before Talia confirms, per the feature's "see the outcome before confirming" shared UI pattern.
- Eligibility (ownership, and Booking.state is Confirmed or Awaiting Outcome) is governed by FEAT-30.SPEC-006 and re-checked on entry and again on confirm.
- The optional private reason is never shown to the client; it is recorded on the Booking for Talia's own record only (per the Brief's Data Notes).

## Edge Cases

- **A client cancels this same booking through FEAT-10 while Talia is viewing this screen** -- Reject-with-refresh: Talia's confirm attempt is denied and she sees the booking's current (client-cancelled) state, per FEAT-30.SPEC-007's Contention handling; nothing is merged.
- **Talia navigates away with the reason field filled in but unconfirmed** -- No confirmation dialog is shown, since no cancellation has been submitted; the reason is discarded, consistent with this being a confirm-only action, not a draft-preserving form.
- **Talia taps "Cancel booking" twice rapidly** -- The second tap is ignored while the first commit is in progress (button in loading state).
- **The booking becomes ineligible (e.g., auto-completes) between screen load and Talia's tap** -- The confirm-time eligibility re-check (FEAT-30.SPEC-006) catches this and denies with the exact current-state message, even though the screen initially showed the action as available.
- **Network failure during the commit** -- Error state shown: "Couldn't cancel this booking. Try again." with Retry; the booking remains exactly as it was.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-30.SPEC-006 (Pro Booking Action Rules) | References (inbound) | Eligibility check on entry and confirm |
| FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) | Triggers (outbound) | Confirm triggers the commit |
| FEAT-12 (Pro Daily Schedule Dashboard) | Navigation (inbound/outbound) | Primary entry point and return destination |
| FEAT-21.SPEC-010 (Pro Recurring Series Management) -- within FEAT-21 (Recurring/Standing Appointments) | Navigation (outbound) | The "Recurring series" link opens the Pro's series set-up and management for this booking; recurring management itself stays outside this feature |
| FEAT-13 (Client Record Management) | Navigation (inbound) | Reaches this screen when deleting a client with an upcoming booking |
| FEAT-29 (Pro Sign-In & Account Lifecycle) | Navigation (inbound) | Reaches this screen for an individual booking during account closure |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-------------------|
| pro_cancel_screen_opened | entry source (dashboard / client_deletion / account_closure) | Screen loads | supports success-metrics.md: "Pro Change Correctness" |
| pro_cancel_confirmed | time from screen open to confirm | Talia taps "Cancel booking" and the commit succeeds | supports success-metrics.md: "Pro Change Correctness" |
| pro_cancel_denied | reason category | Eligibility check denies the action | supports success-metrics.md: "Automatic Refund Correctness" |

## Acceptance Criteria

**FEAT-30.SPEC-001-AC-01:** Given Talia opens this screen for a Confirmed booking she owns, when the screen loads, then she sees the booking summary and the plain statement that the client's deposit will be refunded in full.

**FEAT-30.SPEC-001-AC-02:** Given Talia enters a private reason and taps "Cancel booking", when the commit succeeds, then she sees a confirmation and returns to FEAT-12, and the reason is never shown to Riley.

**FEAT-30.SPEC-001-AC-03:** Given Talia opens this screen for a booking already Completed, when eligibility is checked, then she sees the exact denial message from FEAT-30.SPEC-006 and no confirm control is offered.

**FEAT-30.SPEC-001-AC-04:** Given Riley cancels the same booking through FEAT-10 while Talia is viewing this screen, when Talia taps "Cancel booking", then she sees the booking's current (client-cancelled) state rather than a merged or overwritten outcome.

**FEAT-30.SPEC-001-AC-05:** Given Talia taps "Cancel booking" twice rapidly, when the first tap's commit is in progress, then the second tap has no effect until the first resolves.

**FEAT-30.SPEC-001-AC-06:** Given the commit fails due to a processing error, when the failure occurs, then Talia sees "Couldn't cancel this booking. Try again." with a Retry action, and the booking remains unchanged.

**FEAT-30.SPEC-001-AC-07:** Given Talia loses connectivity while viewing this screen, when she attempts to confirm, then she sees "Check your connection and try again." and no commit is attempted.

**FEAT-30.SPEC-001-AC-08:** Given Talia taps "Never mind", when the navigation completes, then she returns to FEAT-12 with nothing changed.

**FEAT-30.SPEC-001-AC-09:** Given Riley (the Client) has no path to this screen, when the product's screens are reviewed for a reachable control, then none exists.

**FEAT-30.SPEC-001-AC-10:** Given Platform Operator (Support) views this screen through FEAT-19's read-only surfaces, when Support looks for a cancel control, then none is shown.

**FEAT-30.SPEC-001-AC-11:** Given Talia reaches this screen from confirming a client deletion (FEAT-13), when she confirms the cancellation, then the deposit refund proceeds identically to any other entry point.

**FEAT-30.SPEC-001-AC-12:** Given a booking becomes ineligible between screen load and Talia's confirm tap, when the confirm-time eligibility check runs, then it denies with the current, correct reason rather than proceeding against a stale screen state.

**FEAT-30.SPEC-001-AC-13:** Given Talia navigates away from this screen with an unconfirmed reason typed in, when she leaves, then no confirmation dialog appears and the cancellation is not submitted.

**FEAT-30.SPEC-001-AC-14:** Given Talia opens this screen, when the booking's details are being fetched, then she sees a loading placeholder in place of the booking summary and outcome statement.

**FEAT-30.SPEC-001-AC-15:** Given the booking's details fail to load, when the load fails, then Talia sees "Couldn't load this booking. Try again." with a Retry action, and no confirm control is offered until it succeeds.

**FEAT-30.SPEC-001-AC-16:** Given Talia is on this screen for a Confirmed booking, when she taps the "Recurring series" link ("Repeat this booking" for a one-off booking, "Manage recurring series" for a booking in a series), then she is navigated to FEAT-21.SPEC-010 with the Booking reference and nothing is cancelled or changed on this screen.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 8 (loading, ready, ineligible, committing, committed, rejected, error, offline) | 8 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



# Screen Spec: Reschedule Booking (Pro-Initiated)

## Overview

**Name:** Reschedule Booking (Pro-Initiated)
**ID:** FEAT-30.SPEC-002
**Type:** Screen
**Purpose:** Talia picks a new genuinely free time for a client's booking, inside her own notice/horizon exception, seeing that the deposit carries over and a fresh manage link will go to the client.
**Parent Feature:** FEAT-30 -- Pro Booking Management

## Scope and Non-Goals

**In Scope:**
- Re-using the live slot list mechanism (FEAT-03) with the Pro-only notice/horizon exception applied
- Showing that the deposit carries over automatically and a fresh manage link will be issued, before Talia confirms
- Triggering the commit and reflecting its success, rejection, or failure

**Non-Goals:**
- Computing which times are genuinely free -- owned entirely by FEAT-03 (Real-Time Slot Availability Engine); this screen only displays what FEAT-03 returns, with the Pro-only exception layered on by FEAT-30.SPEC-006
- Cancelling the booking instead of rescheduling it -- owned by FEAT-30.SPEC-001 (Cancel Booking, Pro-Initiated)
- Issuing the fresh manage link itself -- owned by FEAT-06 (Client Booking Identity) and FEAT-08 (Automated Booking Messaging), triggered by a successful commit (XBR-18); this screen only states that it will happen

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-12 (Pro Daily Schedule Dashboard) | Talia taps a booking row and chooses "Reschedule" | Booking reference, current appointment time |
| FEAT-17.SPEC-003 (Time Block Conflict Review) | Conflict review confirms with a booking chosen "Reschedule" | The conflicting Booking reference |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen, for bookings she owns only | Select a new time and confirm, or back out with nothing changed | -- |
| The Client (Riley) | No | No | No control on any Client-facing surface reaches this screen; the Client reschedules her own bookings through FEAT-10 |
| Platform Operator (Support) | Full screen, read-only, reached only through FEAT-19's account view | View only | Slot selection and confirm controls are not shown, consistent with SC-05 |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29) |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- an unconfirmed new-time selection is discarded, since no reschedule has been submitted yet |

## Layout and Content

**Header:** Back arrow (returns to FEAT-12) with the title "Reschedule booking" and the client's name and current appointment time shown beneath it as persistent context.

**Body:** A live list of available time slots for the same service, grouped by day, in chronological order -- the same slot-list mechanism as FEAT-05.SPEC-002 and FEAT-30.SPEC-004, with Talia's own notice/horizon exception applied. Below the list, once a new time is selected: a confirmation summary stating "{client_name}'s deposit carries over -- they'll get a new confirmation with this time." with "Confirm reschedule" and "Choose a different time" actions.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Slots list in a single column, grouped by day heading, full width; confirmation summary stacks below.
- **Medium size class and above:** Slots list may show more times per row within the same day grouping; confirmation summary remains single-column, capped at a consistent platform-wide width.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-12 | Screen closes | Standard backward transition |
| Time slot | Tap | Re-validates the candidate against FEAT-03.SPEC-004 (with the Pro-only notice/horizon exception, per FEAT-30.SPEC-006) | Slot selected, confirmation summary appears | If valid: summary shown. If contested: plain "just taken" message, list refreshed |
| "Confirm reschedule" button | Tap | Triggers FEAT-30.SPEC-006 eligibility re-check, then FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) | Button shows loading state during commit | Success: confirmation shown, then navigate to FEAT-12. Rejection: exact denial message shown inline. Failure: retry prompt shown, booking unchanged |
| "Confirm reschedule" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |
| "Choose a different time" link | Tap | Clears the selected time | Returns to the slot list, selection cleared | Slot list remains visible for re-selection |

### Accessibility Notes

- **Focus order:** Back arrow -> current-appointment context -> day groupings top to bottom -> time slots -> (once selected) confirmation summary -> Confirm reschedule button -> Choose a different time link.
- **Announcements:** The confirmation summary and any denial or contention message are announced to assistive technology when they appear.
- **Keyboard alternatives:** Every time slot and action is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Loading | A neutral loading placeholder in place of the slot list | Screen first opens | Slot list loads successfully or a load error occurs |
| Selecting | Slot list rendered, no time yet selected | Live slot data returns at least one available time | Talia taps a slot |
| Slot contested | A plain message "That time was just taken." appears briefly, list refreshes | The tapped slot is lost to another client or booking | Talia picks a different slot |
| Selected | Confirmation summary shown with Confirm/Choose-different-time actions | Talia's candidate slot passes re-validation | Talia confirms or chooses a different time |
| Ineligible | The exact denial message from FEAT-30.SPEC-006 shown in place of the slot list (e.g., booking already Completed) | Eligibility check on screen entry fails | Talia backs out (only path forward) |
| Committing | Confirm button shows a loading state, all inputs disabled | Talia taps "Confirm reschedule" | Commit completes (success, rejection, or failure) |
| Committed | Confirmation shown ("Booking moved. {client_name} will get the new details.") before returning to FEAT-12 | FEAT-30.SPEC-007 reports success | Talia is navigated to FEAT-12 |
| Rejected | Inline message showing the booking's current state | FEAT-30.SPEC-007 reports a conflicting transition already won, or the held slot was lost between selection and commit | Talia backs out, or (for a lost slot) returns to slot selection with a refreshed list |
| Error | Inline error message "Couldn't load available times. Try again." or "Couldn't reschedule this booking. Try again." with Retry | Live slot data fails to load, or the commit reports a processing failure | Talia taps Retry and it succeeds, or she backs out |
| Offline/Degraded | Banner "Check your connection and try again." replaces the slot list or confirm control | Connectivity is lost while the screen is open | Connectivity is restored and Talia can select or confirm |

## Validation Rules

Slot validity governed by FEAT-03.SPEC-004 (Slot Validation & Timing Rules), with the Pro-only notice/horizon exception applied per FEAT-30.SPEC-006. Action eligibility (ownership, Booking.state) governed by FEAT-30.SPEC-006. Both are checked on screen entry and again on confirm.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|-------------------------------------|
| Back arrow tap | FEAT-12 (Pro Daily Schedule Dashboard) | -- |
| Successful reschedule | FEAT-12 (Pro Daily Schedule Dashboard) | -- |

## Data Model

**Reads:** Booking -- state, start_time, service, client, deposit_amount; live slot list for the same service (FEAT-03); Availability Rule -- minimum_booking_notice, booking_horizon (exempted for this screen's re-validation, per FEAT-30.SPEC-006).
**Creates:** None directly.
**Updates:** None directly -- delegated entirely to FEAT-30.SPEC-007 on confirm, which updates Booking.start_time.
**Deletes:** None.

## Business Rules

- XBR-09: a Pro-made reschedule never exposes the client to the cancellation window -- the deposit always carries over automatically, whatever the new time.
- XBR-18: a successful reschedule triggers a fresh manage link for the client, since the previous link's context (the old appointment time) is now stale.
- XBR-03: Talia may reschedule inside her own minimum_booking_notice or beyond her booking_horizon; the candidate slot's fit rule (duration + buffer, no conflict) is never exempted.
- The "see the outcome before confirming" pattern (Brief's Shared UI Patterns) applies here identically to FEAT-30.SPEC-001: the deposit-carries-over outcome is shown before Talia confirms.

## Edge Cases

- **Talia's held candidate slot is lost to a contesting booking between selection and confirm** -- Per FEAT-03.SPEC-005's contention resolution, Talia is returned to slot selection with a refreshed list and a "just taken" message; nothing is committed.
- **A client reschedules or cancels this same booking through FEAT-10 while Talia is mid-selection** -- Reject-with-refresh at confirm time: Talia's commit attempt is denied and she sees the booking's current state, per FEAT-30.SPEC-007's Contention handling.
- **Talia navigates away with a time selected but unconfirmed** -- No confirmation dialog is shown; the selection is discarded and the booking is untouched.
- **Talia taps "Confirm reschedule" twice rapidly** -- The second tap is ignored while the first commit is in progress.
- **The booking becomes ineligible (e.g., cancelled by the client) between screen load and Talia's slot selection** -- The confirm-time eligibility re-check catches this and denies with the exact current-state message.
- **Network failure during the commit** -- Error state shown: "Couldn't reschedule this booking. Try again." with Retry; the booking remains at its original time.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-30.SPEC-006 (Pro Booking Action Rules) | References (inbound) | Notice/horizon exception and eligibility check |
| FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) | Triggers (outbound) | Confirm triggers the commit |
| FEAT-03 (Real-Time Slot Availability Engine) | References (inbound) | Source of the live slot list this screen displays |
| FEAT-12 (Pro Daily Schedule Dashboard) | Navigation (inbound/outbound) | Entry point and return destination |
| FEAT-06 (Client Booking Identity) / FEAT-08 (Automated Booking Messaging) | Affects (outbound) | A successful commit triggers a fresh client manage link |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-------------------|
| pro_reschedule_screen_opened | () | Screen loads | supports success-metrics.md: "Pro Change Correctness" |
| pro_reschedule_slot_selected | time-to-selection since screen open | Talia taps a valid slot | supports success-metrics.md: "Pro Change Correctness" |
| pro_reschedule_confirmed | time from screen open to confirm | The commit succeeds | supports success-metrics.md: "Pro Change Correctness" |
| pro_reschedule_slot_lost_to_contention | () | The selected slot is lost between selection and confirm | supports success-metrics.md: "Zero Double-Booking Confidence" |

## Acceptance Criteria

**FEAT-30.SPEC-002-AC-01:** Given Talia opens this screen for a Confirmed booking, when the screen loads, then she sees a live slot list for the same service with her own notice/horizon exception applied.

**FEAT-30.SPEC-002-AC-02:** Given Talia selects a candidate time inside her own minimum_booking_notice, when the slot is re-validated, then it is accepted, per the Pro-only exception.

**FEAT-30.SPEC-002-AC-03:** Given Talia selects a valid new time, when she taps "Confirm reschedule", then the commit succeeds, the booking's start_time updates, and she sees confirmation before returning to FEAT-12.

**FEAT-30.SPEC-002-AC-04:** Given Talia selects a time, when the confirmation summary appears, then it states the deposit carries over and the client will get a new confirmation with the new time.

**FEAT-30.SPEC-002-AC-05:** Given Talia's selected slot is lost to another booking before she confirms, when she attempts to confirm, then she sees "That time was just taken." and returns to a refreshed slot list.

**FEAT-30.SPEC-002-AC-06:** Given Riley cancels the same booking through FEAT-10 while Talia is mid-selection, when Talia taps "Confirm reschedule", then she sees the booking's current (client-cancelled) state rather than a merged outcome.

**FEAT-30.SPEC-002-AC-07:** Given Talia opens this screen for a booking already Completed, when eligibility is checked, then she sees the exact denial message from FEAT-30.SPEC-006 and no slot list is offered.

**FEAT-30.SPEC-002-AC-08:** Given a successful reschedule commit, when it completes, then Riley receives a fresh manage link, per XBR-18.

**FEAT-30.SPEC-002-AC-09:** Given Talia taps "Confirm reschedule" twice rapidly, when the first tap's commit is in progress, then the second tap has no effect until the first resolves.

**FEAT-30.SPEC-002-AC-10:** Given the commit fails due to a processing error, when the failure occurs, then Talia sees "Couldn't reschedule this booking. Try again." with a Retry action, and the booking remains at its original time.

**FEAT-30.SPEC-002-AC-11:** Given Talia loses connectivity while viewing this screen, when she attempts to select a slot or confirm, then she sees "Check your connection and try again." and nothing is committed.

**FEAT-30.SPEC-002-AC-12:** Given Riley (the Client) has no path to this screen, when the product's screens are reviewed for a reachable control, then none exists.

**FEAT-30.SPEC-002-AC-13:** Given Talia taps "Choose a different time" after selecting a slot, when the tap registers, then the selection clears and she returns to the slot list.

**FEAT-30.SPEC-002-AC-14:** Given Talia navigates away from this screen with a time selected but unconfirmed, when she leaves, then no confirmation dialog appears and the booking is untouched.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 10 (loading, selecting, contested, selected, ineligible, committing, committed, rejected, error, offline) | 10 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Screen Spec: Goodwill Deposit Refund

## Overview

**Name:** Goodwill Deposit Refund
**ID:** FEAT-30.SPEC-003
**Type:** Screen
**Purpose:** Talia confirms a full goodwill refund on a booking's deposit, reached from a no-show prompt, a dispute timeline, or a booking row, available any time before the booking completes.
**Parent Feature:** FEAT-30 -- Pro Booking Management

## Scope and Non-Goals

**In Scope:**
- Showing the deposit outcome (a full refund) before Talia confirms, from any of this action's three entry points
- Capturing an optional private note explaining the goodwill decision
- Triggering the commit and reflecting its success, in-progress, rejection, or failure

**Non-Goals:**
- Deciding whether a goodwill refund is warranted -- excluded per scope-boundaries.md (SC-17): the decision is entirely Talia's own judgment; this screen presents the outcome and captures her confirmation, never an automated recommendation
- Marking or undoing a no-show -- owned by FEAT-11 (No-Show Marking & Deposit Forfeiture); this screen is one of the paths reachable from that feature's prompt, but is a distinct action
- Executing the refund against the payment-processing capability -- owned by FEAT-30.SPEC-011, triggered through FEAT-30.SPEC-009 (Goodwill Refund Commit)

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-11 (No-Show Marking & Deposit Forfeiture) | Talia chooses "Refund as goodwill instead" from the no-show prompt | Booking reference |
| FEAT-16.SPEC-001 (Booking Activity Timeline) (Booking & Payment Activity Record) | Talia decides, from a dispute's timeline, to refund as goodwill | Booking reference |
| FEAT-12 (Pro Daily Schedule Dashboard) | Talia taps a booking row and chooses "Refund as goodwill" | Booking reference |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen, for bookings she owns only | Confirm the goodwill refund, or back out with nothing changed | -- |
| The Client (Riley) | No | No | No control on any Client-facing surface reaches this screen |
| Platform Operator (Support) | Full screen, read-only, reached only through FEAT-19's account view | View only | Confirm control is not shown, consistent with SC-05 |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29) |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- an unconfirmed note is discarded, since no refund has been submitted yet |

## Layout and Content

**Header:** Back arrow (returns to the entry point spec) with the title "Refund as goodwill."

**Body:** A summary of the booking -- client name, service, date and time, deposit amount, and its current status (kept / captured, as applicable) -- followed by a plainly worded outcome statement: "{client_name}'s {deposit_amount} deposit will be refunded in full." An optional single-line text field labeled "Note (private, not shared with {client_name})." Below that, a "Refund as goodwill" confirm button and a "Never mind" link to back out.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described, full width.
- **Medium size class and above:** Content remains single-column, capped at a consistent platform-wide form width and horizontally centered.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate back to the entry point (FEAT-11, FEAT-16, or FEAT-12) | Screen closes | Standard backward transition |
| Note field | Type | Captures optional private text | Field shows entered text | Standard input focus state |
| "Refund as goodwill" button | Tap | Triggers FEAT-30.SPEC-006 eligibility re-check, then FEAT-30.SPEC-009 (Goodwill Refund Commit) | Button shows loading state during commit | Success (immediate): confirmation shown. Success (in progress): "in progress" confirmation shown. Rejection: exact denial message shown inline. Failure: retry prompt shown |
| "Refund as goodwill" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |
| "Never mind" link | Tap | Navigate back to the entry point with nothing changed | Screen closes | Standard backward transition |

### Accessibility Notes

- **Focus order:** Back arrow -> booking summary -> outcome statement -> Note field -> Refund as goodwill button -> Never mind link.
- **Announcements:** The outcome statement and any denial or in-progress message are announced to assistive technology when they appear.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Loading | A neutral loading placeholder in place of the booking summary and outcome statement | Screen opens | Booking and deposit data load successfully or a load error occurs |
| Ready | Booking summary, outcome statement, and confirm/back-out controls shown | Booking and deposit data load successfully and the eligibility check passes (Deposit Transaction Captured or Forfeited, Booking not Completed) | Talia confirms or backs out |
| Ineligible | The exact denial message from FEAT-30.SPEC-006 shown in place of the confirm control | Booking and deposit data load successfully but the eligibility check fails | Talia backs out (only path forward) |
| Committing | Confirm button shows a loading state, all inputs disabled | Talia taps "Refund as goodwill" | Commit completes (success, in-progress, rejection, or failure) |
| Committed | Confirmation shown ("Refund sent. {client_name}'s deposit is on its way back.") before returning to the entry point | FEAT-30.SPEC-009 reports the refund completed immediately | Talia is navigated back to the entry point |
| Committed -- in progress | Confirmation shown ("Refund started. It will complete automatically -- you don't need to do anything.") before returning to the entry point | FEAT-30.SPEC-009 reports the refund cannot complete immediately | Talia is navigated back to the entry point; the dashboard attention flag (FEAT-12) persists until resolved |
| Rejected | Inline message showing the deposit's current, already-resolved status | FEAT-30.SPEC-009 reports the deposit is no longer refundable | Talia backs out |
| Error | Inline error message "Couldn't load this booking. Try again." or "Couldn't process this refund. Try again." with a Retry action | The booking or deposit data fails to load, or FEAT-30.SPEC-009 reports a processing failure | Talia taps Retry and it succeeds, or she backs out |
| Offline/Degraded | Banner "Check your connection and try again." replaces the confirm control; the booking summary remains viewable read-only | Connectivity is lost while the screen is open | Connectivity is restored and Talia can confirm |

## Validation Rules

Eligibility (once-only limit, until-completion window, ownership) governed by FEAT-30.SPEC-006 (Pro Booking Action Rules). See that spec for the exact conditions and denial messages. This screen applies the eligibility check on entry and again on confirm.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|-------------------------------------|
| Back arrow tap | The entry point spec (FEAT-11, FEAT-16, or FEAT-12) | Varies by entry point |
| "Never mind" tap | The entry point spec | Varies by entry point |
| Successful refund (immediate or in-progress) | The entry point spec | Varies by entry point |

## Data Model

**Reads:** Booking -- state; Deposit Transaction -- status, amount.
**Creates:** None directly.
**Updates:** None directly -- delegated entirely to FEAT-30.SPEC-009 on confirm, which updates Deposit Transaction.status.
**Deletes:** None.

## Business Rules

- SC-17: this screen never suggests or adjudicates whether a goodwill refund is warranted -- the decision is entirely Talia's, exercised by reaching this screen and confirming.
- XBR-12: a goodwill refund is available until the booking is completed; this screen's eligibility check enforces that cutoff, per FEAT-30.SPEC-006.
- A goodwill refund is available whether the deposit is currently Captured (offered instead of marking a no-show, or against an unresolved inside-window cancellation) or Forfeited (converting an already-kept deposit into a full refund) -- the outcome statement and confirm flow are identical either way.
- The optional private note is never shown to the client; it is recorded for Talia's own record only.

## Edge Cases

- **Talia reaches this screen from the no-show prompt (FEAT-11) before confirming the no-show mark itself** -- The deposit is at Captured; the refund proceeds as a standard Captured-status goodwill refund with no interaction with FEAT-11's own marking flow, and no no-show mark is ever recorded for this booking.
- **The deposit is refunded automatically by FEAT-09 (an outside-window client cancellation) between Talia opening this screen and her confirm** -- The confirm-time eligibility re-check finds the deposit already Refunded and denies with "This booking's deposit has already been resolved and cannot be refunded again."
- **The booking reaches Completed via the Auto-Completion Sweep between screen open and confirm** -- The confirm-time eligibility re-check denies with "A goodwill refund is no longer available once a booking is completed."
- **Talia taps "Refund as goodwill" twice rapidly** -- The second tap is ignored while the first commit is in progress.
- **Network failure during the commit** -- Error state shown: "Couldn't process this refund. Try again." with Retry; the deposit remains at its prior status.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-30.SPEC-006 (Pro Booking Action Rules) | References (inbound) | Once-only and until-completion eligibility check |
| FEAT-30.SPEC-009 (Goodwill Refund Commit) | Triggers (outbound) | Confirm triggers the commit |
| FEAT-11 (No-Show Marking & Deposit Forfeiture) | Navigation (inbound) | One entry point, reached instead of marking a no-show |
| FEAT-16 (Booking & Payment Activity Record) | Navigation (inbound) | Another entry point, reached from a dispute timeline |
| FEAT-12 (Pro Daily Schedule Dashboard) | Navigation (inbound/outbound) | Another entry point and a possible return destination |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-------------------|
| pro_goodwill_screen_opened | entry source (no_show_prompt / dispute_timeline / booking_row) | Screen loads | supports success-metrics.md: "Pro Change Correctness" |
| pro_goodwill_confirmed | outcome (completed / in_progress) | Talia taps "Refund as goodwill" and the commit succeeds | supports success-metrics.md: "Automatic Refund Correctness" |
| pro_goodwill_denied | reason category | Eligibility check denies the action | supports success-metrics.md: "Automatic Refund Correctness" |

## Acceptance Criteria

**FEAT-30.SPEC-003-AC-01:** Given Talia opens this screen for a booking with a Captured deposit that has not completed, when the screen loads, then she sees the plain statement that the client's deposit will be refunded in full.

**FEAT-30.SPEC-003-AC-02:** Given Talia confirms the refund and the execution completes immediately, when the commit succeeds, then she sees "Refund sent. {client_name}'s deposit is on its way back." and returns to the entry point.

**FEAT-30.SPEC-003-AC-03:** Given the refund execution cannot complete immediately, when the commit processes that outcome, then Talia sees "Refund started. It will complete automatically -- you don't need to do anything."

**FEAT-30.SPEC-003-AC-04:** Given Talia reaches this screen from the no-show prompt before marking the no-show, when she confirms the refund, then it processes normally against the Captured deposit with no no-show mark ever recorded.

**FEAT-30.SPEC-003-AC-05:** Given a booking's deposit is already Forfeited from a no-show mark, when Talia opens this screen, then the goodwill action is still offered and eligible.

**FEAT-30.SPEC-003-AC-06:** Given a booking's deposit has already been automatically refunded by FEAT-09, when Talia opens this screen, then she sees the exact denial message from FEAT-30.SPEC-006 and no confirm control is offered.

**FEAT-30.SPEC-003-AC-07:** Given a booking reaches Completed between screen load and Talia's confirm tap, when the confirm-time eligibility check runs, then it denies with the completed-state message.

**FEAT-30.SPEC-003-AC-08:** Given Talia enters a private note and confirms the refund, when the commit succeeds, then the note is recorded for Talia's own record and never shown to Riley.

**FEAT-30.SPEC-003-AC-09:** Given Talia taps "Refund as goodwill" twice rapidly, when the first tap's commit is in progress, then the second tap has no effect until the first resolves.

**FEAT-30.SPEC-003-AC-10:** Given the commit fails due to a processing error, when the failure occurs, then Talia sees "Couldn't process this refund. Try again." with a Retry action.

**FEAT-30.SPEC-003-AC-11:** Given Talia loses connectivity while viewing this screen, when she attempts to confirm, then she sees "Check your connection and try again." and no commit is attempted.

**FEAT-30.SPEC-003-AC-12:** Given Riley (the Client) has no path to this screen, when the product's screens are reviewed for a reachable control, then none exists.

**FEAT-30.SPEC-003-AC-13:** Given Talia reaches this screen from a dispute timeline (FEAT-16) rather than the no-show prompt, when she confirms, then the refund processes identically regardless of entry point.

**FEAT-30.SPEC-003-AC-14:** Given Talia opens this screen, when the booking and deposit details are being fetched, then she sees a loading placeholder in place of the booking summary and outcome statement.

**FEAT-30.SPEC-003-AC-15:** Given the booking or deposit details fail to load, when the load fails, then Talia sees "Couldn't load this booking. Try again." with a Retry action, and no confirm control is offered until it succeeds.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 9 (loading, ready, ineligible, committing, committed, committed-in-progress, rejected, error, offline) | 9 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Screen Spec: Book Client In

## Overview

**Name:** Book Client In
**ID:** FEAT-30.SPEC-004
**Type:** Screen
**Purpose:** Talia chooses a service, a time, and an existing or new client to book the client in directly, then issues a held deposit request (link or on-screen scan code) instead of collecting payment inside this screen.
**Parent Feature:** FEAT-30 -- Pro Booking Management

## Scope and Non-Goals

**In Scope:**
- Choosing a service, a genuinely free time (with the Pro-only notice/horizon exception), and an existing or new client
- A quick existing-client lookup by phone number
- Capturing a new client's name and phone (and email, if texting is declined)
- Choosing the deposit-request delivery method (link by text/email, or an on-screen code)
- Saving the booking-in action and reflecting its success, contention, or failure

**Non-Goals:**
- Computing which times are genuinely free -- owned by FEAT-03 (Real-Time Slot Availability Engine); this screen only displays what FEAT-03 returns, with the exception FEAT-30.SPEC-006 states
- Creating the Booking record, placing the slot hold, or issuing the deposit request itself -- owned by FEAT-30.SPEC-010 (Pro-Created Booking & Deposit Request Hold) and FEAT-30.SPEC-013 (Deposit Request & Expiry Notice); this screen triggers those but does not implement them
- A full searchable client list -- owned by FEAT-13 (Client Record Management)'s Client List Search & Filter (FEAT-24); this screen offers only a quick inline lookup while booking someone in
- Collecting the client's deposit payment -- owned by FEAT-07 (Deposit Payment at Booking), reached by the client through the issued request, not inside this screen

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-12 (Pro Daily Schedule Dashboard) | Talia opens "Book client in" from the schedule | None -- form starts empty |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | All actions: choose service, time, client, delivery method, and save | -- |
| The Client (Riley) | No | No | No control on any Client-facing surface reaches this screen; a client is booked in by the Pro, never by themselves through this screen |
| Platform Operator (Support) | Full screen, read-only, reached only through FEAT-19's account view | View only | Save control is not shown, consistent with SC-05 |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29) |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- entered form data is preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Back arrow (returns to FEAT-12) with the title "Book client in" and a "Save" action button (right-aligned, disabled until service, time, and client are all set).

**Body:** A single-column flow with the following sections in order:
- Service selector (list of the Pro's Active services, per FEAT-01)
- Time selector: once a service is chosen, a live list of available time slots for that service, grouped by day, with Talia's own notice/horizon exception applied (the same slot-list mechanism as FEAT-05.SPEC-002 and FEAT-30.SPEC-002)
- Client selector: a phone-number lookup field for an existing client, or a "New client" toggle revealing Name (required), Phone (required), and Email (required only if the texting-consent toggle below is left off) fields
- Texting-consent toggle for a new client, unchecked by default (mirroring FEAT-05's own opt-in default)
- Deposit-request delivery choice: "Send a link" (by text if consented, otherwise email) or "Show a code to scan" -- a single selection, defaulting to "Send a link"

**Footer:** None -- Save is in the header.

### Responsive Behavior

- **Compact breakpoint:** Single-column flow as described, full width.
- **Medium size class and above:** Sections remain single-column, capped at a consistent platform-wide form width and horizontally centered; the slot list may show more times per row within the same day grouping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-12 | Screen closes | Standard backward transition |
| Service selector | Select | Sets the chosen service; reveals the time selector | Time selector appears, live slot list requested | Standard selection state |
| Time slot | Tap | Re-validates the candidate against FEAT-03.SPEC-004 (with the Pro-only notice/horizon exception) | Slot selected | If valid: selection shown. If contested: plain "just taken" message, list refreshed |
| Client phone lookup | Type, then select a match | Looks up an existing Client by phone-number match | Existing client's name shown as the selection | Matching client's name displayed; no match shows "No client found -- add as new" |
| "New client" toggle | Tap | Reveals Name, Phone, Email fields for entry | Existing-client lookup hidden | Standard toggle state |
| Name / Phone / Email fields | Type | Captures new client details | Field shows entered text | Standard input focus state |
| Texting-consent toggle | Tap | Sets whether the new client is offered text delivery | Toggle state changes; Email field becomes required if left off | Email field shows required indicator when consent is off |
| Deposit-request delivery choice | Select | Sets link vs. on-screen code for the request | Selected option highlighted | Standard selection state |
| Save button | Tap | 1. Validates all fields via FEAT-30.SPEC-006/FEAT-03.SPEC-004 (slot fit and notice/horizon exception). 2. Triggers FEAT-30.SPEC-010 (Pro-Created Booking & Deposit Request Hold). | Button shows loading state during save | Success: confirmation shown, then navigate to FEAT-12. Contested slot: plain "just taken" message, list refreshed, save not submitted. Failure: retry prompt shown |
| Save button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> Service selector -> Time selector (once revealed) -> Client phone lookup / New client toggle -> Name -> Phone -> Email (when shown) -> Texting-consent toggle -> Deposit-request delivery choice -> Save.
- **Announcements:** Field validation errors, the "just taken" contention message, and save success/failure are announced to assistive technology.
- **Keyboard alternatives:** Every field, toggle, and slot is reachable and selectable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Empty (default) | Service selector shown, all other sections hidden or disabled, Save disabled | Screen first opens | Talia selects a service |
| Selecting time | Live slot list shown for the chosen service | Talia selects a service | Talia selects a time, or changes the service |
| Selecting client | Phone lookup and "New client" toggle available | Talia selects a valid time | Talia selects an existing client or completes new-client fields |
| Ready to save | All required fields complete, Save enabled | Service, time, and client (existing or complete new-client entry) are all set | Talia taps Save or navigates away |
| Slot contested | A plain message "That time was just taken." appears, time selector refreshes | The selected slot is lost to another booking before Save completes | Talia picks a different time |
| Saving | Save button shows loading state, form fields disabled | Talia taps Save | Save completes (success, contested, or failure) |
| Saved | Confirmation shown ("Booking created. {client_name} will get a request to pay their deposit.") before returning to FEAT-12 | FEAT-30.SPEC-010 reports the booking and hold created successfully | Talia is navigated to FEAT-12 |
| Error | Inline error banner "Couldn't save this booking. Try again." with a Retry action | FEAT-30.SPEC-010 reports a processing failure | Talia taps Retry and it succeeds, or she backs out |
| Offline/Degraded | Banner "Check your connection and try again." replaces the Save control; entered form data remains visible and editable | Connectivity is lost while the screen is open | Connectivity is restored and Talia can save |

## Validation Rules

Slot validity and the notice/horizon exception governed by FEAT-03.SPEC-004 and FEAT-30.SPEC-006. See those specs for the exact conditions. The remaining fields on this screen are simple, screen-local validations not warranting a standalone spec:

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| Service selector | Required | On Save | "Choose a service." |
| Time selector | A candidate must pass FEAT-03.SPEC-004's fit rule | On selection and again on Save | "That time doesn't fit -- pick another." |
| Client (existing or new) | Required -- one of an existing-client match or a complete new-client entry | On Save | "Choose an existing client or add a new one." |
| New client -- Name | Required, 1-100 characters, when adding a new client | On blur | "Client name is required." |
| New client -- Phone | Required, valid reachable format, when adding a new client | On blur | "Enter a valid phone number." |
| New client -- Email | Required when the texting-consent toggle is off, optional otherwise | On blur / On submit | "Email is required when texting isn't enabled." |
| Deposit-request delivery choice | Required (defaults to "Send a link") | Always set | -- |

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|-------------------------------------|
| Back arrow tap | FEAT-12 (Pro Daily Schedule Dashboard) | -- |
| Successful save | FEAT-12 (Pro Daily Schedule Dashboard) | -- |

## Data Model

**Creates:** Client (new client path only) -- name, phone, optional email, per FEAT-30.SPEC-010's downstream creation; Booking -- created by FEAT-30.SPEC-010 on save, not by this screen directly.
**Reads:** Service -- Active services list (FEAT-01); live slot list for the chosen service (FEAT-03); Client -- existing-client lookup by phone.
**Updates:** None directly.
**Deletes:** None.

## Business Rules

- XBR-03: Talia may book a client in inside her own minimum_booking_notice or beyond her booking_horizon; the candidate slot's fit rule is never exempted.
- XBR-05: the deposit amount is computed exactly from the chosen service's rule and is never entered or altered by Talia on this screen.
- XBR-15: the deposit-request delivery choice of "Send a link" resolves to text only if the client has active texting consent; otherwise it resolves to email automatically, per FEAT-14's consent rule -- Talia's choice is "link vs. code," not "text vs. email."
- A new client entered here carries the same data-sensitivity treatment as any Client record (SC-03): visible only to Talia, never to any other Pro or client.
- The "just taken" recovery message on a contested slot matches the same wording used by any other booking path (FEAT-05, FEAT-10), per the feature's Shared UI Pattern.

## Edge Cases

- **Talia's chosen slot is taken by a client-side booking between her selection and Save** -- The save re-validates the candidate; if it is no longer available, Talia sees the plain "just taken" message and a refreshed list, never a silent failure or a payment error (XBR-01).
- **Talia looks up an existing client by phone and finds no match** -- She sees "No client found -- add as new" and can switch directly to the new-client fields with the phone number carried over.
- **Talia enters a new client's phone number that already matches an existing client for her account** -- The phone-number match resolves to the existing Client record rather than creating a duplicate (dependency map's Client Contention rule); Talia is shown the matched existing client instead.
- **Talia navigates away with fields partially filled** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Talia taps Save twice rapidly** -- The second tap is ignored while the first save is in progress.
- **Network failure during save** -- Error banner: "Couldn't save this booking. Try again." with a Retry button; entered form data is preserved.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-30.SPEC-006 (Pro Booking Action Rules) | References (inbound) | Notice/horizon exception applied to slot re-validation |
| FEAT-30.SPEC-010 (Pro-Created Booking & Deposit Request Hold) | Triggers (outbound) | Save triggers Booking and hold creation |
| FEAT-03 (Real-Time Slot Availability Engine) | References (inbound) | Source of the live slot list this screen displays |
| FEAT-01 (Service & Pricing Management) | References (inbound) | Source of the Active services list |
| FEAT-13 (Client Record Management) | References (inbound) | The quick existing-client lookup; the full searchable list belongs there |
| FEAT-30.SPEC-013 (Deposit Request & Expiry Notice) | Triggers (outbound) | Talia's delivery-method choice governs which content variant this notification sends |
| FEAT-12 (Pro Daily Schedule Dashboard) | Navigation (inbound/outbound) | Entry point and return destination |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-------------------|
| pro_book_client_in_started | () | Screen loads | supports success-metrics.md: "Pro Change Correctness" |
| pro_book_client_in_saved | client_type (existing / new), delivery_method (link / code) | Save completes successfully | supports success-metrics.md: "Pro Change Correctness" |
| pro_book_client_in_slot_lost_to_contention | () | The chosen slot is lost between selection and save | supports success-metrics.md: "Zero Double-Booking Confidence" |

## Acceptance Criteria

**FEAT-30.SPEC-004-AC-01:** Given Talia opens this screen and selects a service, when the time selector appears, then she sees a live slot list for that service with her own notice/horizon exception applied.

**FEAT-30.SPEC-004-AC-02:** Given Talia selects a service, a time, and an existing client by phone lookup, when she taps Save, then the booking is created, a deposit request is issued, and she sees confirmation before returning to FEAT-12.

**FEAT-30.SPEC-004-AC-03:** Given Talia's phone lookup finds no matching client, when she views the result, then she sees "No client found -- add as new" and can switch to new-client entry with the phone number carried over.

**FEAT-30.SPEC-004-AC-04:** Given Talia enters a new client and leaves the texting-consent toggle off, when she reaches the Email field, then it is required, and Save is blocked until it is filled.

**FEAT-30.SPEC-004-AC-05:** Given Talia enters a new client's phone number that matches an existing client's record, when the match is detected, then she is shown the existing client instead of creating a duplicate.

**FEAT-30.SPEC-004-AC-06:** Given Talia selects a time inside her own minimum_booking_notice, when the candidate is re-validated, then it is accepted, per the Pro-only exception.

**FEAT-30.SPEC-004-AC-07:** Given Talia's selected slot is taken by another booking before she saves, when the save attempt processes, then she sees "That time was just taken." and a refreshed slot list, and no booking is created.

**FEAT-30.SPEC-004-AC-08:** Given Talia chooses "Show a code to scan" as the delivery method, when the save completes, then a scannable code is shown rather than a text or email being sent.

**FEAT-30.SPEC-004-AC-09:** Given Talia chooses "Send a link" for a client without texting consent, when the deposit request is issued, then it delivers by email automatically, per XBR-15.

**FEAT-30.SPEC-004-AC-10:** Given Talia navigates away with fields partially filled, when she attempts to leave, then a confirmation dialog "You have unsaved changes. Discard?" appears with "Discard" and "Keep Editing" options.

**FEAT-30.SPEC-004-AC-11:** Given Talia taps Save twice rapidly, when the first save is in progress, then the second tap has no effect until the first resolves.

**FEAT-30.SPEC-004-AC-12:** Given the save fails due to a processing error, when the failure occurs, then Talia sees "Couldn't save this booking. Try again." with a Retry action, and her entered data is preserved.

**FEAT-30.SPEC-004-AC-13:** Given Talia loses connectivity while filling this screen, when she attempts to save, then she sees "Check your connection and try again." and nothing is submitted.

**FEAT-30.SPEC-004-AC-14:** Given Riley (the Client) has no path to this screen, when the product's screens are reviewed for a reachable control, then none exists.

**FEAT-30.SPEC-004-AC-15:** Given Talia has not yet selected a service, time, and client, when she looks at the Save button, then it is disabled.

**FEAT-30.SPEC-004-AC-16:** Given the deposit amount for the chosen service, when the booking is created, then it is computed exactly from the Service's rule and never entered or altered by Talia on this screen.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 9 | 9 |
| States | 9 (empty, selecting time, selecting client, ready, contested, saving, saved, error, offline) | 9 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Screen Spec: Cancel Several Bookings at Once

## Overview

**Name:** Cancel Several Bookings at Once
**ID:** FEAT-30.SPEC-005
**Type:** Screen
**Purpose:** Talia reviews the set of bookings a new time block conflicts with and confirms cancelling them together, seeing the full-refund outcome for each before confirming.
**Parent Feature:** FEAT-30 -- Pro Booking Management

## Scope and Non-Goals

**In Scope:**
- Reviewing the conflicting bookings a new Time Block overlaps, as handed over by FEAT-17
- Showing the always-full-refund outcome for each booking before Talia confirms
- Capturing an optional shared private cancellation reason
- Triggering the bulk commit and reflecting its per-booking outcome

**Non-Goals:**
- Creating the Time Block itself -- owned by FEAT-17 (Manual Time Blocking); this screen only reviews the bookings that block already conflicts with
- Cancelling a single booking outside a time-block conflict -- owned by FEAT-30.SPEC-001 (Cancel Booking, Pro-Initiated), a distinct one-booking flow
- Letting Talia keep a conflicting booking as an exception to the block instead of cancelling it -- owned by FEAT-17's own conflict-resolution choice, which routes to this screen only when Talia chooses to cancel the affected bookings

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-17.SPEC-003 (Time Block Conflict Review) (Manual Time Blocking) | Talia's new time block conflicts with existing bookings and she chooses to cancel the affected ones | The set of conflicting Booking references |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen, for bookings she owns only | Confirm cancelling the reviewed set, or back out with nothing changed | -- |
| The Client (Riley) | No | No | No control on any Client-facing surface reaches this screen |
| Platform Operator (Support) | Full screen, read-only, reached only through FEAT-19's account view | View only | Confirm control is not shown, consistent with SC-05 |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29) |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- an unconfirmed reason is discarded, since no cancellation has been submitted yet |

## Layout and Content

**Header:** Back arrow (returns to FEAT-17) with the title "Cancel {count} conflicting bookings."

**Body:** A list of the conflicting bookings, one row per booking -- client name, service, date and time, deposit amount -- each row showing the plain outcome statement "{client_name}'s deposit will be refunded in full." Below the list, an optional single-line text field labeled "Reason (private, shared reason for all)." Below that, a "Cancel all" confirm button and a "Never mind" link to back out.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Booking list in a single column, full width, one row per booking; confirm controls stack below the list.
- **Medium size class and above:** List remains single-column, capped at a consistent platform-wide width and horizontally centered; no structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-17 (Manual Time Blocking) | Screen closes | Standard backward transition |
| Reason field | Type | Captures optional shared private text | Field shows entered text | Standard input focus state |
| "Cancel all" button | Tap | Triggers FEAT-30.SPEC-006 eligibility re-check per booking, then FEAT-30.SPEC-008 (Bulk Cancellation Commit) | Button shows loading state during commit | Per-booking outcome shown: succeeded rows marked cancelled, failed rows show their specific denial reason |
| "Cancel all" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |
| A failed booking row's "View" action | Tap | Navigate to FEAT-30.SPEC-001 (Cancel Booking, Pro-Initiated) for that individual booking | Screen navigates | Talia can act on the failed booking separately |
| "Never mind" link | Tap | Navigate to FEAT-17 with nothing changed | Screen closes | Standard backward transition |

### Accessibility Notes

- **Focus order:** Back arrow -> booking list rows top to bottom, each with its outcome statement -> Reason field -> Cancel all button -> Never mind link.
- **Announcements:** The per-booking outcome summary is announced to assistive technology once the bulk commit completes.
- **Keyboard alternatives:** Every action on this screen, including each failed row's "View" action, is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Loading | A neutral loading placeholder in place of the booking list | Screen opens | The conflicting bookings' data loads successfully or a load error occurs |
| Ready | Booking list with per-row outcome statements and confirm/back-out controls shown | The conflicting bookings' data loads successfully with at least one eligible | Talia confirms or backs out |
| All ineligible | Every booking row shows a denial message in place of the outcome statement; "Cancel all" is disabled | Every booking in the set fails eligibility on screen entry | Talia backs out (only path forward) |
| Committing | Confirm button shows a loading state, all inputs disabled | Talia taps "Cancel all" | Bulk commit completes |
| Committed -- all succeeded | Every row shows "Cancelled -- refund confirmed" before returning to FEAT-17 | FEAT-30.SPEC-008 reports every booking succeeded | Talia is navigated to FEAT-17 |
| Committed -- partial success | Succeeded rows show "Cancelled -- refund confirmed"; failed rows show their specific reason with a "View" action to act on them individually | FEAT-30.SPEC-008 reports at least one failure alongside successes | Talia reviews the summary and either backs out or opens a failed row |
| Error | Inline error banner "Couldn't load these bookings. Try again." or "Couldn't process this cancellation. Try again." with a Retry action | The conflicting bookings' data fails to load, or the whole-set commit fails to begin for a processing reason | Talia taps Retry and it succeeds, or she backs out |
| Offline/Degraded | Banner "Check your connection and try again." replaces the confirm control; the booking list remains viewable read-only | Connectivity is lost while the screen is open | Connectivity is restored and Talia can confirm |

## Validation Rules

Eligibility per booking (ownership, and Booking.state is Confirmed or Awaiting Outcome) governed by FEAT-30.SPEC-006 (Pro Booking Action Rules). See that spec for the exact conditions and denial messages. This screen applies the eligibility check per booking on entry and again on confirm.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|-------------------------------------|
| Back arrow tap | FEAT-17 (Manual Time Blocking) | -- |
| "Never mind" tap | FEAT-17 (Manual Time Blocking) | -- |
| Bulk cancellation completes (fully or partially) | FEAT-17 (Manual Time Blocking) | -- |
| Failed row's "View" action | FEAT-30.SPEC-001 (Cancel Booking, Pro-Initiated) | -- |

## Data Model

**Reads:** Booking (state, start_time, service, client, deposit_amount) for each booking in the conflicting set, handed over by FEAT-17; Deposit Transaction -- status, per booking (eligibility precondition).
**Creates:** None directly.
**Updates:** None directly -- delegated entirely to FEAT-30.SPEC-008 on confirm.
**Deletes:** None.

## Business Rules

- XBR-09: every cancellation this screen commits refunds its client's deposit in full, whatever the timing -- shown per booking before Talia confirms, per the feature's "see the outcome before confirming" shared UI pattern.
- Each booking's eligibility and outcome is independent -- one booking's failure never blocks or reverses the others, per FEAT-30.SPEC-008's per-booking processing.
- Eligibility per booking is governed by FEAT-30.SPEC-006 and re-checked on entry and again on confirm.
- The optional shared reason, if provided, is recorded identically on every successfully cancelled booking; it is never shown to any client.

## Edge Cases

- **One booking in the set is already Completed when Talia opens this screen** -- That row shows the completed-state denial message from FEAT-30.SPEC-006 in place of the outcome statement; the remaining eligible bookings can still be cancelled if Talia confirms.
- **A client cancels one of the set's bookings through FEAT-10 while Talia is reviewing this screen** -- The confirm-time eligibility re-check for that specific booking fails (reject-with-refresh); it is reported as a failed outcome while the rest of the set proceeds normally.
- **Talia navigates away with the reason field filled in but unconfirmed** -- No confirmation dialog is shown, since no cancellation has been submitted; nothing changes.
- **Talia taps "Cancel all" twice rapidly** -- The second tap is ignored while the first commit is in progress.
- **The set contains only one conflicting booking** -- The screen and its per-row outcome mechanics render identically to a larger set; FEAT-17 is what decides whether a single conflicting booking routes here or to FEAT-30.SPEC-001 directly.
- **Network failure during the bulk commit** -- Error banner shown: "Couldn't process this cancellation. Try again." with Retry; no booking in the set is affected.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-30.SPEC-006 (Pro Booking Action Rules) | References (inbound) | Per-booking eligibility check |
| FEAT-30.SPEC-008 (Bulk Cancellation Commit) | Triggers (outbound) | Confirm triggers the bulk commit |
| FEAT-30.SPEC-001 (Cancel Booking, Pro-Initiated) | Navigation (outbound) | A failed booking row can be revisited individually here |
| FEAT-17 (Manual Time Blocking) | Navigation (inbound/outbound) | Hands over the conflicting booking set; return destination after review |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-------------------|
| pro_bulk_cancel_screen_opened | booking_count | Screen loads | supports success-metrics.md: "Pro Change Correctness" |
| pro_bulk_cancel_confirmed | booking_count, success_count, failure_count | Talia taps "Cancel all" and the commit completes | supports success-metrics.md: "Pro Change Correctness" |
| pro_bulk_cancel_row_failed | reason category | A booking in the set fails eligibility | supports success-metrics.md: "Automatic Refund Correctness" |

## Acceptance Criteria

**FEAT-30.SPEC-005-AC-01:** Given Talia opens this screen for four bookings her new time block conflicts with, when the screen loads, then she sees each booking listed with the plain statement that its client's deposit will be refunded in full.

**FEAT-30.SPEC-005-AC-02:** Given Talia confirms "Cancel all" and every booking passes eligibility, when the commit completes, then every row shows "Cancelled -- refund confirmed" and she returns to FEAT-17.

**FEAT-30.SPEC-005-AC-03:** Given one booking in the set is already Completed, when the screen loads, then that row shows the completed-state denial message while the others show the standard outcome statement.

**FEAT-30.SPEC-005-AC-04:** Given Talia confirms "Cancel all" and one booking fails eligibility while the others succeed, when the commit completes, then the failed row shows its specific reason with a "View" action, and the succeeded rows show "Cancelled -- refund confirmed."

**FEAT-30.SPEC-005-AC-05:** Given a booking failed within this bulk cancellation, when Talia taps its "View" action, then she is taken to FEAT-30.SPEC-001 for that individual booking.

**FEAT-30.SPEC-005-AC-06:** Given Riley cancels one of the set's bookings through FEAT-10 while Talia is reviewing this screen, when Talia confirms "Cancel all", then that booking is reported as a failed outcome while the rest of the set is cancelled successfully.

**FEAT-30.SPEC-005-AC-07:** Given Talia enters a shared reason and confirms, when the commit succeeds, then that same reason is recorded on every successfully cancelled booking and never shown to any client.

**FEAT-30.SPEC-005-AC-08:** Given every booking in the set fails eligibility, when the screen loads, then "Cancel all" is disabled and every row shows its denial message.

**FEAT-30.SPEC-005-AC-09:** Given Talia taps "Cancel all" twice rapidly, when the first commit is in progress, then the second tap has no effect until the first resolves.

**FEAT-30.SPEC-005-AC-10:** Given the whole-set commit fails due to a processing error, when the failure occurs, then Talia sees "Couldn't process this cancellation. Try again." with a Retry action, and no booking in the set is affected.

**FEAT-30.SPEC-005-AC-11:** Given Talia loses connectivity while reviewing this screen, when she attempts to confirm, then she sees "Check your connection and try again." and no commit is attempted.

**FEAT-30.SPEC-005-AC-12:** Given Riley (the Client) has no path to this screen, when the product's screens are reviewed for a reachable control, then none exists.

**FEAT-30.SPEC-005-AC-13:** Given Talia taps "Never mind", when the navigation completes, then she returns to FEAT-17 with nothing changed.

**FEAT-30.SPEC-005-AC-14:** Given Talia opens this screen, when the conflicting bookings' data is being fetched, then she sees a loading placeholder in place of the booking list.

**FEAT-30.SPEC-005-AC-15:** Given the conflicting bookings' data fails to load, when the load fails, then Talia sees "Couldn't load these bookings. Try again." with a Retry action, and no "Cancel all" control is offered until it succeeds.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 8 (loading, ready, all-ineligible, committing, committed-all, committed-partial, error, offline) | 8 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Pro Booking Action Rules

## Overview

**Name:** Pro Booking Action Rules
**ID:** FEAT-30.SPEC-006
**Type:** Logic/Rule
**Purpose:** Governs the eligibility, ownership, and limits shared by every Pro-initiated booking action -- cancel, reschedule, goodwill refund, book-client-in, and bulk cancel -- so each screen and automation in this feature references one authoritative set of rules instead of restating them.
**Parent Feature:** FEAT-30 -- Pro Booking Management
**Governed Entity:** Booking (the Pro-action eligibility slice: `state` and `start_time`), with Deposit Transaction.status and Availability Rule's notice/horizon fields read as cross-entity preconditions

## Scope and Non-Goals

**In Scope:**
- The Pro-only exception to minimum booking notice and booking horizon for reschedule (FEAT-30.SPEC-002) and book-client-in (FEAT-30.SPEC-004)
- The Pro-created deposit-request hold window (24 hours or 2 hours before the appointment, whichever comes first)
- The once-only refund limit shared by every refund path this feature triggers or executes
- The until-completion availability window for a goodwill refund
- The completed/auto-completed cutoff that blocks cancel, reschedule, and new bulk-cancel actions
- Ownership and authorization for every action this feature defines, by role

**Non-Goals:**
- Determining the deposit outcome's substance (refund vs. keep) for a single Pro cancellation or reschedule -- owned by FEAT-09 (Cancellation & No-Show Policy Engine, XBR-09); this spec governs only whether the triggering action is currently eligible, not the financial outcome it produces
- Creating or expiring the Pro-created deposit-request hold itself -- owned by FEAT-03.SPEC-007 (Pro-Created Deposit Request Hold & Expiration); this spec states the shared window value that spec computes against, not the hold mechanics
- Governing the Booking's transition into `Completed` -- owned by FEAT-12.SPEC-006 (Booking Completion Rules); this spec only reads that boundary as the cutoff for its own actions
- The no-show marking window and its own 24-hour undo grace period -- owned by FEAT-11.SPEC-004 (No-Show Marking Window & Authorization Rules); this spec governs a different action (goodwill refund) that happens to interact with the same Deposit Transaction

## Governed Entity

**Entity:** Booking (Pro-action eligibility slice), with Deposit Transaction and Availability Rule read as cross-entity preconditions
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|--------------|
| state | enum | Pending Payment \| Confirmed \| Awaiting Outcome \| Completed \| No-Show \| Cancelled by Client \| Cancelled by Pro \| Rescheduled \| Expired (unpaid) -- cancel/reschedule eligibility depends on this value |
| start_time | date/time | Appointment start time, Pro's timezone -- the anchor for the deposit-request hold's cutoff and the auto-completion boundary read from FEAT-12.SPEC-006 |
| owning Pro Account | reference | The Pro who owns this booking -- the ownership condition for every action in this spec |
| service, duration, client, price_agreed/deposit_amount, policy_version, attendance_reply, balance_due, source, cancellation/reschedule timestamps and optional private Pro reason | various | Not evaluated as eligibility conditions by this spec; read and written by the automations this spec governs (FEAT-30.SPEC-007/008/009/010) |

**Cross-entity precondition (Deposit Transaction):**

| Field | Data Type | Description |
|-------|-----------|-------------|
| status | enum | Authorized \| Captured \| Applied \| Refunded \| Refund in Progress \| Forfeited \| Disputed -- must be Captured or Forfeited for a goodwill refund to be eligible; a deposit already Refunded, Refund in Progress, or Disputed can never be refunded again (once-only limit) |

**Cross-entity reference (Availability Rule, read-only):**

| Field | Data Type | Description |
|-------|-----------|-------------|
| minimum_booking_notice | number (days) | Exempted for the Pro when rescheduling (FEAT-30.SPEC-002) or booking a client in (FEAT-30.SPEC-004) |
| booking_horizon | number (weeks/months) | Exempted for the Pro when rescheduling or booking a client in |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-30.SPEC-001 | Cancel Booking (Pro-Initiated) | On screen entry (eligibility gate) and again on confirm |
| FEAT-30.SPEC-002 | Reschedule Booking (Pro-Initiated) | On screen entry, on candidate-slot selection (notice/horizon exception), and again on confirm |
| FEAT-30.SPEC-003 | Goodwill Deposit Refund | On screen entry (until-completion, once-only limits) and again on confirm |
| FEAT-30.SPEC-004 | Book Client In | On candidate-slot selection (notice/horizon exception) |
| FEAT-30.SPEC-005 | Cancel Several Bookings at Once | On screen entry, per booking in the reviewed set, and again on confirm |
| FEAT-30.SPEC-007 | Pro Cancel/Reschedule Commit | Re-checks eligibility and ownership immediately before the atomic write |
| FEAT-30.SPEC-008 | Bulk Cancellation Commit | Re-checks eligibility and ownership per booking immediately before each atomic write |
| FEAT-30.SPEC-009 | Goodwill Refund Commit | Re-checks the once-only and until-completion limits immediately before the atomic write |
| FEAT-30.SPEC-010 | Pro-Created Booking & Deposit Request Hold | Reads the deposit-request hold window this spec states, and applies the notice/horizon exception to the candidate slot |
| FEAT-03.SPEC-007 | Pro-Created Deposit Request Hold & Expiration | Consumes this spec's stated hold-window values and notice/horizon exception when creating and expiring the hold |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| Booking.state (cancel/reschedule/bulk-cancel actions) | Must be Confirmed or Awaiting Outcome | Cancel, reschedule, and bulk-cancel actions only | On screen entry and again on confirm | "This booking is already completed and can no longer be changed." (state is Completed or No-Show) / "This booking has already been cancelled or rescheduled." (state is Cancelled by Client, Cancelled by Pro, Rescheduled, or Expired) | Yes |
| Booking.state (goodwill refund action) | Must not be Completed | Goodwill refund only | On screen entry and again on confirm | "A goodwill refund is no longer available once a booking is completed." | Yes |
| Deposit Transaction.status (goodwill refund action) | Must be Captured or Forfeited | Goodwill refund only | On screen entry and again on confirm | "This booking's deposit is not in a state that can be refunded." (already Refunded, Refund in Progress, or Disputed) | Yes |
| owning Pro Account | Must match the requesting Pro's account | All actions | On screen entry and again on confirm | "This booking could not be found." (never reveals another Pro's booking exists, per user-persona.md's "No role can ever see another pro's data") | Yes |
| Candidate slot's minimum_booking_notice (Availability Rule) | Exempted for this feature's Pro-initiated actions | Reschedule (FEAT-30.SPEC-002) and Book Client In (FEAT-30.SPEC-004) only | On candidate-slot re-validation | N/A -- exemption, not a validation failure | No |
| Candidate slot's booking_horizon (Availability Rule) | Exempted for this feature's Pro-initiated actions | Reschedule and Book Client In only | On candidate-slot re-validation | N/A -- exemption, not a validation failure | No |
| Candidate slot's duration + buffer fit | Never exempted -- must pass FEAT-03.SPEC-004's fit rule regardless of who is booking | Reschedule and Book Client In only | On candidate-slot re-validation | "That time doesn't fit -- pick another." (per FEAT-03.SPEC-004's own wording) | Yes |
| service, duration, client, price_agreed/deposit_amount, policy_version, attendance_reply, balance_due, source, cancellation/reschedule timestamps | No validation beyond data type | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Once-only refund limit | Deposit Transaction.status | Any refund this feature triggers or executes (cancel-with-refund, reschedule-carry-over, goodwill, bulk cancel) is refused once the Deposit Transaction has already reached Refunded, Refund in Progress, or Disputed for that booking -- a deposit is refunded at most once (dependency map's Deposit Transaction Contention rule) | "This booking's deposit has already been resolved and cannot be refunded again." |
| Goodwill availability spans both pre- and post-no-show states | Booking.state, Deposit Transaction.status | A goodwill refund is available whenever Deposit Transaction.status is Captured (offered instead of marking a no-show, or against an as-yet-unresolved inside-window client cancellation) or Forfeited (converting an already-kept deposit -- from a no-show mark or an inside-window client cancellation -- into a full refund), for as long as Booking.state has not reached Completed | "A goodwill refund is no longer available once a booking is completed." |
| Deposit-request hold window is the earlier of two limits | Booking.start_time, hold-created timestamp | The Pro-created deposit-request hold's expiry (computed and enforced by FEAT-03.SPEC-007) is the earlier of platform parameter: `deposit-request-hold-max-hours` from creation, or platform parameter: `deposit-request-hold-appointment-cutoff-hours` before the appointment | N/A -- this spec states the value; FEAT-03.SPEC-007 enforces it and owns its own messaging |
| Notice/horizon exception never exempts the fit rule | Candidate slot's duration + buffer, Availability Rule | The Pro-only exception to minimum_booking_notice and booking_horizon never extends to whether the candidate slot's full duration plus buffer genuinely fits an open window with no conflict (XBR-03) | "That time doesn't fit -- pick another." |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Cancel a booking (single) | The Pro | Only bookings the requesting Pro owns, and only while Booking.state is Confirmed or Awaiting Outcome | If ownership fails: "This booking could not be found." If the state condition fails: the exact message from Field Validation Rules above |
| Reschedule a booking | The Pro | Only bookings the requesting Pro owns, only while Booking.state is Confirmed or Awaiting Outcome, and only to a candidate slot that passes FEAT-03.SPEC-004's fit rule (notice/horizon exempted) | Same as Cancel above; a candidate slot that fails the fit rule shows "That time doesn't fit -- pick another." |
| Issue a goodwill refund | The Pro | Only bookings the requesting Pro owns, only while Booking.state has not reached Completed, and only while Deposit Transaction.status is Captured or Forfeited | If ownership fails: "This booking could not be found." If the state or deposit-status condition fails: the exact messages from Field Validation Rules above |
| Book a client in (create a new Booking) | The Pro | Always, for the Pro's own account, to a candidate slot that passes FEAT-03.SPEC-004's fit rule (notice/horizon exempted) | A candidate slot that fails the fit rule shows "That time doesn't fit -- pick another." |
| Cancel several bookings at once | The Pro | Only bookings the requesting Pro owns, evaluated per booking against the same state condition as a single cancellation | Per booking: the exact messages from Field Validation Rules above; a booking that fails is reported as a failed outcome in the reviewed set (FEAT-30.SPEC-005/008) while the others still proceed |
| Any action in this spec | The Client | Never | No control for any of these actions is reachable from any Client-facing surface; the Client's Booking & Payment access is Own-only, exercised entirely through FEAT-10 (Client-Initiated Cancel/Reschedule), never through this feature |
| Any action in this spec | Platform Operator (Support) | Never | No action control exists anywhere on Support's read-only surfaces (FEAT-19); Support's access to Booking & Payment and Cancellation & No-Show Handling is View-only, per SC-05 |
| View the outcome of any action in this spec (not the action itself) | The Client | Own-only, for their own affected booking | -- |
| View the outcome of any action in this spec (not the action itself) | Platform Operator (Support) | View-only, through FEAT-19 | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Deposit-request hold expiry | The earlier of (creation time + platform parameter: `deposit-request-hold-max-hours`) and (appointment start time − platform parameter: `deposit-request-hold-appointment-cutoff-hours`) | Computed by FEAT-03.SPEC-007 whenever FEAT-30.SPEC-010 creates a Pro-created deposit request | No -- one value for every Pro and every booking |
| Notice/horizon exception applicability | Derived from the requesting role: applied automatically whenever the triggering action is FEAT-30.SPEC-002 (reschedule) or FEAT-30.SPEC-004 (book-client-in); never applied to any client-facing booking or reschedule path (FEAT-05, FEAT-10) | On every candidate-slot re-validation for this feature's actions | No |
| Goodwill and once-only refund eligibility | Derived entirely from Booking.state and Deposit Transaction.status at the moment of the action -- never a stored flag or manual toggle | On screen entry and again on confirm, for every refund-bearing action | No |

## Business Rules

- XBR-03: minimum booking notice and booking horizon limit every client-facing booking path; the Pro alone may book inside notice or beyond horizon when booking a client in (FEAT-30.SPEC-004) or rescheduling (FEAT-30.SPEC-002).
- XBR-02: the Pro-created deposit-request hold holds its slot for up to platform parameter: `deposit-request-hold-max-hours` or until platform parameter: `deposit-request-hold-appointment-cutoff-hours` before the appointment, whichever comes first; this spec states the value, FEAT-03.SPEC-007 owns the hold mechanics that enforce it.
- XBR-10: refunds are always full and happen at most once per deposit; this spec's once-only limit is the eligibility gate every refund-bearing action in this feature (cancel, reschedule-inside-window-never-applies-per-XBR-09, goodwill, bulk cancel) checks before proceeding.
- XBR-12: a completed or auto-completed booking can no longer be cancelled or rescheduled; a goodwill refund remains available only until completion. This spec is the shared reference every screen and automation in this feature reads instead of restating the rule.
- Ownership is the sole authorization gate among Pros -- there is no tier among Pro accounts; every Pro has identical authority over their own bookings and none over any other Pro's, consistent with FEAT-11.SPEC-004's identical treatment of the same ownership condition.
- A goodwill refund never changes Booking.state -- only Deposit Transaction.status; this distinguishes it from every other action this spec governs, all of which do transition Booking.state (per the Entity-Lifecycle Coverage Matrix).

## Edge Cases

- **Talia's confirm on a cancel or reschedule arrives the instant the Auto-Completion Sweep (FEAT-12.SPEC-004) completes the same booking** -- The eligibility re-check at confirm time (reject-with-refresh, per the Booking entity's Contention rule) finds the booking already Completed and denies with "This booking is already completed and can no longer be changed."; Talia's screen reflects the current state on refresh.
- **Talia attempts a goodwill refund on a booking whose deposit a client-side outside-window cancellation has already refunded automatically (FEAT-09)** -- The Deposit Transaction.status is already Refunded, so the once-only limit denies with "This booking's deposit has already been resolved and cannot be refunded again."
- **Talia reschedules a client to a time inside her own minimum_booking_notice** -- The candidate slot is not excluded on notice grounds (Pro-only exception applies); the fit rule (duration + buffer, no conflict) still applies unexempted.
- **Two eligibility checks for the same booking (screen entry and confirm) disagree because time passed between them** -- The confirm-time check is authoritative; a window or state condition that closed between entry and confirm is denied at confirm even though the screen initially showed the action as available.
- **Talia issues a goodwill refund on a booking already marked No-Show (Deposit Transaction Forfeited)** -- The refund is eligible per the Cross-Field Rule above (Forfeited is a refundable status), converting the kept deposit to Refunded; per FEAT-11.SPEC-004's own edge case, this also closes the no-show mark's undo window (the Deposit Transaction is no longer Forfeited).
- **A bulk-cancel review set includes one booking that has since been completed by the Auto-Completion Sweep** -- That single booking is denied with the standard completed-state message and reported as a failed outcome in the reviewed set (FEAT-30.SPEC-005/008); the other bookings in the set proceed independently.

## Acceptance Criteria

**FEAT-30.SPEC-006-AC-01:** Given Talia opens the Cancel Booking screen for a Confirmed booking she owns, when eligibility is checked, then the cancel action is available with no denial message.

**FEAT-30.SPEC-006-AC-02:** Given Talia opens the Cancel Booking screen for a booking already Completed, when eligibility is checked, then she sees "This booking is already completed and can no longer be changed." and no cancel action is offered.

**FEAT-30.SPEC-006-AC-03:** Given Talia opens the Reschedule screen for a booking already Cancelled by Client, when eligibility is checked, then she sees "This booking has already been cancelled or rescheduled." and no reschedule action is offered.

**FEAT-30.SPEC-006-AC-04:** Given Talia picks a candidate reschedule time that falls inside her own minimum_booking_notice, when the candidate is re-validated, then it is not excluded on notice grounds, per the Pro-only exception.

**FEAT-30.SPEC-006-AC-05:** Given Talia picks a candidate time for booking a client in beyond her own booking_horizon, when the candidate is re-validated, then it is not excluded on horizon grounds, per the Pro-only exception.

**FEAT-30.SPEC-006-AC-06:** Given Talia picks a candidate reschedule time whose duration and buffer do not fit any open window, when the candidate is re-validated, then she sees "That time doesn't fit -- pick another." regardless of the Pro-only exception.

**FEAT-30.SPEC-006-AC-07:** Given a booking's Deposit Transaction is Captured and Booking.state is Awaiting Outcome, when Talia opens the Goodwill Deposit Refund screen, then the goodwill action is eligible.

**FEAT-30.SPEC-006-AC-08:** Given a booking's Deposit Transaction is Forfeited following a no-show mark, when Talia opens the Goodwill Deposit Refund screen, then the goodwill action is still eligible, converting the kept deposit to a full refund.

**FEAT-30.SPEC-006-AC-09:** Given a booking's Deposit Transaction is already Refunded, when Talia opens the Goodwill Deposit Refund screen, then she sees "This booking's deposit is not in a state that can be refunded." and no goodwill action is offered.

**FEAT-30.SPEC-006-AC-10:** Given a booking has reached Completed, when Talia looks for a goodwill refund option, then none is offered, and a direct attempt is denied with "A goodwill refund is no longer available once a booking is completed."

**FEAT-30.SPEC-006-AC-11:** Given Talia attempts to reach any of this feature's screens for a booking belonging to a different Pro account, when the ownership check runs, then it denies with "This booking could not be found." and never reveals the booking exists.

**FEAT-30.SPEC-006-AC-12:** Given Riley (the Client) has no path to any of this feature's screens, when the product's screens are reviewed for a reachable control, then none exists -- her Booking & Payment access remains Own-only, exercised through FEAT-10.

**FEAT-30.SPEC-006-AC-13:** Given Platform Operator (Support) is viewing a booking through FEAT-19's read-only surfaces, when Support looks for any of this feature's action controls, then none is shown.

**FEAT-30.SPEC-006-AC-14:** Given Talia creates a Pro-created deposit request for an appointment more than 24 hours away, when the hold's expiry is computed by FEAT-03.SPEC-007, then it is set to platform parameter: `deposit-request-hold-max-hours` from creation, per this spec's stated window.

**FEAT-30.SPEC-006-AC-15:** Given Talia creates a Pro-created deposit request for an appointment less than platform parameter: `deposit-request-hold-max-hours` away, when the hold's expiry is computed, then it is set to platform parameter: `deposit-request-hold-appointment-cutoff-hours` before the appointment.

**FEAT-30.SPEC-006-AC-16:** Given Talia's confirm on a cancellation arrives at the exact instant the Auto-Completion Sweep completes the same booking, when the confirm-time eligibility check runs, then it denies with the completed-state message rather than proceeding against a stale screen state.

**FEAT-30.SPEC-006-AC-17:** Given a booking's deposit has already been refunded through an automatic outside-window client cancellation (FEAT-09), when Talia attempts a goodwill refund on the same booking, then she sees "This booking's deposit has already been resolved and cannot be refunded again."

**FEAT-30.SPEC-006-AC-18:** Given Talia reviews a bulk-cancel set where one booking has since been completed, when eligibility is checked per booking, then that booking is denied and reported as a failed outcome while the remaining bookings in the set proceed.

**FEAT-30.SPEC-006-AC-19:** Given Talia owns the booking she is acting on, when the ownership check runs for any action in this spec, then it passes and the action-specific eligibility checks proceed.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 8 | 8 |
| Cross-Field Rules | 4 | 4 |
| Authorization Rules | 9 | 9 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |



# Automation Spec: Pro Cancel/Reschedule Commit

## Overview

**Name:** Pro Cancel/Reschedule Commit
**ID:** FEAT-30.SPEC-007
**Type:** Automation
**Purpose:** Commits a Pro-initiated single cancellation or reschedule to the Booking, coordinating the deposit-outcome handoff, calendar mirroring, activity logging, freed-slot handoff, and client notice this triggers.
**Parent Feature:** FEAT-30 -- Pro Booking Management

## Scope and Non-Goals

**In Scope:**
- Committing a Pro-initiated single cancellation (Confirmed/Awaiting Outcome -> Cancelled by Pro)
- Committing a Pro-initiated single reschedule (updating start_time in place, per the Brief's flagged-not-resolved lifecycle discrepancy, carried forward here for Stage 4)
- Handing off the deposit-outcome determination to FEAT-09 (always full refund for a Pro cancellation or reschedule, per XBR-09)
- Refunding an already-paid Balance Payment in full, if one exists (XBR-23, v1)
- Triggering calendar mirroring, activity logging, freed-slot/waitlist handoff, historical-aggregate maintenance, and the client notice this commit causes
- Requesting a fresh booking-specific manage link on a reschedule (XBR-18), issued by FEAT-08.SPEC-010
- Rejecting a commit attempt against a booking a conflicting transition has already resolved (reject-with-refresh)

**Non-Goals:**
- Determining or executing the deposit refund itself -- owned by FEAT-09 (Cancellation & No-Show Policy Engine, FEAT-09.SPEC-004/FEAT-09.SPEC-005); this automation only hands off the cancellation/reschedule event, per XBR-09 ("any Pro cancellation = full refund")
- The eligibility check for whether this action is currently allowed -- owned by FEAT-30.SPEC-006 (Pro Booking Action Rules); this automation re-checks it once more immediately before the atomic write, per that spec's Enforced By table
- Committing several bookings in one action -- owned by FEAT-30.SPEC-008 (Bulk Cancellation Commit), a distinct per-booking-outcome processing shape
- Composing or delivering the client's change notice -- FEAT-30.SPEC-012 is the trigger-and-audience contract and FEAT-08.SPEC-004 owns the content; this automation triggers it but does not define its content
- Generating or storing the fresh manage link -- owned by FEAT-08.SPEC-010 (Booking-Specific Manage Link Issuance); this automation only requests it after a reschedule commits

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Talia confirms a single cancellation | FEAT-30.SPEC-001 (Cancel Booking, Pro-Initiated) | Fires when Talia taps confirm on the cancel screen | Booking reference, optional private Pro reason |
| Talia confirms a single reschedule | FEAT-30.SPEC-002 (Reschedule Booking, Pro-Initiated) | Fires when Talia taps confirm on the reschedule screen, having selected a validated new time | Booking reference, new start_time, duration (unchanged) |

## Processing Logic

1. Receive the triggering action (cancel or reschedule) and the Booking reference from the triggering screen.
2. Re-check eligibility against FEAT-30.SPEC-006 (ownership, and Booking.state is Confirmed or Awaiting Outcome) immediately before the write. If eligibility fails because a conflicting transition already committed, stop and return the reject-with-refresh outcome.
3. **Cancellation path:** Set Booking.state to Cancelled by Pro; record the cancellation timestamp and any optional private Pro reason.
4. **Reschedule path:** Update Booking.start_time to the new, already-validated time; record the reschedule timestamp and any optional private Pro reason. Booking.state is not changed by a reschedule.
5. Check whether a Balance Payment exists for this Booking with status Succeeded (v1; at MVP this check always finds no record, per SC-16). If one exists, request its full refund (XBR-23) through FEAT-22.SPEC-005 (Balance Charge, Payout Routing & Refund), which owns the outbound balance refund; the deposit leg continues to run through its own refund path.
6. Hand off the cancellation-or-reschedule event to FEAT-09 for deposit-outcome evaluation; FEAT-09.SPEC-004 always determines a full-refund outcome for a Pro-initiated action (XBR-09), and FEAT-09.SPEC-005 executes it.
7. Trigger the Pro's personal calendar mirror to remove (cancellation) or move (reschedule) the corresponding entry (FEAT-04, XBR-13).
8. **Reschedule only:** Trigger FEAT-08.SPEC-010 (Booking-Specific Manage Link Issuance) to issue a fresh manage link for this Booking and supersede the previous one (XBR-18); the fresh link is carried into the client notice in step 12.
9. Write an append-only activity event recording the action, its timestamp, and the actor (Talia) (FEAT-16, XBR-21).
10. **Cancellation only:** Hand the freed slot to FEAT-20's waitlist-priority check before it returns to general public availability (XBR-28).
11. Signal FEAT-25.SPEC-004 (Historical Aggregate Maintenance) that the committed cancellation or in-place reschedule changes the Booking's counted state, so revenue and insights aggregates stay current.
12. Trigger FEAT-30.SPEC-012 (Pro Action Client Notice) with the outcome type (cancelled-with-refund or rescheduled-with-new-time), the resulting deposit status, and (reschedule) the fresh manage link from step 8.
13. Return the committed outcome to the triggering screen for its success feedback.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Cancellation committed | Eligibility passes at write time | Booking.state -> Cancelled by Pro; cancellation timestamp and optional reason recorded | Talia sees the cancellation confirmed on FEAT-30.SPEC-001 and returns to her schedule; Riley receives the cancellation-with-refund notice | FEAT-30.SPEC-001, FEAT-09, FEAT-04, FEAT-16, FEAT-20, FEAT-25.SPEC-004, FEAT-30.SPEC-012 |
| Reschedule committed | Eligibility passes at write time | Booking.start_time updated; reschedule timestamp and optional reason recorded; a fresh manage link issued for the Booking and the previous link superseded (FEAT-08.SPEC-010, XBR-18) | Talia sees the reschedule confirmed on FEAT-30.SPEC-002; Riley receives the reschedule-with-new-time notice carrying the fresh manage link | FEAT-30.SPEC-002, FEAT-09, FEAT-04, FEAT-16, FEAT-08.SPEC-010, FEAT-25.SPEC-004, FEAT-30.SPEC-012 |
| Already-paid balance refunded (v1) | A Balance Payment with status Succeeded exists for this Booking | Balance Payment.state -> Refunded | Included in the same client notice as the deposit outcome, never a separate message | FEAT-22, FEAT-30.SPEC-012 |
| Commit rejected -- conflicting transition already won | A client-side action (FEAT-10) commits a conflicting transition first | No data changes | Talia is shown the booking's current state on FEAT-30.SPEC-001/SPEC-002 and must re-decide; the two transitions are never merged | FEAT-30.SPEC-001, FEAT-30.SPEC-002 |
| Commit rejected -- booking no longer eligible | The eligibility re-check fails for a reason other than a concurrent conflict (e.g., the booking auto-completed moments earlier) | No data changes | Talia sees the exact denial message from FEAT-30.SPEC-006 on the triggering screen | FEAT-30.SPEC-001, FEAT-30.SPEC-002, FEAT-30.SPEC-006 |
| Write failure (processing error) | The commit cannot be written for a reason other than an eligibility conflict | No data changes | Talia sees a retry prompt on the triggering screen; the booking remains exactly as it was | FEAT-30.SPEC-001, FEAT-30.SPEC-002 |

## Data Model

**Reads:** Booking (state, start_time, owning Pro Account); Deposit Transaction (status, for the eligibility pass-through FEAT-09 evaluates); Balance Payment (status, v1).
**Creates:** Activity Event (FEAT-16) -- one per committed action. On a reschedule, a fresh Access Link for the Booking is created by FEAT-08.SPEC-010 (this automation requests it and does not write it).
**Updates:** Booking -- state (cancellation only) and/or start_time (reschedule only), plus cancellation/reschedule timestamp and optional private Pro reason. Balance Payment -- state to Refunded when one exists and has Succeeded (v1).
**Deletes:** None -- a cancelled or rescheduled booking is retained as history (SC-22), never deleted.

## Business Rules

- XBR-09: any Pro cancellation refunds the client's deposit in full, whatever the timing; a Pro-made reschedule never exposes the client to the cancellation window -- the deposit always carries over.
- XBR-13: this commit's calendar mirror is triggered on every successful cancellation or reschedule, never silently skipped.
- XBR-21: every commit writes exactly one append-only activity event; the event is never edited or deleted afterward.
- XBR-23 (v1): an already-succeeded Balance Payment is refunded in full alongside the deposit whenever either party cancels -- never forfeited.
- XBR-28: a cancellation's freed slot is handed to waitlist-priority notification (FEAT-20) before it returns to general public availability; a reschedule does not free a slot in the same sense, since the booking continues to exist at its new time.
- XBR-18: every Pro-made reschedule issues a fresh booking-specific manage link through FEAT-08.SPEC-010, and the client notice carries that fresh link, never the superseded one; a cancellation issues no new link.
- The Booking entity's Contention resolution is reject-with-refresh (dependency map): the first committed transition wins, and the automation never merges a Pro-side and a client-side transition on the same booking.
- A reschedule never changes Booking.state -- only start_time and the reschedule timestamp/reason; this distinguishes it from a cancellation, which does transition state.

## Edge Cases

- **A client cancels the same booking through FEAT-10 in the instant before this commit runs** -- Reject-with-refresh: whichever transition commits first wins; the later commit attempt (this automation's) fails the eligibility re-check and Talia is shown the booking's current (client-cancelled) state, never a merged or overwritten outcome.
- **Talia reschedules a booking to a time that becomes contested between her selection and this commit** -- The candidate slot was already re-validated and held by the triggering screen (FEAT-30.SPEC-002, via FEAT-03); if the hold itself has since been lost, the commit fails and Talia is returned to slot selection with a refreshed list, per FEAT-03.SPEC-005.
- **Concurrent trigger firing (Talia cancels two different bookings at effectively the same time from two schedule rows)** -- Each commit processes independently against its own distinct Booking; no interference occurs since the bookings are unrelated records.
- **Trigger fires while a previous commit for the same booking is still in flight** -- The triggering screen's confirm control is disabled during submission (FEAT-30.SPEC-001/SPEC-002), preventing a duplicate commit request for the same action on the same booking.
- **A Balance Payment exists but has not yet succeeded (Attempted or Failed, v1)** -- No refund is requested against it; only a Succeeded Balance Payment is eligible for the automatic refund this commit triggers (XBR-23), consistent with there being nothing paid to reverse otherwise.
- **The booking being cancelled or rescheduled is the last one on Talia's day** -- No special handling; the commit, calendar mirror, activity log, and client notice proceed identically regardless of how many other bookings exist that day.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-30.SPEC-001 (Cancel Booking, Pro-Initiated) | Triggered by (inbound) / Affects (outbound) | Confirm triggers this commit; a rejected or failed commit is shown here |
| FEAT-30.SPEC-002 (Reschedule Booking, Pro-Initiated) | Triggered by (inbound) / Affects (outbound) | Confirm triggers this commit; a rejected or failed commit is shown here |
| FEAT-30.SPEC-006 (Pro Booking Action Rules) | References (outbound) | Eligibility, ownership, and state-cutoff rules re-checked immediately before the write |
| FEAT-09 (Cancellation & No-Show Policy Engine) | Triggers (outbound) | Hands off the cancellation/reschedule event for deposit-outcome evaluation and execution (XBR-09) |
| FEAT-04.SPEC-005 (Booking-to-Calendar Sync) -- within FEAT-04 (Two-Way Calendar Sync) | Triggers (outbound) | A successful commit removes or moves the corresponding personal-calendar entry |
| FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) | Triggers (outbound) | Every commit writes an append-only activity event |
| FEAT-20.SPEC-005 (Cancellation-Triggered Waitlist Matching) -- within FEAT-20 (Waitlist for Cancelled Slots) | Triggers (outbound) | A cancellation's freed slot is handed to waitlist-priority notification first |
| FEAT-30.SPEC-012 (Pro Action Client Notice) | Triggers (outbound) | A successful commit triggers the client's cancellation or reschedule notice (content owned by FEAT-08.SPEC-004) |
| FEAT-08.SPEC-010 (Booking-Specific Manage Link Issuance) -- within FEAT-08 (Automated Booking Messaging) | Triggers (outbound) | A committed reschedule requests a fresh manage link for the Booking (XBR-18) |
| FEAT-25.SPEC-004 (Historical Aggregate Maintenance) -- within FEAT-25 (Booking & Revenue Insights) | Triggers (outbound) | A committed cancellation or reschedule updates the insights aggregates |
| FEAT-22.SPEC-005 (Balance Charge, Payout Routing & Refund) -- within FEAT-22 (In-App Balance Payment) | Triggers (outbound, v1) | An already-succeeded Balance Payment is refunded alongside the deposit |
| FEAT-10 (Client-Initiated Cancel/Reschedule) | References (outbound) | The source of a conflicting transition this automation may lose to, per the Booking entity's Contention rule |

## Analytics and Success Signals

- **booking_cancelled_by_pro** (has_reason: yes/no) -- supports success-metrics.md: "Pro Change Correctness"
- **booking_rescheduled_by_pro** () -- supports success-metrics.md: "Pro Change Correctness"
- **pro_commit_rejected** (reason: conflicting_transition / no_longer_eligible) -- supports success-metrics.md: "Automatic Refund Correctness" (a rejected commit must never leave the booking or its deposit in an ambiguous state; this event measures how often the reject-with-refresh path is exercised)
- **pro_commit_failed** (action: cancel / reschedule; reason category) -- N/A -- no Stage 2 metric measures processing failures directly; retained so a failed write is never silently unobservable.

## Acceptance Criteria

**FEAT-30.SPEC-007-AC-01:** Given Talia confirms a cancellation on a Confirmed booking she owns, when this commit runs, then Booking.state is set to Cancelled by Pro, the cancellation timestamp is recorded, and Riley's deposit refund is handed off to FEAT-09.

**FEAT-30.SPEC-007-AC-02:** Given Talia confirms a reschedule to an already-validated new time, when this commit runs, then Booking.start_time is updated, Booking.state is unchanged, and Riley's deposit carries over automatically per XBR-09.

**FEAT-30.SPEC-007-AC-03:** Given a successful cancellation or reschedule commit, when it completes, then the corresponding entry on Talia's personal calendar is removed (cancellation) or moved (reschedule).

**FEAT-30.SPEC-007-AC-04:** Given a successful commit, when it completes, then exactly one append-only activity event is written recording the action, timestamp, and Talia as the actor.

**FEAT-30.SPEC-007-AC-05:** Given a successful cancellation commit, when it completes, then the freed slot is handed to FEAT-20's waitlist-priority check before returning to general availability.

**FEAT-30.SPEC-007-AC-06:** Given a successful commit, when it completes, then FEAT-30.SPEC-012 fires the matching client notice for the action taken (cancellation-with-refund or reschedule-with-new-time).

**FEAT-30.SPEC-007-AC-07:** Given a booking already has a Succeeded Balance Payment (v1), when Talia cancels or reschedules it, then the Balance Payment is refunded in full alongside the deposit, per XBR-23.

**FEAT-30.SPEC-007-AC-08:** Given a booking has only an Attempted or Failed Balance Payment, when Talia cancels it, then no Balance Payment refund is requested, since nothing was actually paid.

**FEAT-30.SPEC-007-AC-09:** Given Riley cancels the same booking through FEAT-10 moments before Talia's commit runs, when this commit's eligibility re-check executes, then it is rejected with reject-with-refresh, and Talia is shown the booking's current (client-cancelled) state.

**FEAT-30.SPEC-007-AC-10:** Given a booking has auto-completed since Talia opened the cancel screen, when this commit's eligibility re-check runs, then it is rejected with the completed-state message from FEAT-30.SPEC-006, and no data changes.

**FEAT-30.SPEC-007-AC-11:** Given the commit cannot be written due to a processing error, when the write fails, then Talia sees a retry prompt on the triggering screen and the booking remains exactly as it was.

**FEAT-30.SPEC-007-AC-12:** Given Talia cancels two different bookings from two different schedule rows at effectively the same time, when both commits run, then each succeeds independently with no interference.

**FEAT-30.SPEC-007-AC-13:** Given a cancellation commit is already in flight for a booking, when Talia's confirm control is tapped again before it resolves, then no duplicate commit is submitted, since the control is disabled during submission.

**FEAT-30.SPEC-007-AC-14:** Given Talia's reschedule candidate slot loses its hold between selection and this commit's write, when the commit attempts to proceed, then it fails and Talia returns to FEAT-30.SPEC-002 with a refreshed slot list, per FEAT-03.SPEC-005.

**FEAT-30.SPEC-007-AC-15:** Given Talia confirms a reschedule and the commit succeeds, when the commit completes, then FEAT-08.SPEC-010 issues a fresh manage link for that Booking, the previous link no longer resolves as current, and the client notice triggered by FEAT-30.SPEC-012 carries the fresh link; a cancellation commit issues no new link.

**FEAT-30.SPEC-007-AC-16:** Given a cancellation or in-place reschedule commit succeeds, when it completes, then FEAT-25.SPEC-004 is signalled once with the Booking's Pro Account, service, and start_time so the insights aggregates reflect the change.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 | 2 |
| Outcome Paths | 6 | 6 |
| Business Rules | 8 | 8 |
| Edge Cases | 6 | 6 |



# Automation Spec: Bulk Cancellation Commit

## Overview

**Name:** Bulk Cancellation Commit
**ID:** FEAT-30.SPEC-008
**Type:** Automation
**Purpose:** Commits a Pro-initiated cancellation of several bookings at once, reporting a per-booking outcome and coordinating each booking's refund, calendar removal, and client notice independently.
**Parent Feature:** FEAT-30 -- Pro Booking Management

## Scope and Non-Goals

**In Scope:**
- Committing Cancelled by Pro to every booking Talia selects from a reviewed conflict set
- Processing each booking independently so one booking's failure never blocks or rolls back the others
- Reporting a per-booking success/failure outcome back to the triggering screen
- Refunding any already-paid Balance Payment per affected booking (XBR-23, v1)
- Coordinating each booking's calendar removal, activity logging, freed-slot handoff, and client notice

**Non-Goals:**
- Reviewing or presenting the conflicting booking set to Talia -- owned by FEAT-30.SPEC-005 (Cancel Several Bookings at Once), which this automation is triggered by
- A single-booking cancellation or reschedule -- owned by FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit), a distinct processing shape (one booking, one outcome) from this automation's per-item outcome tracking
- Determining or executing the deposit refund itself for each booking -- owned by FEAT-09 for the refund-outcome evaluation this automation hands off to, per XBR-09
- Rolling back bookings that already succeeded when a later booking in the same set fails -- excluded per the Brief's States field ("a multi-booking cancellation reports the outcome for each booking"): each booking's outcome is independent and final once committed, never undone by a sibling's failure

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Talia confirms "cancel all" on the bulk-cancellation review screen | FEAT-30.SPEC-005 (Cancel Several Bookings at Once) | Fires when Talia confirms cancelling the reviewed set of bookings a new Time Block conflicts with | The set of Booking references to cancel, optional shared private Pro reason |

## Processing Logic

1. Receive the set of Booking references from FEAT-30.SPEC-005's confirm action.
2. For each Booking in the set, independently:
   a. Re-check eligibility against FEAT-30.SPEC-006 (ownership, and Booking.state is Confirmed or Awaiting Outcome) immediately before the write.
   b. If eligibility fails (a conflicting transition already committed, or the booking is no longer eligible for another reason), mark this booking's outcome as Failed with the specific denial reason and continue to the next booking without affecting it.
   c. If eligibility passes, set Booking.state to Cancelled by Pro and record the cancellation timestamp and any shared private Pro reason.
   d. Check whether a Balance Payment exists with status Succeeded (v1); if so, request its full refund (XBR-23) through FEAT-22.SPEC-005 (Balance Charge, Payout Routing & Refund).
   e. Hand off the cancellation event to FEAT-09 for deposit-outcome evaluation (always full refund, per XBR-09).
   f. Trigger the calendar mirror to remove the corresponding personal-calendar entry (FEAT-04, XBR-13).
   g. Write an append-only activity event for this booking (FEAT-16, XBR-21).
   h. Hand the freed slot to FEAT-20's waitlist-priority check before it returns to general availability (XBR-28).
   i. Trigger FEAT-30.SPEC-012 (Pro Action Client Notice) for this booking's client.
   j. Mark this booking's outcome as Succeeded.
3. Once every booking in the set has been processed, return the complete per-booking outcome report to FEAT-30.SPEC-005.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| All bookings cancelled | Every booking in the set passes eligibility | Every Booking.state -> Cancelled by Pro | Talia sees a per-booking success summary on FEAT-30.SPEC-005; each client receives a cancellation-with-refund notice | FEAT-30.SPEC-005, FEAT-09, FEAT-04, FEAT-16, FEAT-20, FEAT-30.SPEC-012 |
| Partial success -- one or more bookings fail | At least one booking in the set fails eligibility while others pass | Only the passing bookings' states change | Talia sees which specific booking(s) failed and why, alongside the successful cancellations, on FEAT-30.SPEC-005; she can act on the failed one separately (e.g., via FEAT-30.SPEC-001) | FEAT-30.SPEC-005, FEAT-30.SPEC-001 |
| A single booking's Balance Payment refunded (v1) | That booking has a Succeeded Balance Payment | Balance Payment.state -> Refunded for that booking | Included in that booking's client notice, never a separate message | FEAT-22, FEAT-30.SPEC-012 |
| Whole-set write failure (processing error before any booking commits) | The commit cannot begin for a reason other than a per-booking eligibility conflict | No data changes | Talia sees a retry prompt on FEAT-30.SPEC-005; no booking in the set is affected | FEAT-30.SPEC-005 |

## Data Model

**Reads:** Booking (state, start_time, owning Pro Account) for each booking in the set; Deposit Transaction (status, per booking); Balance Payment (status, per booking, v1).
**Creates:** Activity Event (FEAT-16) -- one per successfully cancelled booking.
**Updates:** Booking -- state to Cancelled by Pro, plus cancellation timestamp and optional shared reason, per successfully processed booking. Balance Payment -- state to Refunded where one exists and has Succeeded, per booking (v1).
**Deletes:** None -- every cancelled booking is retained as history (SC-22).

## Business Rules

- Each booking in the set is processed and evaluated fully independently -- no booking's outcome depends on or is rolled back by another's, per the Brief's per-booking outcome-reporting requirement.
- XBR-09, XBR-13, XBR-21, XBR-23, and XBR-28 apply identically to each booking in the set as they do to a single Pro-initiated cancellation (FEAT-30.SPEC-007) -- this automation differs only in operating over several bookings under one confirm action.
- A booking that fails eligibility is reported, never silently dropped -- Talia always sees exactly which booking failed and why, consistent with pipeline-rules.md's output-completeness expectation carried into product behavior.
- The set's shared private Pro reason (if provided) is recorded identically on every successfully cancelled booking in the set; it is never inferred or altered per booking.

## Edge Cases

- **One booking in the set was already completed by the Auto-Completion Sweep moments before this commit runs** -- That booking's eligibility check fails with the completed-state message from FEAT-30.SPEC-006; it is reported as a failed outcome while the remaining bookings in the set are cancelled normally.
- **A client cancels one of the set's bookings through FEAT-10 in the instant before this commit processes it** -- Reject-with-refresh for that single booking: it is reported as a failed outcome (already resolved by the client), and every other booking in the set is unaffected.
- **Every booking in the set fails eligibility** -- The outcome report shows every booking as failed with its specific reason; no client notices fire, and Talia sees the full set needs a separate look.
- **The set contains only one booking** -- Processed identically to a full multi-booking set; the per-booking outcome mechanics do not special-case a set of size one, though the triggering screen (FEAT-30.SPEC-005) is the one that decides when this path versus FEAT-30.SPEC-007 applies.
- **Concurrent trigger firing (two different bulk-cancel confirms from two different time-block conflicts, with no overlapping bookings)** -- Each bulk commit processes its own distinct set independently; no interference occurs since the underlying bookings do not overlap.
- **Trigger fires while a previous bulk commit for the same set is still in flight** -- FEAT-30.SPEC-005's confirm control is disabled during submission, preventing a duplicate bulk-commit request for the same reviewed set.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-30.SPEC-005 (Cancel Several Bookings at Once) | Triggered by (inbound) / Affects (outbound) | Confirm triggers this commit; the per-booking outcome report is shown here |
| FEAT-30.SPEC-006 (Pro Booking Action Rules) | References (outbound) | Per-booking eligibility, ownership, and state-cutoff rules re-checked before each write |
| FEAT-30.SPEC-001 (Cancel Booking, Pro-Initiated) | Affects (outbound) | A failed booking in the set can be revisited individually through this screen |
| FEAT-09 (Cancellation & No-Show Policy Engine) | Triggers (outbound) | Hands off each booking's cancellation event for deposit-outcome evaluation and execution (XBR-09) |
| FEAT-04.SPEC-005 (Booking-to-Calendar Sync) -- within FEAT-04 (Two-Way Calendar Sync) | Triggers (outbound) | Each successfully cancelled booking's calendar entry is removed |
| FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) | Triggers (outbound) | Each successfully cancelled booking writes its own activity event |
| FEAT-20.SPEC-005 (Cancellation-Triggered Waitlist Matching) -- within FEAT-20 (Waitlist for Cancelled Slots) | Triggers (outbound) | Each freed slot is handed to waitlist-priority notification first |
| FEAT-30.SPEC-012 (Pro Action Client Notice) | Triggers (outbound) | Each successfully cancelled booking's client receives the cancellation-with-refund notice |
| FEAT-22.SPEC-005 (Balance Charge, Payout Routing & Refund) -- within FEAT-22 (In-App Balance Payment) | Triggers (outbound, v1) | Any already-succeeded Balance Payment per booking is refunded alongside the deposit |
| FEAT-17 (Manual Time Blocking) | Triggered by (inbound, indirect) | The Time Block whose conflicting bookings Talia reviewed on FEAT-30.SPEC-005 originates here |

## Analytics and Success Signals

- **bulk_cancellation_completed** (booking_count, success_count, failure_count) -- supports success-metrics.md: "Pro Change Correctness"
- **booking_cancelled_by_pro** (context: bulk) -- supports success-metrics.md: "Pro Change Correctness" (per-booking event, emitted once for each booking successfully cancelled within the set)
- **bulk_cancellation_booking_failed** (reason category) -- supports success-metrics.md: "Automatic Refund Correctness" (a failed booking within a bulk action must never be silently dropped; this event measures how often the per-booking failure path is exercised)

## Acceptance Criteria

**FEAT-30.SPEC-008-AC-01:** Given Talia confirms "cancel all" on four bookings her new time block conflicts with, when this commit runs and all four pass eligibility, then all four transition to Cancelled by Pro, and Talia sees a success summary for all four on FEAT-30.SPEC-005.

**FEAT-30.SPEC-008-AC-02:** Given one of the four bookings was already completed before this commit runs, when the set is processed, then that booking is reported as a failed outcome with the completed-state reason while the other three are cancelled successfully.

**FEAT-30.SPEC-008-AC-03:** Given a booking in the set is successfully cancelled, when the commit processes it, then its personal-calendar entry is removed, an activity event is written, its freed slot is handed to FEAT-20, and its client receives the cancellation-with-refund notice.

**FEAT-30.SPEC-008-AC-04:** Given a booking in the set already has a Succeeded Balance Payment (v1), when it is cancelled, then that Balance Payment is refunded in full alongside its deposit.

**FEAT-30.SPEC-008-AC-05:** Given Talia provides a shared private reason when confirming the bulk cancellation, when each booking is cancelled, then that same reason is recorded on every successfully cancelled booking in the set.

**FEAT-30.SPEC-008-AC-06:** Given a client cancels one of the set's bookings through FEAT-10 moments before this commit processes it, when that booking is evaluated, then it is reported as a failed outcome (reject-with-refresh) and the remaining bookings in the set are unaffected.

**FEAT-30.SPEC-008-AC-07:** Given every booking in the reviewed set fails eligibility, when the commit runs, then every booking is reported as failed with its specific reason and no client notices fire.

**FEAT-30.SPEC-008-AC-08:** Given the set contains exactly one booking, when this automation processes it, then the single-booking outcome mechanics apply identically to a larger set.

**FEAT-30.SPEC-008-AC-09:** Given the whole-set commit cannot begin due to a processing error, when the failure occurs, then Talia sees a retry prompt on FEAT-30.SPEC-005 and no booking in the set is affected.

**FEAT-30.SPEC-008-AC-10:** Given two different bulk-cancel confirms fire at effectively the same time for two non-overlapping sets, when both commits run, then each processes its own set independently with no interference.

**FEAT-30.SPEC-008-AC-11:** Given a bulk commit is already in flight for a reviewed set, when Talia's confirm control is tapped again before it resolves, then no duplicate commit is submitted, since the control is disabled during submission.

**FEAT-30.SPEC-008-AC-12:** Given a booking that failed within a bulk cancellation, when Talia looks for a way to act on it separately, then she can reach FEAT-30.SPEC-001 for that individual booking.

**FEAT-30.SPEC-008-AC-13:** Given all bookings in the set succeed, when the final outcome is reported, then Talia's summary distinguishes success from failure per booking rather than showing one aggregate result.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 4 | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Automation Spec: Goodwill Refund Commit

## Overview

**Name:** Goodwill Refund Commit
**ID:** FEAT-30.SPEC-009
**Type:** Automation
**Purpose:** Processes a confirmed goodwill refund against a booking's deposit -- a standalone Pro override, independent of the cancellation window and available until the booking completes.
**Parent Feature:** FEAT-30 -- Pro Booking Management

## Scope and Non-Goals

**In Scope:**
- Determining that a confirmed goodwill refund is due, governed by FEAT-30.SPEC-006's once-only and until-completion limits
- Requesting the refund through FEAT-30.SPEC-011 (Goodwill & Bulk-Cancellation Refund Execution)
- Writing the activity event and triggering the client notice this action causes
- Never changing Booking.state -- only Deposit Transaction.status

**Non-Goals:**
- Deciding whether a goodwill refund is warranted -- excluded per scope-boundaries.md (SC-17): the goodwill decision is entirely Talia's own judgment, exercised through FEAT-30.SPEC-003; this automation only processes a decision Talia has already confirmed
- Executing the refund request against the payment-processing capability -- owned by FEAT-30.SPEC-011; this automation determines the refund is due and hands off the request
- Evaluating a client cancellation's or no-show's own deposit outcome -- owned by FEAT-09; a goodwill refund is not a Pro cancellation and is never evaluated by FEAT-09's outcome-evaluation automation, per the Brief's own Analyst-Discovered rationale for this spec
- Marking or undoing a no-show mark -- owned by FEAT-11; a goodwill refund can follow either a no-show mark or an inside-window client cancellation, but this automation never itself marks or unmarks a no-show

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Talia confirms a goodwill refund | FEAT-30.SPEC-003 (Goodwill Deposit Refund) | Fires when Talia taps confirm, reached from a no-show prompt (FEAT-11), a dispute timeline (FEAT-16), or a booking row (FEAT-12) | Booking reference, optional private Pro reason |

## Processing Logic

1. Receive the Booking reference and any optional private Pro reason from FEAT-30.SPEC-003's confirm action.
2. Re-check eligibility against FEAT-30.SPEC-006 (ownership, Booking.state has not reached Completed, and Deposit Transaction.status is Captured or Forfeited) immediately before the write.
3. If eligibility fails, stop and return the specific denial reason to FEAT-30.SPEC-003 without changing any data.
4. If eligibility passes, request the full refund through FEAT-30.SPEC-011 (Goodwill & Bulk-Cancellation Refund Execution), carrying the Deposit Transaction reference and the Pro's payout account reference.
5. On the execution's immediate confirmation, set Deposit Transaction.status to Refunded and record the outcome_reason as goodwill and the refund timestamp; on a not-yet-completable report, set Deposit Transaction.status to Refund in Progress (FEAT-30.SPEC-011 owns the resulting retry).
6. Write an append-only activity event recording the goodwill refund, its timestamp, the actor (Talia), and any optional private reason (FEAT-16, XBR-21).
7. Trigger FEAT-30.SPEC-012 (Pro Action Client Notice) with the goodwill-refund outcome (or the "in progress" variant, if not yet complete).
8. Return the committed outcome to FEAT-30.SPEC-003 for its success feedback.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Goodwill refund completed immediately | Eligibility passes and FEAT-30.SPEC-011 confirms the refund on the first attempt | Deposit Transaction.status -> Refunded; outcome_reason set to goodwill | Talia sees the refund confirmed on FEAT-30.SPEC-003; Riley receives the goodwill-refund notice | FEAT-30.SPEC-003, FEAT-30.SPEC-011, FEAT-16, FEAT-30.SPEC-012 |
| Goodwill refund entered in progress | Eligibility passes but FEAT-30.SPEC-011 reports it cannot complete immediately | Deposit Transaction.status -> Refund in Progress | Talia sees the refund confirmed as "in progress, will complete automatically" on FEAT-30.SPEC-003 and her dashboard attention flag; Riley receives the "in progress" client notice | FEAT-30.SPEC-003, FEAT-30.SPEC-011, FEAT-12, FEAT-30.SPEC-012 |
| Commit rejected -- deposit no longer refundable | Deposit Transaction.status is already Refunded, Refund in Progress, or Disputed at write time | No data changes | Talia sees "This booking's deposit has already been resolved and cannot be refunded again." on FEAT-30.SPEC-003 | FEAT-30.SPEC-003, FEAT-30.SPEC-006 |
| Commit rejected -- booking already completed | Booking.state has reached Completed at write time | No data changes | Talia sees "A goodwill refund is no longer available once a booking is completed." | FEAT-30.SPEC-003, FEAT-30.SPEC-006 |
| Write failure (processing error) | The commit cannot be written for a reason other than an eligibility conflict | No data changes | Talia sees a retry prompt on FEAT-30.SPEC-003; the deposit remains at its prior status | FEAT-30.SPEC-003 |

## Data Model

**Reads:** Booking (state, owning Pro Account); Deposit Transaction (status).
**Creates:** Activity Event (FEAT-16) -- one per committed goodwill refund.
**Updates:** Deposit Transaction -- status (Refunded or Refund in Progress), outcome_reason, refund timestamp.
**Deletes:** None.

## Business Rules

- SC-17: this automation never adjudicates whether a goodwill refund is warranted -- that judgment is made entirely by Talia through FEAT-30.SPEC-003 before this automation ever runs.
- XBR-10: the refund this automation triggers is always full, happens at most once per deposit, and a not-yet-completable attempt is retried automatically and never dropped, per FEAT-30.SPEC-011's execution and retry behavior.
- A goodwill refund never changes Booking.state -- only Deposit Transaction.status, per the Entity-Lifecycle Coverage Matrix; the booking's own cancellation/no-show/completion history is untouched by this action.
- A goodwill refund is available whenever Deposit Transaction.status is Captured or Forfeited and Booking.state has not reached Completed (FEAT-30.SPEC-006) -- independent of the cancellation policy window, since it is a Pro override rather than a window-based outcome.
- FEAT-09's outcome-evaluation automation never fires for a goodwill refund -- it is a standalone Pro-initiated path with its own commit, distinct from the automatic outside-window or Pro-cancellation refund paths FEAT-09 owns.

## Edge Cases

- **A client-side outside-window cancellation refunds this same deposit automatically (FEAT-09) in the instant before Talia confirms goodwill** -- The eligibility re-check finds Deposit Transaction.status already Refunded and denies with "This booking's deposit has already been resolved and cannot be refunded again."; no duplicate refund is requested.
- **The booking is marked Completed by the Auto-Completion Sweep in the instant before Talia confirms goodwill** -- The eligibility re-check finds Booking.state already Completed and denies with the completed-state message; the deposit is left exactly as it was.
- **Talia issues a goodwill refund reached from a no-show prompt (FEAT-11), before the no-show mark itself is confirmed** -- Talia chose goodwill instead of marking a no-show; the booking's Deposit Transaction is at Captured (never having been Forfeited), and this automation processes it as a standard Captured-status goodwill refund with no interaction with FEAT-11's own marking flow.
- **Talia issues a goodwill refund on a deposit already Forfeited from a no-show mark** -- Eligible per FEAT-30.SPEC-006 (Forfeited is a refundable status); the refund converts the kept deposit to Refunded, and per FEAT-11.SPEC-004's own edge case, this also closes that no-show mark's 24-hour undo window.
- **Concurrent trigger firing (Talia confirms goodwill refunds on two different bookings at effectively the same time)** -- Each commit processes independently against its own distinct Deposit Transaction; no interference occurs.
- **Trigger fires while a previous goodwill commit for the same booking is still in flight** -- FEAT-30.SPEC-003's confirm control is disabled during submission, preventing a duplicate commit request for the same booking's deposit.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-30.SPEC-003 (Goodwill Deposit Refund) | Triggered by (inbound) / Affects (outbound) | Confirm triggers this commit; the outcome or denial is shown here |
| FEAT-30.SPEC-006 (Pro Booking Action Rules) | References (outbound) | Once-only and until-completion eligibility re-checked before the write |
| FEAT-30.SPEC-011 (Goodwill & Bulk-Cancellation Refund Execution) | Triggers (outbound) | The determined-due refund is requested through this integration |
| FEAT-16 (Booking & Payment Activity Record) | Triggers (outbound) | A successful commit writes an append-only activity event |
| FEAT-30.SPEC-012 (Pro Action Client Notice) | Triggers (outbound) | A completed or in-progress goodwill refund triggers the client's notice |
| FEAT-11 (No-Show Marking & Deposit Forfeiture) | References (inbound) | One of this action's entry points; a goodwill refund can be chosen instead of marking a no-show |
| FEAT-16 (Booking & Payment Activity Record) | References (inbound) | Another of this action's entry points, from a no-show dispute timeline |

## Analytics and Success Signals

- **goodwill_refund_issued** (source: no_show_prompt / dispute_timeline / booking_row; outcome: completed / in_progress) -- supports success-metrics.md: "Pro Change Correctness"
- **goodwill_refund_completed** () -- supports success-metrics.md: "Automatic Refund Correctness"
- **goodwill_refund_rejected** (reason: not_refundable / already_completed) -- N/A -- no Stage 2 metric measures rejected goodwill attempts directly; retained so a denied action is never silently unobservable.

## Acceptance Criteria

**FEAT-30.SPEC-009-AC-01:** Given Talia confirms a goodwill refund on a booking whose deposit is Captured and not yet Completed, when this commit runs and the execution confirms immediately, then Deposit Transaction.status is set to Refunded and Riley receives the goodwill-refund notice.

**FEAT-30.SPEC-009-AC-02:** Given the refund execution reports it cannot complete immediately, when this commit processes that outcome, then Deposit Transaction.status is set to Refund in Progress, Talia's dashboard shows the attention flag, and Riley sees "in progress."

**FEAT-30.SPEC-009-AC-03:** Given a booking's Deposit Transaction is already Refunded through an automatic client cancellation, when Talia confirms a goodwill refund on it, then the commit is rejected with "This booking's deposit has already been resolved and cannot be refunded again."

**FEAT-30.SPEC-009-AC-04:** Given a booking has reached Completed, when Talia confirms a goodwill refund on it, then the commit is rejected with "A goodwill refund is no longer available once a booking is completed."

**FEAT-30.SPEC-009-AC-05:** Given a successful goodwill refund commit, when it completes, then exactly one append-only activity event is written recording the action and Talia as the actor.

**FEAT-30.SPEC-009-AC-06:** Given Talia reaches the goodwill refund screen from a no-show prompt and confirms before marking the no-show, when this commit runs, then it processes the Captured-status deposit normally with no interaction with the no-show marking flow.

**FEAT-30.SPEC-009-AC-07:** Given a booking's deposit is already Forfeited from a no-show mark, when Talia confirms a goodwill refund on it, then the commit succeeds, converting the deposit to Refunded.

**FEAT-30.SPEC-009-AC-08:** Given a goodwill refund converts a Forfeited deposit to Refunded, when Talia later attempts to undo the original no-show mark, then the undo is denied per FEAT-11.SPEC-004, since the deposit is no longer Forfeited.

**FEAT-30.SPEC-009-AC-09:** Given the commit cannot be written due to a processing error, when the failure occurs, then Talia sees a retry prompt and the deposit remains at its prior status.

**FEAT-30.SPEC-009-AC-10:** Given Talia confirms goodwill refunds on two different bookings at effectively the same time, when both commits run, then each succeeds independently with no interference.

**FEAT-30.SPEC-009-AC-11:** Given a goodwill commit is already in flight for a booking, when Talia's confirm control is tapped again before it resolves, then no duplicate commit is submitted, since the control is disabled during submission.

**FEAT-30.SPEC-009-AC-12:** Given Talia reaches this action from a no-show dispute timeline (FEAT-16) rather than a no-show prompt, when she confirms the refund, then this commit processes it identically regardless of entry point.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 5 | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Automation Spec: Pro-Created Booking & Deposit Request Hold

## Overview

**Name:** Pro-Created Booking & Deposit Request Hold
**ID:** FEAT-30.SPEC-010
**Type:** Automation
**Purpose:** Creates the pending Booking from a Pro-entered service, time, and client, invokes the slot hold that reserves the time, confirms the booking on payment, and reflects the hold's expiry if the deposit is never paid.
**Parent Feature:** FEAT-30 -- Pro Booking Management

## Scope and Non-Goals

**In Scope:**
- Creating the Booking record in Pending Payment state, with source = "Pro booked-in", from the service, time, and client Talia selects on FEAT-30.SPEC-004
- Invoking the slot-hold mechanism that reserves the candidate time for this booking
- Issuing the deposit request (link or on-screen code) once the Booking and its hold are created
- Confirming the Booking (Pending Payment -> Confirmed) when the client pays, exactly as a client-initiated booking confirms
- Relying on the Booking's transition to Expired (unpaid) when its hold lapses unpaid: FEAT-03.SPEC-007 is the sole writer of that transition (XBR-02), and this automation only reads the resulting state
- Triggering the personal-calendar write when the Booking is confirmed (FEAT-04.SPEC-005)

**Non-Goals:**
- Computing the hold's expiry timestamp, re-validating the candidate slot against live availability, or actually expiring the hold record -- owned entirely by FEAT-03.SPEC-007 (Pro-Created Deposit Request Hold & Expiration), which this automation triggers and whose outcomes it reflects onto the Booking it created; this automation never duplicates that spec's timing mechanics
- Selecting the service, time, and client -- owned by FEAT-30.SPEC-004 (Book Client In), the triggering screen
- Capturing the client's deposit payment itself -- owned by FEAT-07 (Deposit Payment at Booking); this automation only reacts to that capability's confirmation to transition the Booking
- Writing the Booking -> Expired (unpaid) transition -- owned solely by FEAT-03.SPEC-007 (XBR-02 authority: FEAT-03); this automation never performs, repeats, or races that write
- Composing or delivering the deposit-request content -- owned by FEAT-30.SPEC-013 (Deposit Request & Expiry Notice); this automation triggers it but does not define its content
- Composing or delivering the Pro's expiry notice -- FEAT-03.SPEC-007 is the trigger and FEAT-08.SPEC-006 (Pro Attention Alert) is the content owner

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Talia saves a new Pro-created booking | FEAT-30.SPEC-004 (Book Client In) | Fires when Talia confirms the service, time, and existing-or-new client for booking someone in | Service ID, chosen start time and duration, Client reference (existing or newly entered) |
| FEAT-03.SPEC-007 marks the Booking Expired (unpaid) | FEAT-03.SPEC-007 (Pro-Created Deposit Request Hold & Expiration) | Fires after that spec's expiration path has determined the deposit was never paid in time and has itself written Booking.state = Expired (unpaid) | The owning Booking reference (already Expired (unpaid)) |
| The client pays the deposit request | FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation) | Fires when that spec confirms the deposit payment against this Booking's hold | The owning Booking reference, payment outcome |

## Processing Logic

1. Receive the candidate Service ID, start time, duration, and Client reference from FEAT-30.SPEC-004's save action.
2. Create the Booking record: service, start_time, duration, client, price_agreed and deposit_amount (computed from the Service's rule, per XBR-05), policy_version (the Pro's current Cancellation Policy), state = Pending Payment, source = "Pro booked-in".
3. Invoke FEAT-03.SPEC-007 to create the owning slot hold against this new Booking, applying the Pro-only notice/horizon exception (FEAT-30.SPEC-006) to the candidate slot's re-validation.
4. If FEAT-03.SPEC-007 reports the candidate slot is contested (already held, booked, blocked, or busy), do not create the Booking; return the "just taken" outcome to FEAT-30.SPEC-004 for its inline recovery message.
5. If the hold is created successfully, issue the deposit request through FEAT-30.SPEC-013, by the delivery choice Talia selected on FEAT-30.SPEC-004 (link by text/email, or an on-screen code).
6. **On payment path:** When FEAT-07 confirms the deposit payment against this Booking's hold, transition Booking.state from Pending Payment to Confirmed, exactly as a client-initiated booking confirms.
7. **On expiry path:** When FEAT-03.SPEC-007 reports it has marked this Booking Expired (unpaid), take no state-writing step: FEAT-03.SPEC-007 is the sole writer of that transition. Read the resulting state so FEAT-30.SPEC-004's confirmation and FEAT-12 show the booking as expired. The Pro's expiry notice is triggered by FEAT-03.SPEC-007 with its content owned by FEAT-08.SPEC-006; this automation sends nothing.
8. **After the payment path (step 6):** Trigger FEAT-04.SPEC-005 (Booking-to-Calendar Sync) for the newly Confirmed Booking so it is written to Talia's connected personal calendar (XBR-13).
9. Return the created Booking's outcome to FEAT-30.SPEC-004 for its confirmation feedback.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Booking created, hold placed, deposit request issued | The candidate slot passes re-validation (with the Pro-only exception) and FEAT-03.SPEC-007 places the hold | New Booking created in Pending Payment, source "Pro booked-in" | Talia sees the booking on her dashboard awaiting payment; the client receives the deposit request (FEAT-30.SPEC-013) | FEAT-30.SPEC-004, FEAT-03.SPEC-007, FEAT-30.SPEC-013 |
| Candidate slot contested at creation | FEAT-03.SPEC-007 reports the candidate time is already held, booked, blocked, or busy | No Booking created | Talia sees the plain "just taken" message on FEAT-30.SPEC-004 and re-selects a time | FEAT-30.SPEC-004, FEAT-03.SPEC-007 |
| Deposit paid before expiry | The client completes payment while the hold is still Active | Booking.state -> Confirmed; calendar write triggered | Talia and the client both see the booking confirmed | FEAT-07, FEAT-12, FEAT-04.SPEC-005 |
| Hold expired -- deposit never paid | FEAT-03.SPEC-007's expiration path has lapsed the hold unpaid and written Booking.state = Expired (unpaid) | None by this automation -- the state is written solely by FEAT-03.SPEC-007; this automation reads it | Talia is notified via her dashboard and a message (triggered by FEAT-03.SPEC-007, content owned by FEAT-08.SPEC-006); the client receives no further reminder for this booking | FEAT-03.SPEC-007, FEAT-08.SPEC-006 |
| Booking creation failure | The Booking record cannot be written (e.g., a processing error) | No Booking created; no hold placed | Talia sees a retry prompt on FEAT-30.SPEC-004; no deposit-request link or code is issued | FEAT-30.SPEC-004 |

## Data Model

**Reads:** Service (price, duration, deposit_rule); Cancellation Policy (current version); Client (existing lookup) or none (new client, created inline by FEAT-30.SPEC-004).
**Creates:** Booking -- service, start_time, duration, client, price_agreed, deposit_amount, policy_version, state (Pending Payment), source ("Pro booked-in").
**Updates:** Booking.state -- Pending Payment -> Confirmed (on payment) only. The -> Expired (unpaid) transition on unpaid hold expiry is written solely by FEAT-03.SPEC-007 and is only read here.
**Deletes:** None -- an expired Pro-created booking is retained as history (SC-22), never deleted; only its owning slot hold (a distinct, transient record owned by FEAT-03.SPEC-007) is deleted on expiry.

## Business Rules

- XBR-05: the deposit amount is computed once, exactly, from the Service's rule in the Pro's account currency, and cannot be altered by Talia, identical to any client-initiated booking.
- XBR-02: the slot hold this automation invokes reserves the candidate time for up to platform parameter: `deposit-request-hold-max-hours` or until platform parameter: `deposit-request-hold-appointment-cutoff-hours` before the appointment, whichever comes first -- the value is stated by FEAT-30.SPEC-006 and enforced entirely by FEAT-03.SPEC-007; this automation never computes or enforces the expiry itself.
- The Pro-only notice/horizon exception (FEAT-30.SPEC-006) applies to the candidate slot's re-validation, since the triggering action is always a Pro-side booking; the duration+buffer fit rule is never exempted.
- A Booking this automation creates is never confirmed by anything other than FEAT-07's payment confirmation -- there is no "mark paid manually" path, consistent with the product's correctness-over-convenience stance (SC-21).
- FEAT-03.SPEC-007 is the sole writer of Booking -> Expired (unpaid) (XBR-02); this automation never sets that state, so two writers can never race on the same Booking.
- An expired Pro-created booking is marked Expired (unpaid) (by FEAT-03.SPEC-007), never silently deleted -- the record that a booking was attempted and lapsed is preserved, distinct from the transient slot hold itself, which FEAT-03.SPEC-007 deletes.

## Edge Cases

- **The candidate slot is contested by a client-side booking that completes payment first** -- FEAT-03.SPEC-007's contention resolution applies (first committed wins, per XBR-01); no Booking is created for Talia's attempt, and she sees the "just taken" message and re-selects a time.
- **Talia cancels the Pro-created booking herself before the deposit is paid or the hold expires** -- The cancellation (via FEAT-30.SPEC-007, the single-cancellation commit) transitions the Booking and, through FEAT-03.SPEC-007, deletes the hold directly; no expiration notification fires for a Pro-initiated cancellation.
- **The appointment is scheduled less than platform parameter: `deposit-request-hold-appointment-cutoff-hours` away at the moment of creation** -- The Pro-only notice exception permits creating the booking itself; the resulting hold's own window is correspondingly short, per FEAT-03.SPEC-007's own handling of this case.
- **The client pays and the hold expires at effectively the same instant** -- FEAT-03.SPEC-007's own precedence rule applies: the completed payment takes priority, and this automation confirms the Booking rather than expiring it.
- **Concurrent trigger firing (Talia books two different clients into two different, non-overlapping slots at effectively the same time)** -- Each booking-creation attempt is processed independently against its own distinct candidate slot; no interference occurs.
- **Trigger fires while a previous booking-creation attempt for the same booking-in action is still in flight** -- FEAT-30.SPEC-004's save control is disabled during submission, preventing a duplicate Booking-and-hold creation request for the same client and slot.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-30.SPEC-004 (Book Client In) | Triggered by (inbound) / Affects (outbound) | Save triggers Booking creation; the outcome (created, contested, or failed) is shown here |
| FEAT-30.SPEC-006 (Pro Booking Action Rules) | References (outbound) | States the notice/horizon exception and the hold-window values this automation invokes |
| FEAT-03.SPEC-007 (Pro-Created Deposit Request Hold & Expiration) | Triggers (outbound) / Triggered by (inbound) | This automation invokes hold creation on save; that spec is the sole writer of Booking -> Expired (unpaid) and this automation only relies on the result |
| FEAT-07 (Deposit Payment at Booking) | Triggered by (inbound) | Payment confirmation against this Booking's hold transitions it to Confirmed |
| FEAT-30.SPEC-013 (Deposit Request & Expiry Notice) | Triggers (outbound) | A created hold issues the deposit request |
| FEAT-08.SPEC-006 (Pro Attention Alert) | References (outbound) | Owns the Pro's expiry-notice content, triggered by FEAT-03.SPEC-007 |
| FEAT-04.SPEC-005 (Booking-to-Calendar Sync) -- within FEAT-04 (Two-Way Calendar Sync) | Triggers (outbound) | A Pro-created Booking confirmed by payment is written to Talia's personal calendar |
| FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) | Talia sees the booking's awaiting-payment, confirmed, or expired status on her schedule |

## Analytics and Success Signals

- **pro_booking_created** (service_id) -- supports success-metrics.md: "Pro Change Correctness"
- **deposit_request_paid** (time_to_pay) -- supports success-metrics.md: "Pro Change Correctness" (the target's "at least 70% of deposit requests the pro sends when rebooking at the chair are paid before the hold expires" is measured against deposit_request_expired, emitted below when this automation reads the Expired (unpaid) state FEAT-03.SPEC-007 writes)
- **deposit_request_expired** () -- supports success-metrics.md: "Pro Change Correctness"
- **pro_booking_creation_failed** (reason category) -- N/A -- no Stage 2 metric measures booking-creation failures directly; retained so a failed save is never silently unobservable.

## Acceptance Criteria

**FEAT-30.SPEC-010-AC-01:** Given Talia saves a new booking for an existing client at a genuinely free time, when this automation runs, then a Booking is created in Pending Payment with source "Pro booked-in", and a slot hold is placed via FEAT-03.SPEC-007.

**FEAT-30.SPEC-010-AC-02:** Given the candidate slot is contested by another booking that lands first, when the hold-creation attempt runs, then no Booking is created and Talia sees the "just taken" message on FEAT-30.SPEC-004.

**FEAT-30.SPEC-010-AC-03:** Given a Booking and its hold are created successfully, when the save completes, then the deposit request is issued through FEAT-30.SPEC-013 by Talia's chosen delivery method.

**FEAT-30.SPEC-010-AC-04:** Given the client completes the deposit payment while the hold is Active, when FEAT-07 confirms the payment, then this Booking's state transitions from Pending Payment to Confirmed.

**FEAT-30.SPEC-010-AC-05:** Given the hold's computed expiry passes with the deposit never paid, when FEAT-03.SPEC-007's expiration path fires, then FEAT-03.SPEC-007 (and no other spec) sets this Booking's state to Expired (unpaid), this automation performs no state write, and Talia is notified via her dashboard and a message with content owned by FEAT-08.SPEC-006.

**FEAT-30.SPEC-010-AC-06:** Given Talia books a client in for a time inside her own minimum_booking_notice, when the candidate is re-validated, then it is not excluded on notice grounds, per the Pro-only exception (FEAT-30.SPEC-006).

**FEAT-30.SPEC-010-AC-07:** Given Talia cancels a Pro-created booking herself before its hold expires, when the cancellation completes via FEAT-30.SPEC-007, then the hold is deleted directly and no expiration notification fires.

**FEAT-30.SPEC-010-AC-08:** Given the deposit payment and the hold's expiry occur at effectively the same moment, when both are evaluated, then the completed payment takes precedence and the Booking is Confirmed, not Expired.

**FEAT-30.SPEC-010-AC-09:** Given the Booking record cannot be created due to a processing error, when Talia attempts to save the booking-in flow, then she sees a retry prompt and no deposit-request link or code is issued.

**FEAT-30.SPEC-010-AC-10:** Given Talia books two different clients into two different, non-overlapping slots at effectively the same time, when both booking-creation attempts run, then each succeeds independently with no interference.

**FEAT-30.SPEC-010-AC-11:** Given a booking-creation attempt is already in flight for a booking-in action, when Talia's save control is tapped again before it resolves, then no duplicate Booking-and-hold is created, since the control is disabled during submission.

**FEAT-30.SPEC-010-AC-12:** Given a Pro-created booking expires unpaid, when Riley next requests the same service's slot list, then the freed time appears as open, per FEAT-03.SPEC-007.

**FEAT-30.SPEC-010-AC-13:** Given a Pro-created booking's deposit is computed from the Service's rule, when the Booking is created, then the deposit_amount matches exactly what FEAT-07 would compute for a client-initiated booking of the same service.

**FEAT-30.SPEC-010-AC-14:** Given a Pro-created booking's deposit is paid and the Booking transitions to Confirmed, when this automation completes the payment path, then FEAT-04.SPEC-005 is triggered once for that Booking so it appears on Talia's connected personal calendar.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 | 3 |
| Outcome Paths | 5 | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Integration Spec: Goodwill & Bulk-Cancellation Refund Execution

## Overview

**Name:** Goodwill & Bulk-Cancellation Refund Execution
**ID:** FEAT-30.SPEC-011
**Type:** Integration
**Purpose:** Requests each goodwill or bulk-cancellation refund from the payment-processing capability, drawing on the Pro's connected payout account, guarantees each refund completes exactly once with automatic retry, and reports back any refund that cannot complete immediately.
**Parent Feature:** FEAT-30 -- Pro Booking Management

## Scope and Non-Goals

**In Scope:**
- Requesting a full refund for a specific Deposit Transaction once FEAT-30.SPEC-009 (goodwill) or FEAT-30.SPEC-008 (bulk cancellation, per booking) determines one is due
- Receiving and applying the refund outcome (succeeded, cannot complete immediately) to the Deposit Transaction
- Guaranteeing at most one successful refund per Deposit Transaction, with automatic, indefinite retry until a not-yet-completable attempt resolves
- User-facing behavior when the payment-processing capability is slow, unavailable, or rejects the refund request
- Disclosure of what data this refund request shares with the capability

**Non-Goals:**
- Determining that a refund is due in the first place, or the Pro's own decision to issue one -- owned by FEAT-30.SPEC-009 (Goodwill Refund Commit) and FEAT-30.SPEC-008 (Bulk Cancellation Commit); this spec only executes a refund already determined
- Executing a single Pro-initiated cancellation's or reschedule's refund -- owned by FEAT-09.SPEC-005 (Automatic Deposit Refund), which handles that case per the External Touchpoints table's assignment; this spec covers only the refund calls FEAT-09 never evaluates or batches: goodwill refunds and the per-booking refund set a bulk cancellation produces
- Choosing the payment-processing vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate
- Capturing the original deposit charge, or routing it to the Pro's payout account in the first place -- owned by FEAT-07 (Deposit Payment at Booking); this spec only reverses an already-captured amount

## Capability Category

**Category:** Payment processing
**Dependency Source:** ASMP-31 -- "Payment-processing capability... required to take client deposits, verify each pro's identity and bank details for a connected payout account, pay deposits out to the pro, issue refunds, notify the product of card-issuer disputes, and bill the pro's own monthly subscription." (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Payment processing -- client card charges and refunds (deposits; from v1 balances; Later tips)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-07, FEAT-09, FEAT-30, FEAT-22, FEAT-23; this spec is named directly as "FEAT-30.SPEC-011 (goodwill refunds and the per-booking refund set of a Pro bulk cancellation, drawing on the Pro's payout account, with not-yet-completable refunds reported back for retry; a single Pro cancellation's refund is executed by FEAT-09.SPEC-005)")
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Talia issues a goodwill refund and Riley sees her deposit refunded in full | Refund a deposit in full as goodwill | FEAT-30.SPEC-003 (Goodwill Deposit Refund) shows the confirmed refund |
| Talia cancels several bookings at once and every affected client's deposit refunds | Cancel several bookings at once | FEAT-30.SPEC-005 (Cancel Several Bookings at Once) shows the per-booking refund outcome |
| If a refund cannot complete immediately, both Talia and Riley see it as "in progress" rather than silently failing | Refund a deposit in full as goodwill; cancel several bookings at once | FEAT-12 (Pro Daily Schedule Dashboard) attention flag; Riley's own booking status view |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Refund amount and currency | Deposit Transaction -- amount, currency | FEAT-30.SPEC-009 or FEAT-30.SPEC-008 determines a refund is due | The capability must know exactly how much to return, matching the original captured amount |
| Deposit reference | Deposit Transaction -- the reference tying it to the original capture | Refund is requested | Ties the refund to the specific original charge so the capability reverses the correct transaction |
| Payout account reference | Payout Account -- processor_account_reference | Refund is requested | Identifies which of the Pro's connected accounts the refund draws against |
| Idempotency key | Derived -- an identifier tied to this specific refund attempt on this specific Deposit Transaction | Every refund request, including retries | Lets the capability recognize a resubmitted request as the same attempt rather than a second refund |

Booking details (service, client note, appointment time), the Client's contact fields, and every other Deposit Transaction field beyond amount, currency, and the deposit reference never leave the product for this integration.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Refund succeeded confirmation, with refund timestamp | The capability completes the refund | Deposit Transaction -- status (Refunded), refund timestamp |
| Refund cannot complete immediately (e.g., the Pro's payout balance cannot yet cover it), with a retry-eligibility signal | The capability reports the refund could not be completed on this attempt | Deposit Transaction -- status (Refund in Progress); scheduled for this spec's own automatic retry |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Refund succeeded | The capability completes the requested refund | Deposit Transaction.status set to Refunded; refund timestamp recorded | Included in the relevant confirmation content (FEAT-30.SPEC-012), not a separate standalone notice; Riley sees her deposit refunded on her own booking view; the refund outcome is passed to FEAT-25.SPEC-004 (Historical Aggregate Maintenance) so revenue aggregates net out the refunded deposit | FEAT-30.SPEC-009, FEAT-30.SPEC-008, FEAT-30.SPEC-012, FEAT-25.SPEC-004 |
| Refund could not complete immediately | The capability reports it cannot complete the refund on this attempt (for example, the Pro's payout balance cannot yet cover it) | Deposit Transaction.status set to Refund in Progress; a retry is scheduled on a fixed cadence (platform parameter: `refund-retry-interval-hours`) | Talia's dashboard shows a clear attention flag (FEAT-08.SPEC-006); Riley's booking view shows the refund as "in progress," never as failed or silent (FEAT-30.SPEC-012) | FEAT-12, FEAT-30.SPEC-012, FEAT-25.SPEC-004 |
| Refund completes after a retry | A scheduled retry succeeds on a later attempt | Deposit Transaction.status set to Refunded; refund timestamp recorded | Talia's attention flag clears; Riley's "in progress" status updates to refunded, and the follow-up confirmation reflects the completed refund; FEAT-25.SPEC-004 is signalled of the completed refund | FEAT-12, FEAT-30.SPEC-012, FEAT-25.SPEC-004 |

## Degradation Behavior

This integration is triggered by FEAT-30.SPEC-009's and FEAT-30.SPEC-008's automations, not directly by a screen action, so no screen sends the refund request itself; the rows below cover the screens where this capability's trouble is visible to a user.

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-12 (Pro Daily Schedule Dashboard, attention list) | No visible change -- the refund request is not user-initiated on this screen, so a slow response produces no waiting state here; the attention flag simply has not yet appeared | If the capability is unreachable when the refund is requested, the attention flag reads "A refund for {client name}'s cancelled booking is in progress and will complete automatically." -- Talia sees no action she needs to take, and the flag persists until this spec's retry succeeds | If the capability explicitly rejects the refund request (for example, the payout account is no longer valid), the attention flag reads "A refund for {client name}'s cancelled booking needs attention -- your payout account may need reconnecting." with a link into FEAT-28 |
| Riley's own booking status view (FEAT-06/FEAT-10) | No visible change -- Riley sees no waiting state for a refund that has not yet been requested to fail or succeed | Riley's booking shows "Your deposit refund is in progress and will complete automatically." -- never a failure message | Riley's booking shows the same "in progress" wording; she is never shown the capability's rejection reason directly, since the resolution (e.g., reconnecting the payout account) is Talia's action, not hers |
| FEAT-30.SPEC-003 (Goodwill Deposit Refund) | No visible change -- Talia's confirm has already completed (Committing) by the time this integration's request reaches the capability, so no additional waiting state appears on the screen itself | Talia's confirm resolves to the screen's "Committed -- in progress" state ("Refund started. It will complete automatically -- you don't need to do anything.") rather than the immediate "Refund sent" confirmation, and FEAT-12's attention flag persists until this integration's retry succeeds | Same "Committed -- in progress" outcome is shown to Talia; the rejection itself surfaces on FEAT-12's attention flag (with the payout-reconnection link into FEAT-28), never as a failure on this screen, and the goodwill refund's already-committed decision is never reversed |
| FEAT-30.SPEC-005 (Cancel Several Bookings at Once, outcome summary) | No visible change -- the bulk commit's confirm has already completed by the time refunds process | A booking whose refund cannot complete immediately still shows in the per-booking summary as "Cancelled -- refund in progress", never as a failure | Same "refund in progress" wording; a rejected refund never blocks or reverses that booking's already-committed cancellation |

## Consent and Disclosure

- **No new disclosure moment for the refund itself** -- Riley already agreed, at the moment she paid her deposit (FEAT-07), that the payment-processing capability holds and processes her payment method; reversing that same capture through the same capability requires no additional consent screen.
- **Payout account reference disclosure** -- Talia was told, when she connected her payout account (FEAT-28), that it would be used to receive deposits and to fund refunds and payouts; this integration's use of that same reference for a goodwill or bulk-cancellation refund draws on that existing disclosure and requires no repeated notice.
- **What is never shared** -- Booking details, the Client's contact fields, and every Deposit Transaction field beyond amount, currency, and the deposit reference stay inside the product; the refund request never carries client contact information to the capability.

## Edge Cases

- **A refund-succeeded event arrives for a Deposit Transaction already marked Refunded** -- The second delivery changes nothing: the Deposit Transaction stays Refunded with its original refund timestamp, and no duplicate confirmation content fires, per the idempotency key discipline.
- **A refund-could-not-complete event arrives after a refund-succeeded event for the same deposit (out-of-order delivery)** -- The Deposit Transaction reflects the most recent true state, not arrival order: since a deposit can be refunded only once, a genuine refund-succeeded event is authoritative and a stale not-yet-completed report arriving late is treated as superseded and produces no status change.
- **Two overlapping retry attempts for the same Deposit Transaction are both processed** -- Only one results in a Refunded status; the other, whichever resolves second, finds the Deposit Transaction already Refunded via the idempotency key and is treated as a no-op with no duplicate refund.
- **Talia's payout account is reconnected mid-retry (the blocking condition resolves before the next scheduled attempt)** -- The next scheduled retry (platform parameter: `refund-retry-interval-hours` after the previous attempt) picks up the now-resolved payout account automatically; the refund completes on that attempt with no separate action from Talia beyond reconnecting.
- **A refund stays in progress for an extended period because the underlying blocking condition never resolves** -- The retry continues indefinitely on its fixed cadence; the Deposit Transaction is never silently abandoned or moved to a terminal failure state, and Talia's attention flag persists throughout, per XBR-10's "never dropped" guarantee.
- **The Booking or Client the refund relates to is deleted or de-identified before the refund event arrives** -- The event is still applied to the retained, de-identified financial record (per SC-22); no user feedback fires since there is no longer an active client-facing view to show it to.
- **One booking in a bulk cancellation's refund cannot complete immediately while its siblings' refunds succeed** -- Each Deposit Transaction is tracked and retried independently; the slow booking's "in progress" status never blocks or delays the confirmed refunds of the other bookings in the same bulk action.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-30.SPEC-009 (Goodwill Refund Commit) | Triggered by (inbound) | A determined-due goodwill refund initiates this integration's request |
| FEAT-30.SPEC-008 (Bulk Cancellation Commit) | Triggered by (inbound) | Each booking's determined-due refund within a bulk cancellation initiates its own request here |
| FEAT-28 (Payout Account Connection & Payout Visibility) | References (outbound) | Refunds draw on the Pro's connected payout account balance |
| FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) | A refund that cannot complete immediately is flagged clearly on the Pro's dashboard until it resolves |
| FEAT-08.SPEC-006 (Pro Attention Alert) | Triggers (outbound) | A refund failure on this integration's first attempt fires the Pro-facing attention alert |
| FEAT-30.SPEC-012 (Pro Action Client Notice) | Triggers (outbound) | The refund outcome (confirmed or in progress) feeds the client notice this integration's callers trigger |
| FEAT-09.SPEC-005 (Automatic Deposit Refund) | References (outbound) | The sibling integration that executes a single Pro cancellation's or reschedule's own refund; this spec never duplicates that path |
| FEAT-25.SPEC-004 (Historical Aggregate Maintenance) -- within FEAT-25 (Booking & Revenue Insights) | Triggers (outbound) | Each Deposit Transaction outcome this integration applies (Refund in Progress, Refunded) is an inbound event to that automation, which updates the insights aggregates |
| FEAT-23.SPEC-003 (Tip Payout & Refund Rule) -- within FEAT-23 (Tipping at Checkout) | Enforces (outbound) | Enforces that rule for the refund set this integration executes: where a booking has a tipped, succeeded Balance Payment, the full-refund-including-tip guarantee applies, and the tip is refunded only as part of the Balance Payment's own Refunded transition (FEAT-22.SPEC-005), never as a standalone refund; this integration adds no tip-handling logic of its own |
| FEAT-09.SPEC-006 (Refund Idempotency & Retry Rule) | References (outbound) | This spec's own idempotency-key and retry-cadence discipline mirrors that spec's guarantee for FEAT-09's refund path, per the Brief's Shared Validation note |

## Analytics and Success Signals

- **goodwill_bulk_refund_requested** (source: goodwill / bulk_cancellation) -- supports success-metrics.md: "Automatic Refund Correctness"
- **goodwill_bulk_refund_outcome_received** (outcome: succeeded / could-not-complete) -- supports success-metrics.md: "Automatic Refund Correctness"
- **goodwill_bulk_refund_retry_scheduled** (attempt_number) -- supports success-metrics.md: "Automatic Refund Correctness" (a refund that cannot complete immediately must still be shown as "in progress" and complete without either party chasing it; this event measures how often the retry path is exercised)

## Acceptance Criteria

**FEAT-30.SPEC-011-AC-01:** Given FEAT-30.SPEC-009 determines a goodwill refund is due, when this integration requests it and the capability confirms immediately, then the Deposit Transaction is set to Refunded with a refund timestamp, and Riley sees the confirmation reflecting her refund.

**FEAT-30.SPEC-011-AC-02:** Given FEAT-30.SPEC-008 determines a refund is due for one booking in a bulk cancellation, when this integration requests it and the capability confirms immediately, then that booking's Deposit Transaction is set to Refunded independently of the other bookings in the set.

**FEAT-30.SPEC-011-AC-03:** Given a refund request is sent to the capability, when the capability reports it cannot complete on this attempt, then the Deposit Transaction is set to Refund in Progress, Talia's dashboard shows the attention flag, and Riley's booking shows "in progress," never a failure.

**FEAT-30.SPEC-011-AC-04:** Given the capability is unreachable when a refund is requested, then Talia's dashboard shows "A refund for {client name}'s cancelled booking is in progress and will complete automatically." with no action required from her yet.

**FEAT-30.SPEC-011-AC-05:** Given the capability explicitly rejects a refund request because the payout account is no longer valid, then Talia's dashboard shows "A refund for {client name}'s cancelled booking needs attention -- your payout account may need reconnecting." with a link into FEAT-28.

**FEAT-30.SPEC-011-AC-06:** Given a refund that could not complete immediately, when the next scheduled retry runs (platform parameter: `refund-retry-interval-hours` after the previous attempt), then a new refund request is submitted carrying the same attempt's idempotency key.

**FEAT-30.SPEC-011-AC-07:** Given a Deposit Transaction is already Refunded, when the same refund-succeeded event is delivered again, then nothing changes and no duplicate confirmation content fires.

**FEAT-30.SPEC-011-AC-08:** Given two overlapping retry attempts for the same Deposit Transaction both resolve, then only one results in a Refunded status, and the other is treated as a no-op with no duplicate refund.

**FEAT-30.SPEC-011-AC-09:** Given Talia reconnects her payout account while a goodwill refund sits at Refund in Progress, when the next scheduled retry runs, then the refund completes automatically with no further action from Talia.

**FEAT-30.SPEC-011-AC-10:** Given a refund's blocking condition never resolves, when successive scheduled retries continue to fail, then the Deposit Transaction remains Refund in Progress indefinitely rather than moving to a terminal failure state, and Talia's attention flag persists throughout.

**FEAT-30.SPEC-011-AC-11:** Given the Client whose deposit is being refunded has since been deleted, when the refund event arrives, then it is applied to the retained de-identified financial record with no user feedback fired.

**FEAT-30.SPEC-011-AC-12:** Given one booking's refund within a bulk cancellation cannot complete immediately while its siblings' refunds succeed, when the outcomes are reported, then the slow booking shows "refund in progress" while the others show refunded, independently.

**FEAT-30.SPEC-011-AC-13:** Given this integration requests a refund, when the request is composed, then only the refund amount, currency, deposit reference, payout account reference, and idempotency key are sent -- Booking details and Client contact fields are never included.

**FEAT-30.SPEC-011-AC-14:** Given a refund that could not complete immediately eventually succeeds after a retry, then the Deposit Transaction is set to Refunded, Talia's attention flag clears, and Riley's "in progress" status updates to reflect the completed refund.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 3 | 3 |
| Inbound Events | 3 | 3 |
| Degradation Paths | 12 (4 screens x 3 conditions) | 12 |
| Consent and Disclosure | 3 | 3 |
| Edge Cases | 7 | 7 |



# Notification Spec: Pro Action Client Notice

## Overview

**Name:** Pro Action Client Notice
**ID:** FEAT-30.SPEC-012
**Type:** Notification
**Purpose:** The trigger-and-audience contract that ensures the client is told when the Pro cancelled their booking (with refund status), rescheduled it (with the new time and a fresh manage link), or issued a goodwill refund -- so no client is ever left wondering whether a Pro-initiated change went through or what happened to their deposit. The message content is owned by FEAT-08.SPEC-004; this spec defines when it fires, for whom, and what data this feature supplies.
**Parent Feature:** FEAT-30 -- Pro Booking Management

## Scope and Non-Goals
**In Scope:**
- The trigger events in this feature that require a client notice: single cancellation, single reschedule, bulk cancellation (per affected booking), and goodwill refund (completed or in-progress)
- The audience contract: exactly one client recipient per event, the Client tied to the affected Booking
- The data this feature supplies to FEAT-08.SPEC-004 for each event (outcome type, deposit outcome, new time, fresh manage link)
- The Pro-side counterpart trigger: the same events also reach FEAT-08.SPEC-005 (Pro Booking Activity Notification), which owns the Pro-facing content

**Non-Goals:**
- Message wording, channels, placeholders, and text/email variants -- owned by FEAT-08.SPEC-004 (Booking Change & Refund Notice), which already defines the client-facing content for every Pro-initiated event; this spec defines no duplicate content
- The client-facing notice for a client-initiated cancellation or reschedule -- triggered by FEAT-10.SPEC-006 and rendered by FEAT-08.SPEC-004; this spec covers only Pro-initiated events
- The Pro-facing content of any of these events -- owned by FEAT-08.SPEC-005 (routine activity) and FEAT-08.SPEC-006 (a refund failure needing attention); this spec is client-facing only
- Deciding the deposit outcome itself -- owned by FEAT-09 (Cancellation & No-Show Policy Engine, XBR-09) for a Pro cancellation or reschedule, and by FEAT-30.SPEC-009/FEAT-30.SPEC-011 for a goodwill refund; this spec only reports the outcome those specs determine
- Issuing the fresh manage link -- owned by FEAT-08.SPEC-010; the actual text/email send mechanics -- owned by FEAT-08.SPEC-012 (text) and FEAT-08.SPEC-013 (email)

## Channels
| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Text | The client has active Messaging Consent for texting (per FEAT-14, evaluated by FEAT-08.SPEC-011's channel-selection rule) | Channel choice and wording are defined by FEAT-08.SPEC-004; a Pro-initiated cancellation, reschedule, or refund is time-sensitive and financially material, so this contract always requires the notice to go out |
| Email | The client has not granted texting consent | Ensures the notice always reaches the client, per BRIEF.md's stated email fallback; delivered per FEAT-08.SPEC-004 |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Talia's single cancellation or reschedule commits | FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) | Always, on a successfully saved Pro-initiated cancellation or reschedule; this contract passes the event on to FEAT-08.SPEC-004 (client) and FEAT-08.SPEC-005 (Pro) | Booking (updated state, new time if rescheduled), Deposit Transaction (outcome), fresh manage link (reschedule only, from FEAT-08.SPEC-010) |
| A booking within a bulk cancellation commits | FEAT-30.SPEC-008 (Bulk Cancellation Commit) | Fires once per successfully cancelled booking in the reviewed set | Booking (updated state), Deposit Transaction (outcome) |
| A goodwill refund completes or enters progress | FEAT-30.SPEC-009 (Goodwill Refund Commit) via FEAT-30.SPEC-011 (Goodwill & Bulk-Cancellation Refund Execution) | On the Deposit Transaction reaching Refunded or Refund in Progress from a goodwill action; passed on to FEAT-08.SPEC-004 | Deposit Transaction (status, outcome_reason: goodwill) |

## Audience and Preferences

**Recipients:** The Client tied to the affected Booking (Access Matrix: Booking & Payment = Own-only), the sole recipient. Platform Operator (Support) has View-only access to delivery status only, per the Access Matrix.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Texting consent (governs channel, not whether this notice sends) | Granted / Revoked | Captured at booking | FEAT-06 at booking; changed via FEAT-14 |

This notice carries no separate opt-out: a change to the client's own paid booking, made by the Pro, is transactional information the client cannot decline to receive (consistent with FEAT-08.SPEC-004).

**Quiet Hours:** N/A -- this notice is the direct, expected report of a change the Pro just made to the client's own booking, not an unprompted interruption; it sends immediately regardless of time of day. XBR-16's daytime window governs only the discretionary pre-appointment reminder (FEAT-08.SPEC-002), not this transactional notice, and FEAT-08.SPEC-004 applies the same rule.

## Content Definition
The client-facing message content (text bodies, email subject and body, CTA, and placeholders) is owned entirely by FEAT-08.SPEC-004 (Booking Change & Refund Notice); this spec restates none of it. For each event this feature raises, the table states which FEAT-08.SPEC-004 content applies and what data this feature must supply so that content renders correctly.

| Event Raised By This Feature | FEAT-08.SPEC-004 Content Applied | Data This Feature Supplies |
|------------------------------|----------------------------------|----------------------------|
| Single Pro cancellation (FEAT-30.SPEC-007) | The Pro-initiated cancellation variant (always a full refund, XBR-09) | Booking reference, original start_time, deposit amount, deposit outcome = refunded in full |
| Single Pro reschedule (FEAT-30.SPEC-007) | The Pro-initiated reschedule variant | Booking reference, new start_time, deposit outcome = carried over (XBR-09), the fresh manage link issued by FEAT-08.SPEC-010 (XBR-18) |
| Each booking cancelled in a bulk cancellation (FEAT-30.SPEC-008) | The Pro-initiated cancellation variant, once per booking | Same data as a single cancellation, per booking |
| Goodwill refund completed (FEAT-30.SPEC-009 via FEAT-30.SPEC-011) | The refund outcome content for a completed Pro-issued refund | Booking reference, deposit amount, deposit outcome = refunded, outcome_reason = goodwill |
| Goodwill refund in progress (FEAT-30.SPEC-009 via FEAT-30.SPEC-011) | The refund-in-progress variant | Booking reference, deposit amount, deposit outcome = in progress |

The Pro-facing counterpart of these events (routine activity) is composed by FEAT-08.SPEC-005 from the same trigger; a refund that fails to complete is flagged to the Pro by FEAT-08.SPEC-006.

## Delivery Rules

**Batching:** None -- each change event (a cancellation, a reschedule, a bulk-cancellation booking, or a goodwill-refund state transition) produces its own single notice at the moment it occurs. A bulk cancellation of four bookings produces four independent notices, one per affected client, never one combined message.
**Deduplication:** At most one notice per triggering event. A goodwill refund transitioning from "in progress" to "completed" is itself a second, distinct event and produces its own follow-up notice.
**Retry on failure:** Governed by FEAT-08.SPEC-009 and executed by FEAT-08.SPEC-004: a failed text is retried once, then falls back to email, with the gap flagged on the Pro's dashboard (FEAT-08.SPEC-006).
**Expiry:** None -- a change or refund notice never becomes not-worth-sending; it reports a fact about the client's own money and appointment that remains true and relevant no matter when it is finally delivered.

## Edge Cases

- **Talia cancels a booking that Riley had already tried to cancel moments earlier through FEAT-10** -- Per the Booking entity's Contention resolution (reject-with-refresh), only the first committed transition applies; this notice reports the transition that actually committed, and Riley receives exactly one notice for it, not two.
- **A goodwill refund fails outright rather than merely being slow** -- Riley still sees only the "refund in progress" wording, never a failure message; the underlying failure is retried automatically (FEAT-30.SPEC-011) and flagged only to Talia (FEAT-08.SPEC-006), per XBR-10.
- **One booking within a bulk cancellation fails eligibility while its siblings succeed** -- Only the successfully cancelled bookings' clients receive this notice; the client of the failed booking receives no notice for an action that never committed.
- **Riley's texting consent is revoked between booking and this notice** -- The notice honors the consent state current at send time (FEAT-08.SPEC-011), routing to email if consent is no longer active.
- **A Pro-made reschedule lands on a booking whose fresh manage link has not yet been issued at the instant this notice would fire** -- FEAT-30.SPEC-007 requests the link from FEAT-08.SPEC-010 before raising this event (XBR-18), and the event carries that fresh link, so the manage link in the message FEAT-08.SPEC-004 renders is never empty or a stale, superseded link.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) | Triggered by (inbound) | A single Pro cancellation or reschedule fires this notice |
| FEAT-30.SPEC-008 (Bulk Cancellation Commit) | Triggered by (inbound) | Each successfully cancelled booking within a bulk action fires this notice independently |
| FEAT-30.SPEC-009 (Goodwill Refund Commit) | Triggered by (inbound) | A completed or in-progress goodwill refund fires this notice |
| FEAT-30.SPEC-011 (Goodwill & Bulk-Cancellation Refund Execution) | Triggered by (inbound) | A refund outcome (completed or in-progress) from this integration feeds the content this notice reports |
| FEAT-08.SPEC-004 (Booking Change & Refund Notice) | Triggers (outbound) | Content owner: renders and delivers the client message for every event this contract raises; FEAT-08.SPEC-004 lists this spec as its trigger-and-audience contract |
| FEAT-08.SPEC-005 (Pro Booking Activity Notification) | Triggers (outbound) | Pro-facing counterpart: composes the Pro's routine-activity notification from the same events |
| FEAT-08.SPEC-010 (Booking-Specific Manage Link Issuance) | References (inbound) | Source of the fresh manage link supplied with a reschedule event (XBR-18) |
| FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule) | References (inbound) | Decides text vs. email for this send |
| FEAT-08.SPEC-012 / FEAT-08.SPEC-013 (Text / Email Capabilities) | References (outbound) | Perform the actual send, via FEAT-08.SPEC-004 |
| FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | References (outbound) | Governs retry and fallback on failure |

## Analytics and Success Signals

- **pro_action_client_notice_sent** (change_type: pro_cancel / pro_reschedule / bulk_cancel / goodwill_refund / goodwill_refund_in_progress; channel) -- supports success-metrics.md: "Pro Change Correctness"
- **pro_action_client_notice_refund_outcome_shown** (outcome: refunded / carried_over / in_progress) -- supports success-metrics.md: "Automatic Refund Correctness"
- **pro_action_client_notice_cta_tapped** (destination: manage_link) -- supports success-metrics.md: "Self-Service Access Success"

## Acceptance Criteria

**FEAT-30.SPEC-012-AC-01:** Given Talia cancels Riley's booking, when FEAT-30.SPEC-007 completes the cancellation, then this contract raises the cancellation event to FEAT-08.SPEC-004 with deposit outcome "refunded in full", and Riley receives the notice FEAT-08.SPEC-004 defines for it, regardless of timing.

**FEAT-30.SPEC-012-AC-02:** Given Talia reschedules Riley's booking to a new time, when FEAT-30.SPEC-007 completes the reschedule, then this contract raises the reschedule event to FEAT-08.SPEC-004 with the new date and time and deposit outcome "carried over", and Riley receives the notice FEAT-08.SPEC-004 defines for it.

**FEAT-30.SPEC-012-AC-03:** Given Talia cancels four bookings at once through FEAT-30.SPEC-008, when each booking successfully commits, then each affected client receives their own independent cancellation notice, never one combined message.

**FEAT-30.SPEC-012-AC-04:** Given Talia issues a goodwill refund that completes immediately, when FEAT-30.SPEC-011 confirms it, then this contract raises the completed-refund event to FEAT-08.SPEC-004 with outcome_reason goodwill, and Riley receives the notice FEAT-08.SPEC-004 defines for it.

**FEAT-30.SPEC-012-AC-05:** Given a goodwill refund for Riley's booking cannot complete immediately, when FEAT-30.SPEC-011 sets the Deposit Transaction to Refund in Progress, then this contract raises the refund-in-progress event to FEAT-08.SPEC-004, and Riley receives the in-progress notice FEAT-08.SPEC-004 defines for it.

**FEAT-30.SPEC-012-AC-06:** Given a goodwill refund that was "in progress" for Riley later completes, when the Deposit Transaction updates to Refunded, then this contract raises a second, distinct completed-refund event, and Riley receives the follow-up notice FEAT-08.SPEC-004 defines for it.

**FEAT-30.SPEC-012-AC-07:** Given Riley cancelled her own booking through FEAT-10 moments before Talia's cancellation commit runs, when only the first transition commits, then Riley receives exactly one notice, reflecting the committed transition.

**FEAT-30.SPEC-012-AC-08:** Given Talia's bulk cancellation reports one booking as failed, when the outcome is processed, then that failed booking's client receives no notice, since the cancellation never committed.

**FEAT-30.SPEC-012-AC-09:** Given Riley has revoked texting consent since booking, when a Pro-action notice for her booking is triggered, then it is sent by email, honoring her current consent state.

**FEAT-30.SPEC-012-AC-10:** Given a text notice to Riley fails to deliver, when FEAT-08.SPEC-009's retry-then-fallback runs, then Riley still receives the notice by email.

**FEAT-30.SPEC-012-AC-11:** Given Talia's reschedule triggers a fresh manage link for Riley, when this notice sends, then the event this contract raises carries that fresh link from FEAT-08.SPEC-010, and the notice never carries the booking's previous, now-superseded link.

**FEAT-30.SPEC-012-AC-12:** Given Riley taps "Manage my booking" from this notice, when the tap registers, then the pro_action_client_notice_cta_tapped event fires and she is taken to her booking through the manage link that FEAT-08.SPEC-004 places in the notice.

**FEAT-30.SPEC-012-AC-13:** Given Talia's single cancellation commits, when this contract fires, then FEAT-08.SPEC-005 also receives the event so Talia's own routine-activity notification is composed there, and this spec defines no Pro-facing content.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (text, email) | 2 |
| Trigger Paths | 3 (single commit, bulk-per-booking, goodwill outcome) | 3 |
| Preference States | 2 (consent granted, consent revoked/declined) | 2 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |



# Notification Spec: Deposit Request & Expiry Notice

## Overview

**Name:** Deposit Request & Expiry Notice
**ID:** FEAT-30.SPEC-013
**Type:** Notification
**Purpose:** Delivers a Pro-created deposit request to the client -- by text, by email, or as an on-screen code to scan -- and, when an unpaid request expires, stops any pending delivery for it. The Pro's expiry notice is not defined here: FEAT-03.SPEC-007 is its trigger and FEAT-08.SPEC-006 is its content owner.
**Parent Feature:** FEAT-30 -- Pro Booking Management

## Scope and Non-Goals

**In Scope:**
- The client-facing deposit-request delivery, on its three delivery paths (text link, email link, on-screen code)
- Stopping any still-pending delivery of a deposit request once its hold expires unpaid (the Pro's expiry notice itself is referenced, not defined, here)
- The channel-decision rule for choosing among the three delivery paths

**Non-Goals:**
- Creating the Booking, placing the slot hold, or computing the hold's expiry -- owned by FEAT-30.SPEC-010 (Pro-Created Booking & Deposit Request Hold) and FEAT-03.SPEC-007; this spec only delivers the request once those specs create it, and reports the expiry outcome those specs determine
- Capturing the client's deposit payment itself -- owned by FEAT-07 (Deposit Payment at Booking); this notice's link or code hands off to that flow, it does not process payment
- The Pro-facing expiry notice for an unpaid request -- triggered by FEAT-03.SPEC-007 (the sole writer of Booking -> Expired (unpaid), XBR-02) and content-owned by FEAT-08.SPEC-006 (Pro Attention Alert); this spec defines no duplicate content, channels, or preferences for it
- The outcome notice for a cancellation, reschedule, or goodwill refund -- owned by FEAT-30.SPEC-012 (Pro Action Client Notice), a distinct content class from a payment request
- Choosing the delivery channel's underlying send mechanics (text/email) -- owned by FEAT-08.SPEC-012/FEAT-08.SPEC-013, the category-level transactional messaging capability every notification in the product sends through

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Text | The client has active Messaging Consent for texting (FEAT-14) | The fastest way for a client to act on a time-limited deposit request while the moment (e.g., booking their next visit at the chair) is fresh |
| Email | The client has not granted texting consent | Ensures the request always reaches the client, per BRIEF.md's stated email fallback |
| On-screen code | Talia chooses to show the request on her own screen rather than send it remotely (e.g., the client is standing at the chair) | Lets a client without a phone number capture step, or one who prefers to act immediately, pay on the spot by scanning |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A Pro-created deposit request is issued | FEAT-30.SPEC-010 (Pro-Created Booking & Deposit Request Hold) | Fires the instant the Booking and its slot hold are created successfully | Booking (service, start_time, deposit_amount), Client (name, phone or email), delivery choice Talia selected on FEAT-30.SPEC-004 |
| A Pro-created deposit request expires unpaid (reference only) | FEAT-03.SPEC-007 (Pro-Created Deposit Request Hold & Expiration) -- the trigger for the Pro's expiry notice, whose content FEAT-08.SPEC-006 owns | Fires when the hold's computed expiry passes with the deposit never paid; this spec's only action is to withdraw any still-pending deposit-request delivery for that Booking | Booking reference |

## Audience and Preferences

**Recipients:** The Client Talia is booking in (Access Matrix: Booking & Payment = Own-only for the Client's own booking; created here by Talia's Full access). The Pro's expiry notice, addressed to Talia (Access Matrix: Booking & Payment = Full for the Pro), is defined by FEAT-08.SPEC-006, not here. Platform Operator (Support) has View-only access to delivery status only, per the Access Matrix.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Texting consent (governs the client's delivery channel, not whether the request is delivered) | Granted / Revoked | Captured at the client's first booking, or granted inline if this is a brand-new client entered by Talia | FEAT-06 at booking; changed via FEAT-14 |
| Delivery method choice (text/email link vs. on-screen code) | Link (text or email, per consent) / On-screen code | Talia's explicit choice on FEAT-30.SPEC-004 for each booking-in action -- no stored default | FEAT-30.SPEC-004 |

The deposit request itself carries no client opt-out: it is the client's own payment request for a booking Talia entered on their behalf, transactional by nature. The Pro's channel preferences for the expiry notice (set in FEAT-27) are applied by FEAT-08.SPEC-006, not by this spec.

**Quiet Hours:** N/A -- the deposit request is delivered immediately regardless of time of day, since it is the direct, expected consequence of Talia's own booking-in action at the moment she takes it (often at the chair, in front of the client).

## Content Definition

**Text (deposit request):**
- **Body:** {pro_display_name} has booked you in for {service_name} on {appointment_date} at {appointment_time}. Pay your {deposit_amount} deposit to confirm: {deposit_link}. This link expires in {hold_window_description}.

**Email (deposit request):**
- **Subject:** Confirm your appointment with {pro_display_name}
- **Body:** Hi {client_first_name}, {pro_display_name} has booked you in for {service_name} on {appointment_date} at {appointment_time}. Pay your {deposit_amount} deposit to confirm your spot.
- **CTA (button):** Pay deposit -- deep-links to FEAT-07 (Deposit Payment at Booking) for this Booking

**On-screen code (shown on Talia's device):**
- **Title:** Scan to pay your deposit
- **Body:** {client_first_name}, scan this code to pay your {deposit_amount} deposit for {service_name} on {appointment_date}.
- **CTA:** The code itself deep-links to FEAT-07 (Deposit Payment at Booking) for this Booking when scanned

**Pro expiry notice:** Not defined in this spec. When a deposit request's hold expires unpaid, FEAT-03.SPEC-007 triggers FEAT-08.SPEC-006 (Pro Attention Alert), which owns the in-app alert, text, and email content and the Pro's channel handling.

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {pro_display_name} | Pro Account -- display_name | Talia | Never empty (required field) |
| {client_first_name} | Client -- name (first token) | Riley | Renders the full name field if no separable first token exists |
| {service_name} | Service -- name | Full Set Lashes | Never empty (required field) |
| {appointment_date} / {appointment_time} | Booking -- start_time | Oct 4, 2026 / 2:30 PM | Never empty -- fixed at booking |
| {deposit_amount} | Deposit -- computed once from the Service's rule (FEAT-07) | $40.00 | Never empty -- computed at booking creation |
| {deposit_link} | Derived -- the deposit-payment deep link for this specific Booking (FEAT-07) | chairtime.app/pay/9c1f2a | Never empty -- generated the instant the Booking and its hold are created |
| {hold_window_description} | Derived -- a plain-language rendering of whichever of platform parameter: `deposit-request-hold-max-hours` or platform parameter: `deposit-request-hold-appointment-cutoff-hours` governs this specific request | 24 hours / 2 hours | Never empty -- always resolves to one of the two governing limits |

## Delivery Rules

**Batching:** None -- each deposit request is its own single send tied to one Booking; Talia booking in several clients in succession produces one independent request per client, never a combined message.
**Deduplication:** At most one deposit-request send per booking-in action, and no expiry message from this spec (the Pro's expiry notice belongs to FEAT-08.SPEC-006). Talia re-sending the same request (e.g., the client asks her to resend the link) triggers a fresh send of the same content, not treated as a new booking or a duplicate expiry.
**Retry on failure:** Governed by FEAT-08.SPEC-009 for the client-facing text/email delivery: a failed text is retried once, then falls back to email; the on-screen code path has no delivery-failure mode, since it renders directly on Talia's own device. 
**Expiry:** The deposit-request notice itself never separately "expires" -- its content states the governing hold window, and the underlying Booking's hold expiring is what FEAT-03.SPEC-007 and FEAT-30.SPEC-010 own; once that hold expires, no further reminder is sent for the same request (per the Brief's Side-Effect Inventory: "the client receives no further reminder for this booking"). 

## Edge Cases

- **Talia chooses the on-screen code instead of a link** -- No text or email is sent at all for the request itself; the code renders directly on her device, and the client scans it in person. If the code is never scanned in time, the hold expires as usual and Talia is notified by FEAT-08.SPEC-006.
- **The client pays the deposit before this notice's text or email delivery completes** -- The pending send is superseded: no further reminder about paying is delivered once FEAT-07 confirms the payment, since the request has already served its purpose.
- **A new client is entered by Talia with no texting consent captured yet (declined during entry, per FEAT-30.SPEC-004)** -- The deposit request delivers by email, using the required-when-texting-declined email address captured at entry (per FEAT-05's equivalent rule, applied here for a Pro-entered client).
- **The deposit request expires while a send is still pending or Talia is mid-way through booking another client** -- Any still-pending delivery for the expired request is withdrawn, and the expiry (whose Pro notice FEAT-08.SPEC-006 delivers independently) does not interrupt or merge with Talia's in-progress second booking-in action or its own deposit request.
- **Talia re-sends the same deposit request after the client says they didn't receive it** -- A fresh send of the identical content goes out on the same delivery method originally chosen; this does not reset the underlying hold's expiry, which FEAT-03.SPEC-007 continues to track from the hold's original creation time.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-30.SPEC-010 (Pro-Created Booking & Deposit Request Hold) | Triggered by (inbound) | A created Booking and hold fires the deposit-request delivery; an expired hold withdraws any pending delivery |
| FEAT-03.SPEC-007 (Pro-Created Deposit Request Hold & Expiration) | Triggered by (inbound) | The hold-expiry event withdraws any pending delivery here; the same event is the trigger for the Pro's expiry notice, owned by FEAT-08.SPEC-006 |
| FEAT-08.SPEC-006 (Pro Attention Alert) | References (outbound) | Content owner of the Pro's expiry notice; this spec defines no duplicate content |
| FEAT-30.SPEC-004 (Book Client In) | References (inbound) | Talia's delivery-method choice (link vs. on-screen code) governs which content variant sends |
| FEAT-07 (Deposit Payment at Booking) | Navigation (outbound) | Every client-facing variant's CTA deep-links here to complete payment |
| FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule) | References (inbound) | Decides text vs. email for the client-facing send |
| FEAT-08.SPEC-012 / FEAT-08.SPEC-013 (Text / Email Capabilities) | Triggers (outbound) | Perform the actual sends for the request |
| FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | References (outbound) | Governs retry and fallback on failure for the request |

## Analytics and Success Signals

- **deposit_request_delivered** (channel: text / email / on_screen_code) -- supports success-metrics.md: "Pro Change Correctness"
- **deposit_request_paid** (channel) -- supports success-metrics.md: "Pro Change Correctness"
- **pro_expiry_notice_signal** () -- N/A -- the Pro's expiry notice and its signal (pro_attention_alert_sent, condition: deposit_request_expired) belong to FEAT-08.SPEC-006; this spec sends no expiry notice
- **deposit_request_cta_tapped** (channel) -- N/A -- no Stage 2 metric measures deposit-request tap rate directly; retained alongside deposit_request_paid so the funnel between delivery and payment is observable rather than measured only at the endpoints.

## Acceptance Criteria

**FEAT-30.SPEC-013-AC-01:** Given Talia books Riley in and chooses to send a text deposit request, when FEAT-30.SPEC-010 creates the Booking and hold, then Riley receives a text naming the service, date, time, deposit amount, and a payment link with the governing hold window stated.

**FEAT-30.SPEC-013-AC-02:** Given Riley has not granted texting consent, when the deposit request is issued, then it is delivered by email instead, using her email address on file.

**FEAT-30.SPEC-013-AC-03:** Given Talia chooses the on-screen code instead of a link, when the Booking and hold are created, then a scannable code renders on Talia's device and no text or email is sent for the request.

**FEAT-30.SPEC-013-AC-04:** Given a Pro-created deposit request's hold expires unpaid, when FEAT-03.SPEC-007's expiration path fires, then any still-pending delivery of that request is withdrawn, and Talia's expiry notice is triggered by FEAT-03.SPEC-007 and delivered with the content and channels FEAT-08.SPEC-006 defines; this spec sends no expiry message itself.

**FEAT-30.SPEC-013-AC-05:** Given a deposit request expires unpaid, when the expiry occurs, then this spec sends Talia no text, email, or in-app message and defines no wording for one, since FEAT-08.SPEC-006 applies her notification_preferences to the expiry notice.

**FEAT-30.SPEC-013-AC-06:** Given Riley completes the deposit payment before this notice's text delivery finishes retrying, when payment is confirmed, then no further payment-reminder content is sent for the same request.

**FEAT-30.SPEC-013-AC-07:** Given Talia enters a brand-new client who declines texting during entry, when the deposit request is issued, then it delivers by email to the address captured at entry.

**FEAT-30.SPEC-013-AC-08:** Given a text deposit request fails to deliver, when FEAT-08.SPEC-009's retry-then-fallback runs, then Riley still receives the request by email.

**FEAT-30.SPEC-013-AC-09:** Given Talia re-sends the same deposit request after the client reports not receiving it, when the resend completes, then the identical content is delivered again on the same originally chosen method, and the hold's expiry timing is unaffected.

**FEAT-30.SPEC-013-AC-10:** Given a deposit request's text delivery is still retrying when its hold expires unpaid, when the expiry occurs, then no further delivery attempt for that request is made and no payment link is sent after expiry.

**FEAT-30.SPEC-013-AC-11:** Given the deposit request's on-screen code is scanned, when the client follows it, then they land on FEAT-07 (Deposit Payment at Booking) for that specific Booking.

**FEAT-30.SPEC-013-AC-12:** Given Talia is mid-way through booking a second client when a first client's deposit request expires, when the expiry occurs, then the second booking-in action and its own deposit request are unaffected.

**FEAT-30.SPEC-013-AC-13:** Given a deposit request's governing hold window is the appointment-proximity cutoff rather than the 24-hour cap, when the request's content renders, then {hold_window_description} states the shorter, correct window rather than always showing 24 hours.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 3 (text, email, on-screen code) | 3 |
| Trigger Paths | 2 (request issued, request expired -- reference only) | 2 |
| Preference States | 3 (consent granted, consent revoked/declined, delivery method choice) | 3 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |


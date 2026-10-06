---
document_type: feature-overview
feature_number: FEAT-30
feature_name: Pro Booking Management
feature_slug: pro-booking-management
priority_tier: Core
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 13
screen_count: 5
automation_count: 4
logic_rule_count: 1
integration_count: 1
notification_count: 2
---

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

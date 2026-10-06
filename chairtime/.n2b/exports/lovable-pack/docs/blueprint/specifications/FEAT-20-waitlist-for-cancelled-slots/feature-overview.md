---
document_type: feature-overview
feature_number: FEAT-20
feature_name: Waitlist for Cancelled Slots
feature_slug: waitlist-for-cancelled-slots
priority_tier: Nice-to-Have
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 9
screen_count: 2
automation_count: 3
logic_rule_count: 2
integration_count: 0
notification_count: 2
---

# Feature Breakdown Brief: Waitlist for Cancelled Slots

## Summary

**Feature:** Waitlist for Cancelled Slots
**ID:** FEAT-20
**Description:** A client can ask to be notified if a specific service and day opens up from someone else's cancellation, instead of repeatedly checking the booking page.
**Priority:** Nice-to-Have
**Phase:** v1
**Type:** User-Facing
**Rationale:** BRIEF.md's Open Questions names this exactly: "should a pro be able to offer a waitlist for slots that open up from cancellations?" As Visionary judgment: valuable but not required for the core one-minute-booking promise, and it depends on Client-Initiated Cancel/Reschedule (FEAT-10) already existing to generate openings. Phased to v1, once the core cancellation flow is proven. [RESEARCH-INFORMED: waitlists are offered by the closest solo-focused competitor as part of its business tools, from independent review aggregators (1 profile, MEDIUM confidence), confirming the pattern without making it a baseline expectation]

**Key Capabilities:**
- Join a waitlist for a specific service/day when no slot is currently free
- Get notified the moment a matching slot opens from a cancellation
- Book directly from the notification before anyone else can grab the slot
- Leave a waitlist at any time from their own booking view [AUDIT-ADDED: 3 -- entity coverage: Waitlist Entry had no client-side removal]

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-20.SPEC-001 | Join Waitlist | Screen | The Client | Client joins the waitlist for a service and day (or up to a 7-day range) from a fully booked service |
| FEAT-20.SPEC-002 | My Waitlists | Screen | The Client | Client views their own waitlist entries (position/status), sees the empty state when they hold none, and leaves any entry |
| FEAT-20.SPEC-003 | Waitlist Entry Validation & Limits | Logic/Rule | The Client | Governs what a valid waitlist join looks like -- service/date-range shape, the 3-active-entries-per-Pro cap, and the notice/horizon bounds a joined date range must respect |
| FEAT-20.SPEC-004 | Waitlist Priority & Claim Window Rule | Logic/Rule | The Client, The Pro | Governs which Requested entries match a freed slot, the 30-minute claim window, how the window interacts with general public availability, and how contested or withdrawn claims resolve |
| FEAT-20.SPEC-005 | Cancellation-Triggered Waitlist Matching | Automation | The Client, The Pro, Platform Operator (Support) | On a freed-slot signal from a cancellation, finds every matching Requested entry, transitions each to Notified, and hands off to the opening notification |
| FEAT-20.SPEC-006 | Waitlist Claim Conversion | Automation | The Client, Platform Operator (Support) | When a notified client completes the ordinary booking flow for the matching slot, converts their entry to Converted and leaves the other notified entries untouched |
| FEAT-20.SPEC-007 | Waitlist Entry Expiry | Automation | The Client, Platform Operator (Support) | Expires a Notified entry whose 30-minute claim window lapses unclaimed, and separately expires a Requested entry whose joined date range elapses with no matching opening ever found |
| FEAT-20.SPEC-008 | Waitlist Opening Notification | Notification | The Client | Notifies a matching client the moment their slot opens, states the 30-minute claim window, and carries the claim link into the booking flow |
| FEAT-20.SPEC-009 | Waitlist Expiry Notification | Notification | The Client | Informs a client that their waitlist entry has expired -- either an unclaimed opening or an unmatched date range -- so they are never left wondering |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Join a waitlist for a specific service/day when no slot is currently free | FEAT-20.SPEC-001, FEAT-20.SPEC-003 | Join Waitlist screen creates the entry; the validation/limits rule governs the service+date-range shape and the 3-per-Pro cap before it is accepted | Phase 2 (Explicit) |
| Get notified the moment a matching slot opens from a cancellation | FEAT-20.SPEC-005, FEAT-20.SPEC-008 | The matching automation finds and transitions the entry to Notified the instant a cancellation frees a slot; the notification spec sends the message | Phase 2 (Explicit) |
| Book directly from the notification before anyone else can grab the slot | FEAT-20.SPEC-008, FEAT-20.SPEC-006 | The opening notification carries a claim link into FEAT-05's ordinary booking flow; completing that booking is recognized by the claim-conversion automation, which marks the entry Converted | Phase 2 (Explicit) |
| Leave a waitlist at any time from their own booking view [AUDIT-ADDED: 3] | FEAT-20.SPEC-002 | Leave action on the My Waitlists screen, governed by SPEC-004's rule that a leave request wins over a pending notification | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-20.SPEC-004 | Waitlist Priority & Claim Window Rule | Phase 5 (Rule Discovery) | The Validation & Limits field's 30-minute claim window, the Alternate's simultaneous-notification/first-to-act contention, and XBR-28's window-vs-public-availability interaction are conditional rules referenced by three different specs -- past the standalone threshold |
| FEAT-20.SPEC-007 | Waitlist Entry Expiry | Phase 6 (Negative/Failure Analysis) | The feature description's own Alternate ("the waitlist entry expires unclaimed after a set period with no matching opening") and the States field's implied timeout are unhappy-path behavior the Key Capabilities never name directly |
| FEAT-20.SPEC-009 | Waitlist Expiry Notification | Phase 4 (Notification surfacing) | The Alternate's "the client is informed rather than left wondering indefinitely" is a communication with real audience and content rules, not a bare in-app toast |
| FEAT-20.SPEC-003 | Waitlist Entry Validation & Limits | Phase 5 (Rule Discovery) | The Validation & Limits field states multiple, interacting constraints (date-range shape, the 3-entry cap, notice/horizon bounds) shared by the join screen and, indirectly, the claim path -- past the standalone threshold |

## Entity-Lifecycle Coverage Matrix

**Entity: Waitlist Entry**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-20.SPEC-001 | Join Waitlist screen -- client submits a service and date (or up to 7-day range), validated by SPEC-003 before the entry is accepted in state Requested | -- |
| Read (single) | FEAT-20.SPEC-002 | My Waitlists screen loads each of the client's own entries to show its position/status | -- |
| Read (list) | FEAT-20.SPEC-002 | My Waitlists screen lists all of the client's own active entries, with the "you're not on any waitlists" empty state when none exist | Aggregate counts across clients are read by FEAT-12 for the Pro's Attention List -- a cross-feature read, not a screen of this feature (coordination note 7) |
| Update | FEAT-20.SPEC-005, FEAT-20.SPEC-006, FEAT-20.SPEC-007 | State transitions only -- Requested to Notified (SPEC-005), Notified to Converted (SPEC-006), Requested-or-Notified to Expired (SPEC-007). No other fields are ever edited after creation | -- |
| Delete/Archive | FEAT-20.SPEC-002 | Hard delete -- leaving removes the record entirely (dependency map: "Deleted by FEAT-20 (client leaves)"), matching the Entity Inventory's four lifecycle states holding no fifth "left" state. No restore path: a client who changes their mind rejoins fresh through SPEC-001, counted freshly against the 3-per-Pro cap. No cascade: an entry that never converted has no dependent records, and a converted entry's resulting Booking is a fully independent record unaffected by any later, unrelated leave. Retention/purge: N/A -- nothing persists once deleted, so no purge policy is needed for this path | Decision recorded per coordination note 8: leaving is treated as delete, not a fifth state |
| State Transition | FEAT-20.SPEC-005, FEAT-20.SPEC-006, FEAT-20.SPEC-007 | Requested -> Notified (matching automation) -> Converted (claim conversion) or Expired (claim-window or unmatched-range expiry); Requested can also go directly to Expired if the joined date range elapses with no match ever found; a leave (delete) can occur from Requested or Notified and wins over a pending notification per SPEC-004's contention rule | Converted and Expired entries are not deleted -- retained indefinitely as a bounded historical record (see Non-Goals: no automatic purge of terminal entries) |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Service | FEAT-20.SPEC-001, FEAT-20.SPEC-003 | Read to show the service being waitlisted and to validate that the requested service exists and is Active; this feature never edits a Service |
| Booking | FEAT-20.SPEC-006 | Read to confirm the completed booking matches the notified slot before converting the entry; the booking itself is created by FEAT-05, never by this feature |
| Availability Rule (via FEAT-03) | FEAT-20.SPEC-004, FEAT-20.SPEC-005 | Consulted indirectly through FEAT-03's live slot truth to confirm an opening genuinely matches a waitlisted service/day; this feature never reads the rule directly |
| Messaging Consent | FEAT-20.SPEC-008, FEAT-20.SPEC-009 | Read to determine text-vs-email channel for both notifications (XBR-15); this feature never changes consent |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Client submits a waitlist join request | Validate service/date-range shape, the 3-active-entries-per-Pro cap, and notice/horizon bounds | Standalone Logic/Rule | FEAT-20.SPEC-003 |
| Client submits a valid waitlist join request | Create the Waitlist Entry in state Requested | Inline in triggering screen | FEAT-20.SPEC-001 |
| Client submits an invalid join request (cap exceeded, bad range) | Show a plain reason and keep the client on the join form | Inline in triggering screen | FEAT-20.SPEC-001 |
| A cancellation frees a slot (inbound from FEAT-10 or FEAT-30) | Find every Requested entry whose service/date-range matches the opening | Standalone Automation | FEAT-20.SPEC-005 |
| A match is found | Transition the matching entries to Notified and hand off to the opening notification | Standalone Automation | FEAT-20.SPEC-005 |
| Matching entries are notified | Send the opening notification with the 30-minute claim window and claim link | Standalone Notification | FEAT-20.SPEC-008 |
| Client taps the claim link | Route into FEAT-05's ordinary booking flow for the opened slot | Cross-feature -- logged in touchpoints | FEAT-05 responsibility |
| A notified client completes the booking for the matching slot | Convert that client's entry to Converted | Standalone Automation | FEAT-20.SPEC-006 |
| A second notified client tries to claim the same slot after the first already booked | Show the plain "just taken" message (XBR-01) and leave that client's entry Requested for the next opportunity | Cross-feature -- logged in touchpoints | FEAT-03 responsibility (XBR-01), reflected in FEAT-20.SPEC-004 |
| A Notified entry's 30-minute claim window lapses unclaimed | Transition the entry to Expired; the slot returns to ordinary public availability | Standalone Automation | FEAT-20.SPEC-007 |
| A Requested entry's joined date range elapses with no matching opening ever found | Transition the entry to Expired | Standalone Automation | FEAT-20.SPEC-007 |
| An entry transitions to Expired (either path) | Send the expiry notification so the client is not left wondering | Standalone Notification | FEAT-20.SPEC-009 |
| Client leaves a waitlist entry that has a pending (Notified) claim | Delete the entry immediately; the leave wins over the pending notification and the slot proceeds as if that client were never notified | Standalone Logic/Rule (contention) + inline delete | FEAT-20.SPEC-004 governs; FEAT-20.SPEC-002 executes |
| Client leaves a waitlist entry with no pending claim | Delete the entry immediately | Inline in triggering screen | FEAT-20.SPEC-002 |
| Entry transitions occur (joined, notified, converted, expired) | Emit the corresponding analytics signal | Inline in triggering spec | FEAT-20.SPEC-001 / SPEC-005 / SPEC-006 / SPEC-007 |
| Waitlist activity changes the demand count for a Pro's day | Pro's Attention List reflects the current aggregate count | Cross-feature -- logged in touchpoints | FEAT-12 responsibility |
| Client attempts to join, leave, or claim while offline or connectivity drops | Plain message that connectivity is required; nothing is submitted (States field: Offline-degraded N/A -- requires connectivity) | Inline in triggering screen | FEAT-20.SPEC-001 / SPEC-002 |
| A failed join is submitted | Client can retry from the same form (States field: Error -- a failed join is retried) | Inline in triggering screen | FEAT-20.SPEC-001 |

## Shared Context

**Shared Entities:**
- Waitlist Entry -- created by SPEC-001, read/listed by SPEC-002, updated by SPEC-005/SPEC-006/SPEC-007, deleted by SPEC-002. Fields in scope here: service, date or range (up to 7 days), state (Requested | Notified | Converted | Expired), claim_deadline (30 minutes after notification).
- Service (read-only) -- read by SPEC-001 and SPEC-003 to validate the join target and display its name.
- Messaging Consent (read-only) -- read by SPEC-008 and SPEC-009 to choose the notification channel.

**Shared UI Patterns:**
- Position/status list pattern -- SPEC-002 presents each waitlist entry's status plainly (waiting, notified-and-claimable, converted, expired) alongside a leave action; the empty state ("you're not on any waitlists") follows the same plain, non-alarming tone the feature's other empty states use elsewhere in the product.
- Join form pattern -- SPEC-001 reuses the same service and date-picker conventions as the ordinary booking flow (FEAT-05) so a client who just saw "fully booked" recognizes the same controls immediately.

**Shared Validation:**
- FEAT-20.SPEC-003 (Waitlist Entry Validation & Limits) is referenced, not duplicated, by SPEC-001 at join time.
- FEAT-20.SPEC-004 (Waitlist Priority & Claim Window Rule) is referenced, not duplicated, by SPEC-005 (who gets notified and when), SPEC-006 (how a contested claim resolves), and SPEC-007 (when the window or the range has lapsed).

**Recorded readings (per the Requirements Architect's coordination notes -- decisions, not resolutions of ambiguity):**
- **Reschedule does not generate an opening.** XBR-28's wording is "a slot freed by a cancellation," and FEAT-10's own Side-Effect Inventory scopes the waitlist hand-off specifically to "Booking Update Commit succeeds (cancellation)," not to a reschedule's vacated original time. This feature therefore only receives and acts on cancellation-sourced freed-slot signals from FEAT-10.SPEC-004 and FEAT-30.SPEC-007/008; a reschedule's vacated time is not a waitlist opening. Recorded as a Non-Goal below.
- **How the 30-minute window interacts with public availability.** XBR-28 states the slot "becomes publicly bookable immediately" and that "matching waitlisted clients are notified first and have a 30-minute priority window before it returns to general availability"; the journey's Failure/Recovery Variant says it then "returns to ordinary public availability." This Brief reads the two together as: the underlying slot truth (FEAT-03) treats the slot as free immediately (no hold blocks it from ever being booked), but for the 30-minute window the only path to it is the claim link sent to matching waitlisted clients -- it does not yet appear on the general public slot list (FEAT-05's service page). Once the window lapses with no claim (or immediately, if no client matched), it appears on the general public slot list like any other open time. This reading governs SPEC-004 and is a decision record, not a change to XBR-28 or the journey text.
- **Opening notification and daytime-hours (ASMP-29).** ASMP-29 restricts *automatic reminders* to roughly 8am-9pm in the Pro's timezone. The opening notification is time-critical by design -- the entire 30-minute claim window depends on immediate delivery -- so this Brief treats it as outside ASMP-29's reminder scope and sends it the moment the match occurs, at any hour, same as a payment confirmation. The expiry notification is not time-critical (the window has already closed) and follows ASMP-29's daytime-hours rule.

## Internal Dependency Map

```
SPEC-001 (Join Waitlist) -> [validates against] -> SPEC-003 (Waitlist Entry Validation & Limits)
SPEC-001 (Join Waitlist) -> [client submits a valid request] -> Waitlist Entry created (Requested) -> [appears on] -> SPEC-002 (My Waitlists)
SPEC-002 (My Waitlists) -> [client taps Leave] -> [checked against] -> SPEC-004 (Waitlist Priority & Claim Window Rule) -> [entry deleted]
SPEC-005 (Cancellation-Triggered Waitlist Matching) -> [matches entries using] -> SPEC-004 (Waitlist Priority & Claim Window Rule)
SPEC-005 (Cancellation-Triggered Waitlist Matching) -> [entry transitions to Notified] -> SPEC-008 (Waitlist Opening Notification)
SPEC-008 (Waitlist Opening Notification) -> [client taps claim link] -> FEAT-05 booking flow -> [booking completes] -> SPEC-006 (Waitlist Claim Conversion)
SPEC-006 (Waitlist Claim Conversion) -> [resolves contested claims using] -> SPEC-004 (Waitlist Priority & Claim Window Rule)
SPEC-007 (Waitlist Entry Expiry) -> [governed by] -> SPEC-004 (Waitlist Priority & Claim Window Rule)
SPEC-007 (Waitlist Entry Expiry) -> [entry transitions to Expired] -> SPEC-009 (Waitlist Expiry Notification)
SPEC-006 (Waitlist Claim Conversion) / SPEC-007 (Waitlist Entry Expiry) -> [entry leaves active state] -> SPEC-002 (My Waitlists) [list reflects the new status]
```

**Default Entry:** This feature has no single default landing screen -- SPEC-001 (Join Waitlist) is reached only from FEAT-05's fully booked service outbound link, and SPEC-002 (My Waitlists) is reached only from FEAT-06's My Bookings outbound link; a client never navigates to this feature area directly.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-20.SPEC-001 | Inbound | FEAT-05 (Public Booking Page & Booking Flow) | Client taps the outbound join-waitlist path on a fully booked service | Client sees no free slot for their chosen service/day |
| FEAT-20.SPEC-002 | Inbound | FEAT-06 (Client Booking Identity) | Client taps the outbound leave-waitlist action from My Bookings | Client opens My Bookings |
| FEAT-20.SPEC-006 | Outbound | FEAT-05 (Public Booking Page & Booking Flow) | Claim link routes the client into the ordinary booking flow for the opened slot, including policy acknowledgment and deposit | Client taps the claim link on the opening notification |
| FEAT-20.SPEC-005 | Inbound | FEAT-10 (Client-Initiated Cancel/Reschedule) | A client cancellation frees a slot and hands off the opening signal (cancellation only, not reschedule -- see Shared Context recorded reading) | FEAT-10.SPEC-004 Booking Update Commit succeeds (cancellation) |
| FEAT-20.SPEC-005 | Inbound | FEAT-30 (Pro Booking Management) | A Pro single or bulk cancellation frees a slot and hands off the opening signal | FEAT-30.SPEC-007 / SPEC-008 |
| FEAT-20.SPEC-004 / SPEC-005 / SPEC-006 | Outbound | FEAT-03 (Real-Time Slot Availability Engine) | Relies on FEAT-03's slot truth for what genuinely matches, and on FEAT-03.SPEC-005 for first-to-pay-wins resolution of contested claims (XBR-01) | A matching slot opens / a claim is contested |
| FEAT-20.SPEC-001 / SPEC-006 | Outbound | FEAT-02 (Availability & Working Hours Setup) | A joined date range and a claim must both respect minimum booking notice and booking horizon (XBR-03) | Client joins a waitlist or claims an opening |
| FEAT-20.SPEC-008 / SPEC-009 | Outbound | FEAT-08 (Automated Booking Messaging) | Both notifications are delivered through the shared transactional text/email transports (FEAT-08.SPEC-012, FEAT-08.SPEC-013), not a spec of this feature's own | Opening or expiry notification fires |
| FEAT-20.SPEC-008 / SPEC-009 | Outbound | FEAT-14 (Messaging Consent Management) | Channel choice (text vs. email) follows the client's current textability determination (XBR-15, FEAT-14.SPEC-007) | Before either notification is sent |
| FEAT-20.SPEC-004 | Outbound | FEAT-12 (Pro Daily Schedule Dashboard) | Supplies the aggregate waitlist-demand count consumed by FEAT-12.SPEC-005's Attention Flag Aggregation | Pro views their Attention List |
| FEAT-20 (feature-wide) | Inbound | FEAT-19 (Platform Support Read-Only Access) | Support's view-only access to Waitlist Entry state is surfaced through FEAT-19's own screen, never through this feature's screens | Support opens a Pro's account after a help request |
| FEAT-20.SPEC-005 / SPEC-006 / SPEC-007 | Outbound | FEAT-16 (Booking & Payment Activity Record) | If waitlist events are recorded on a booking's timeline, they route through FEAT-16.SPEC-002 rather than a separate event store this feature would own | Entry is notified, converted, or expires |

## Non-Functional Notes

**Data volumes / growth:** Bounded and small by design -- each client may hold at most 3 active entries per Pro, and each Pro serves roughly 100-500 clients (scope-boundaries SC-19), so per-Pro waitlist volume stays in the low hundreds at most even at full adoption. This is why the States field marks Loading as N/A ("small dataset").

**Responsiveness:** The opening notification must go out essentially the moment a match is found -- its entire value depends on the client having the full 30-minute claim window (XBR-02) to act, consistent with ASMP-21's product-wide correctness-and-speed bar. Joining, leaving, and viewing the My Waitlists list follow the same roughly-one-second responsiveness expectation as any other client-facing screen (ASMP-21).

**Data sensitivity / privacy:** A Waitlist Entry is personal data -- it reveals a client's desired appointment times (dependency map, Data Sensitivity). It is Own-only for the Client; the Pro sees aggregate counts only, never which specific clients are waitlisted (Access Matrix); Platform Operator (Support) has view-only access, surfaced through FEAT-19.

**Compliance flags:** ASMP-24 (US SMS-consent rules) governs both notifications -- text only with active consent, otherwise email, per XBR-15. ASMP-29's daytime-hours rule is read as not applying to the time-critical opening notification (recorded reading above) but does apply to the expiry notification, which carries no time pressure.

**Signals:** waitlist_joined (SPEC-001, on successful join), waitlist_notified (SPEC-005, on Requested-to-Notified transition), waitlist_converted_to_booking (SPEC-006, on successful claim conversion), waitlist_expired (SPEC-007, on either expiry path) -- these four Stage 2 signals are fully covered and require no additional instrumentation beyond what each spec's transition already emits.

## Non-Goals

- **Treating a rescheduled booking's vacated original time as a waitlist opening** -- Excluded per the recorded reading of XBR-28's literal "freed by a cancellation" wording and FEAT-10's own Side-Effect Inventory, which scopes its waitlist hand-off specifically to cancellation, not reschedule; this feature only acts on cancellation-sourced freed-slot signals.
- **A second Pro-facing screen for waitlist demand** -- Excluded per the Requirements Architect's coordination note: the Pro's View access (Access Matrix) is satisfied by supplying an aggregate count into FEAT-12's existing Attention List; this feature builds no dashboard of its own.
- **Sequential, turn-based waitlist claiming** -- Excluded per the feature's own Alternate flow, which describes simultaneous notification of every matching entry with first-to-act winning (not a queued, one-at-a-time turn system); introducing a turn order would be an invented behavior beyond what Stage 2 defines.
- **Automatic purge of Converted or Expired waitlist entries** -- Intentional lifecycle decision surfaced by the CRUD matrix: the dependency map names deletion only for the "client leaves" path, so terminal entries are retained indefinitely as a historical record; volume stays bounded by the 3-active-entries-per-Pro cap, so this carries no unbounded-growth risk.
- **Waitlist notifications treated as promotional messaging** -- Excluded per scope-boundaries SC-15: both notifications are transactional (booking-availability related), sent under the same booking-specific texting consent as any other product text, never a marketing or promotional campaign.

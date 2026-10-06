# FEAT-20 — Waitlist for Cancelled Slots

This chapter covers Waitlist for Cancelled Slots (FEAT-20), a Nice-to-Have-tier feature. It carries 9 specifications carrying 130 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-20.SPEC-001 | Join Waitlist | screen | 17 |
| FEAT-20.SPEC-002 | My Waitlists | screen | 15 |
| FEAT-20.SPEC-003 | Waitlist Entry Validation & Limits | logic-rule | 16 |
| FEAT-20.SPEC-004 | Waitlist Priority & Claim Window Rule | logic-rule | 15 |
| FEAT-20.SPEC-005 | Cancellation-Triggered Waitlist Matching | automation | 14 |
| FEAT-20.SPEC-006 | Waitlist Claim Conversion | automation | 13 |
| FEAT-20.SPEC-007 | Waitlist Entry Expiry | automation | 13 |
| FEAT-20.SPEC-008 | Waitlist Opening Notification | notification | 14 |
| FEAT-20.SPEC-009 | Waitlist Expiry Notification | notification | 13 |

The feature breakdown brief follows, then every specification in full.


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



# Screen Spec: Join Waitlist

## Overview

**Name:** Join Waitlist
**ID:** FEAT-20.SPEC-001
**Type:** Screen
**Purpose:** Riley joins the waitlist for a specific service and a day (or up to a 7-day range) when the public booking page shows no free time, so she is notified the moment a cancellation opens a matching slot.
**Parent Feature:** FEAT-20 -- Waitlist for Cancelled Slots

## Scope and Non-Goals

**In Scope:**
- Capturing the requested service (carried in from the fully booked service page) and the requested date or date range
- Capturing the identity fields (name, phone, texting opt-in, optional email) needed to create or match Riley's Client record for this Pro, since this screen is reached before any booking has ever put her on this Pro's Client list
- Submitting the join request for validation and creating the Waitlist Entry on success
- The plain reasons shown when a join request is invalid

**Non-Goals:**
- Defining the validation rules themselves (date-range shape, the 3-active-entries-per-Pro cap, notice/horizon bounds) -- owned by FEAT-20.SPEC-003; this screen only submits to that rule and displays its stated reasons
- Viewing or leaving an existing waitlist entry -- owned by FEAT-20.SPEC-002 (My Waitlists), reached separately through FEAT-06's My Bookings, per this Brief's Default Entry note that a client never navigates to this feature area directly
- Selecting a different service to check for a free slot -- that is FEAT-05.SPEC-001/SPEC-002's own service and slot browsing, which this screen is reached from, not re-implemented here

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-05.SPEC-002 (Slot Selection) | Riley taps "Join the waitlist" from the Empty (fully booked) state | The chosen Service reference (name, ID); no date pre-filled |

## Access and Visibility

single-role product screen for this feature (only the Client acts here); the roles below are the product's full closed role set, applied to this one open, unauthenticated screen.

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Full screen | Full -- submit a join request | -- |
| The Pro (Talia) | Full screen, exactly as any visitor sees it -- this screen carries no Pro-specific view or elevated content | Same as any visitor; the product defines no reason for the Pro to use this path, since her own View access to waitlist demand is FEAT-12's aggregate count, not this screen | -- (no restriction; nothing here is hidden from or unlocked for the Pro) |
| Platform Operator (Support) | Full screen, exactly as any visitor sees it | Same as any visitor; Support's own View-only access to Waitlist Entry state is exercised through FEAT-19, never through this public screen | -- |
| Unauthenticated | Full screen | Full -- join action requires no sign-in, matching BRIEF.md's no-signup-wall stance carried from FEAT-05 | -- |
| Expired session | N/A | N/A | N/A -- this is a stateless, unauthenticated public screen with no session concept to expire; each visit is independent |

## Layout and Content

**Header:** Screen title "Join the waitlist" with a back arrow (returns to FEAT-05.SPEC-002's slot list for the same service) and the service name shown below the title, non-editable (e.g., "for Full Set -- Lashes").

**Body:** A single-column form:
- **Date section** -- a mode toggle "One day" / "A range of days" (defaults to "One day"); when "One day" is selected, a single date picker; when "A range of days" is selected, a start-date and end-date picker pair, spanning at most platform parameter: `waitlist-join-range-max-days`. Both pickers reuse the same date-picker convention as FEAT-05's slot browsing so the control is immediately familiar.
- **Contact section** -- Name (text input, required), Phone (text input, required), a texting opt-in checkbox (unchecked by default, matching FEAT-05.SPEC-003's own opt-in convention), and an Email field shown only when texting opt-in is unchecked (required in that case, optional otherwise).
- **Join button** -- full width, at the bottom of the form.

**Footer:** None -- Join is the form's own trailing element.

### Responsive Behavior

- **Compact breakpoint:** Single-column form as described, full width; the date-mode toggle stacks above its picker(s).
- **Medium size class and above:** Form remains single-column, capped at the same platform-wide form width FEAT-05's booking form uses, horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-05.SPEC-002 for the same service | Screen closes | Animated transition back to slot list |
| Date-mode toggle | Tap | Switches between single-date and range picker layout | Body re-renders with the chosen picker | Selected mode highlighted |
| Date picker(s) | Select | Captures the requested date or date range | Field shows chosen date(s) | Selected date(s) displayed |
| Name input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Phone input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Texting opt-in checkbox | Tap | Toggles opt-in; shows or hides the Email field | Email field appears/disappears | Checkbox state changes |
| Email input (when shown) | Type | Captures text input | Field shows entered text | Standard input focus state |
| Join button | Tap | 1. Validate all fields via FEAT-20.SPEC-003 (join-shape rules) and standard identity-field format checks (matching FEAT-05.SPEC-003's own field formats). 2. If valid, submit the join request, creating or matching Riley's Client record by phone for this Pro, and creating the Waitlist Entry in state Requested. | Button shows loading state during submit | Success: confirmation screen (see States). Failure: inline error per Validation Rules |
| Join button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> date-mode toggle -> date picker(s) -> Name -> Phone -> texting opt-in checkbox -> Email (when shown) -> Join.
- **Validation announcements:** When a field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Confirmation announcement:** The join-succeeded confirmation content is announced on arrival.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (default) | Form fields empty except the fixed service name; date mode defaults to "One day"; Join enabled | Screen first opens | Riley begins entering any field |
| Filling | Form fields contain entered input | Riley enters any field | Riley taps Join or navigates away |
| Submitting | Join button shows a loading spinner, form fields disabled | Riley taps Join with valid input | Submission completes (success or failure) |
| Validation Error | Failed fields show inline error messages per FEAT-20.SPEC-003's stated reasons | Submission is rejected by validation | Riley corrects the field(s) and resubmits |
| Confirmed | Form is replaced by a confirmation message ("You're on the waitlist for {service_name} on {date/date range}. We'll text/email you the moment a matching time opens.") with a link to request an access link (FEAT-06.SPEC-001) to manage this waitlist entry later | Submission succeeds | Riley taps the access-link request, or navigates away |
| Error | Error banner "We couldn't join the waitlist. Try again." with a Retry option; entered values preserved | Submission fails for a reason other than validation (e.g., a transient write failure) | Riley taps Retry or navigates away |
| Offline/Degraded | Banner "You're offline -- connect to join the waitlist." appears; the Join button is disabled; nothing is submitted or queued | Connectivity is lost while this screen is open | Connectivity is restored -- banner clears and Join re-enables |

## Validation Rules

**Option A -- Reference Logic/Rule spec:**
Date-range shape, the 3-active-entries-per-Pro cap, and notice/horizon bounds are governed by FEAT-20.SPEC-003 (Waitlist Entry Validation & Limits). See that spec for the exact conditions and error messages.

**Option B -- Inline (identity-field formats, matching FEAT-05.SPEC-003's own conventions since this feature does not duplicate FEAT-05's field rules):**

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| Name | Required, 1-100 characters | On blur | "Name is required." |
| Phone | Required, valid reachable format | On blur | "Enter a valid phone number." |
| Email | Required when texting opt-in is unchecked; otherwise optional | On submit | "Enter an email address, or opt in to texts." |

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-05.SPEC-002 (Slot Selection) | FEAT-05 (Public Booking Page & Booking Flow) |
| Confirmed state's access-link link | FEAT-06.SPEC-001 (Access Link Request) | FEAT-06 (Client Booking Identity) |

## Data Model

**Creates:** Client record -- matched by phone for this Pro per the Client entity's Contention resolution (phone-number match within a Pro resolves to a single record), or created fresh if no match exists, with name, phone, and email (when provided) set from form input; a new Waitlist Entry -- service (carried from entry context), date or date range (up to platform parameter: `waitlist-join-range-max-days`), state set to Requested.
**Reads:** Service -- name and Active status, for the fixed service header and for FEAT-20.SPEC-003's existence check.
**Updates:** None.
**Deletes:** None.

## Business Rules

- FEAT-20.SPEC-003 (Waitlist Entry Validation & Limits) governs every condition on the submitted join request -- the service/date-range shape, the 3-active-entries-per-Pro cap, and the notice/horizon bounds -- and this screen cannot be saved with a request that rule rejects.
- **Coordination note (Client entity creator list):** the Feature Dependency Map's Client entity lifecycle names FEAT-05 and FEAT-30 as its creators; this screen is a third creation path the map does not yet list, since a client can reach the waitlist before ever completing a booking with this Pro. This follows the identical pattern the dependency map already documents for the Contention resolution ("phone-number match within a Pro resolves to a single Client record"); it is flagged here as a carry-forward item for reconciliation to add FEAT-20.SPEC-001 to the Client entity's Creators list, rather than silently contradicting the map.
- A join request never creates a Booking or takes a payment -- joining is free and carries no obligation.

## Edge Cases

- **Riley reaches this screen with no service context (a malformed or direct link)** -- The screen cannot render its fixed service header; Riley is routed to FEAT-05.SPEC-001 (service list) with the plain message "Choose a service to join its waitlist."
- **Riley's phone matches an existing Client record for this Pro** -- The existing Client record is reused (name and email are not overwritten from this form if they differ; the existing record's identity stands per the Client entity's phone-match resolution).
- **Riley already holds 3 active entries with this Pro** -- Rejected per FEAT-20.SPEC-003 with its stated cap message; the form is preserved so she can adjust or leave an existing entry first (via FEAT-06 -> FEAT-20.SPEC-002).
- **Riley taps Join twice rapidly** -- Second tap is ignored while the first submission is in progress (button in loading state).
- **Network failure during submission** -- Error banner "We couldn't join the waitlist. Try again." with Retry; form data preserved.
- **Riley closes the page mid-fill** -- Nothing is submitted; no draft is preserved, since a waitlist join has no partial or resumable state.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-05.SPEC-002 (Slot Selection) | Navigation (inbound) | Riley arrives here from the fully booked empty state, carrying the Service reference |
| FEAT-20.SPEC-003 (Waitlist Entry Validation & Limits) | References (outbound) | Join-shape, cap, and notice/horizon validation |
| FEAT-06.SPEC-001 (Access Link Request) | Navigation (outbound) | The confirmed state offers a path to request an access link for later management |
| FEAT-20.SPEC-002 (My Waitlists) | References (outbound) | Where the created entry subsequently appears, once Riley requests an access link and opens My Bookings |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| waitlist_joined | service ID, date_mode (single/range), range length in days | Join succeeds and the Waitlist Entry is created | N/A -- no success-metrics.md metric names waitlist adoption directly; retained per the Brief's Non-Functional Notes, which names waitlist_joined as one of the four Stage 2 signals this feature's transitions must emit |
| waitlist_join_rejected | reason (cap_exceeded / invalid_range / notice_horizon) | Join is rejected by FEAT-20.SPEC-003 | N/A -- no success-metrics.md metric measures rejected join attempts; retained for operational visibility into how often the cap or range rules are hit |

## Acceptance Criteria

**FEAT-20.SPEC-001-AC-01:** Given Riley sees "No open times right now for this service" on FEAT-05.SPEC-002, when she taps "Join the waitlist", then she lands on this screen with the service name fixed at the top.

**FEAT-20.SPEC-001-AC-02:** Given Riley is on this screen, when she selects "One day" and picks a date, phone, name, and opts into texts, and taps Join, then the Waitlist Entry is created in state Requested and she sees the Confirmed state.

**FEAT-20.SPEC-001-AC-03:** Given Riley selects "A range of days" spanning platform parameter: `waitlist-join-range-max-days`, when she submits, then the join succeeds with that full range recorded.

**FEAT-20.SPEC-001-AC-04:** Given Riley selects a range longer than platform parameter: `waitlist-join-range-max-days`, when she taps Join, then FEAT-20.SPEC-003's stated range-shape error is shown and no entry is created.

**FEAT-20.SPEC-001-AC-05:** Given Riley already holds 3 active waitlist entries with this Pro, when she submits a fourth, then FEAT-20.SPEC-003's cap message is shown and no entry is created.

**FEAT-20.SPEC-001-AC-06:** Given Riley leaves the phone field empty and blurs it, then the field shows "Enter a valid phone number." and Join is blocked until corrected.

**FEAT-20.SPEC-001-AC-07:** Given Riley does not opt into texts and leaves email empty, when she taps Join, then she sees "Enter an email address, or opt in to texts." and the join does not proceed.

**FEAT-20.SPEC-001-AC-08:** Given Riley's phone number matches an existing Client record with this Pro, when she submits, then the existing Client record is reused rather than a duplicate created.

**FEAT-20.SPEC-001-AC-09:** Given Riley reaches this screen with no service context, then she is redirected to FEAT-05.SPEC-001 with the message "Choose a service to join its waitlist."

**FEAT-20.SPEC-001-AC-10:** Given Riley taps Join twice rapidly, then the second tap is ignored while the first submission is in progress.

**FEAT-20.SPEC-001-AC-11:** Given a network failure occurs during submission, then Riley sees "We couldn't join the waitlist. Try again." with a Retry option and her entered values preserved.

**FEAT-20.SPEC-001-AC-12:** Given Riley loses connectivity while filling the form, when she taps Join, then the offline banner appears, the Join button is disabled, and nothing is submitted.

**FEAT-20.SPEC-001-AC-13:** Given Riley reaches the Confirmed state, when she taps the access-link link, then she is taken to FEAT-06.SPEC-001 to request an access link for later management.

**FEAT-20.SPEC-001-AC-14:** Given Talia (the Pro) opens this screen's link, then she sees exactly the same screen any visitor would, with no Pro-specific content unlocked.

**FEAT-20.SPEC-001-AC-15:** Given a Support operator opens this screen's link, then it renders exactly as any visitor would see it -- Support's own account access is exercised only through FEAT-19.

**FEAT-20.SPEC-001-AC-16:** Given Riley closes the page partway through filling the form, when she returns via the same entry link, then the form is empty again -- no draft is preserved.

**FEAT-20.SPEC-001-AC-17:** Given Riley taps the back arrow, then she is returned to FEAT-05.SPEC-002 for the same service.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 9 | 9 |
| States | 7 (empty, filling, submitting, validation error, confirmed, error, offline) | 7 |
| Business Rules | 3 | 3 |
| Edge Cases | 6 | 6 |



# Screen Spec: My Waitlists

## Overview

**Name:** My Waitlists
**ID:** FEAT-20.SPEC-002
**Type:** Screen
**Purpose:** Riley views her own waitlist entries (position/status), sees the plain empty state when she holds none, and leaves any entry.
**Parent Feature:** FEAT-20 -- Waitlist for Cancelled Slots

## Scope and Non-Goals

**In Scope:**
- Listing every active Waitlist Entry belonging to Riley with this Pro, with its current status
- The plain "you're not on any waitlists" empty state
- Leaving (deleting) an entry, including the contention case where a pending Notified claim exists

**Non-Goals:**
- Joining a new waitlist -- owned by FEAT-20.SPEC-001, reached only from a fully booked service; this screen offers no "join" entry point of its own, since a client never navigates to this feature area directly (Brief, Default Entry)
- Establishing Riley's identity -- owned by FEAT-06 (Client Booking Identity); this screen is reached only after FEAT-06's My Bookings List has already matched Riley's phone to her Client record
- Showing which specific clients are waitlisted to the Pro -- excluded per the Access Matrix and this Brief's Non-Goals: the Pro sees only an aggregate count via FEAT-12, never individual entries or this screen

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-06.SPEC-003 (My Bookings List) | Riley's My Bookings List renders its Waitlist section (shown only when she has active entries) | Riley's matched Client identity for this Pro (from FEAT-06.SPEC-008) |
| FEAT-20.SPEC-009 (Waitlist Expiry Notification) | Client taps "View my waitlists" in a waitlist expiry notice | The client's access-link identity; list loads that client's own entries with this Pro |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Her own active waitlist entries with this one Pro only | Leave any of her own entries | -- |
| The Pro (Talia) | None -- this screen shows no aggregate or individual waitlist data to the Pro; her View access to waitlist demand is fulfilled by FEAT-12's aggregate count, not this screen | None | This screen is not reachable through any Pro-facing navigation; a Pro account has no path to it |
| Platform Operator (Support) | None on this screen -- Support's View-only access to Waitlist Entry state is surfaced through FEAT-19's own screen, never here | None | Support has no path to this screen; troubleshooting a waitlist entry goes through FEAT-19 |
| Unauthenticated | No | No | Cannot reach this screen without first redeeming a valid access link (FEAT-06.SPEC-002); an unauthenticated visitor is sent to FEAT-06.SPEC-001 to request one |
| Expired session | No | No | The access link this screen depends on has expired per FEAT-06.SPEC-007; Riley sees FEAT-06's "request a new link" prompt and any pending "Leave" action on this screen is not carried over |

## Layout and Content

**Header:** Screen title "My Waitlists," reached as a section within FEAT-06.SPEC-003's My Bookings List rather than a standalone top-level screen; a back element returns to My Bookings.

**Body:** A list of rows, one per active Waitlist Entry:
- Service name
- Requested date or date range
- Status label: "Waiting" (Requested), "A time opened -- claim it" (Notified, with the countdown described below), "Booked" (Converted, shown briefly before the entry rolls off this list per its Data Notes), or "Expired" (shown briefly before rolling off)
- A "Leave" action next to each Requested or Notified row (not shown for Converted or Expired rows, which are terminal and not leaveable)
- For a Notified row: a visible countdown to the claim deadline and a "Claim now" link

**Empty state content:** "You're not on any waitlists" in the same plain, non-alarming tone the product's other empty states use.

### Responsive Behavior

- **Compact breakpoint:** Rows stack vertically, full width, each with its status label directly below the service/date line and the Leave action right-aligned.
- **Medium size class and above:** Uniform scaling, no structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back element | Tap | Navigate to FEAT-06.SPEC-003 (My Bookings List) | Screen closes | Animated transition back |
| Waitlist row (Requested or Expired) | Tap | Display-only -- no detail view beyond what the row already shows | None | -- |
| "Leave" action | Tap | Opens a confirmation dialog "Leave the waitlist for {service}?" | Dialog appears | Dialog with "Leave" and "Cancel" |
| "Leave" confirmed | Tap | 1. Re-check via FEAT-20.SPEC-004 whether the entry has a pending (Notified) claim. 2. Delete the Waitlist Entry immediately regardless of pending-claim state, per FEAT-20.SPEC-004's rule that a leave wins over a pending notification. | Entry removed from the list | Row disappears; toast "You've left the waitlist for {service}." |
| "Leave" cancelled | Tap | Closes the dialog | Dialog closes | No change |
| "Claim now" link (Notified rows only) | Tap | Navigate into FEAT-05's booking flow for the opened slot | Screen closes | Routes to the same destination as the opening notification's claim link (FEAT-20.SPEC-008) |

### Accessibility Notes

- **Focus order:** Back element -> each waitlist row in list order -> each row's Leave (and Claim now, when present) action.
- **Dynamic announcements:** When a row is removed after a confirmed Leave, its removal is announced to assistive technology; the Notified countdown updates are not individually announced (a static remaining-time value is sufficient) to avoid interrupting screen-reader users repeatedly.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty | "You're not on any waitlists" message, no list | Riley has zero active entries with this Pro | An entry is created (via FEAT-20.SPEC-001) and the list is reloaded |
| Populated | List of active entries with status labels and actions | One or more active entries exist | Riley navigates away, or the list changes (leave, notify, convert, expire) |
| Leave confirming | Confirmation dialog open over the list | Riley taps Leave | Riley confirms or cancels |
| Error | Error banner "We couldn't load your waitlists. Try again." with Retry | List fails to load | Riley taps Retry |
| Offline/Degraded | The already-loaded list remains visible read-only; a banner "You're offline -- reconnect to leave a waitlist." appears; Leave and Claim now actions are disabled | Connectivity is lost while this screen is open | Connectivity is restored -- banner clears and actions re-enable |

## Validation Rules

Validation governed by FEAT-20.SPEC-004 (Waitlist Priority & Claim Window Rule) for the leave-vs-pending-claim contention. No field input exists on this screen beyond the Leave confirmation choice.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back element tap | FEAT-06.SPEC-003 (My Bookings List) | FEAT-06 (Client Booking Identity) |
| "Claim now" tap | FEAT-05 booking flow for the opened slot | FEAT-05 (Public Booking Page & Booking Flow) |

## Data Model

**Reads:** Waitlist Entry -- service, date or date range, state (Requested / Notified / Converted / Expired), claim_deadline (for the Notified countdown); scoped to the matched Client with this Pro (per FEAT-06.SPEC-008).
**Updates:** None directly by this screen beyond triggering the delete below.
**Deletes:** Waitlist Entry -- when Riley confirms Leave, the entry is deleted immediately (hard delete, matching the Entity-Lifecycle Coverage Matrix: no restore path, no cascade to any resulting Booking).

## Business Rules

- A leave request always wins over a pending (Notified) claim, per FEAT-20.SPEC-004's contention rule -- Riley is never blocked from leaving because a notification is in flight.
- Converted and Expired entries are retained per the Brief's Non-Goals (no automatic purge) but are not shown indefinitely on this screen -- see Edge Cases for the exact rolling-off behavior this spec defines.
- The Waitlist section on FEAT-06.SPEC-003 only appears at all when at least one active (Requested or Notified) entry exists; this screen's own Empty state is reached only if Riley navigates to it directly with zero entries (e.g., her last entry just left, converted, or expired).

## Edge Cases

- **A cancellation frees a matching slot while Riley has this screen open (entry transitions Requested -> Notified)** -- The row updates in place to the Notified status and countdown the next time the list refreshes; this screen is a snapshot, not live-updating, so the change may not appear until Riley reopens or refreshes it.
- **Riley taps Leave on an entry that has just become Notified (a pending claim exists)** -- The confirmation dialog and outcome are identical regardless of state; the leave is honored and the entry is deleted, per FEAT-20.SPEC-004.
- **Riley's claim window lapses (entry transitions Notified -> Expired) while this screen is open** -- The countdown reaching zero does not itself update the row; the row reflects Expired the next time the list reloads, and the Expired row itself rolls off the list on Riley's next visit after that (see below).
- **An entry converts (another notified client books first, or Riley's own claim succeeds) while this screen is open** -- If Riley's own claim succeeded, she is already mid-navigation into FEAT-05's booking flow and does not return to this exact state; if a different notified client's claim converted, Riley's own remaining entries are unaffected and continue to show their own status.
- **Expired or Converted rows persist beyond the visit in which they last changed** -- This screen shows a terminal (Expired or Converted) row for one visit after the transition so Riley sees the outcome, then it no longer appears in this list on the next load (the record itself is retained per the Brief's Non-Goals; only this screen's display rolls it off).
- **Riley leaves the same entry twice in rapid succession (double-tap on Leave confirmed)** -- The second confirmation is a no-op; the entry is already deleted after the first.
- **Riley's last active entry is left, converted, or expired while this screen is open** -- The list transitions to the Empty state on next reload.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-003 (My Bookings List) | Navigation (inbound) | Riley arrives here from the Waitlist section of My Bookings |
| FEAT-20.SPEC-001 (Join Waitlist) | References (outbound) | Where an entry shown here was originally created |
| FEAT-20.SPEC-004 (Waitlist Priority & Claim Window Rule) | References (outbound) | Governs the leave-vs-pending-claim contention outcome |
| FEAT-20.SPEC-005, FEAT-20.SPEC-006, FEAT-20.SPEC-007 | Affects (inbound) | These automations' state transitions (Notified, Converted, Expired) are what this screen's status labels reflect |
| FEAT-05 (Public Booking Page & Booking Flow) | Navigation (outbound) | "Claim now" routes into the booking flow for the opened slot |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| my_waitlists_viewed | active entry count | Screen loads with at least one active entry | N/A -- no success-metrics.md metric measures waitlist screen views; retained for operational visibility into how often clients check their status |
| waitlist_leave_confirmed | reason: pending_claim / no_pending_claim | Riley confirms Leave on an entry | N/A -- no success-metrics.md metric measures voluntary waitlist departures; retained so the leave path (Key Capability: "Leave a waitlist at any time") is observable, matching FEAT-06.SPEC-003's identical event for its own outbound Leave action |

## Acceptance Criteria

**FEAT-20.SPEC-002-AC-01:** Given Riley has no active waitlist entries with Talia, when she opens this screen, then she sees "You're not on any waitlists."

**FEAT-20.SPEC-002-AC-02:** Given Riley has one Requested entry, when the screen loads, then she sees the service, requested date, and status "Waiting," with a Leave action.

**FEAT-20.SPEC-002-AC-03:** Given Riley has one Notified entry, when the screen loads, then she sees "A time opened -- claim it," a countdown to the claim deadline, a "Claim now" link, and a Leave action.

**FEAT-20.SPEC-002-AC-04:** Given Riley taps "Claim now" on a Notified entry, then she is routed into FEAT-05's booking flow for the opened slot.

**FEAT-20.SPEC-002-AC-05:** Given Riley taps Leave on a Requested entry with no pending claim, when she confirms in the dialog, then the entry is deleted immediately and the row disappears with the toast "You've left the waitlist for {service}."

**FEAT-20.SPEC-002-AC-06:** Given Riley taps Leave on a Notified entry with a pending claim, when she confirms, then the leave still wins per FEAT-20.SPEC-004 -- the entry is deleted immediately regardless of the pending notification.

**FEAT-20.SPEC-002-AC-07:** Given Riley taps Leave and then Cancel in the confirmation dialog, then the dialog closes and the entry remains unchanged.

**FEAT-20.SPEC-002-AC-08:** Given an entry converted to a Booking on Riley's last visit, when she reopens this screen on a later visit, then that row no longer appears (the underlying record is retained, but this screen has rolled it off).

**FEAT-20.SPEC-002-AC-09:** Given the list fails to load, when the screen attempts to render, then Riley sees "We couldn't load your waitlists. Try again." with a Retry option.

**FEAT-20.SPEC-002-AC-10:** Given Riley loses connectivity while this screen is open with entries already loaded, then the list remains visible read-only, a banner appears, and Leave and Claim now are disabled.

**FEAT-20.SPEC-002-AC-11:** Given Riley's access link has expired, when she attempts to reach this screen, then she sees FEAT-06's "request a new link" prompt instead.

**FEAT-20.SPEC-002-AC-12:** Given Talia (the Pro) has no path in her own account to this screen, when she looks for individual waitlist entries, then she finds none -- only FEAT-12's aggregate count is available to her.

**FEAT-20.SPEC-002-AC-13:** Given Riley double-taps the Leave confirmation, then the second tap is a no-op since the entry is already deleted after the first.

**FEAT-20.SPEC-002-AC-14:** Given Riley's last active entry is left, when the confirmation completes, then this screen transitions to the Empty state on next reload.

**FEAT-20.SPEC-002-AC-15:** Given Riley taps the back element, then she returns to FEAT-06.SPEC-003 (My Bookings List).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 5 (empty, populated, leave confirming, error, offline) | 5 |
| Business Rules | 3 | 3 |
| Edge Cases | 7 | 7 |



# Logic/Rule Spec: Waitlist Entry Validation & Limits

## Overview

**Name:** Waitlist Entry Validation & Limits
**ID:** FEAT-20.SPEC-003
**Type:** Logic/Rule
**Purpose:** Governs what a valid waitlist join looks like -- the service/date-range shape, the 3-active-entries-per-Pro cap, and the notice/horizon bounds a joined date range must respect.
**Parent Feature:** FEAT-20 -- Waitlist for Cancelled Slots
**Governed Entity:** Waitlist Entry

## Scope and Non-Goals

**In Scope:**
- Field validation for every Waitlist Entry field set at creation
- The 3-active-entries-per-Pro cap (across Requested and Notified states)
- Minimum booking notice and booking horizon bounds applied to a joined date or date range (XBR-03)
- Authorization for who may create, read, and delete a Waitlist Entry

**Non-Goals:**
- The claim window, matching priority, and contention resolution once an entry is Notified -- owned by FEAT-20.SPEC-004 (Waitlist Priority & Claim Window Rule); this spec governs only what makes a join valid at creation time, not what happens after
- Identity-field format rules (name, phone, email) -- those are ordinary contact-detail formats matching FEAT-05.SPEC-003's own conventions, enforced inline on FEAT-20.SPEC-001 as noted there, not a waitlist-specific rule this spec owns
- Deciding which entries match a freed slot -- owned by FEAT-20.SPEC-005 (Cancellation-Triggered Waitlist Matching); this spec only governs whether an entry was valid to create, not how it is later matched

## Governed Entity

**Entity:** Waitlist Entry
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| service | reference | The Service this entry is waitlisted for |
| date_mode | enum (single day, range) | Whether the entry names one day or a range of up to platform parameter: `waitlist-join-range-max-days` |
| start_date | date | The requested day, or the first day of a requested range |
| end_date | date | Equal to start_date for a single day; the last day of a requested range otherwise |
| state | enum (Requested, Notified, Converted, Expired) | The entry's current lifecycle state |
| claim_deadline | date/time | 30 minutes after notification; set only once the entry is Notified (owned by FEAT-20.SPEC-004/SPEC-005, not by this spec) |

**Referenced (read-only):** Service -- to confirm the requested service exists and is Active; Availability Rule (via FEAT-03) -- to confirm the notice and horizon bounds current at join time.

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|--------------------|
| FEAT-20.SPEC-001 | Join Waitlist | On Join button submit -- the sole point this spec's rules are evaluated, since a Waitlist Entry is never edited after creation |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| service | Must reference an existing, Active Service belonging to this Pro | Always | On submit | "This service is no longer available. Choose another service." | Yes |
| date_mode | Must be one of "single day" or "range" | Always | On submit | No user-facing error -- the screen's toggle only offers these two values | No |
| start_date | Required; must not be before the earliest day permitted by the Pro's current minimum booking notice (XBR-03, via FEAT-03) | Always | On submit | "The earliest day you can join for is {earliest_permitted_date}." | Yes |
| end_date | Required; equal to start_date in single-day mode; in range mode, must be on or after start_date and at most platform parameter: `waitlist-join-range-max-days` minus one day after it | Always | On submit | "A waitlist range can span at most {waitlist-join-range-max-days} days." | Yes |
| end_date | Must not be after the latest day permitted by the Pro's current booking horizon (XBR-03, via FEAT-03) | Always | On submit | "The latest day you can join for is {latest_permitted_date}, based on how far ahead this Pro takes bookings." | Yes |
| state | No validation beyond data type -- always set to Requested on creation by this spec; every later value is set exclusively by FEAT-20.SPEC-005/SPEC-006/SPEC-007 | Always | -- | -- | -- |
| claim_deadline | No validation beyond data type at creation -- always empty at creation; set only by FEAT-20.SPEC-005 when the entry transitions to Notified | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Date-range shape | date_mode, start_date, end_date | If date_mode is "single day," end_date must equal start_date; if "range," end_date must be on or after start_date and the span must not exceed platform parameter: `waitlist-join-range-max-days` | "A waitlist range can span at most {waitlist-join-range-max-days} days." |
| Active-entries cap | service (via the owning Client-Pro relationship), state | The requesting Client may not hold more than platform parameter: `waitlist-max-active-entries-per-pro` entries in state Requested or Notified with this Pro at the moment of a new join | "You're already on {waitlist-max-active-entries-per-pro} waitlists with this Pro. Leave one before joining another." |
| Notice/horizon bounds | start_date, end_date | The entire requested span (start_date through end_date) must fall within the window the Pro's current minimum booking notice and booking horizon permit (XBR-03), evaluated at the moment of join, not re-evaluated later as those settings may change | "This date range falls outside the times this Pro currently takes bookings." |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create a Waitlist Entry | The Client (Riley) | Always, subject to the field and cross-field rules above | The specific rule's error message from the tables above |
| Create a Waitlist Entry | The Pro (Talia) | Never -- the product defines no Pro-initiated waitlist join | No control exists for the Pro to create an entry on a client's behalf; this action is not offered anywhere in the Pro's account |
| Create a Waitlist Entry | Platform Operator (Support) | Never | No control exists for Support to create an entry |
| Read own Waitlist Entry (state, position/status) | The Client (Riley) | Only entries she herself created (Own-only, per the Access Matrix) | She never sees another client's entry |
| Read aggregate waitlist demand for a day | The Pro (Talia) | Always, as a count only, via FEAT-12 -- never individual entries or client identities | -- |
| Read individual Waitlist Entry state | Platform Operator (Support) | View-only, for troubleshooting a specific Pro's reported issue, via FEAT-19 only | -- |
| Delete (leave) a Waitlist Entry | The Client (Riley) | Only entries she herself created, in state Requested or Notified | -- (a Converted or Expired entry offers no Leave action per FEAT-20.SPEC-002, since it is already terminal) |
| Delete (leave) a Waitlist Entry | The Pro (Talia) | Never -- the Pro cannot remove a client's waitlist entry on her own initiative | No control exists for the Pro to remove an entry |
| Update any field after creation | Any role | Never -- per the Entity-Lifecycle Coverage Matrix, no field is ever edited after creation; only the state transitions FEAT-20.SPEC-005/SPEC-006/SPEC-007 own occur | No edit control exists anywhere in the product for a created entry's service or date range |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| state | Set to Requested | On creation, once all validation passes | No |
| service | Carried unmodified from the fully booked service page's context (FEAT-05.SPEC-002 -> FEAT-20.SPEC-001) | On creation | No -- the client cannot pick a different service on the Join Waitlist screen itself |
| claim_deadline | Left unset | On creation | No -- set only by FEAT-20.SPEC-005 when the entry is later Notified |

## Business Rules

- XBR-03 governs the notice/horizon bounds entirely: the Pro's current minimum booking notice and booking horizon (owned by FEAT-02) bound every client-facing booking path, including this one; a Pro-side change to those settings after an entry was created does not retroactively invalidate it (the entry's date range was valid when joined; only new joins are checked against current settings).
- The 3-active-entries-per-Pro cap (platform parameter: `waitlist-max-active-entries-per-pro`) counts only Requested and Notified entries -- Converted and Expired entries never count against it, and a client who leaves an entry immediately frees a slot in the cap for a new join.
- A join request is evaluated once, atomically, at submission -- there is no partial or draft Waitlist Entry state; either every rule passes and the entry is created in Requested, or none of it is created.
- This spec's rules apply identically regardless of date_mode -- a single-day join and a range join are evaluated by the same cross-field logic, with a single day simply being the degenerate one-day case of a range.

## Edge Cases

- **A requested range's start_date is exactly at the minimum-notice boundary** -- Passes; the boundary day itself is permitted, consistent with FEAT-02's own boundary-inclusive convention for notice and horizon.
- **A requested range's end_date is exactly platform parameter: `waitlist-join-range-max-days` minus one day after start_date** -- Passes, since this is the maximum permitted span, not one day beyond it.
- **A requested range's end_date is exactly platform parameter: `waitlist-join-range-max-days` after start_date** -- Rejected with the range-shape error, since this spans one day more than the maximum permitted.
- **The Client holds exactly platform parameter: `waitlist-max-active-entries-per-pro` minus one active entries and submits one more** -- Passes; this is the boundary case that reaches, but does not exceed, the cap.
- **The Client holds exactly the cap and submits one more** -- Rejected with the cap message.
- **The Pro's booking horizon shortens between when Riley started filling the form and when she submits** -- The submitted range is checked against the Pro's current horizon at the moment of submit, not at the moment the form was opened; a range that was valid when she started but is no longer valid at submit is rejected with the notice/horizon message, and she can adjust her requested dates and resubmit.
- **The requested Service is archived between when Riley arrived on the Join Waitlist screen and when she submits** -- Rejected with "This service is no longer available. Choose another service." per the service-existence rule; she is routed back to FEAT-05.SPEC-001's current service list.
- **A single-day join where start_date and end_date are somehow submitted as different values (a malformed submission)** -- Rejected by the date-range-shape rule, since single-day mode requires them to be equal; this is treated identically to any other range-shape violation.

## Acceptance Criteria

**FEAT-20.SPEC-003-AC-01:** Given Riley submits a join for an Active service with a valid single-day date within notice and horizon, when validation runs, then the entry is created in state Requested.

**FEAT-20.SPEC-003-AC-02:** Given Riley submits a join for a service that is not Active, when validation runs, then she sees "This service is no longer available. Choose another service." and no entry is created.

**FEAT-20.SPEC-003-AC-03:** Given Riley submits a range spanning exactly platform parameter: `waitlist-join-range-max-days` minus one day, when validation runs, then it passes as the maximum permitted span.

**FEAT-20.SPEC-003-AC-04:** Given Riley submits a range spanning platform parameter: `waitlist-join-range-max-days`, when validation runs, then she sees "A waitlist range can span at most {waitlist-join-range-max-days} days." and no entry is created.

**FEAT-20.SPEC-003-AC-05:** Given Riley's requested start_date falls before the Pro's current minimum booking notice, when validation runs, then she sees the earliest-permitted-date message and no entry is created.

**FEAT-20.SPEC-003-AC-06:** Given Riley's requested end_date falls beyond the Pro's current booking horizon, when validation runs, then she sees the latest-permitted-date message and no entry is created.

**FEAT-20.SPEC-003-AC-07:** Given Riley already holds platform parameter: `waitlist-max-active-entries-per-pro` minus one active entries with this Pro, when she submits one more valid join, then it succeeds, reaching the cap.

**FEAT-20.SPEC-003-AC-08:** Given Riley already holds platform parameter: `waitlist-max-active-entries-per-pro` active entries with this Pro, when she submits another, then she sees the cap message and no entry is created.

**FEAT-20.SPEC-003-AC-09:** Given Riley leaves one of her active entries and then submits a new join while still holding the cap minus one, then the new join succeeds, since the leave freed a slot in the cap.

**FEAT-20.SPEC-003-AC-10:** Given Talia (the Pro) looks for a way to create a waitlist entry on a client's behalf, then no such control exists anywhere in her account.

**FEAT-20.SPEC-003-AC-11:** Given Talia views her Attention List, when she looks at waitlist demand for a day, then she sees only an aggregate count, never individual client identities or entries.

**FEAT-20.SPEC-003-AC-12:** Given Support opens FEAT-19's read-only view of a Pro's account, when they inspect waitlist activity, then they see entry state but never a control to edit or delete an entry.

**FEAT-20.SPEC-003-AC-13:** Given Riley's requested Service is archived between screen load and submit, when she submits, then she sees "This service is no longer available. Choose another service." and no entry is created.

**FEAT-20.SPEC-003-AC-14:** Given the Pro's booking horizon shortens between Riley starting the form and submitting it, when she submits a range that was valid at load but is no longer valid at submit, then it is rejected against the current, not the original, horizon.

**FEAT-20.SPEC-003-AC-15:** Given a single-day submission is malformed with unequal start_date and end_date, when validation runs, then it is rejected by the date-range-shape rule.

**FEAT-20.SPEC-003-AC-16:** Given Riley attempts to edit an existing entry's service or date range after creation, when she looks for a way to do so, then no edit control exists anywhere in the product.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 6 | 6 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 8 | 8 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 8 | 8 |



# Logic/Rule Spec: Waitlist Priority & Claim Window Rule

## Overview

**Name:** Waitlist Priority & Claim Window Rule
**ID:** FEAT-20.SPEC-004
**Type:** Logic/Rule
**Purpose:** Governs which Requested entries match a freed slot, the 30-minute claim window, how the window interacts with general public availability, and how contested or withdrawn claims resolve.
**Parent Feature:** FEAT-20 -- Waitlist for Cancelled Slots
**Governed Entity:** Waitlist Entry -- specifically its matching, notification-window, and contention behavior once a slot frees

## Scope and Non-Goals

**In Scope:**
- The matching test that decides whether a Requested entry corresponds to a freed slot
- The 30-minute claim window's exact boundaries and what happens at each edge
- How the claim window interacts with the slot's general public availability (the recorded reading in this Brief's Shared Context)
- How a leave request resolves against a pending Notified claim (the contention rule this spec is Enforced By FEAT-20.SPEC-002 for)
- How a contested claim (two or more Notified entries for the same freed slot) resolves

**Non-Goals:**
- Detecting that a cancellation has freed a slot in the first place -- owned by FEAT-20.SPEC-005 (Cancellation-Triggered Waitlist Matching); this spec supplies the matching test and window rule that automation applies, not the detection of the triggering cancellation itself
- Converting a claimed slot into a Booking -- owned by FEAT-20.SPEC-006 (Waitlist Claim Conversion); this spec governs only which entry is eligible to claim and for how long, not the booking-completion mechanics
- The underlying free/not-free slot truth and the generic first-to-pay-wins tie-break mechanism for any booking path -- owned by FEAT-03.SPEC-005 (Slot Contention Resolution Rules, XBR-01); this spec builds the waitlist-specific priority window on top of that mechanism, per FEAT-03.SPEC-005's own stated Non-Goal excluding the waitlist window from its scope

## Governed Entity

**Entity:** Waitlist Entry, plus its contention against the freed slot's general availability
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|--------------|
| service | reference | The Service this entry is waitlisted for |
| start_date / end_date | date | The requested day or range this entry matches against |
| state | enum (Requested, Notified, Converted, Expired) | The entry's current lifecycle state |
| claim_deadline | date/time | Set to the moment of notification plus platform parameter: `waitlist-claim-window-minutes`, once Notified |

**Referenced (read-only):** Booking (freed slot's service, date, and time), via FEAT-03's live slot truth.

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|--------------------|
| FEAT-20.SPEC-005 | Cancellation-Triggered Waitlist Matching | At the moment a freed-slot signal arrives -- applies the matching test to find every eligible Requested entry |
| FEAT-20.SPEC-002 | My Waitlists | At the moment Riley confirms Leave on an entry that may have a pending Notified claim |
| FEAT-20.SPEC-006 | Waitlist Claim Conversion | At the moment a notified client completes a booking for the matching slot -- resolves any contested claim |
| FEAT-20.SPEC-007 | Waitlist Entry Expiry | At the moment a Notified entry's claim_deadline passes unclaimed |
| FEAT-03.SPEC-005 | Slot Contention Resolution Rules | Supplies the underlying free/not-free slot truth this spec's window rule builds on |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| service | No validation beyond data type -- already validated at creation by FEAT-20.SPEC-003; this spec only reads it for matching | Always | -- | -- | -- |
| start_date / end_date | No validation beyond data type -- already validated at creation; this spec only reads them for matching | Always | -- | -- | -- |
| state | Must be Requested for an entry to be eligible for matching by FEAT-20.SPEC-005 | Always | At the moment a freed-slot signal is evaluated | N/A -- read-only evaluation, no user-facing error; a non-Requested entry is simply excluded from the match set | No |
| claim_deadline | Must be unset (entry not yet Notified) for a match to set it; once set, it is never recomputed or extended | Always | At the moment of notification | N/A -- read-only evaluation | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Matching test | service, start_date, end_date, state | An entry matches a freed slot when: state is Requested, the freed slot's service equals the entry's service, and the freed slot's date falls within [start_date, end_date] inclusive | N/A -- a computed boolean, not a user-facing error |
| Claim window boundary | claim_deadline | A Notified entry remains claimable strictly before claim_deadline; at or after claim_deadline, the claim window has lapsed and the entry is no longer eligible to claim (governed for expiry by FEAT-20.SPEC-007) | N/A -- expressed to the client as the countdown on FEAT-20.SPEC-002 and the deadline stated in FEAT-20.SPEC-008 |
| Priority-before-public rule | claim_deadline | For the full platform parameter: `waitlist-claim-window-minutes` following notification, the freed slot does not appear on FEAT-05's general public slot list; it is reachable only through a matching client's claim link. Once the window lapses (or immediately, if zero entries matched), the slot appears on the general public list like any other open time -- this is the recorded reading of XBR-28 in this Brief's Shared Context, distinguishing the underlying slot truth (always free the instant it is freed, per FEAT-03) from public *listing* (deferred for the window) | N/A -- this is a display/listing rule, not a validation failure |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Be matched and notified for a freed slot | The Client (Riley) | Only her own Requested entries that pass the matching test | -- |
| Claim a notified opening | The Client (Riley) | Only while her own entry's claim_deadline has not yet passed, and only for the exact matched slot | If the window has lapsed: FEAT-20.SPEC-007's expiry handling applies (no claim action remains available; see that spec) |
| Claim a notified opening after another notified client already booked it | The Client (Riley) | Never for that specific slot -- the slot is gone the instant the first booking completes | Riley sees FEAT-03's plain "just taken" message (XBR-01), and her own entry remains Requested for the next opportunity (per this spec's contention resolution below), not deleted or expired by this event |
| Leave (delete) an entry with a pending Notified claim | The Client (Riley) | Always -- a leave request wins over a pending notification, with no condition attached | -- (the leave always succeeds; there is no denial path for this action) |
| See who else is notified for the same opening | The Client (Riley) | Never | She is never shown whether or how many other clients were notified for the same slot |
| See individual notified clients for a freed slot | The Pro (Talia) | Never -- her View access is the aggregate demand count only (FEAT-12), never per-client notification state | -- |
| View a specific client's notification/claim state | Platform Operator (Support) | View-only, for troubleshooting, via FEAT-19 | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| claim_deadline | Derived: the exact moment of notification plus platform parameter: `waitlist-claim-window-minutes` | Set once, at the moment FEAT-20.SPEC-005 transitions the entry to Notified | No -- fixed and never extended, paused, or recalculated for any reason |
| match set | Derived: every Requested entry (across all clients waitlisted with this Pro) whose service and date satisfy the Matching test | Computed fresh each time a freed-slot signal arrives | No |

## Business Rules

- XBR-28 governs this spec entirely: a slot freed by a cancellation becomes publicly bookable immediately at the underlying slot-truth level (FEAT-03 never blocks it), while matching waitlisted clients are notified first and have platform parameter: `waitlist-claim-window-minutes` of priority before it returns to general public listing (FEAT-05).
- Notification is simultaneous, not sequential: every matching Requested entry is transitioned to Notified and notified at the same moment (FEAT-20.SPEC-005) -- there is no queue or turn order, per this Brief's Non-Goals ("Sequential, turn-based waitlist claiming" is explicitly excluded).
- First-to-complete-booking wins a contested claim (XBR-01, via FEAT-03.SPEC-005's underlying tie-break mechanism): when two or more Notified clients attempt to claim the same freed slot, whichever completes payment first wins it; every other notified client's entry remains Requested, unconverted and unexpired by this event, so they remain eligible for the next matching opening.
- A leave request unconditionally wins over a pending Notified claim (enforced by FEAT-20.SPEC-002): there is no scenario in which a leave is refused or delayed because a notification is in flight.
- The claim window is fixed at platform parameter: `waitlist-claim-window-minutes` for every entry, every Pro, and every slot -- it is never configured per Pro or extended for any individual claim.

## Edge Cases

- **Zero Requested entries match a freed slot** -- The slot appears on the general public list immediately, with no priority window elapsing, since there is no one to notify (per the Priority-before-public rule's "or immediately, if zero entries matched" clause).
- **Exactly one Requested entry matches** -- That entry is Notified alone; the window and its claim behave identically to the multi-match case, just with a single eligible claimant.
- **Multiple Requested entries match the same freed slot** -- All are transitioned to Notified simultaneously and all receive the opening notification (FEAT-20.SPEC-008) at the same moment; the first to complete a booking wins per the contention rule above.
- **A matching entry belongs to a client who already holds another Notified entry for a different freed slot at the same time** -- Each entry and its claim window are independent; claiming one has no effect on the other, and both remain separately actionable within their own windows.
- **A client's Notified entry's claim_deadline is reached at the exact instant she taps "Claim now"** -- Whichever event's timestamp is earlier governs: if the claim action reaches the booking flow before claim_deadline, it proceeds as an ordinary in-window claim; if claim_deadline has already passed, FEAT-20.SPEC-007's expiry handling applies and the claim link no longer completes a priority-window booking (the slot is then evaluated against general availability like any other attempt).
- **A client leaves an entry that has already converted (a rare race where Leave is tapped just as her own claim commits)** -- Not possible in practice: FEAT-20.SPEC-002 disables the Leave action the instant a claim conversion (FEAT-20.SPEC-006) commits for that entry, since a Converted entry is terminal and offers no Leave action.
- **The Pro cancels her own cancellation policy's window mid-flight while entries are Notified** -- Has no effect on this spec's rules: the claim window and matching logic depend only on the Waitlist Entry and the freed slot, never on the Cancellation Policy that produced the original cancellation.

## Acceptance Criteria

**FEAT-20.SPEC-004-AC-01:** Given a cancellation frees a slot for a service and date matching Riley's Requested entry, when FEAT-20.SPEC-005 evaluates the matching test, then Riley's entry is included in the match set.

**FEAT-20.SPEC-004-AC-02:** Given a freed slot's service does not match Riley's Requested entry's service, when the matching test runs, then her entry is excluded from the match set.

**FEAT-20.SPEC-004-AC-03:** Given a freed slot's date falls outside Riley's requested [start_date, end_date] range, when the matching test runs, then her entry is excluded.

**FEAT-20.SPEC-004-AC-04:** Given Riley's entry is matched and Notified, when claim_deadline is computed, then it is set to the notification moment plus platform parameter: `waitlist-claim-window-minutes`, and never recalculated afterward.

**FEAT-20.SPEC-004-AC-05:** Given no Requested entry matches a freed slot, when the matching test finds zero matches, then the slot appears on FEAT-05's general public list immediately with no priority window elapsing.

**FEAT-20.SPEC-004-AC-06:** Given one or more entries are Notified for a freed slot, when the claim window is in effect, then the slot does not appear on FEAT-05's general public list until the window lapses.

**FEAT-20.SPEC-004-AC-07:** Given two clients are both Notified for the same freed slot, when one completes a booking first, then that client's entry converts (FEAT-20.SPEC-006) and the other's entry remains Requested, unconverted and unexpired.

**FEAT-20.SPEC-004-AC-08:** Given a second notified client attempts to claim a slot after the first already booked it, then she sees the plain "just taken" message (XBR-01) and her entry remains Requested.

**FEAT-20.SPEC-004-AC-09:** Given Riley has a pending Notified claim, when she confirms Leave on FEAT-20.SPEC-002, then her entry is deleted immediately regardless of the pending notification.

**FEAT-20.SPEC-004-AC-10:** Given Talia (the Pro) views her Attention List, when she looks for which specific clients are notified for an opening, then she sees only the aggregate demand count, never individual notification state.

**FEAT-20.SPEC-004-AC-11:** Given Riley's claim window has already lapsed when she taps "Claim now," then FEAT-20.SPEC-007's expiry handling applies rather than a priority-window booking.

**FEAT-20.SPEC-004-AC-12:** Given Riley holds two separate Notified entries for two different freed slots at the same time, when she claims one, then the other's window and claimability are entirely unaffected.

**FEAT-20.SPEC-004-AC-13:** Given three Requested entries all match the same freed slot, when the matching runs, then all three are transitioned to Notified simultaneously with no queue or turn order among them.

**FEAT-20.SPEC-004-AC-14:** Given Support views a Pro's account via FEAT-19, when they inspect a specific client's notification/claim state, then they see it as View-only, with no action available.

**FEAT-20.SPEC-004-AC-15:** Given a Notified entry converts to a Booking, when Riley then looks for a Leave action on that entry in FEAT-20.SPEC-002, then none is offered, since the entry is already terminal.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 7 | 7 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |



# Automation Spec: Cancellation-Triggered Waitlist Matching

## Overview

**Name:** Cancellation-Triggered Waitlist Matching
**ID:** FEAT-20.SPEC-005
**Type:** Automation
**Purpose:** On a freed-slot signal from a cancellation, finds every matching Requested entry, transitions each to Notified, and hands off to the opening notification.
**Parent Feature:** FEAT-20 -- Waitlist for Cancelled Slots

## Scope and Non-Goals

**In Scope:**
- Receiving a cancellation-sourced freed-slot signal from FEAT-10 (client cancellation) or FEAT-30 (Pro cancellation)
- Applying the matching test (FEAT-20.SPEC-004) to find every eligible Requested entry
- Transitioning every matched entry to Notified and setting its claim_deadline
- Handing off to FEAT-20.SPEC-008 for the opening notification

**Non-Goals:**
- Treating a reschedule's vacated original time as a freed slot -- excluded per this Brief's recorded reading of XBR-28's "freed by a cancellation" wording and FEAT-10's own Side-Effect Inventory, which scopes its waitlist hand-off specifically to cancellation; this automation is never triggered by a reschedule
- Defining the matching test itself, the claim window, or contention resolution -- owned by FEAT-20.SPEC-004; this automation applies that spec's rules, it does not define them
- Sending the notification content -- owned by FEAT-20.SPEC-008; this automation only triggers it once entries are Notified

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A client cancellation commits | FEAT-10.SPEC-004 (Booking Update Commit) | Fires only for the Cancellation committed outcome (never a reschedule outcome, per this Brief's recorded reading) | Freed slot's service, date, and start time |
| A Pro single cancellation commits | FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) | Fires only for a cancel outcome (never the reschedule branch) | Freed slot's service, date, and start time |
| A Pro bulk cancellation commits | FEAT-30.SPEC-008 (Bulk Cancellation Commit) | Fires once per booking cancelled in the bulk action | Freed slot's service, date, and start time, per affected booking |

## Processing Logic

1. Receive the freed-slot signal (service, date, start time) from the triggering cancellation commit.
2. Apply the matching test (FEAT-20.SPEC-004): find every Waitlist Entry with this Pro in state Requested whose service equals the freed slot's service and whose [start_date, end_date] range includes the freed slot's date.
3. If zero entries match, take no further action -- the slot proceeds directly to general public availability with no priority window (FEAT-20.SPEC-004's zero-match rule).
4. If one or more entries match, transition each matched entry's state to Notified and set its claim_deadline to the current moment plus platform parameter: `waitlist-claim-window-minutes`, simultaneously for every matched entry.
5. Trigger FEAT-20.SPEC-008 (Waitlist Opening Notification) once per matched entry, carrying the freed slot's details and that entry's claim_deadline.
6. Report completion; no data is returned to the triggering cancellation commit beyond acknowledgment, since FEAT-10.SPEC-004 and FEAT-30.SPEC-007/SPEC-008 do not wait on this automation's outcome to complete their own commit.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| No match found | Zero Requested entries match the freed slot | None | None -- the slot simply becomes generally available per FEAT-03/FEAT-05's ordinary path | FEAT-03, FEAT-05 |
| One or more entries matched and notified | 1+ Requested entries match | Each matched entry: state -> Notified, claim_deadline set | Each matched client receives the opening notification (FEAT-20.SPEC-008) | FEAT-20.SPEC-008, FEAT-20.SPEC-002 (status updates on next load) |
| Matching runs but the notification hand-off fails for one or more matched entries | The state transition to Notified succeeds but FEAT-20.SPEC-008 cannot be triggered for one or more of the matched entries (a transient failure) | The affected entries remain Notified with claim_deadline already set | The affected client sees no notification arrive, but her entry still shows as Notified with a countdown on FEAT-20.SPEC-002 if she checks that screen directly; the claim link itself is only ever delivered by the notification, so a client who never receives it cannot claim within the window through this path | FEAT-20.SPEC-002, FEAT-20.SPEC-008 |
| Automation failure (the matching step itself cannot run) | A transient failure prevents evaluating the freed-slot signal at all | No entry is transitioned | No client is notified for this opening; the slot proceeds to general availability once FEAT-03's own timing allows it, exactly as if zero entries had matched -- this is a silent miss from the waitlist's perspective, not a blocking failure of the triggering cancellation | FEAT-03, FEAT-05 |

## Data Model

**Reads:** Waitlist Entry -- service, start_date, end_date, state, scoped to this Pro; Booking -- the freed slot's service, date, and start time from the triggering cancellation.
**Creates:** None.
**Updates:** Waitlist Entry -- state (Requested -> Notified) and claim_deadline, for every matched entry.
**Deletes:** None.

## Business Rules

- XBR-28 governs this automation's entire purpose: matching waitlisted clients are notified first, with platform parameter: `waitlist-claim-window-minutes` of priority before the slot returns to general availability.
- This automation is triggered only by a cancellation-sourced freed-slot signal (FEAT-10.SPEC-004's Cancellation committed outcome, or FEAT-30.SPEC-007/SPEC-008's cancel outcomes) -- never by a reschedule, per this Brief's recorded reading of XBR-28 and FEAT-10's own scoping.
- Notification is simultaneous across every matched entry -- there is no queue or sequential notification order (this Brief's Non-Goals).
- This automation's own responsibility ends once matched entries are transitioned to Notified and the notification is triggered; it does not itself decide who wins a subsequently contested claim (FEAT-20.SPEC-006 and FEAT-20.SPEC-004 own that).
- Platform Operator (Support) can view every transition this automation writes (Requested -> Notified, and the freed-slot signal that produced it) through FEAT-19's read-only account view, per XBR-24 -- Support never triggers, delays, or overrides a match.

## Edge Cases

- **A single Pro bulk cancellation frees several slots at once (FEAT-30.SPEC-008)** -- Each freed booking's slot is processed as its own independent freed-slot signal, with its own matching pass, its own set of Notified entries (if any), and its own claim windows; slots freed in the same bulk action never share a claim window or notification batch.
- **Two cancellations for the same service and overlapping dates commit at effectively the same time** -- Each freed-slot signal is processed independently; if both signals could match the same waitlisted entry (a range join covering both freed dates), that entry is Notified once per matching freed slot it actually satisfies, since a Requested entry can only be matched while it remains in Requested state -- the moment it is Notified for the first freed slot, it is no longer eligible to be Notified again for a second freed slot until it resolves (converts, expires, or the client leaves and rejoins).
- **A Requested entry's date range covers a freed slot, but the entry converts or expires between the freed-slot signal arriving and this automation's matching step actually running** -- Not possible in practice: an entry only transitions to Notified through this automation itself, so at the moment matching evaluates state, an entry still in Requested is genuinely eligible; a stale read is avoided by evaluating state fresh at trigger time.
- **The notification hand-off (step 5) fails for one matched entry among several** -- The other matched entries' notifications proceed independently; only the failed entry's client experiences the "Notified but never notified" gap described in the Outcome Definitions above.
- **Concurrent trigger firing (two different bookings, for two different services, are cancelled at effectively the same time)** -- Each freed-slot signal runs its own independent matching pass; neither is delayed by the other.
- **Trigger fires while a previous run is in flight for the same freed slot** -- Not possible in practice: a given Booking can only be cancelled once (its Contention resolution is reject-with-refresh, first committed transition wins), so exactly one freed-slot signal is ever produced per cancelled booking.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-10.SPEC-004 (Booking Update Commit) | Triggered by (inbound) | A client cancellation commit fires this automation |
| FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) | Triggered by (inbound) | A Pro single-cancellation commit fires this automation |
| FEAT-30.SPEC-008 (Bulk Cancellation Commit) | Triggered by (inbound) | Each cancelled booking in a Pro bulk cancellation fires this automation |
| FEAT-20.SPEC-004 (Waitlist Priority & Claim Window Rule) | References (outbound) | Supplies the matching test and claim-window computation this automation applies |
| FEAT-20.SPEC-008 (Waitlist Opening Notification) | Triggers (outbound) | Fires once per matched, Notified entry |
| FEAT-20.SPEC-002 (My Waitlists) | Affects (outbound) | The status change (Requested -> Notified) is what that screen's next load reflects |
| FEAT-03 (Real-Time Slot Availability Engine) | References (outbound) | Supplies the freed slot's live truth and governs when it joins general availability |

## Analytics and Success Signals

- **waitlist_notified** (service ID, matched_entry_count) -- N/A -- no success-metrics.md metric directly measures waitlist matching; retained per the Brief's Non-Functional Notes naming waitlist_notified as one of the four Stage 2 signals this feature's transitions must emit.
- **waitlist_match_none_found** (service ID) -- N/A -- no Stage 2 metric measures unmatched openings; retained for operational visibility into how often a freed slot has no waiting client.
- **waitlist_notification_handoff_failed** (service ID) -- N/A -- no Stage 2 metric measures this failure path; retained so a silent notification gap is observable rather than invisible, consistent with the product's correctness-first bar (ASMP-21).

## Acceptance Criteria

**FEAT-20.SPEC-005-AC-01:** Given a client cancellation commits (FEAT-10.SPEC-004) for a service and date matching Riley's Requested entry, when this automation runs, then her entry transitions to Notified with claim_deadline set to platform parameter: `waitlist-claim-window-minutes` from that moment, and FEAT-20.SPEC-008 is triggered.

**FEAT-20.SPEC-005-AC-02:** Given a Pro single cancellation commits (FEAT-30.SPEC-007) for a freed slot with no matching Requested entries, when this automation runs, then no entry is transitioned and the slot proceeds to general availability.

**FEAT-20.SPEC-005-AC-03:** Given a Pro bulk cancellation (FEAT-30.SPEC-008) frees three bookings' slots, when this automation processes them, then each freed slot's matching runs independently with its own set of Notified entries.

**FEAT-20.SPEC-005-AC-04:** Given three Requested entries match the same freed slot, when this automation runs, then all three are transitioned to Notified simultaneously and each triggers its own FEAT-20.SPEC-008 notification.

**FEAT-20.SPEC-005-AC-05:** Given a booking's vacated original time results from a reschedule rather than a cancellation, then this automation is never triggered for that vacated time.

**FEAT-20.SPEC-005-AC-06:** Given this automation transitions an entry to Notified, when the notification hand-off to FEAT-20.SPEC-008 fails, then the entry still shows as Notified with a countdown on FEAT-20.SPEC-002, even though no notification was delivered.

**FEAT-20.SPEC-005-AC-07:** Given the matching step itself fails to run for a freed slot, when the failure occurs, then no entry is transitioned and the slot proceeds to general availability exactly as if zero entries had matched.

**FEAT-20.SPEC-005-AC-08:** Given a client's range-joined entry could match two different freed slots that arrive at the same time, when the first freed-slot signal is processed, then the entry transitions to Notified for that slot and becomes ineligible to be matched again until it resolves.

**FEAT-20.SPEC-005-AC-09:** Given two different bookings for two different services are cancelled at effectively the same time, when this automation processes both, then each freed-slot signal's matching runs independently and neither is delayed by the other.

**FEAT-20.SPEC-005-AC-10:** Given a Booking can only be cancelled once due to its own reject-with-refresh contention resolution, then this automation is never triggered twice for the same cancelled booking.

**FEAT-20.SPEC-005-AC-11:** Given zero entries match a freed slot, when this automation completes, then the slot appears on FEAT-05's general public list without any priority window elapsing.

**FEAT-20.SPEC-005-AC-12:** Given one or more entries are matched and notified, when the claim window is in effect, then the freed slot does not appear on FEAT-05's general public list until the window lapses, per FEAT-20.SPEC-004.

**FEAT-20.SPEC-005-AC-13:** Given a matched entry's claim_deadline is computed, then it is set exactly once, at the moment of transition to Notified, and this automation never recomputes it afterward.

**FEAT-20.SPEC-005-AC-14:** Given this automation reports completion back to the triggering cancellation commit, then that commit (FEAT-10.SPEC-004 or FEAT-30.SPEC-007/SPEC-008) proceeds and completes without waiting on this automation's own outcome.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 | 3 |
| Outcome Paths | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |



# Automation Spec: Waitlist Claim Conversion

## Overview

**Name:** Waitlist Claim Conversion
**ID:** FEAT-20.SPEC-006
**Type:** Automation
**Purpose:** When a notified client completes the ordinary booking flow for the matching slot, converts their entry to Converted and leaves the other notified entries untouched.
**Parent Feature:** FEAT-20 -- Waitlist for Cancelled Slots

## Scope and Non-Goals

**In Scope:**
- Recognizing that a completed Booking corresponds to a Notified Waitlist Entry's claimed slot
- Transitioning that entry to Converted
- Confirming the other Notified entries for the same freed slot (if any) are left unconverted and unexpired by this event

**Non-Goals:**
- Creating the Booking itself, or handling the deposit payment -- owned entirely by FEAT-05 (Public Booking Page & Booking Flow); this automation reacts to a booking FEAT-05 already completed, it never creates one
- Resolving which of several notified clients wins a contested slot -- that resolution is FEAT-03.SPEC-005's first-to-pay-wins mechanism (XBR-01), applied by the ordinary booking flow itself; this automation only records the outcome for the winning client's Waitlist Entry once FEAT-05 reports the booking complete
- Expiring the other, still-Requested notified entries -- owned by FEAT-20.SPEC-007; this automation only converts the one entry whose client won the slot

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A booking completes via the claim link | FEAT-05 (Public Booking Page & Booking Flow), reached via the claim link on FEAT-20.SPEC-008 | Fires when FEAT-05's booking flow reports a completed Booking whose service, date, and start time match a Notified Waitlist Entry's claimed slot for the same Client | Booking reference, service, date, start time, Client reference |

## Processing Logic

1. Receive the completed-Booking signal from FEAT-05, carrying the Client reference and the booked service/date/start time.
2. Find the requesting Client's Waitlist Entry in state Notified whose matched slot (service, date, start time, as set by FEAT-20.SPEC-005) equals the completed Booking's service, date, and start time.
3. If found, confirm the entry's claim_deadline has not yet passed at the moment the Booking completed (re-checked here as the authoritative gate, even though FEAT-05's own slot re-validation already confirmed the slot was bookable at payment time).
4. If the window check passes, transition that Waitlist Entry's state to Converted.
5. Take no action on any other Notified entry for the same freed slot -- they remain Notified, each still governed by its own independent claim_deadline, per FEAT-20.SPEC-004's contention rule that the losing entries remain eligible for the next opportunity.
6. If no matching Notified entry is found for this Client and this slot (the booking was an ordinary booking unrelated to any waitlist claim), take no action -- this is the expected outcome for the overwhelming majority of bookings, which never touch this automation at all.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Claim converted | A completed Booking matches a Notified entry for the same Client, within its claim window | Waitlist Entry: state -> Converted | The client's entry shows "Booked" on FEAT-20.SPEC-002 on next load; no separate notification is sent beyond the ordinary booking confirmation (FEAT-08.SPEC-001), since the client is already looking at the confirmation | FEAT-20.SPEC-002 |
| No matching entry found | The completed Booking does not correspond to any Notified entry for this Client (an ordinary booking, unrelated to the waitlist) | None | None -- this is the expected outcome for ordinary bookings | -- |
| Claim window had already lapsed at booking completion | A matching Notified entry exists, but its claim_deadline passed before the Booking completed | No conversion -- the entry is left exactly as FEAT-20.SPEC-007's own expiry processing will handle it | The client still keeps the Booking she just completed (it was validated as bookable through the ordinary, general-availability path, not the priority window); her waitlist entry's own fate is governed separately by FEAT-20.SPEC-007 | FEAT-20.SPEC-007 |
| Automation failure (the conversion write itself fails) | A transient failure prevents the state write from completing | No change to the Waitlist Entry | The Booking itself is unaffected and remains completed; the entry may still show as Notified until the next successful processing pass or until FEAT-20.SPEC-007's expiry naturally resolves it | FEAT-20.SPEC-002, FEAT-20.SPEC-007 |

## Data Model

**Reads:** Waitlist Entry -- state, service, start_date/end_date, claim_deadline, scoped to the Client who completed the Booking; Booking -- service, date, start_time, Client reference, to confirm the match.
**Creates:** None.
**Updates:** Waitlist Entry -- state (Notified -> Converted), for the one matched entry only.
**Deletes:** None.

## Business Rules

- XBR-01 governs the underlying contention this automation observes the outcome of: the first client to complete payment wins a contested slot; this automation records that outcome for the winner's Waitlist Entry, it does not itself decide the winner.
- Only the client whose completed Booking matches a Notified entry's exact slot receives a conversion -- every other Notified entry for the same freed slot is left untouched by this event, per FEAT-20.SPEC-004's rule that losing entries remain eligible for the next opportunity, never automatically expired or deleted by someone else's successful claim.
- A Converted entry is terminal and retained indefinitely per this Brief's Non-Goals (no automatic purge); it is never re-activated or re-matched.
- This automation performs no notification of its own -- the client already sees the ordinary booking confirmation (FEAT-08.SPEC-001) from completing the booking flow; a redundant "you claimed it" message would be noise on top of a confirmation she is already looking at.
- Platform Operator (Support) can view a Converted entry and the Booking it produced through FEAT-19's read-only account view, per XBR-24, useful for a dispute or confusion around who claimed a given opening; Support never performs or reverses a conversion.

## Edge Cases

- **Two notified clients race to claim the same freed slot** -- Whichever completes payment first triggers this automation's conversion for her own entry; the second client's payment attempt is refused by FEAT-03.SPEC-005's contention resolution before it ever reaches this automation, so only one conversion is ever produced per freed slot.
- **A client completes a booking for the same service and date through the ordinary booking flow, coincidentally, without ever having been notified (she never joined the waitlist for this slot)** -- No matching Notified entry exists for her, so this automation takes no action; this is simply an ordinary booking.
- **A client holds a Notified entry for one freed slot but books a completely different time for the same service through the general booking flow** -- The completed Booking's date/time does not match the Notified entry's claimed slot, so no conversion occurs; her Notified entry remains active and subject to its own claim window and eventual expiry.
- **The claim window lapses between the client tapping "Claim now" and completing payment** -- FEAT-05's own slot re-validation at checkout (mirroring FEAT-03.SPEC-006's re-validation pattern) determines whether the slot is still reachable through the priority path at that moment; if her window lapsed first, this automation's window check in Processing Logic step 3 also fails, and no conversion is recorded -- her entry is instead handled by FEAT-20.SPEC-007's expiry.
- **Concurrent trigger firing (two different clients each complete a claim for two different freed slots at the same time)** -- Each conversion runs independently; neither is delayed by the other.
- **Trigger fires while a previous run is in flight for the same Client and entry** -- Not possible in practice: a given Waitlist Entry converts at most once (state moves from Notified to the terminal Converted and never back), so a second completed-Booking signal for an already-Converted entry finds no eligible Notified entry to match and takes no action.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-05 (Public Booking Page & Booking Flow) | Triggered by (inbound) | A completed Booking through the claim link fires this automation |
| FEAT-20.SPEC-008 (Waitlist Opening Notification) | References (inbound) | The claim link this automation's trigger originates from is the one that notification carries |
| FEAT-20.SPEC-004 (Waitlist Priority & Claim Window Rule) | References (outbound) | Governs the window check and the losing entries' contention resolution |
| FEAT-20.SPEC-002 (My Waitlists) | Affects (outbound) | The Converted status is what that screen's next load reflects |
| FEAT-20.SPEC-007 (Waitlist Entry Expiry) | References (outbound) | Governs what happens to an entry whose window lapsed before this automation could convert it |
| FEAT-08.SPEC-001 (Booking Confirmation Message) | References (outbound) | Supplies the client's booking confirmation; this automation triggers no separate message of its own |

## Analytics and Success Signals

- **waitlist_converted_to_booking** (service ID, time from notification to conversion) -- N/A -- no success-metrics.md metric directly measures waitlist conversion; retained per the Brief's Non-Functional Notes naming waitlist_converted_to_booking as one of the four Stage 2 signals this feature's transitions must emit.
- **waitlist_claim_window_lapsed_before_conversion** (service ID) -- N/A -- no Stage 2 metric measures near-miss claims; retained for operational visibility into how often a claim attempt arrives just after its window closes.

## Acceptance Criteria

**FEAT-20.SPEC-006-AC-01:** Given Riley taps the claim link on her opening notification and completes the booking flow within her claim window, when this automation processes the completed Booking, then her Waitlist Entry transitions to Converted.

**FEAT-20.SPEC-006-AC-02:** Given Riley's entry converts, when she next views FEAT-20.SPEC-002, then it shows "Booked" for that entry.

**FEAT-20.SPEC-006-AC-03:** Given two clients were both Notified for the same freed slot and Riley completes payment first, when this automation processes her completed Booking, then only her entry converts and the other client's entry remains Notified, unconverted.

**FEAT-20.SPEC-006-AC-04:** Given Riley completes an ordinary booking unrelated to any waitlist entry, when this automation evaluates it, then no matching Notified entry is found and no action is taken.

**FEAT-20.SPEC-006-AC-05:** Given Riley holds a Notified entry for one slot but books a different time for the same service, when this automation evaluates the completed Booking, then it does not match her Notified entry, and that entry remains active.

**FEAT-20.SPEC-006-AC-06:** Given Riley's claim window lapses before her payment completes, when this automation checks the window at completion, then no conversion is recorded and her entry is instead handled by FEAT-20.SPEC-007.

**FEAT-20.SPEC-006-AC-07:** Given a second notified client's payment attempt is refused by FEAT-03.SPEC-005's contention resolution after the first client already won the slot, then this automation is never triggered for the second client's failed attempt.

**FEAT-20.SPEC-006-AC-08:** Given Riley's entry converts, when the conversion completes, then no separate waitlist-specific notification is sent -- only the ordinary booking confirmation (FEAT-08.SPEC-001) she already sees.

**FEAT-20.SPEC-006-AC-09:** Given the conversion write itself fails due to a transient error, when the failure occurs, then Riley's completed Booking is unaffected and her entry's state is resolved by the next successful pass or by FEAT-20.SPEC-007's expiry.

**FEAT-20.SPEC-006-AC-10:** Given two different clients each complete a claim for two different freed slots at effectively the same time, when this automation processes both, then each conversion runs independently.

**FEAT-20.SPEC-006-AC-11:** Given an entry has already converted, when a second completed-Booking signal somehow arrives referencing the same entry, then no eligible Notified entry is found and no further action is taken.

**FEAT-20.SPEC-006-AC-12:** Given Riley's Notified entry converts, when the entry is later inspected, then it is retained indefinitely as a terminal, historical record per this Brief's Non-Goals.

**FEAT-20.SPEC-006-AC-13:** Given a Converted entry exists, when anyone looks for a way to re-activate or re-match it, then no such control exists anywhere in the product.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Automation Spec: Waitlist Entry Expiry

## Overview

**Name:** Waitlist Entry Expiry
**ID:** FEAT-20.SPEC-007
**Type:** Automation
**Purpose:** Expires a Notified entry whose 30-minute claim window lapses unclaimed, and separately expires a Requested entry whose joined date range elapses with no matching opening ever found.
**Parent Feature:** FEAT-20 -- Waitlist for Cancelled Slots

## Scope and Non-Goals

**In Scope:**
- Detecting a Notified entry whose claim_deadline has passed without converting
- Detecting a Requested entry whose end_date has passed with no matching opening ever found
- Transitioning each to Expired and handing off to the expiry notification

**Non-Goals:**
- Converting a claim that completes within the window -- owned by FEAT-20.SPEC-006; this automation only handles the unclaimed-lapse case
- Deciding what makes an entry match a freed slot in the first place -- owned by FEAT-20.SPEC-004/SPEC-005; this automation only observes that no match ever occurred before the range elapsed
- Composing or sending the expiry notification's content -- owned by FEAT-20.SPEC-009; this automation only triggers it

## Trigger Definition

| Trigger | Category | Source Spec | Conditions | Available Data |
|---------|----------|------------|------------|----------------|
| A Notified entry's claim_deadline passes | Schedule-based | system (evaluated continuously against each Notified entry's own claim_deadline) | Fires the moment the current time reaches a Notified entry's claim_deadline without that entry having converted (FEAT-20.SPEC-006) | Waitlist Entry reference, service, matched slot detail, claim_deadline |
| A Requested entry's end_date passes | Schedule-based | system (evaluated continuously against each Requested entry's own end_date) | Fires the moment the current time passes the end of a Requested entry's joined date range without it ever having been matched (FEAT-20.SPEC-005) | Waitlist Entry reference, service, start_date, end_date |

## Processing Logic

1. **Claim-window path:** Identify every Waitlist Entry in state Notified whose claim_deadline has passed.
2. Re-confirm each identified entry has not converted in the interval since claim_deadline was checked (avoiding a race with FEAT-20.SPEC-006 processing a last-second claim).
3. Transition each confirmed entry's state to Expired.
4. Trigger FEAT-20.SPEC-009 (Waitlist Expiry Notification) for each, with the reason "unclaimed opening."
5. **Unmatched-range path:** Identify every Waitlist Entry in state Requested whose end_date has passed.
6. Transition each identified entry's state to Expired.
7. Trigger FEAT-20.SPEC-009 for each, with the reason "unmatched date range."
8. Both paths run independently and on their own schedule; an entry is only ever processed by the one path that applies to its current state at the moment of evaluation.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Notified entry expired (unclaimed) | claim_deadline passed without conversion | Waitlist Entry: state -> Expired | Client receives the expiry notification (FEAT-20.SPEC-009) stating the opening was unclaimed; entry shows "Expired" on FEAT-20.SPEC-002 for one visit, per that screen's roll-off rule | FEAT-20.SPEC-009, FEAT-20.SPEC-002 |
| Requested entry expired (unmatched range) | end_date passed with no match ever found | Waitlist Entry: state -> Expired | Client receives the expiry notification stating no matching opening was ever found; entry shows "Expired" on FEAT-20.SPEC-002 for one visit | FEAT-20.SPEC-009, FEAT-20.SPEC-002 |
| No entries due for expiry | Neither condition is met for any entry at the current evaluation moment | None | None | -- |
| Automation failure (the expiry write itself fails) | A transient failure prevents the state write from completing | No change to the affected entry | The entry remains in its prior state (Notified past its window, or Requested past its range) until the next successful evaluation pass re-attempts the same transition | FEAT-20.SPEC-002 |

## Data Model

**Reads:** Waitlist Entry -- state, claim_deadline, start_date, end_date, across all entries.
**Creates:** None.
**Updates:** Waitlist Entry -- state (Notified -> Expired, or Requested -> Expired).
**Deletes:** None -- Expired entries are retained per this Brief's Non-Goals (no automatic purge), not deleted.

## Business Rules

- The claim window is fixed at platform parameter: `waitlist-claim-window-minutes` and never extended for any reason (FEAT-20.SPEC-004) -- an entry that reaches claim_deadline unclaimed expires without exception.
- A Requested entry's own joined date range is its natural lifetime bound: once end_date passes with no match ever found, the entry has no further chance to match (the range it asked about is now in the past) and expires.
- Both expiry paths are mutually exclusive per entry at any given moment: an entry is either Requested (subject to the unmatched-range path) or Notified (subject to the claim-window path), never both at once, since matching (FEAT-20.SPEC-005) is the only transition from Requested to Notified.
- Every expiry, from either path, triggers the expiry notification (FEAT-20.SPEC-009) -- an entry is never silently retired without informing the client, per this Brief's own Alternate flow: "the client is informed rather than left wondering indefinitely."
- Platform Operator (Support) can view an Expired entry and which path produced it through FEAT-19's read-only account view, per XBR-24, useful when a client reports "I never heard back"; Support never reactivates or extends an expired entry.

## Edge Cases

- **A Notified entry's claim_deadline passes at the exact instant FEAT-20.SPEC-006 is processing a completed Booking for it** -- The re-confirmation step (Processing Logic step 2) checks for a conversion that may have just landed; if the conversion already committed, this automation takes no action on that entry, since FEAT-20.SPEC-006's transition to Converted takes precedence over a late-arriving expiry evaluation for the same entry.
- **A Requested entry's range elapses on the same day a matching opening would have appeared, but the cancellation that would have produced it never occurs** -- The entry expires exactly as any other unmatched-range entry; the automation has no way to know a "near miss" almost happened and does not treat it differently.
- **A range-joined entry's end_date passes while the entry is mid-match (a freed-slot signal is being evaluated for it by FEAT-20.SPEC-005 at the same moment)** -- Whichever transition commits first wins: if FEAT-20.SPEC-005 already transitioned the entry to Notified before this automation's unmatched-range check runs, the entry is no longer Requested and this automation's unmatched-range path does not apply to it (it is now subject to the claim-window path instead, on its own new timeline).
- **Two different Notified entries reach their independent claim_deadlines at effectively the same time** -- Each is expired independently; neither is delayed by the other.
- **Concurrent trigger firing (the claim-window path and the unmatched-range path evaluate at the same moment across many entries)** -- Both paths run independently across the full set of entries each governs; there is no shared lock between them since they never target the same entry at the same time (per the mutual-exclusivity business rule above).
- **Trigger fires while a previous evaluation pass is still processing the same entry** -- Not possible in practice: an entry transitions to Expired at most once (a terminal state), so a second evaluation of an already-Expired entry finds it no longer eligible for either path and takes no action.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-20.SPEC-004 (Waitlist Priority & Claim Window Rule) | References (outbound) | Supplies the fixed claim-window value this automation checks against |
| FEAT-20.SPEC-005 (Cancellation-Triggered Waitlist Matching) | References (outbound) | The transition to Notified this automation's claim-window path watches for expiry against |
| FEAT-20.SPEC-006 (Waitlist Claim Conversion) | References (outbound) | The competing transition this automation's re-confirmation step checks for before expiring a Notified entry |
| FEAT-20.SPEC-009 (Waitlist Expiry Notification) | Triggers (outbound) | Fires once per expired entry, with the applicable reason |
| FEAT-20.SPEC-002 (My Waitlists) | Affects (outbound) | The Expired status is what that screen's next load (and one-visit roll-off) reflects |

## Analytics and Success Signals

- **waitlist_expired** (service ID, reason: unclaimed_opening / unmatched_range) -- N/A -- no success-metrics.md metric directly measures waitlist expiry; retained per the Brief's Non-Functional Notes naming waitlist_expired as one of the four Stage 2 signals this feature's transitions must emit.

## Acceptance Criteria

**FEAT-20.SPEC-007-AC-01:** Given Riley's Notified entry's claim window lapses without her completing a booking, when this automation evaluates it, then her entry transitions to Expired and FEAT-20.SPEC-009 is triggered with reason "unclaimed opening."

**FEAT-20.SPEC-007-AC-02:** Given Riley's Requested entry's joined date range elapses with no matching opening ever found, when this automation evaluates it, then her entry transitions to Expired and FEAT-20.SPEC-009 is triggered with reason "unmatched date range."

**FEAT-20.SPEC-007-AC-03:** Given Riley's Notified entry converts to a Booking just before this automation's claim-window check runs, when the re-confirmation step executes, then no expiry is recorded, since the conversion already committed.

**FEAT-20.SPEC-007-AC-04:** Given Riley's Requested entry transitions to Notified via FEAT-20.SPEC-005 at the same moment its end_date would otherwise trigger the unmatched-range path, when this automation evaluates it, then the unmatched-range path does not apply, and the entry is instead subject to the claim-window path on its new timeline.

**FEAT-20.SPEC-007-AC-05:** Given no entries are due for expiry at the current evaluation moment, when this automation runs, then no entry is transitioned and no notification is triggered.

**FEAT-20.SPEC-007-AC-06:** Given the expiry write fails for an entry due to a transient error, when the failure occurs, then the entry remains in its prior state until the next successful evaluation pass.

**FEAT-20.SPEC-007-AC-07:** Given an entry has already expired, when a later evaluation pass considers it again, then it is no longer eligible for either expiry path and no further action is taken.

**FEAT-20.SPEC-007-AC-08:** Given two different Notified entries reach their independent claim_deadlines at effectively the same time, when this automation processes both, then each expires independently.

**FEAT-20.SPEC-007-AC-09:** Given Riley's Expired entry, when she next views FEAT-20.SPEC-002, then it shows "Expired" for one visit before rolling off the list on a subsequent visit.

**FEAT-20.SPEC-007-AC-10:** Given an entry is Requested (never yet matched), when this automation evaluates it, then it is only ever subject to the unmatched-range path, never the claim-window path.

**FEAT-20.SPEC-007-AC-11:** Given an entry is Notified (already matched), when this automation evaluates it, then it is only ever subject to the claim-window path, never the unmatched-range path.

**FEAT-20.SPEC-007-AC-12:** Given every expiry this automation produces, when the transition completes, then FEAT-20.SPEC-009 is triggered without exception -- no entry expires silently.

**FEAT-20.SPEC-007-AC-13:** Given an Expired entry, when anyone looks for a way to reactivate or restore it, then no such control exists anywhere in the product -- a client who wants back on the waitlist rejoins fresh through FEAT-20.SPEC-001.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 | 2 |
| Outcome Paths | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Notification Spec: Waitlist Opening Notification

## Overview

**Name:** Waitlist Opening Notification
**ID:** FEAT-20.SPEC-008
**Type:** Notification
**Purpose:** Notifies a matching client the moment their slot opens, states the 30-minute claim window, and carries the claim link into the booking flow.
**Parent Feature:** FEAT-20 -- Waitlist for Cancelled Slots

## Scope and Non-Goals

**In Scope:**
- The notification delivered the instant a Requested entry is matched and transitioned to Notified
- Stating the exact claim window and carrying the claim link
- Delivery timing, retry, and expiry behavior for this specific, time-critical send

**Non-Goals:**
- Deciding which entries are matched or how the claim window is computed -- owned by FEAT-20.SPEC-004/SPEC-005; this spec only sends the notification those specs' transitions trigger
- The booking flow the claim link opens into -- owned by FEAT-05; this spec only carries the client there
- Applying to entries whose window has already lapsed -- that case is FEAT-20.SPEC-009's expiry notification, a distinct communication

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Text | The matched client has active texting consent for this Pro (FEAT-14.SPEC-007, XBR-15) | Riley's entire claim window is only 30 minutes; a text is the channel most likely to reach her within it, matching this Brief's own decision that this notification is time-critical, same as a payment confirmation |
| Email | The matched client does not have active texting consent | The fallback channel every other transactional communication in this product uses when texting consent is absent (XBR-15) |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A Waitlist Entry transitions to Notified | FEAT-20.SPEC-005 (Cancellation-Triggered Waitlist Matching) | Fires once per matched entry, immediately at the moment of transition | Client reference, service name, matched slot date/time, claim_deadline, claim link destination |

## Audience and Preferences

**Recipients:** The Client (Riley) -- the sole recipient the Access Matrix's Waitlist row (Own-only for the Client) supports; this notification is never sent to the Pro or to Platform Operator (Support), consistent with the Pro's aggregate-only View access and Support's read-only troubleshooting access.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Texting consent (channel eligibility only, not an on/off switch for this notification) | Granted / Revoked | Whatever the client's current consent state is with this Pro | FEAT-06.SPEC-005 (Consent & Email Preferences) |

There is no on/off preference for this notification itself -- it is a transactional communication about the client's own waitlist entry, not a promotional or discretionary send (scope-boundaries.md SC-15), so it cannot be turned off independently of leaving the waitlist entirely (FEAT-20.SPEC-002).

**Quiet Hours:** N/A -- this Brief's recorded reading treats the opening notification as time-critical by design (its entire value depends on the client having the full claim window to act) and therefore outside ASMP-29's daytime-hours scope, the same as a payment confirmation; it is sent the moment the match occurs, at any hour.

## Content Definition

**Text:**
- **Body:** A spot opened for {service_name} on {matched_date} at {matched_time} -- you're on the waitlist! Claim it within {claim_window_minutes} minutes before it's offered to others: {claim_link}
- **CTA:** Claim it -- deep-links to FEAT-05's booking flow (FEAT-05.SPEC-002 onward) for the opened slot

**Email:**
- **Subject:** A spot opened for {service_name} -- claim it within {claim_window_minutes} minutes
- **Body:**
  A spot just opened up for {service_name} on {matched_date} at {matched_time}, and you're on the waitlist.

  You have {claim_window_minutes} minutes to claim it before it's offered to everyone else.
- **CTA (button):** Claim this spot -- deep-links to FEAT-05's booking flow (FEAT-05.SPEC-002 onward) for the opened slot

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {service_name} | Service -- name | Full Set -- Lashes | Never empty -- required at service creation (FEAT-01) |
| {matched_date} | Waitlist Entry -- derived, the freed slot's date from the matching signal (FEAT-20.SPEC-005) | Thursday, March 12 | Never empty -- the notification is triggered only once a match with a concrete date exists |
| {matched_time} | Waitlist Entry -- derived, the freed slot's start time, in the Pro's timezone (XBR-25) | 2:30 PM | Never empty -- same as above |
| {claim_window_minutes} | platform parameter: `waitlist-claim-window-minutes` | 30 | Never empty -- one value for every Pro |
| {claim_link} | Access Link (booking-specific, issued for the claim), or the direct route into FEAT-05.SPEC-002 for the matched slot | (link) | Never empty -- generated fresh at the moment this notification is composed |

## Delivery Rules

**Batching:** None -- each matched entry produces its own independent notification the instant it is matched; multiple openings for the same client (across different entries) are never combined into one message, since each carries its own distinct slot and claim window.
**Deduplication:** At most one opening notification per Waitlist Entry per match -- a Requested entry is only ever matched and Notified once per freed slot (FEAT-20.SPEC-004/SPEC-005); if that same entry is later matched again after expiring and being rejoined fresh, the new entry produces its own new notification, entirely independent of the prior one.
**Retry on failure:** Text delivery failure is retried and falls back to email per FEAT-08.SPEC-009's product-wide retry-and-fallback rule (platform parameter: `message-delivery-retry-count`), applied here exactly as for any other transactional text -- but because this notification is time-critical, a fallback that completes after a meaningful fraction of the 30-minute window has already elapsed still delivers (a late-but-real chance to claim is better than none), and the claim_deadline itself is never extended to compensate for delivery delay.
**Expiry:** This notification is never re-sent or held past the moment it is composed -- if delivery ultimately fails on both channels, the client simply does not learn of the opening within the window; her Waitlist Entry still expires normally at claim_deadline (FEAT-20.SPEC-007) and she receives the expiry notification (FEAT-20.SPEC-009) explaining the entry expired, so she is never left permanently wondering even if this specific message never arrived.

## Edge Cases

- **The matched Waitlist Entry converts or is left (deleted) before this notification is delivered (a fast client checks FEAT-20.SPEC-002 directly and claims, or leaves, before the message lands)** -- The notification is still delivered as composed; a client who already acted on the opening by other means simply receives a redundant confirmation-adjacent message, since cancelling an in-flight send for a fast-acting client would risk the opposite failure (a client who needed the message not receiving it) and this product's correctness bar treats over-delivery as the safer default here.
- **The claim window lapses before the text retry or email fallback completes** -- The message still delivers if it can; a late-arriving notification for a lapsed window tells the client honestly that the moment has passed rather than showing a live, actionable link -- FEAT-05's own slot re-validation at the claim link's destination independently confirms whether the priority window is still open, so a stale link never grants a claim past the deadline it never should have.
- **Quiet hours vs. expiry collision** -- Not applicable: this notification carries no quiet-hours hold at all (see Quiet Hours above), so there is no collision to resolve.
- **A preference change mid-flight (texting consent is revoked between the match and the send)** -- FEAT-08.SPEC-011's channel-selection rule (via FEAT-14.SPEC-007's fresh-every-read textability determination) is evaluated at send time, not at match time, so a revoke that lands before the send routes this notification to email automatically.
- **Multiple entries for the same client are matched by the same freed slot signal (not possible under the matching test, since a slot has one service/date, but two different clients' entries can both match)** -- Each matched client's entry produces its own independent notification; this is not a batching case, since the recipients differ.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-20.SPEC-005 (Cancellation-Triggered Waitlist Matching) | Triggered by (inbound) | The transition to Notified fires this notification |
| FEAT-20.SPEC-004 (Waitlist Priority & Claim Window Rule) | References (inbound) | Supplies the claim window and the matched-slot data this content presents |
| FEAT-05 (Public Booking Page & Booking Flow) | Navigation (outbound) | The claim link's destination |
| FEAT-14.SPEC-007 (Textability Determination Rule) | References (inbound) | Determines the channel at send time |
| FEAT-08.SPEC-012 (Transactional Text Messaging Capability) | References (inbound) | Delivers the text channel |
| FEAT-08.SPEC-013 (Transactional Email Capability) | References (inbound) | Delivers the email channel |
| FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | References (inbound) | Governs retry and fallback behavior for this send |
| FEAT-20.SPEC-006 (Waitlist Claim Conversion) | Affects (outbound) | A completed claim through this notification's link is what that automation converts |
| FEAT-20.SPEC-007 (Waitlist Entry Expiry) | References (outbound) | Governs what happens if the window lapses without a claim |

## Analytics and Success Signals

- **waitlist_opening_notification_delivered** (channel: text / email) -- N/A -- no success-metrics.md metric directly measures waitlist notification delivery; retained because a silent delivery gap here would defeat the entire feature's value proposition without any metric detecting why.
- **waitlist_opening_notification_claim_link_tapped** () -- N/A -- no Stage 2 metric names this specific tap; retained as the leading indicator behind waitlist_converted_to_booking (FEAT-20.SPEC-006).
- **waitlist_opening_notification_delivery_failed** (channel: text / email) -- N/A -- no Stage 2 metric measures delivery failure specifically for this notification; retained for operational visibility given how time-sensitive a failed send here is (ASMP-21's correctness-and-speed bar).

## Acceptance Criteria

**FEAT-20.SPEC-008-AC-01:** Given Riley's Requested entry transitions to Notified and she has active texting consent with this Pro, when this notification fires, then she receives a text stating the service, matched date/time, and the platform parameter: `waitlist-claim-window-minutes` window, with the claim link.

**FEAT-20.SPEC-008-AC-02:** Given Riley does not have active texting consent, when this notification fires, then she receives the email variant with the identical content and CTA.

**FEAT-20.SPEC-008-AC-03:** Given Riley taps the claim link, then she is routed into FEAT-05's booking flow for the exact opened slot.

**FEAT-20.SPEC-008-AC-04:** Given this notification fires at 3am in the Pro's timezone, when delivery is evaluated, then it is sent immediately, unaffected by ASMP-29's daytime-hours rule, since this Brief treats it as time-critical.

**FEAT-20.SPEC-008-AC-05:** Given the text send fails, when the retry-and-fallback rule (FEAT-08.SPEC-009) processes it, then it retries per platform parameter: `message-delivery-retry-count` and falls back to email if the retry also fails.

**FEAT-20.SPEC-008-AC-06:** Given the fallback email completes after part of the claim window has already elapsed, when delivery finishes, then the message still delivers, and the claim_deadline itself is not extended to compensate.

**FEAT-20.SPEC-008-AC-07:** Given Riley's claim window fully lapses before any channel succeeds, when the final failure is confirmed, then Riley receives no live notification for this opening, and her entry proceeds to FEAT-20.SPEC-007's expiry path, which triggers FEAT-20.SPEC-009 instead.

**FEAT-20.SPEC-008-AC-08:** Given Riley revokes texting consent between her entry being matched and this notification being sent, when the send executes, then the channel-selection rule (evaluated fresh at send time) routes the message to email.

**FEAT-20.SPEC-008-AC-09:** Given two different clients' entries both match the same freed slot, when this notification fires, then each client receives her own independent notification, never a combined or batched send.

**FEAT-20.SPEC-008-AC-10:** Given Riley leaves her waitlist entry moments after being matched but before this notification is delivered, when the send proceeds anyway, then she still receives the message, since the product does not cancel an in-flight send for a fast-acting client.

**FEAT-20.SPEC-008-AC-11:** Given Riley taps a claim link whose window has already lapsed by the time she opens it, when FEAT-05's slot re-validation runs, then it does not grant her a priority-window booking past the deadline.

**FEAT-20.SPEC-008-AC-12:** Given Riley's entry is matched a second time after a prior entry for the same service expired and she rejoined fresh, when the new match occurs, then a new, independent notification is sent, unrelated to any prior one.

**FEAT-20.SPEC-008-AC-13:** Given Talia (the Pro) is not a recipient of this notification under any condition, when a slot on her own calendar opens and matches a client, then she receives no copy of this notification -- only her own aggregate demand count on FEAT-12 reflects the activity.

**FEAT-20.SPEC-008-AC-14:** Given Support views a Pro's account via FEAT-19, when they look at this notification's delivery status, then they see it View-only, consistent with Support's read-only access to Message delivery status.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (text, email) | 2 |
| Trigger Paths | 1 | 1 |
| Preference States | 2 (consent granted, consent revoked) | 2 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |



# Notification Spec: Waitlist Expiry Notification

## Overview

**Name:** Waitlist Expiry Notification
**ID:** FEAT-20.SPEC-009
**Type:** Notification
**Purpose:** Informs a client that their waitlist entry has expired -- either an unclaimed opening or an unmatched date range -- so they are never left wondering.
**Parent Feature:** FEAT-20 -- Waitlist for Cancelled Slots

## Scope and Non-Goals

**In Scope:**
- The notification delivered when either expiry path (FEAT-20.SPEC-007) transitions an entry to Expired
- Both variants: unclaimed opening, and unmatched date range
- Delivery timing, retry, and expiry behavior for this non-time-critical send

**Non-Goals:**
- Deciding when an entry expires -- owned by FEAT-20.SPEC-007; this spec only sends the notification that automation's transitions trigger
- The opening notification itself -- owned by FEAT-20.SPEC-008, a distinct, time-critical communication with its own channel and quiet-hours behavior
- Offering a one-tap rejoin action -- product-features.md and this Brief describe no such shortcut; a client who wants back on the waitlist rejoins fresh through FEAT-20.SPEC-001, counted freshly against the cap, matching the Entity-Lifecycle Coverage Matrix's "no restore path" decision

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Text | The client has active texting consent for this Pro (FEAT-14.SPEC-007, XBR-15) | Consistent with every other transactional message this product sends this client |
| Email | The client does not have active texting consent | The product-wide fallback channel (XBR-15) |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A Notified entry expires unclaimed | FEAT-20.SPEC-007 (Waitlist Entry Expiry) | Fires when the claim-window path transitions an entry to Expired | Client reference, service name, the opening's matched date, reason: unclaimed_opening |
| A Requested entry expires unmatched | FEAT-20.SPEC-007 (Waitlist Entry Expiry) | Fires when the unmatched-range path transitions an entry to Expired | Client reference, service name, the joined date range, reason: unmatched_range |

## Audience and Preferences

**Recipients:** The Client (Riley) -- the sole recipient, matching the Access Matrix's Own-only Waitlist access; never the Pro or Platform Operator (Support).

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Texting consent (channel eligibility only) | Granted / Revoked | Whatever the client's current consent state is with this Pro | FEAT-06.SPEC-005 (Consent & Email Preferences) |

There is no on/off preference for this notification -- it is a transactional communication informing the client of her own entry's outcome (scope-boundaries.md SC-15), not a discretionary or promotional send.

**Quiet Hours:** This notification carries no time pressure (the window it concerns has already closed), so it follows ASMP-29's ordinary daytime-hours rule: held if it would otherwise be delivered outside platform parameter: `reminder-window-start-hour` to platform parameter: `reminder-window-end-hour` in the Pro's timezone, and delivered at the window's start once it opens.

## Content Definition

**Text (unclaimed-opening variant):**
- **Body:** Your {claim_window_minutes}-minute window to claim the {service_name} spot on {matched_date} has passed. You're still on the waitlist for the rest of your requested range if it hasn't ended -- see your status: {my_waitlists_link}
- **CTA:** View my waitlists -- deep-links to FEAT-20.SPEC-002 (My Waitlists)

**Text (unmatched-range variant):**
- **Body:** No matching opening came up for {service_name} between {start_date} and {end_date}, so that waitlist request has ended. Want to try again? {join_waitlist_link}
- **CTA:** Join again -- deep-links to FEAT-05.SPEC-001 (service list, the entry point that leads back to FEAT-20.SPEC-001)

**Email (unclaimed-opening variant):**
- **Subject:** Your waitlist window for {service_name} has closed
- **Body:**
  Your {claim_window_minutes}-minute window to claim the {service_name} spot on {matched_date} has passed, so it's now been offered more broadly.

  If your original requested range hasn't ended yet, you're still waitlisted for it -- check your status any time.
- **CTA (button):** View my waitlists -- deep-links to FEAT-20.SPEC-002 (My Waitlists)

**Email (unmatched-range variant):**
- **Subject:** Your waitlist request for {service_name} has ended
- **Body:**
  No matching opening came up for {service_name} between {start_date} and {end_date}, so that waitlist request has ended.

  You're welcome to join again for a new date or range any time.
- **CTA (button):** Join the waitlist again -- deep-links to FEAT-05.SPEC-001

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {service_name} | Service -- name | Full Set -- Lashes | Never empty -- required at service creation (FEAT-01) |
| {claim_window_minutes} | platform parameter: `waitlist-claim-window-minutes` | 30 | Never empty -- one value for every Pro |
| {matched_date} | Waitlist Entry -- derived, the freed slot's date from the matching signal that produced the now-lapsed Notified state | Thursday, March 12 | Never empty -- only present in the unclaimed-opening variant, which only fires for an entry that was genuinely matched |
| {start_date} / {end_date} | Waitlist Entry -- start_date, end_date | March 10 / March 17 | Never empty -- required at join (FEAT-20.SPEC-003) |
| {my_waitlists_link} | Access Link (booking-specific, into FEAT-20.SPEC-002), reached via FEAT-06 | (link) | If the client's access link cannot be freshly issued at send time, the CTA still renders and routes into FEAT-06.SPEC-001 to request one, rather than omitting the link entirely |
| {join_waitlist_link} | Direct route into FEAT-05.SPEC-001 | (link) | Never empty -- FEAT-05's service list requires no identity or link state to reach |

## Delivery Rules

**Batching:** None -- each expiry is its own distinct entry outcome (a different service, date, or reason), and this Brief's own communication tone treats each as worth its own message rather than a rolled-up summary; a client with two entries expiring on the same day receives two separate notifications.
**Deduplication:** At most one expiry notification per Waitlist Entry -- an entry transitions to Expired exactly once (a terminal state), so this notification fires exactly once per entry.
**Retry on failure:** Text delivery failure is retried and falls back to email per FEAT-08.SPEC-009's product-wide retry-and-fallback rule (platform parameter: `message-delivery-retry-count`), identical to any other transactional text in this product.
**Expiry:** This notification is held, not dropped, if quiet hours are in effect at the moment it would otherwise send, and delivers at the next allowed hour; if both channels ultimately fail after the ordinary retry/fallback chain, no further attempt is made -- the client's entry still shows "Expired" on FEAT-20.SPEC-002 for one visit, which is the surviving signal even if this specific message is never delivered.

## Edge Cases

- **The underlying Waitlist Entry is later inspected and no longer shows on FEAT-20.SPEC-002 (it rolled off after one visit) by the time the client reads a delayed notification** -- The notification's content stands on its own (it names the service and dates directly) and does not depend on the entry still being visible on that screen; the CTA still routes correctly to FEAT-20.SPEC-002, which simply shows no matching row by then, an outcome consistent with that screen's own roll-off rule.
- **Quiet hours extend past the point the client might reasonably expect this notification** -- The held notification delivers at the next allowed window-start hour, exactly as any other daytime-bound notification in this product (FEAT-08.SPEC-007's pattern); there is no separate expiry cutoff for the notification itself distinct from the entry's own already-final Expired state.
- **A preference change mid-flight (texting consent is revoked between the expiry transition and the send)** -- The channel-selection rule evaluates fresh at send time (FEAT-14.SPEC-007), so a revoke landing before the send routes this notification to email automatically.
- **The client rejoins the waitlist for the same service before this notification is delivered** -- The notification still delivers as composed, describing the entry that actually expired; it is not cancelled or altered by a subsequent, unrelated new join, since the two are independent entries.
- **Both the unclaimed-opening and unmatched-range paths could plausibly apply to the same entry (a range-joined entry that was matched partway through its range and then its claim lapses)** -- Not possible under FEAT-20.SPEC-007's mutual-exclusivity rule: once an entry is Notified, it is no longer subject to the unmatched-range path at all, so only the unclaimed-opening variant ever fires for that entry.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-20.SPEC-007 (Waitlist Entry Expiry) | Triggered by (inbound) | Either expiry path fires this notification with its variant reason |
| FEAT-20.SPEC-002 (My Waitlists) | Navigation (outbound) | The unclaimed-opening variant's CTA deep-links here |
| FEAT-05.SPEC-001 (Public Booking Page & Service List) | Navigation (outbound) | The unmatched-range variant's CTA deep-links here, leading back toward FEAT-20.SPEC-001 |
| FEAT-06.SPEC-001 (Access Link Request) | Navigation (outbound) | Fallback destination when a fresh My Waitlists link cannot be issued at send time |
| FEAT-14.SPEC-007 (Textability Determination Rule) | References (inbound) | Determines the channel at send time |
| FEAT-08.SPEC-007 (Reminder Scheduling & Timing Window Enforcement) | References (inbound) | Supplies the daytime-hours pattern this notification's quiet-hours behavior follows |
| FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | References (inbound) | Governs retry and fallback behavior for this send |
| FEAT-08.SPEC-012 (Transactional Text Messaging Capability) | References (inbound) | Delivers the text channel |
| FEAT-08.SPEC-013 (Transactional Email Capability) | References (inbound) | Delivers the email channel |

## Analytics and Success Signals

- **waitlist_expiry_notification_delivered** (channel: text / email; reason: unclaimed_opening / unmatched_range) -- N/A -- no success-metrics.md metric directly measures waitlist expiry communication; retained so the "client is informed rather than left wondering" commitment is observable rather than assumed.
- **waitlist_expiry_notification_delivery_failed** (channel: text / email; reason) -- N/A -- no Stage 2 metric measures this failure path; retained for operational visibility, consistent with the product's own "never silently dropped" pattern for other notifications (XBR-17).

## Acceptance Criteria

**FEAT-20.SPEC-009-AC-01:** Given Riley's Notified entry's claim window lapses unclaimed, when FEAT-20.SPEC-007 expires it, then she receives the unclaimed-opening variant naming {service_name} and {matched_date}.

**FEAT-20.SPEC-009-AC-02:** Given Riley's Requested entry's joined range elapses with no match ever found, when FEAT-20.SPEC-007 expires it, then she receives the unmatched-range variant naming {service_name}, {start_date}, and {end_date}.

**FEAT-20.SPEC-009-AC-03:** Given Riley has active texting consent, when either variant fires, then she receives it by text.

**FEAT-20.SPEC-009-AC-04:** Given Riley does not have active texting consent, when either variant fires, then she receives the corresponding email variant.

**FEAT-20.SPEC-009-AC-05:** Given this notification would otherwise be sent at 11pm in the Pro's timezone, when the send is evaluated, then it is held per ASMP-29's daytime-hours rule and delivered at the next allowed hour.

**FEAT-20.SPEC-009-AC-06:** Given Riley taps "View my waitlists" on the unclaimed-opening variant, then she is routed to FEAT-20.SPEC-002.

**FEAT-20.SPEC-009-AC-07:** Given Riley taps "Join again" on the unmatched-range variant, then she is routed to FEAT-05.SPEC-001.

**FEAT-20.SPEC-009-AC-08:** Given the text send fails, when the retry-and-fallback rule processes it, then it retries per platform parameter: `message-delivery-retry-count` and falls back to email if the retry also fails.

**FEAT-20.SPEC-009-AC-09:** Given both channels ultimately fail, when no further attempt is possible, then Riley's entry still shows "Expired" on FEAT-20.SPEC-002 for one visit, even though this notification never arrived.

**FEAT-20.SPEC-009-AC-10:** Given Riley revokes texting consent between her entry's expiry and this notification's send, when the send executes, then the channel-selection rule routes it to email.

**FEAT-20.SPEC-009-AC-11:** Given Riley rejoins the waitlist for the same service before this notification is delivered, when the send proceeds, then it still describes the entry that actually expired, unaffected by the new, independent join.

**FEAT-20.SPEC-009-AC-12:** Given an entry was Notified and its claim lapsed, when this notification fires, then only the unclaimed-opening variant is sent -- the unmatched-range variant never applies to a Notified entry.

**FEAT-20.SPEC-009-AC-13:** Given a fresh My Waitlists access link cannot be issued at send time, when the unclaimed-opening variant's CTA is rendered, then it routes into FEAT-06.SPEC-001 to request one rather than omitting the link.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (text, email) | 2 |
| Trigger Paths | 2 (unclaimed opening, unmatched range) | 2 |
| Preference States | 2 (consent granted, consent revoked) | 2 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |


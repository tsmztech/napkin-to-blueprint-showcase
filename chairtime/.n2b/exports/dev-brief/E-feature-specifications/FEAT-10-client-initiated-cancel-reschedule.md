# FEAT-10 — Client-Initiated Cancel/Reschedule

This chapter covers Client-Initiated Cancel/Reschedule (FEAT-10), a Core-tier feature. It carries 6 specifications carrying 91 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-10.SPEC-001 | Cancel Booking | screen | 17 |
| FEAT-10.SPEC-002 | Reschedule -- Select New Time | screen | 15 |
| FEAT-10.SPEC-003 | Reschedule -- Outcome & Confirm | screen | 17 |
| FEAT-10.SPEC-004 | Booking Update Commit | automation | 16 |
| FEAT-10.SPEC-005 | Cancellation Window & Eligibility Rule | logic-rule | 14 |
| FEAT-10.SPEC-006 | Cancellation/Reschedule Notification | notification | 12 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Client-Initiated Cancel/Reschedule

## Summary

**Feature:** Client-Initiated Cancel/Reschedule
**ID:** FEAT-10
**Description:** A client can cancel or reschedule their own booking, within the Pro's stated policy, without a phone call or a DM -- from the reminder's one-tap option or the client's own booking-management link.
**Priority:** Core
**Phase:** MVP
**Type:** User-Facing
**Rationale:** BRIEF.md's Target Users & Roles states the Client "reschedules or cancels within the policy window" as a defined capability, and the Vision's reminder flow ("I need to reschedule") depends on it existing. Research-informed market validation adds that self-service client booking meaningfully reduces phone and DM interruptions during the workday (Capterra/SoftwareAdvice-aggregated reviews, MEDIUM confidence).

**Key Capabilities:**
- Cancel an upcoming booking and see the deposit outcome before confirming
- Reschedule to a new genuinely free time for the same service, without a new deposit charge if within policy
- See the applicable cancellation window countdown before acting

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-10.SPEC-001 | Cancel Booking | Screen | Client | Client views the cancellation window countdown and deposit outcome for their own upcoming booking and confirms or backs out of cancelling it |
| FEAT-10.SPEC-002 | Reschedule -- Select New Time | Screen | Client | Client picks a new, genuinely free time for the same service from the same live slot list a fresh booking would use |
| FEAT-10.SPEC-003 | Reschedule -- Outcome & Confirm | Screen | Client | Client sees the deposit outcome for the chosen new time (carried-over deposit, or late-reschedule deposit-kept-plus-new-deposit-needed) before confirming |
| FEAT-10.SPEC-004 | Booking Update Commit | Automation | Client, Pro, Support | Commits the client's cancellation or reschedule to the booking, coordinating the deposit outcome, calendar mirroring, activity logging, and freed-slot handoff this triggers in other features |
| FEAT-10.SPEC-005 | Cancellation Window & Eligibility Rule | Logic/Rule | Client | Governs whether a booking is currently eligible to be cancelled or rescheduled by its client and computes the window countdown that determines which deposit-outcome branch applies |
| FEAT-10.SPEC-006 | Cancellation/Reschedule Notification | Notification | Client, Pro | Sends the client a cancellation or reschedule confirmation and the Pro a change notice once the update commits |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Cancel an upcoming booking and see the deposit outcome before confirming | FEAT-10.SPEC-001 | Primary purpose of the Cancel Booking screen, using SPEC-005's window calculation and FEAT-09's outcome computation for the preview | Phase 2 (Explicit) |
| Reschedule to a new genuinely free time for the same service, without a new deposit charge if within policy | FEAT-10.SPEC-002, FEAT-10.SPEC-003 | Select New Time applies the same slot validation as a new booking (FEAT-03); Outcome & Confirm shows the carry-over (outside window) or new-deposit-needed (inside window) result before committing | Phase 2 (Explicit) |
| See the applicable cancellation window countdown before acting | FEAT-10.SPEC-001, FEAT-10.SPEC-003, FEAT-10.SPEC-005 | Countdown is computed by the eligibility rule and surfaced on both the Cancel screen and the Reschedule outcome screen | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-10.SPEC-004 | Booking Update Commit | Phase 4 (Trigger-Response) | Confirming a cancel or reschedule triggers cross-entity, cross-feature processing (deposit outcome, calendar mirror, activity log, freed-slot/waitlist handoff, notification) that exceeds a simple inline data write -- the Booking entity's High contention and reject-with-refresh resolution (dependency map) also requires a dedicated processing spec |
| FEAT-10.SPEC-005 | Cancellation Window & Eligibility Rule | Phase 5 (Rule Discovery) | The Validation & Limits field ("a booking already marked completed or no-show cannot be cancelled or rescheduled") and the States field's window-countdown behavior are rules shared across all three screens -- a rule referenced by multiple specs crosses the standalone threshold |
| FEAT-10.SPEC-006 | Cancellation/Reschedule Notification | Phase 4 (Notification surfacing) | The Communications field names a two-audience message (client confirmation, Pro change notice) sent through FEAT-08's messaging mechanism with real audience and content rules -- not a bare success toast |

## Entity-Lifecycle Coverage Matrix

**Entity: Booking**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-10.SPEC-004 (late reschedule only) | Bookings are otherwise created by FEAT-05, FEAT-21, and FEAT-30. FEAT-10.SPEC-004 creates one new Booking record, only on an inside-window (late) reschedule, for the chosen new time (same service, duration, and client, freshly acknowledged policy version, flagged as requiring its own deposit via FEAT-07); cancellations and outside-window reschedules create nothing | Created atomically with the original's move to Rescheduled; an unpaid new Booking follows FEAT-03's standard hold/expiry rules (XBR-02) |
| Read (single) | FEAT-10.SPEC-001, FEAT-10.SPEC-002, FEAT-10.SPEC-003 | Each screen loads the one booking the client is acting on (already identified via FEAT-06's access link) to display its current time, service, and policy | -- |
| Read (list) | N/A | Browsing multiple bookings is FEAT-06's My Bookings List; this feature always acts on one already-identified booking | -- |
| Update | FEAT-10.SPEC-004 | Writes the cancellation state (cancel), the new start_time in place on the same record (outside-window reschedule), or the Rescheduled state on the original record (late reschedule), plus the cancellation/reschedule timestamp, once the client confirms | -- |
| Delete/Archive | N/A | Bookings are never deleted by this feature or any feature -- kept for the life of the account per SC-22; a cancelled or rescheduled booking remains as history, never removed. Recorded as an explicit non-goal below rather than a silent gap | -- |
| State Transition | FEAT-10.SPEC-004 | Confirmed/Awaiting Outcome -> Cancelled by Client (cancel); Confirmed/Awaiting Outcome -> Rescheduled (terminal) on the original Booking for an inside-window (late) reschedule, with a new Booking created at the new time; an outside-window reschedule updates start_time in place with the state unchanged. All governed by FEAT-10.SPEC-005's eligibility gate and the Booking entity's reject-with-refresh contention rule (dependency map) | Resolves the earlier flagged discrepancy to match FEAT-10.SPEC-004: the late reschedule is a compound write (original to Rescheduled plus new Booking) that succeeds or fails as one unit. The dependency map's Booking Creators line still omits FEAT-10 (map-level delta carried to Stage 4, SG-01) |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Deposit Transaction | FEAT-10.SPEC-001, FEAT-10.SPEC-003 | Read for the deposit outcome preview shown before the client confirms; the outcome itself is derived and applied by FEAT-09, never computed by this feature |
| Cancellation Policy | FEAT-10.SPEC-001, FEAT-10.SPEC-003, FEAT-10.SPEC-005 | Read for the window_hours value (countdown calculation) and the plain-language wording shown to the client; this feature never edits the policy |
| Access Link | FEAT-10.SPEC-001, FEAT-10.SPEC-002, FEAT-10.SPEC-003 | The client's identity and booking scope are already validated by FEAT-06 before any of this feature's screens load; this feature performs no independent link validation |
| Availability Rule | FEAT-10.SPEC-002 | Read indirectly through FEAT-03's live slot computation when re-checking availability for a reschedule -- this feature never reads the rule directly |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Client opens Cancel Booking or Reschedule Outcome & Confirm | Compute the cancellation window countdown and eligibility (booking not Completed/No-Show) | Standalone Logic/Rule | FEAT-10.SPEC-005 |
| Client opens Cancel Booking or Reschedule Outcome & Confirm | Read the current deposit outcome preview from FEAT-09 for display | Cross-feature -- logged in touchpoints | FEAT-09 responsibility |
| Client picks a new time on Select New Time | Re-validate the chosen slot against live availability (full duration plus buffer, no conflicts) | Cross-feature -- logged in touchpoints | FEAT-03 responsibility (XBR-01) |
| Client's first-choice new time disappears before they confirm it | Show a plain "that time was just taken" message and keep the client on the same live slot list | Inline in triggering screen | FEAT-10.SPEC-002 |
| Client confirms a reschedule outside the cancellation window | Deposit carries over to the new time; no new deposit charge | Inline in triggering screen (uses SPEC-005's window result) | FEAT-10.SPEC-003 |
| Client confirms a reschedule inside the cancellation window | Show plainly that the original deposit is kept and a new deposit is needed for the new time, before confirming | Inline in triggering screen, then handed to FEAT-07 for the new deposit charge | FEAT-10.SPEC-003 |
| Client backs out of a late-reschedule confirmation | Nothing changes; client returns to slot selection | Inline in triggering screen | FEAT-10.SPEC-003 |
| Client confirms cancel or reschedule | Commit the state transition on Booking; for a late reschedule, move the original to Rescheduled and create a new Booking for the new time as one atomic write | Standalone Automation | FEAT-10.SPEC-004 |
| Booking Update Commit succeeds | Apply the deposit outcome (refund, forfeiture, or late-reschedule new-deposit determination) | Cross-feature -- logged in touchpoints | FEAT-09 responsibility (XBR-09) |
| Booking Update Commit succeeds | Move or remove the entry on the Pro's personal calendar | Cross-feature -- logged in touchpoints | FEAT-04 responsibility (XBR-13) |
| Booking Update Commit succeeds | Write an append-only activity event | Cross-feature -- logged in touchpoints | FEAT-16 responsibility (XBR-21) |
| Booking Update Commit succeeds (cancellation) | Freed slot becomes publicly bookable immediately; matching waitlisted clients notified first with a 30-minute priority window | Cross-feature -- logged in touchpoints | FEAT-20 responsibility (XBR-28) |
| Booking Update Commit succeeds | Trigger the client confirmation and the Pro's change notice | Standalone Notification | FEAT-10.SPEC-006 |
| Booking Update Commit fails to save | Original booking remains untouched and intact; client sees an error and can retry | Standalone Automation (failure handling) | FEAT-10.SPEC-004 |
| A Pro-side action (FEAT-30) commits a conflicting transition first | Reject-with-refresh: the client is shown the booking's current state and must re-decide; the two transitions are never merged | Standalone Automation (failure handling) | FEAT-10.SPEC-004 |
| Client attempts to act on a booking already Completed or No-Show | Action is blocked; client sees a plain ineligibility message | Standalone Logic/Rule | FEAT-10.SPEC-005 |
| Client attempts any action while offline or connectivity drops | Plain message that connectivity is required; nothing is submitted | Inline in triggering screen | FEAT-10.SPEC-001 / SPEC-002 / SPEC-003 |

## Shared Context

**Shared Entities:**
- Booking -- read by SPEC-001, SPEC-002, and SPEC-003; updated by SPEC-004, which also creates a new Booking on a late reschedule (original goes to Rescheduled). Fields in scope here: state, start_time, policy_version, cancellation/reschedule timestamps.
- Deposit Transaction (read-only) -- read by SPEC-001 and SPEC-003 for the outcome preview; the authoritative outcome value and its application are owned by FEAT-09.
- Cancellation Policy (read-only) -- read by SPEC-001, SPEC-003, and SPEC-005 for window_hours and plain_language_wording.
- Access Link (read-only, consumed not managed) -- the client's identity and booking scope arrive pre-validated from FEAT-06 into every screen in this feature.

**Shared UI Patterns:**
- "See the outcome before confirming" pattern -- SPEC-001 (cancellation) and SPEC-003 (late reschedule) both show the deposit consequence plainly, with an explicit confirm step and a no-penalty way to back out with nothing changed. Spec Writers for both should keep this pattern's plainness and ordering (outcome shown, then confirm) consistent, per the feature's Alternate flows.
- Live slot list reuse -- SPEC-002 presents the identical real-time slot list mechanism a fresh booking uses (FEAT-03), including the "just taken" recovery message, so a client never sees a stale or misleading option.

**Shared Validation:**
- FEAT-10.SPEC-005 (Cancellation Window & Eligibility Rule) is referenced, not duplicated, by SPEC-001 (cancel eligibility and countdown), SPEC-002 (eligibility gate before offering reschedule), and SPEC-003 (which deposit-outcome branch applies).

## Internal Dependency Map

```
SPEC-001 (Cancel Booking) -> [checks eligibility and countdown using] -> SPEC-005 (Cancellation Window & Eligibility Rule)
SPEC-001 (Cancel Booking) -> [client confirms cancellation] -> SPEC-004 (Booking Update Commit)
SPEC-002 (Reschedule -- Select New Time) -> [checks eligibility using] -> SPEC-005 (Cancellation Window & Eligibility Rule)
SPEC-002 (Reschedule -- Select New Time) -> [client picks a new time] -> SPEC-003 (Reschedule -- Outcome & Confirm)
SPEC-003 (Reschedule -- Outcome & Confirm) -> [determines outcome branch using] -> SPEC-005 (Cancellation Window & Eligibility Rule)
SPEC-003 (Reschedule -- Outcome & Confirm) -> [client confirms reschedule] -> SPEC-004 (Booking Update Commit)
SPEC-003 (Reschedule -- Outcome & Confirm) -> [client backs out] -> SPEC-002 (Reschedule -- Select New Time) [nothing changed]
SPEC-004 (Booking Update Commit) -> [commit succeeds] -> SPEC-006 (Cancellation/Reschedule Notification)
SPEC-004 (Booking Update Commit) -> [commit fails or a conflicting transition wins] -> SPEC-001 (Cancel Booking) / SPEC-003 (Reschedule -- Outcome & Confirm) [original or current state re-shown]
```

**Default Entry:** This feature has no single default landing screen -- the client arrives already viewing one specific booking (via FEAT-06.SPEC-004, Booking Detail) and chooses Cancel (routes to SPEC-001) or Reschedule (routes to SPEC-002); a reminder's one-tap "I need to reschedule" option (FEAT-08) routes directly into SPEC-002, bypassing the choice.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-10.SPEC-001 / SPEC-002 | Inbound | FEAT-06 (Client Booking Identity) | Client, already viewing their own booking, chooses to cancel or reschedule it | Client chooses cancel or reschedule on Booking Detail |
| FEAT-10.SPEC-002 | Inbound | FEAT-08 (Automated Booking Messaging) | Client taps the reminder's "I need to reschedule" one-tap option | Client taps reminder action |
| FEAT-10.SPEC-002 | Outbound | FEAT-03 (Real-Time Slot Availability Engine) | Reschedule requests and re-validates the live slot list for the same service | Client opens Select New Time / confirms a slot |
| FEAT-10.SPEC-001 / SPEC-003 / SPEC-005 | Outbound | FEAT-09 (Cancellation & No-Show Policy Engine) | Reads the cancellation policy window and the derived deposit outcome for preview and eligibility | Client opens Cancel or Reschedule Outcome & Confirm |
| FEAT-10.SPEC-004 | Outbound | FEAT-09 (Cancellation & No-Show Policy Engine) | Hands off deposit outcome application (refund, forfeiture, or late-reschedule new-deposit determination) | Booking update commits |
| FEAT-10.SPEC-004 | Outbound | FEAT-04 (Two-Way Calendar Sync) | Commit causes the Pro's personal calendar entry to move or be removed | Booking cancelled or rescheduled |
| FEAT-10.SPEC-004 | Outbound | FEAT-16 (Booking & Payment Activity Record) | Commit writes an append-only activity event | Booking cancelled or rescheduled |
| FEAT-10.SPEC-004 | Outbound | FEAT-20 (Waitlist for Cancelled Slots) | A cancellation frees the slot and triggers waitlist priority notification before general availability | Booking cancelled |
| FEAT-10.SPEC-003 / SPEC-004 | Outbound | FEAT-07 (Deposit Payment at Booking) | A late reschedule (inside the window) creates a new Booking (SPEC-004) that requires a new deposit charge for the new time, handled by the standard deposit-payment mechanism | Client confirms a late reschedule |
| FEAT-10.SPEC-006 | Outbound | FEAT-08 (Automated Booking Messaging) | Notification content is delivered through FEAT-08's transactional text/email messaging capability, which owns the delivery contract | Booking Update Commit succeeds |

## Non-Functional Notes

**Data volumes / growth:** Cancel/reschedule volume is a subset of overall booking traffic -- a few hundred Pros in year one, each with roughly 100-500 clients and 20-40 bookings a week (scope-boundaries SC-19). The Self-Service Reschedule Rate success metric targets at least 85% of client cancellations/reschedules completing entirely through this feature, so it should be expected to carry a meaningful share of weekly booking-adjacent traffic per Pro, not an edge case.

**Responsiveness:** Re-checking live availability for a reschedule follows the product-wide slot-search responsiveness bar -- available slots appear within roughly one second of selection and the list updates within roughly one second of a slot being taken (Slot Search Responsiveness success metric; ASMP-21). A full booking-equivalent action here should complete without a perceptible wait once the client confirms, consistent with ASMP-21's under-one-minute booking benchmark.

**Data sensitivity / privacy:** The booking, deposit outcome, and policy data this feature displays are personal and financial data linked to an identifiable client, visible only to that client and their one Pro (ASMP-23); Platform Operator (Support) has view-only access to the resulting state and never uses or bypasses the client's own access link to reach it.

**Compliance flags:** N/A -- this feature applies no compliance regime of its own; the money movement it triggers (refund, forfeiture, or a new deposit charge) is a category-level payment-processing capability (ASMP-31) whose contract and any related compliance handling belong to FEAT-07 and FEAT-09, not to this feature.

**Signals:** This feature emits booking_cancelled_by_client and booking_rescheduled_by_client on Booking Update Commit (FEAT-10.SPEC-004), and late_reschedule_warning_shown when the Outcome & Confirm screen (FEAT-10.SPEC-003) displays the inside-window, new-deposit-needed warning -- these three signals are the analytics basis for the Self-Service Reschedule Rate success metric.

## Non-Goals

- **Pro-initiated cancellation or reschedule** -- Excluded per the feature entry's own MODIFIED note: the Pro's own cancel/reschedule actions were moved to Pro Booking Management (FEAT-30) so client-side and Pro-side deposit rules are each owned by one feature; this feature covers only the client-initiated path.
- **Partial refunds or tiered cancellation schedules** -- Excluded per scope-boundaries SC-18: BRIEF.md's Business Context defines a binary deposit rule (kept inside the window, refunded outside it); this feature's outcome preview and commit never expose a percentage-based or tiered outcome.
- **Charging a client's card later, or keeping one on file, for a cancellation fee** -- Excluded per scope-boundaries SC-13: the product protects the Pro with a deposit paid up front; when a late reschedule needs a new deposit, that charge goes through the same standard deposit-payment mechanism (FEAT-07) as any other booking, never a stored-card charge made after the fact.
- **Automatic purge or deletion of cancelled/rescheduled booking history** -- Intentional lifecycle decision surfaced by the CRUD matrix: bookings are retained for the life of the account per SC-22, so a cancelled or rescheduled booking is never deleted, only left as history with an updated state.
- **Support acting on a client's or Pro's behalf to cancel or reschedule** -- Excluded per scope-boundaries SC-05 and the Access Matrix: Platform Operator (Support) has view-only access to the outcome and never performs the cancellation or reschedule itself, even to help resolve a support request.



# Screen Spec: Cancel Booking

## Overview

**Name:** Cancel Booking
**ID:** FEAT-10.SPEC-001
**Type:** Screen
**Purpose:** Client views the cancellation window countdown and deposit outcome for their own upcoming booking and confirms or backs out of cancelling it.
**Parent Feature:** FEAT-10 -- Client-Initiated Cancel/Reschedule

## Scope and Non-Goals

**In Scope:**
- The default landing screen when a client chooses to act on a booking from FEAT-06.SPEC-004 ("Cancel or Reschedule")
- Showing the cancellation window countdown and the deposit outcome preview (refund vs. kept) before the client commits to cancelling
- The explicit confirm step and a no-penalty way to back out with nothing changed
- Offering the reschedule path as an alternative to cancelling, without leaving the client stuck choosing wrong
- Blocking the action entirely, with a plain message, when the booking is no longer eligible (already Completed or No-Show)

**Non-Goals:**
- Computing the cancellation window countdown and eligibility itself -- owned by FEAT-10.SPEC-005 (Cancellation Window & Eligibility Rule); this screen only displays what that rule returns
- Deriving the deposit outcome value -- owned by FEAT-09 (Cancellation & No-Show Policy Engine); this screen reads and displays FEAT-09.SPEC-003's rule result, never computes it
- Committing the cancellation to the Booking record -- owned by FEAT-10.SPEC-004 (Booking Update Commit), which this screen triggers but does not implement
- Selecting a new time -- owned by FEAT-10.SPEC-002 (Reschedule -- Select New Time), reached via this screen's "Reschedule instead" link

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-06.SPEC-004 (Booking Detail via Manage Link) | Client taps "Cancel or Reschedule" | The selected Booking's reference |
| FEAT-10.SPEC-004 (Booking Update Commit) | Cancellation fails to save, or a conflicting Pro-side transition wins first | Same Booking reference, refreshed to its current state; an error or conflict message |
| FEAT-10.SPEC-003 (Reschedule -- Outcome & Confirm) | Client backs out of a late-reschedule confirmation and instead chooses to cancel from that screen's "or cancel instead" link | Same Booking reference |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Full screen, for a booking that belongs to their own matched Client record with this Pro | Confirm cancel, back out, navigate to reschedule instead | -- |
| The Pro (Talia) | No | No | This is not the Pro's own change surface; the Pro's own cancel/reschedule actions are through Pro Booking Management (FEAT-30), never through this screen |
| Platform Operator (Support) | No | No | Support access never uses or bypasses a client's access link (scope-boundaries SC-05); no support entry point exists here |
| Unauthenticated | No | No | Reachable only via FEAT-06.SPEC-004's already-authenticated navigation; a direct, unauthenticated attempt is redirected to FEAT-06.SPEC-001 |
| Expired session | No | No | The underlying access link governs the viewing session (FEAT-06); once it has transitioned to Used or Expired, reloading this screen is treated as unauthenticated and redirected to FEAT-06.SPEC-001 with the "request a new link" prompt |

## Layout and Content

**Header:** Screen title "Cancel Booking" with a back arrow (returns to FEAT-06.SPEC-004).

**Body:**
- Appointment summary: service name, date, time, duration
- Cancellation window countdown: plain-language statement of how much time remains before the booking enters the cancellation window (or that it has already passed), computed by FEAT-10.SPEC-005
- Deposit outcome preview: a plain statement of what happens to the deposit if the client cancels right now -- "Your deposit will be refunded" (outside the window) or "Your deposit will be kept, per the cancellation policy you agreed to" (inside the window), derived from FEAT-09.SPEC-003
- Policy wording: the exact plain-language cancellation policy text acknowledged at booking (FEAT-09.SPEC-002)
- "Cancel Booking" button (primary action)
- "Reschedule instead" link, below the Cancel button
- "Keep my booking" link, returning to FEAT-06.SPEC-004 with nothing changed

Appointment summary, cancellation window countdown, deposit outcome preview, and policy wording are display-only text with no interaction of their own; only the back arrow, "Cancel Booking," "Reschedule instead," and "Keep my booking" are interactive elements on this screen.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Sections stack vertically in the order given above, full width; "Cancel Booking" and "Reschedule instead" are full-width, stacked.
- **Medium size class and above:** Same vertical section order, content column capped at a consistent platform-wide reading width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-06.SPEC-004 | Screen closes | Standard navigation transition |
| "Cancel Booking" button | Tap | Opens the confirm dialog showing the same deposit outcome preview one more time | Dialog appears | Dialog title "Cancel this booking?" with the deposit outcome restated, and "Confirm Cancellation" / "Keep Booking" options |
| "Confirm Cancellation" (in dialog) | Tap | Triggers FEAT-10.SPEC-004 (Booking Update Commit) for a cancellation | Button shows loading state; dialog stays open during commit | Success: navigate to FEAT-06.SPEC-004, showing the booking's now-Cancelled state. Failure: dialog shows the error and a Retry option (see States, Error) |
| "Keep Booking" (in dialog) | Tap | Closes the dialog; nothing changes | Dialog closes | Client returns to this screen exactly as before |
| "Reschedule instead" link | Tap | Navigate to FEAT-10.SPEC-002 (Reschedule -- Select New Time) for this Booking | Screen transitions | Standard navigation transition |
| "Keep my booking" link | Tap | Navigate to FEAT-06.SPEC-004 | Screen closes | Standard navigation transition |
| "Confirm Cancellation" (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> appointment summary -> cancellation window countdown -> deposit outcome preview -> policy wording -> "Cancel Booking" -> "Reschedule instead" -> "Keep my booking".
- **Dynamic announcements:** The confirm dialog's appearance and its deposit outcome restatement are announced to assistive technology when it opens; the concurrent-edit conflict message (see Edge Cases) is announced as soon as it appears; a commit failure's error text is announced and focus moves to the Retry action.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | A brief in-place loading indicator where the countdown and outcome preview will appear | Screen first opens | Data (booking, window, outcome preview) finishes loading |
| Populated | Full countdown, outcome preview, and policy wording shown as described in Layout and Content | Data loads successfully and the booking is eligible | Client navigates away or taps Cancel Booking |
| Ineligible | Plain message "This booking can no longer be cancelled or rescheduled." replaces the Cancel/Reschedule actions; appointment summary and status remain visible | FEAT-10.SPEC-005 reports the booking as ineligible (state is Completed, No-Show, Cancelled by Client, Cancelled by Pro, Rescheduled, Pending Payment, or Expired (unpaid) -- the full ineligibility scope defined by FEAT-10.SPEC-005 AC-05 and AC-14) | Client navigates back to FEAT-06.SPEC-004 (no path forward on this screen) |
| Confirming | Confirm dialog open, showing the restated outcome | Client taps "Cancel Booking" | Client taps "Confirm Cancellation" or "Keep Booking" |
| Cancelling | "Confirm Cancellation" button shows a loading state; dialog remains open and non-dismissible | Client taps "Confirm Cancellation" | Commit succeeds or fails |
| Error | Dialog shows "We couldn't cancel this booking. Try again." with a Retry action; the original booking remains untouched and intact | FEAT-10.SPEC-004 reports the commit failed to save | Client taps Retry and the commit succeeds, or navigates away |
| Load Error | Error banner "We couldn't load this booking. Try again." with a retry action | The initial data load fails | Client taps Retry and the load succeeds |
| Offline/Degraded | Banner "Connectivity is required to cancel or reschedule a booking." appears; the countdown and outcome preview (if already loaded) remain visible read-only; "Cancel Booking" and "Reschedule instead" are disabled | Connectivity is lost while this screen is open, or the screen is opened without connectivity | Connectivity is restored -- the banner clears and actions re-enable |

## Validation Rules

Validation governed by FEAT-10.SPEC-005 (Cancellation Window & Eligibility Rule). See that spec for the eligibility gate this screen enforces before offering the Cancel action.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-06.SPEC-004 (Booking Detail via Manage Link) | FEAT-06 |
| Successful cancellation | FEAT-06.SPEC-004 (Booking Detail via Manage Link) | FEAT-06 |
| "Reschedule instead" tap | FEAT-10.SPEC-002 (Reschedule -- Select New Time) | -- |
| "Keep my booking" tap | FEAT-06.SPEC-004 (Booking Detail via Manage Link) | FEAT-06 |

## Data Model

**Creates:** None.
**Reads:** Booking -- service, start_time, duration, state, policy_version, scoped to the matched Client with this Pro (FEAT-06.SPEC-008). Deposit Transaction -- amount, for the outcome preview. Cancellation Policy -- window_hours and plain_language_wording, via the bound version (FEAT-09.SPEC-002).
**Updates:** None directly -- the cancellation itself is performed by FEAT-10.SPEC-004, which this screen triggers.
**Deletes:** None.

## Business Rules

- FEAT-10.SPEC-005 governs whether this booking is currently eligible to be cancelled and computes the window countdown shown here; this screen enforces its result but never re-derives it.
- The deposit outcome preview reflects FEAT-09.SPEC-003's Rule 1 (outside window: Refund Due) or Rule 2 (inside window: Forfeiture Flagged), read live at the moment this screen loads.
- XBR-12: a Completed or No-Show booking can never be cancelled from this screen; the Ineligible state applies instead.
- "See the outcome before confirming" pattern: this screen shows the deposit consequence plainly, with an explicit confirm step and a no-penalty way to back out with nothing changed, consistent with FEAT-10.SPEC-003's identical pattern for a late reschedule.

## Edge Cases

- **The Pro cancels, reschedules, or marks this booking no-show while Riley is viewing this screen** -- Per the Booking entity's Contention resolution (reject-with-refresh, feature-dependency-map.md), if Riley then taps "Confirm Cancellation," the commit is rejected because the Pro's transition already committed first; Riley sees the dialog "This booking's details changed. Refresh to see the latest before continuing." with a "Refresh" action that reloads the booking's current state -- her original tap is never carried through against stale data.
- **Riley taps "Cancel Booking" twice rapidly** -- The second tap is ignored while the confirm dialog is already open.
- **Riley taps "Confirm Cancellation" twice rapidly** -- The second tap is ignored while the first commit is in progress (button in loading state).
- **The countdown crosses from outside to inside the window while this screen is open** -- The screen is a snapshot view; the countdown and outcome preview do not silently update mid-view. The values shown reflect the moment the screen loaded; the values used at commit time are recomputed fresh by FEAT-10.SPEC-004/FEAT-10.SPEC-005 at the instant of the confirm tap, so the actual outcome applied is always current even if the displayed preview was taken moments earlier. If the recomputed outcome at commit time differs from what was shown, the confirm dialog is re-shown once with the updated outcome before the commit proceeds, rather than silently applying a different outcome than the client just confirmed.
- **Riley navigates away with the confirm dialog open and returns later** -- The dialog does not persist; the screen reloads fresh data as on any new visit.
- **Riley loses connectivity while the confirm dialog is open** -- The dialog closes and the Offline/Degraded banner appears; no cancellation is submitted.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-004 (Booking Detail via Manage Link) | Navigation (inbound/outbound) | Entry point into this screen and the destination on back/cancel/keep |
| FEAT-10.SPEC-005 (Cancellation Window & Eligibility Rule) | References (inbound) | Supplies the window countdown and eligibility gate |
| FEAT-09.SPEC-003 (Deposit Outcome Rules) | References (inbound) | Supplies the deposit outcome preview |
| FEAT-09.SPEC-002 (Policy Versioning & Cutoff Rendering) | References (inbound) | Supplies the rendered cutoff time and plain-language wording |
| FEAT-10.SPEC-004 (Booking Update Commit) | Triggers (outbound) | "Confirm Cancellation" triggers the commit |
| FEAT-10.SPEC-002 (Reschedule -- Select New Time) | Navigation (outbound) | "Reschedule instead" starts that flow |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| cancel_screen_viewed | window_state (outside/inside), outcome_preview (refund/kept) | Screen finishes loading with data | supports success-metrics.md: "Self-Service Reschedule Rate" |
| cancel_confirmed | window_state, outcome | Client confirms the cancellation and the commit succeeds | supports success-metrics.md: "Self-Service Reschedule Rate" |
| cancel_abandoned | reason (kept_booking / rescheduled_instead / navigated_away) | Client leaves this screen without cancelling | supports success-metrics.md: "Self-Service Reschedule Rate" |
| cancel_screen_conflict_shown | -- | Riley's confirm is rejected because a Pro-side change committed first | supports success-metrics.md: "Self-Service Reschedule Rate" |

## Acceptance Criteria

**FEAT-10.SPEC-001-AC-01:** Given Riley opens Cancel Booking for an upcoming booking outside the cancellation window, when the screen loads, then she sees the countdown, "Your deposit will be refunded" as the outcome preview, and the acknowledged policy wording.

**FEAT-10.SPEC-001-AC-02:** Given Riley opens Cancel Booking for a booking inside the cancellation window, when the screen loads, then she sees "Your deposit will be kept, per the cancellation policy you agreed to" as the outcome preview.

**FEAT-10.SPEC-001-AC-03:** Given Riley taps "Cancel Booking," when the confirm dialog opens, then it restates the same deposit outcome she saw on the screen.

**FEAT-10.SPEC-001-AC-04:** Given Riley taps "Confirm Cancellation" in the dialog, when the commit succeeds, then FEAT-10.SPEC-004 records the cancellation and she is returned to FEAT-06.SPEC-004 showing the booking as Cancelled.

**FEAT-10.SPEC-001-AC-05:** Given Riley taps "Keep Booking" in the dialog, when the tap registers, then the dialog closes and nothing about the booking changes.

**FEAT-10.SPEC-001-AC-06:** Given Riley taps "Reschedule instead," when the tap registers, then she is taken to FEAT-10.SPEC-002 for the same booking.

**FEAT-10.SPEC-001-AC-07:** Given Riley opens Cancel Booking for a booking already marked Completed, when the screen loads, then it shows the Ineligible state with the message "This booking can no longer be cancelled or rescheduled." and no Cancel or Reschedule action is offered.

**FEAT-10.SPEC-001-AC-08:** Given Riley opens Cancel Booking for a booking already marked No-Show, when the screen loads, then it shows the same Ineligible state.

**FEAT-10.SPEC-001-AC-09:** Given the initial data load for this screen fails, when the failure occurs, then the error banner "We couldn't load this booking. Try again." appears with a retry action.

**FEAT-10.SPEC-001-AC-10:** Given Riley taps "Confirm Cancellation" and the commit fails to save, when the failure occurs, then the dialog shows "We couldn't cancel this booking. Try again." with a Retry option, and the original booking remains untouched.

**FEAT-10.SPEC-001-AC-11:** Given Talia cancels this same booking on her side while Riley is viewing this screen, when Riley then taps "Confirm Cancellation," then her commit is rejected with the dialog "This booking's details changed. Refresh to see the latest before continuing." and no stale cancellation is recorded.

**FEAT-10.SPEC-001-AC-12:** Given Riley loses connectivity while viewing this screen, when connectivity drops, then the banner "Connectivity is required to cancel or reschedule a booking." appears and "Cancel Booking" and "Reschedule instead" are disabled.

**FEAT-10.SPEC-001-AC-13:** Given Riley taps "Cancel Booking" twice in rapid succession, when the second tap registers, then it is ignored because the confirm dialog is already open.

**FEAT-10.SPEC-001-AC-14:** Given Riley taps "Confirm Cancellation" twice in rapid succession, when the second tap registers, then it is ignored while the first commit is in progress.

**FEAT-10.SPEC-001-AC-15:** Given a client without a valid access link attempts to reach this screen directly, when the attempt is made, then they are redirected to FEAT-06.SPEC-001 and shown no booking data.

**FEAT-10.SPEC-001-AC-16:** Given Riley taps the back arrow, when the tap registers, then she returns to FEAT-06.SPEC-004 and nothing about the booking has changed.

**FEAT-10.SPEC-001-AC-17:** Given Riley taps "Keep my booking," when the tap registers, then she returns to FEAT-06.SPEC-004 and nothing about the booking has changed.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 7 | 7 |
| States | 8 (loading, populated, ineligible, confirming, cancelling, error, load error, offline) | 8 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Screen Spec: Reschedule -- Select New Time

## Overview

**Name:** Reschedule -- Select New Time
**ID:** FEAT-10.SPEC-002
**Type:** Screen
**Purpose:** Client picks a new, genuinely free time for the same service from the same live slot list a fresh booking would use.
**Parent Feature:** FEAT-10 -- Client-Initiated Cancel/Reschedule

## Scope and Non-Goals

**In Scope:**
- Requesting and displaying the live slot list for the booking's existing service, using the identical real-time mechanism a fresh booking uses (FEAT-03)
- Letting the client pick a new genuinely free time for the same service and same duration
- The eligibility gate before offering a reschedule at all (a Completed or No-Show booking cannot be rescheduled)
- Recovering plainly when the client's first-choice time disappears before they confirm it
- Both entry paths: from FEAT-10.SPEC-001's "Reschedule instead" link, and directly from a reminder's "I need to reschedule" one-tap option (FEAT-08)

**Non-Goals:**
- Computing which times are genuinely free -- owned entirely by FEAT-03 (Real-Time Slot Availability Engine); this screen only displays what FEAT-03 returns and never derives availability itself
- Changing the service or duration being booked -- a reschedule keeps the same service and duration as the original booking; changing service is not offered, since that would functionally be a new booking, out of this feature's scope
- Showing the deposit outcome for the chosen time -- owned by FEAT-10.SPEC-003 (Reschedule -- Outcome & Confirm), the next step
- Computing the cancellation window countdown and eligibility itself -- owned by FEAT-10.SPEC-005 (Cancellation Window & Eligibility Rule); this screen only enforces the eligibility gate it returns

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-10.SPEC-001 (Cancel Booking) | Client taps "Reschedule instead" | The Booking reference, its service and duration |
| FEAT-08 (Automated Booking Messaging) | Client taps "I need to reschedule" on a reminder | The Booking reference, its service and duration -- bypasses FEAT-10.SPEC-001 entirely |
| FEAT-10.SPEC-003 (Reschedule -- Outcome & Confirm) | Client backs out of a late-reschedule confirmation | Same Booking reference; slot list refreshed |
| FEAT-10.SPEC-004 (Booking Update Commit) | Reschedule fails to save, or a conflicting Pro-side transition wins first | Same Booking reference, refreshed to its current state; an error or conflict message |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Full screen, for a booking that belongs to their own matched Client record with this Pro | Select any genuinely free time shown for the same service | -- |
| The Pro (Talia) | No | No | This is not the Pro's own change surface; the Pro's own reschedule is through Pro Booking Management (FEAT-30), never through this screen |
| Platform Operator (Support) | No | No | Support access never uses or bypasses a client's access link (scope-boundaries SC-05); no support entry point exists here |
| Unauthenticated | No | No | Reachable only via FEAT-10.SPEC-001's or FEAT-08's already-authenticated navigation; a direct, unauthenticated attempt is redirected to FEAT-06.SPEC-001 |
| Expired session | No | No | The underlying access link governs the viewing session (FEAT-06); once it has transitioned to Used or Expired, reloading this screen is treated as unauthenticated and redirected to FEAT-06.SPEC-001 with the "request a new link" prompt |

## Layout and Content

**Header:** Back arrow (returns to FEAT-10.SPEC-001 when arrived from there, or to FEAT-06.SPEC-003 My Bookings List when arrived directly from a reminder tap) with the service's name, price, and duration shown as persistent context beneath it, and a note that this is a reschedule of the client's existing booking (its current date and time shown for reference).

**Body:** A live list of available time slots for the booking's service, grouped by day, in chronological order, identical in structure to FEAT-05.SPEC-002's slot list. Each slot is a single tappable time element. If no times are available for the visible range, the list shows a plain message in place of slots (see States, Empty).

The service/price/duration and current-appointment context in the header are display-only text with no interaction of their own; only the back arrow and each time slot are interactive elements on this screen.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Slots list in a single column, grouped by day heading, full width.
- **Medium size class and above:** Slots list may show more times per row (a grid rather than a single column) within the same day grouping; no change to grouping or day-heading structure.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to the entry source (FEAT-10.SPEC-001 or FEAT-06.SPEC-003) | Screen closes | Standard backward transition |
| Time slot | Tap | Re-validates the chosen slot against live availability (full duration plus buffer, no conflicts), per XBR-01, using the same mechanism a fresh booking uses (FEAT-03) | Slot shows a brief "checking availability" loading indicator | On success: navigate to FEAT-10.SPEC-003 (Reschedule -- Outcome & Confirm) with the chosen time. On the slot being lost to contention: plain "That time was just taken" message and refreshed list, slot removed from the list |
| Time slot (while a re-validation attempt is in flight for the same client) | Tap | No action -- debounced | None | Slot remains in its loading indicator state |

### Accessibility Notes

- **Focus order:** Back arrow -> service/current-appointment context -> day groupings in chronological order, each slot in time order within its day.
- **Dynamic announcements:** The "That time was just taken" message is announced to assistive technology as soon as it appears; the Ineligible message (see States) is announced on screen load if it applies.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | A brief in-place loading indicator where the slot list will appear | Screen first opens | Slot list finishes loading |
| Populated | Full slot list shown, grouped by day | Slot list loads with one or more available times | Client taps a slot or navigates away |
| Empty | Plain message "No available times found for this service right now." in place of the slot list | The live slot list returns zero available times for the visible range | A new slot becomes available and the list is refreshed (client-initiated refresh or re-entry) |
| Ineligible | Plain message "This booking can no longer be cancelled or rescheduled." replaces the slot list entirely | FEAT-10.SPEC-005 reports the booking as ineligible (state is Completed, No-Show, Cancelled by Client, Cancelled by Pro, Rescheduled, Pending Payment, or Expired (unpaid) -- the full ineligibility scope defined by FEAT-10.SPEC-005 AC-05 and AC-14) | Client navigates back (no path forward on this screen) |
| Re-validating | The tapped slot shows a "checking availability" loading indicator | Client taps a time slot | Re-validation completes (available or just-taken) |
| Load Error | Error banner "We couldn't load available times. Try again." with a retry action | The initial slot list load fails | Client taps Retry and the load succeeds |
| Offline/Degraded | Banner "Connectivity is required to reschedule a booking." appears; any already-loaded slot list remains visible read-only; slot selection is disabled | Connectivity is lost while this screen is open, or the screen is opened without connectivity | Connectivity is restored -- the banner clears and slot selection re-enables |

## Validation Rules

Validation governed by FEAT-10.SPEC-005 (Cancellation Window & Eligibility Rule) for the eligibility gate, and by FEAT-03's slot validation rules (XBR-01, XBR-02, XBR-03) for whether a chosen time is genuinely free, applied identically to a fresh booking's slot selection.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-10.SPEC-001 (Cancel Booking) or FEAT-06.SPEC-003 (My Bookings List), matching the entry source | FEAT-06 (when arrived directly) |
| Slot re-validated successfully | FEAT-10.SPEC-003 (Reschedule -- Outcome & Confirm) | -- |

## Data Model

**Creates:** None.
**Reads:** Booking -- service, duration, start_time (for the "current appointment" reference context), state, scoped to the matched Client with this Pro (FEAT-06.SPEC-008). Availability Rule -- read indirectly through FEAT-03's live slot computation; this screen never reads the rule directly.
**Updates:** None -- the actual reschedule commit is performed by FEAT-10.SPEC-004, reached from FEAT-10.SPEC-003.
**Deletes:** None.

## Business Rules

- FEAT-10.SPEC-005 governs whether this booking is currently eligible to be rescheduled at all; this screen enforces its result before showing any slot list.
- XBR-01: a time is offered only if it passes the live slot check (full duration plus buffer inside an open window, no conflicting booking, block, recurring reservation, or personal-calendar busy time); the first client to complete the reschedule wins a contested slot.
- XBR-03: the Pro's minimum booking notice and booking horizon apply to this reschedule exactly as they would to a fresh booking; the client cannot reschedule inside notice or beyond horizon.
- "Live slot list reuse" pattern: this screen presents the identical real-time slot list mechanism a fresh booking uses (FEAT-03), including the "just taken" recovery message, so the client never sees a stale or misleading option.

## Edge Cases

- **The client's first-choice new time disappears before they confirm it** -- The slot's re-validation reports it is no longer free; the client sees the plain message "That time was just taken" and remains on this same live slot list, refreshed, rather than being sent to an error screen.
- **The Pro cancels, reschedules, or marks this booking no-show while Riley is viewing this screen** -- Per the Booking entity's Contention resolution (reject-with-refresh), if Riley then taps a slot, the re-validation step still succeeds (it only checks the new time's availability), but the subsequent commit at FEAT-10.SPEC-004 is rejected because the Pro's transition already committed first; Riley is returned here (or to FEAT-06.SPEC-004) with the current booking state and the dialog "This booking's details changed. Refresh to see the latest before continuing."
- **Riley taps a slot twice rapidly** -- The second tap is ignored while the first re-validation is in progress (slot in loading indicator state).
- **Riley navigates away and returns** -- The slot list is re-fetched fresh on return; no stale list is shown.
- **No available times exist for the visible range at all (e.g., the Pro is fully booked for the horizon)** -- The Empty state is shown; the client can navigate back and try again later, or cancel instead via FEAT-10.SPEC-001.
- **Riley arrives directly from a reminder's "I need to reschedule" tap and taps back** -- She returns to FEAT-06.SPEC-003 (My Bookings List), since there is no FEAT-10.SPEC-001 in her navigation history for this session.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-10.SPEC-001 (Cancel Booking) | Navigation (inbound/outbound) | "Reschedule instead" link enters here; back arrow returns there |
| FEAT-08 (Automated Booking Messaging) | Navigation (inbound) | Reminder's "I need to reschedule" one-tap option enters here directly |
| FEAT-03.SPEC-001 (Slot Availability Computation) -- within FEAT-03 (Real-Time Slot Availability Engine) | References (outbound) | Supplies the live slot list and re-validates the chosen slot |
| FEAT-10.SPEC-005 (Cancellation Window & Eligibility Rule) | References (inbound) | Gates whether this screen offers a slot list at all |
| FEAT-10.SPEC-003 (Reschedule -- Outcome & Confirm) | Navigation (outbound) | A successfully re-validated slot proceeds here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| reschedule_slot_list_viewed | entry source (cancel_screen / reminder_tap), slot_count | Screen finishes loading with data | supports success-metrics.md: "Self-Service Reschedule Rate" |
| reschedule_slot_selected | -- | Client taps a time slot | supports success-metrics.md: "Self-Service Reschedule Rate" |
| reschedule_slot_lost_to_contention | -- | The chosen slot's re-validation reports it is no longer free | supports success-metrics.md: "Slot Search Responsiveness" |

## Acceptance Criteria

**FEAT-10.SPEC-002-AC-01:** Given Riley taps "Reschedule instead" on FEAT-10.SPEC-001, when this screen loads, then it shows the live slot list for the same service as her existing booking.

**FEAT-10.SPEC-002-AC-02:** Given Riley taps "I need to reschedule" on a reminder, when this screen loads, then it shows the same live slot list directly, bypassing FEAT-10.SPEC-001.

**FEAT-10.SPEC-002-AC-03:** Given Riley taps an available time slot, when re-validation confirms it is still free, then she is taken to FEAT-10.SPEC-003 with that time.

**FEAT-10.SPEC-002-AC-04:** Given Riley taps a time slot that another client books moments earlier, when re-validation runs, then she sees "That time was just taken" and remains on a refreshed slot list.

**FEAT-10.SPEC-002-AC-05:** Given Riley opens this screen for a booking already marked Completed, when the screen loads, then it shows the Ineligible state with "This booking can no longer be cancelled or rescheduled." and no slot list is shown.

**FEAT-10.SPEC-002-AC-06:** Given Riley opens this screen for a booking already marked No-Show, when the screen loads, then it shows the same Ineligible state.

**FEAT-10.SPEC-002-AC-07:** Given the live slot list returns zero available times, when the screen loads, then the Empty state message "No available times found for this service right now." is shown.

**FEAT-10.SPEC-002-AC-08:** Given the initial slot list load fails, when the failure occurs, then the error banner "We couldn't load available times. Try again." appears with a retry action.

**FEAT-10.SPEC-002-AC-09:** Given Talia cancels this same booking on her side while Riley is viewing this screen, when Riley taps a slot and the subsequent commit is attempted at FEAT-10.SPEC-004, then it is rejected with "This booking's details changed. Refresh to see the latest before continuing." rather than silently committing against a stale booking.

**FEAT-10.SPEC-002-AC-10:** Given Riley loses connectivity while viewing this screen, when connectivity drops, then the banner "Connectivity is required to reschedule a booking." appears and slot selection is disabled.

**FEAT-10.SPEC-002-AC-11:** Given Riley taps a time slot twice in rapid succession, when the second tap registers, then it is ignored while the first re-validation is in progress.

**FEAT-10.SPEC-002-AC-12:** Given Riley attempts to select a time inside the Pro's minimum booking notice or beyond the booking horizon, when the live slot list is computed, then that time is never offered, per XBR-03.

**FEAT-10.SPEC-002-AC-13:** Given Riley backs out of the late-reschedule confirmation on FEAT-10.SPEC-003, when she returns here, then the slot list is refreshed and her prior selection is not pre-applied.

**FEAT-10.SPEC-002-AC-14:** Given a client without a valid access link attempts to reach this screen directly, when the attempt is made, then they are redirected to FEAT-06.SPEC-001 and shown no slot data.

**FEAT-10.SPEC-002-AC-15:** Given Riley taps the back arrow, when the tap registers, then she returns to the entry source (FEAT-10.SPEC-001 or FEAT-06.SPEC-003) and nothing about the booking has changed.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 3 | 3 |
| States | 7 (loading, populated, empty, ineligible, re-validating, load error, offline) | 7 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Screen Spec: Reschedule -- Outcome & Confirm

## Overview

**Name:** Reschedule -- Outcome & Confirm
**ID:** FEAT-10.SPEC-003
**Type:** Screen
**Purpose:** Client sees the deposit outcome for the chosen new time (carried-over deposit, or late-reschedule deposit-kept-plus-new-deposit-needed) before confirming.
**Parent Feature:** FEAT-10 -- Client-Initiated Cancel/Reschedule

## Scope and Non-Goals

**In Scope:**
- Showing the deposit outcome for the chosen new time before the client confirms: carried-over deposit (outside the window) or the compound late-reschedule outcome (inside the window)
- The applicable cancellation window countdown, computed against the original booking's start_time
- The explicit confirm step and a no-penalty way to back out with nothing changed
- Committing the reschedule once confirmed

**Non-Goals:**
- Selecting the new time itself -- owned by FEAT-10.SPEC-002 (Reschedule -- Select New Time), the previous step
- Deriving the deposit outcome value -- owned by FEAT-09 (Cancellation & No-Show Policy Engine, FEAT-09.SPEC-003); this screen reads and displays that rule set's result, never computes it
- Collecting the new deposit payment for a late reschedule -- handed off to FEAT-07 (Deposit Payment at Booking), the standard deposit-payment mechanism, once this screen's confirm step completes
- Committing the reschedule to the Booking record -- owned by FEAT-10.SPEC-004 (Booking Update Commit), which this screen triggers but does not implement

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-10.SPEC-002 (Reschedule -- Select New Time) | Client's chosen time is re-validated as still free | The Booking reference, the chosen new time |
| FEAT-10.SPEC-004 (Booking Update Commit) | Reschedule fails to save, or a conflicting Pro-side transition wins first | Same Booking reference and chosen time, refreshed to current state; an error or conflict message |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Full screen, for a booking that belongs to their own matched Client record with this Pro | Confirm the reschedule, back out to slot selection, or (on a late reschedule) cancel instead | -- |
| The Pro (Talia) | No | No | This is not the Pro's own change surface; the Pro's own reschedule is through Pro Booking Management (FEAT-30), never through this screen |
| Platform Operator (Support) | No | No | Support access never uses or bypasses a client's access link (scope-boundaries SC-05); no support entry point exists here |
| Unauthenticated | No | No | Reachable only via FEAT-10.SPEC-002's already-authenticated navigation; a direct, unauthenticated attempt is redirected to FEAT-06.SPEC-001 |
| Expired session | No | No | The underlying access link governs the viewing session (FEAT-06); once it has transitioned to Used or Expired, reloading this screen is treated as unauthenticated and redirected to FEAT-06.SPEC-001 with the "request a new link" prompt |

## Layout and Content

**Header:** Screen title "Confirm Reschedule" with a back arrow (returns to FEAT-10.SPEC-002).

**Body:**
- Appointment summary: service name, the original date/time struck through or clearly marked "current," and the new chosen date/time
- Cancellation window countdown: plain-language statement of how much time remains before the original booking would have entered the cancellation window (computed against the original start_time, per FEAT-10.SPEC-005)
- Deposit outcome, one of two variants:
  - **Outside the window:** "Your deposit carries over -- nothing further to pay now."
  - **Inside the window (late reschedule):** "Your original deposit is kept, per the cancellation policy you agreed to, and a new deposit is needed for this new time." followed by the new deposit amount
- "Confirm Reschedule" button (primary action)
- "Choose a different time" link, returning to FEAT-10.SPEC-002
- On a late reschedule only: "Cancel instead" link

Appointment summary and the cancellation window countdown are display-only text with no interaction of their own; only the back arrow, "Confirm Reschedule," "Choose a different time," and "Cancel instead" (when shown) are interactive elements on this screen.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Sections stack vertically in the order given above, full width; "Confirm Reschedule" is full-width.
- **Medium size class and above:** Same vertical section order, content column capped at a consistent platform-wide reading width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-10.SPEC-002 | Screen closes | Standard backward transition |
| "Confirm Reschedule" button | Tap | Triggers FEAT-10.SPEC-004 (Booking Update Commit) for a reschedule | Button shows loading state | Success (outside window): navigate to FEAT-06.SPEC-004 showing the updated time. Success (inside window): navigate into FEAT-07's deposit-payment step for the new booking's fresh deposit. Failure: error message and Retry option (see States, Error) |
| "Choose a different time" link | Tap | Navigate to FEAT-10.SPEC-002 (Reschedule -- Select New Time) | Screen transitions | Standard navigation transition; nothing changes |
| "Cancel instead" link (late reschedule only) | Tap | Navigate to FEAT-10.SPEC-001 (Cancel Booking) for the original booking | Screen transitions | Standard navigation transition; nothing changes |
| "Confirm Reschedule" (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> appointment summary -> cancellation window countdown -> deposit outcome -> "Confirm Reschedule" -> "Choose a different time" -> "Cancel instead" (when shown).
- **Dynamic announcements:** The deposit outcome variant is announced on screen load; the concurrent-edit conflict message and any "just taken" recovery message are announced as soon as they appear; a commit failure's error text is announced and focus moves to the Retry action.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | A brief in-place loading indicator where the outcome will appear | Screen first opens | Outcome computation finishes loading |
| Ineligible | Plain message "This booking can no longer be cancelled or rescheduled." replaces the deposit outcome and Confirm action; the appointment summary remains visible | FEAT-10.SPEC-005 reports the booking as ineligible (state is Completed, No-Show, Cancelled by Client, Cancelled by Pro, Rescheduled, Pending Payment, or Expired (unpaid)) on this screen's load -- covering the case where the booking became ineligible between FEAT-10.SPEC-002's slot selection and this screen's load | Client navigates back to FEAT-10.SPEC-002 (no path forward on this screen) |
| Populated -- Outside Window | Carried-over deposit outcome shown | Loaded outcome is Rule 5 (no change, carryover) | Client confirms or navigates away |
| Populated -- Inside Window (Late Reschedule) | Compound outcome shown: original deposit kept plus new deposit needed, with the "Cancel instead" link visible | Loaded outcome is Rule 6 (compound outcome) | Client confirms or navigates away |
| Confirming | "Confirm Reschedule" button shows a loading state | Client taps "Confirm Reschedule" | Commit succeeds or fails |
| Error | Error banner "We couldn't reschedule this booking. Try again." with a Retry option; the original booking remains untouched and intact | FEAT-10.SPEC-004 reports the commit failed to save | Client taps Retry and the commit succeeds, or navigates away |
| Slot No Longer Available | Plain message "That time was just taken. Choose another." replaces the confirm action | The chosen new time is re-validated at commit time and found no longer free | Client taps "Choose a different time," returning to a refreshed FEAT-10.SPEC-002 |
| Load Error | Error banner "We couldn't load the reschedule outcome. Try again." with a retry action | The initial outcome computation fails to load | Client taps Retry and the load succeeds |
| Offline/Degraded | Banner "Connectivity is required to reschedule a booking." appears; the loaded outcome (if any) remains visible read-only; "Confirm Reschedule" is disabled | Connectivity is lost while this screen is open, or the screen is opened without connectivity | Connectivity is restored -- the banner clears and "Confirm Reschedule" re-enables |

## Validation Rules

Validation governed by FEAT-10.SPEC-005 (Cancellation Window & Eligibility Rule), which this screen enforces at two points: on screen load (gating whether the outcome and Confirm action are shown at all, per the Ineligible state) and again at the moment of confirm (the authoritative re-check performed by FEAT-10.SPEC-004). FEAT-09.SPEC-003 (Deposit Outcome Rules) governs which outcome branch applies once eligibility passes. The chosen slot is re-validated one final time at the moment of commit (FEAT-10.SPEC-004), per FEAT-03's slot rules, to guard against the time being taken between FEAT-10.SPEC-002's selection and this screen's confirm.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-10.SPEC-002 (Reschedule -- Select New Time) | -- |
| Successful reschedule, outside window | FEAT-06.SPEC-004 (Booking Detail via Manage Link) | FEAT-06 |
| Successful reschedule, inside window (late reschedule) | Deposit payment step for the new booking | FEAT-07 (Deposit Payment at Booking) |
| "Choose a different time" tap | FEAT-10.SPEC-002 (Reschedule -- Select New Time) | -- |
| "Cancel instead" tap (late reschedule only) | FEAT-10.SPEC-001 (Cancel Booking) | -- |

## Data Model

**Creates:** None on this screen directly -- a new Booking record for a late reschedule is created by FEAT-10.SPEC-004 at commit time, not here.
**Reads:** Booking -- service, original start_time, state, policy_version, scoped to the matched Client with this Pro (FEAT-06.SPEC-008). Deposit Transaction -- amount, for the outcome preview. Cancellation Policy -- window_hours and plain_language_wording, via the bound version (FEAT-09.SPEC-002).
**Updates:** None directly -- the reschedule itself is performed by FEAT-10.SPEC-004, which this screen triggers.
**Deletes:** None.

## Business Rules

- FEAT-10.SPEC-005 governs the window comparison used here, computed against the Booking's *original* start_time (not the newly chosen time), per FEAT-09.SPEC-003's Rule 5/Rule 6 definitions.
- FEAT-10.SPEC-005 also governs whether this screen is reachable at all: eligibility is checked on this screen's load (the Ineligible state applies if the booking became Completed, No-Show, or otherwise ineligible since FEAT-10.SPEC-002's slot selection) and re-checked again at the instant of confirm by FEAT-10.SPEC-004, consistent with FEAT-10.SPEC-005's "Enforced By" table naming both points for this screen.
- Outside the window: FEAT-09.SPEC-003 Rule 5 applies -- the existing deposit carries over to the new appointment time; no new deposit charge.
- Inside the window: FEAT-09.SPEC-003 Rule 6 applies -- the original deposit's disposition becomes Forfeiture Flagged (treated as a late cancellation) and the newly created Booking requires its own fresh deposit, shown to the client before they confirm; both halves of this compound outcome are always shown together, per the Brief's Cross-Field Rules ("Reschedule-inside-window compound outcome").
- "See the outcome before confirming" pattern: this screen shows the deposit consequence plainly, with an explicit confirm step and a no-penalty way to back out with nothing changed, consistent with FEAT-10.SPEC-001's identical pattern for a cancellation.
- XBR-01: the chosen slot is re-validated one final time at commit, since availability can change between selection (FEAT-10.SPEC-002) and confirm (this screen).

## Edge Cases

- **The Pro cancels, reschedules, or marks this booking no-show while Riley is viewing this screen** -- Per the Booking entity's Contention resolution (reject-with-refresh), if Riley then taps "Confirm Reschedule," the commit is rejected because the Pro's transition already committed first; Riley sees the dialog "This booking's details changed. Refresh to see the latest before continuing." with a "Refresh" action, and her original tap is never carried through against stale data.
- **The chosen new time is taken by another client between FEAT-10.SPEC-002's re-validation and this screen's confirm tap** -- The commit attempt reports the slot is no longer free; the screen shows the "Slot No Longer Available" state with "That time was just taken. Choose another." and a link back to a refreshed FEAT-10.SPEC-002, per XBR-01's "never a payment error" guarantee.
- **Riley taps "Confirm Reschedule" twice rapidly** -- The second tap is ignored while the first commit is in progress (button in loading state).
- **Riley taps "Cancel instead" on a late reschedule** -- Nothing about the reschedule attempt is committed; she is taken to FEAT-10.SPEC-001 to evaluate cancelling the original booking on its own terms.
- **The window boundary is crossed between screen load and confirm tap (e.g., the client sits on this screen past the exact cutoff moment)** -- The outcome used is the one computed fresh at the moment of commit (FEAT-10.SPEC-004/FEAT-10.SPEC-005), not the one displayed when the screen first loaded; if the recomputed outcome differs from what was shown, the confirm proceeds once with the updated outcome re-shown for one additional explicit confirm, rather than silently applying a different outcome than the client saw.
- **Riley loses connectivity while viewing this screen** -- The Offline/Degraded banner appears and "Confirm Reschedule" is disabled; no reschedule is submitted.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-10.SPEC-002 (Reschedule -- Select New Time) | Navigation (inbound/outbound) | Entry point into this screen; "Choose a different time" returns there |
| FEAT-10.SPEC-005 (Cancellation Window & Eligibility Rule) | References (inbound) | Supplies the window comparison against the original start_time |
| FEAT-09.SPEC-003 (Deposit Outcome Rules) | References (inbound) | Supplies which outcome branch (Rule 5 or Rule 6) applies |
| FEAT-10.SPEC-004 (Booking Update Commit) | Triggers (outbound) | "Confirm Reschedule" triggers the commit |
| FEAT-10.SPEC-001 (Cancel Booking) | Navigation (outbound) | "Cancel instead" (late reschedule only) starts that flow |
| FEAT-07 (Deposit Payment at Booking) | Navigation (outbound) | A late reschedule's new deposit is collected here after confirm |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| late_reschedule_warning_shown | -- | Screen loads showing the inside-window compound outcome | supports success-metrics.md: "Self-Service Reschedule Rate" |
| reschedule_outcome_viewed | window_state (outside/inside) | Screen finishes loading with data | supports success-metrics.md: "Self-Service Reschedule Rate" |
| reschedule_confirmed | window_state | Client confirms and the commit succeeds | supports success-metrics.md: "Self-Service Reschedule Rate" |
| reschedule_abandoned | reason (chose_different_time / cancelled_instead / navigated_away) | Client leaves this screen without confirming | supports success-metrics.md: "Self-Service Reschedule Rate" |

## Acceptance Criteria

**FEAT-10.SPEC-003-AC-01:** Given Riley picks a time outside the cancellation window, when this screen loads, then she sees "Your deposit carries over -- nothing further to pay now."

**FEAT-10.SPEC-003-AC-02:** Given Riley picks a time that would put her reschedule inside the cancellation window, when this screen loads, then she sees both halves of the compound outcome together: the original deposit is kept, and a new deposit is needed.

**FEAT-10.SPEC-003-AC-03:** Given Riley is on the inside-window outcome, when she looks for a way to back out without penalty, then a "Cancel instead" link is shown alongside "Choose a different time."

**FEAT-10.SPEC-003-AC-04:** Given Riley taps "Confirm Reschedule" on the outside-window outcome, when the commit succeeds, then FEAT-10.SPEC-004 updates the booking to the new time and she lands on FEAT-06.SPEC-004 showing it.

**FEAT-10.SPEC-003-AC-05:** Given Riley taps "Confirm Reschedule" on the inside-window outcome, when the commit succeeds, then the original booking's deposit is flagged forfeited and she is taken into FEAT-07's deposit-payment step for the new booking's fresh deposit.

**FEAT-10.SPEC-003-AC-06:** Given Riley taps "Choose a different time," when the tap registers, then she returns to FEAT-10.SPEC-002 and nothing about the original booking has changed.

**FEAT-10.SPEC-003-AC-07:** Given Riley taps "Cancel instead" on the inside-window outcome, when the tap registers, then she is taken to FEAT-10.SPEC-001 and no reschedule was committed.

**FEAT-10.SPEC-003-AC-08:** Given the chosen new time is taken by another client between selection and confirm, when Riley taps "Confirm Reschedule," then she sees "That time was just taken. Choose another." and is returned to a refreshed FEAT-10.SPEC-002, never a payment error.

**FEAT-10.SPEC-003-AC-09:** Given Talia cancels this booking on her side while Riley is viewing this screen, when Riley then taps "Confirm Reschedule," then her commit is rejected with "This booking's details changed. Refresh to see the latest before continuing."

**FEAT-10.SPEC-003-AC-10:** Given Riley taps "Confirm Reschedule" and the commit fails to save for a reason other than slot contention, when the failure occurs, then the error banner "We couldn't reschedule this booking. Try again." appears with a Retry option, and the original booking remains untouched.

**FEAT-10.SPEC-003-AC-11:** Given the initial outcome computation fails to load, when the failure occurs, then the error banner "We couldn't load the reschedule outcome. Try again." appears with a retry action.

**FEAT-10.SPEC-003-AC-12:** Given Riley loses connectivity while viewing this screen, when connectivity drops, then the banner "Connectivity is required to reschedule a booking." appears and "Confirm Reschedule" is disabled.

**FEAT-10.SPEC-003-AC-13:** Given Riley taps "Confirm Reschedule" twice in rapid succession, when the second tap registers, then it is ignored while the first commit is in progress.

**FEAT-10.SPEC-003-AC-14:** Given Riley reaches exactly the cutoff moment while this screen is displayed, when she taps "Confirm Reschedule," then the outcome applied is the one recomputed at that exact moment, treated as outside the window per the inclusive-boundary rule (FEAT-10.SPEC-005).

**FEAT-10.SPEC-003-AC-15:** Given a client without a valid access link attempts to reach this screen directly, when the attempt is made, then they are redirected to FEAT-06.SPEC-001 and shown no outcome data.

**FEAT-10.SPEC-003-AC-16:** Given the booking becomes ineligible (e.g., already Completed, No-Show, or transitioned by the Pro) between Riley's slot selection on FEAT-10.SPEC-002 and this screen's load, when this screen loads, then it shows the Ineligible state with "This booking can no longer be cancelled or rescheduled." and no deposit outcome or Confirm action is offered.

**FEAT-10.SPEC-003-AC-17:** Given Riley taps the back arrow, when the tap registers, then she returns to FEAT-10.SPEC-002 and nothing about the booking has changed.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 9 (loading, ineligible, populated-outside, populated-inside, confirming, error, slot-no-longer-available, load error, offline) | 9 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |



# Automation Spec: Booking Update Commit

## Overview

**Name:** Booking Update Commit
**ID:** FEAT-10.SPEC-004
**Type:** Automation
**Purpose:** Commits the client's confirmed cancellation or reschedule to the Booking record, coordinating the deposit outcome, calendar mirroring, activity logging, and freed-slot handoff this triggers in other features.
**Parent Feature:** FEAT-10 -- Client-Initiated Cancel/Reschedule

## Scope and Non-Goals

**In Scope:**
- Re-checking eligibility (FEAT-10.SPEC-005) at the instant of commit, as the authoritative gate
- Writing the Booking state transition: Cancelled by Client, or the in-place time update for an outside-window reschedule, or the compound Rescheduled-plus-new-Booking transition for an inside-window (late) reschedule
- Handing off the recorded action to FEAT-09.SPEC-004 for deposit outcome evaluation
- Handling commit failure so the original booking is left untouched and intact
- Resolving a conflicting concurrent transition via reject-with-refresh

**Non-Goals:**
- Determining eligibility or the window comparison itself -- owned by FEAT-10.SPEC-005; this automation re-checks that spec's result at commit time but never redefines it
- Deriving or applying the deposit outcome -- owned by FEAT-09.SPEC-003 (rule set) and FEAT-09.SPEC-004 (evaluation); this automation only records the action and hands off, per the Side-Effect Inventory's disposition of that response as "FEAT-09 responsibility (XBR-09)"
- Moving or removing the Pro's personal calendar entry -- owned by FEAT-04 (Two-Way Calendar Sync, XBR-13); this automation's commit is what FEAT-04 reacts to
- Writing the append-only activity event -- owned by FEAT-16 (Booking & Payment Activity Record, XBR-21); this automation's commit is what FEAT-16 reacts to
- Notifying the client or the Pro -- owned by FEAT-10.SPEC-006 (Cancellation/Reschedule Notification), which this automation triggers on success
- Making the freed slot bookable and notifying the waitlist -- owned by FEAT-20 (Waitlist for Cancelled Slots, XBR-28); this automation's cancellation commit is what FEAT-20 reacts to
- Maintaining the Pro's rolling booking-count and revenue aggregates -- owned by FEAT-25.SPEC-004 (Historical Aggregate Maintenance); this automation's cancellation, in-place reschedule, and terminal-Rescheduled commits are the events FEAT-25.SPEC-004 reacts to, and this automation performs no aggregate write itself

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Client confirms a cancellation | FEAT-10.SPEC-001 (Cancel Booking) | Client taps "Confirm Cancellation" in the confirm dialog | Booking reference, cancellation timestamp |
| Client confirms a reschedule | FEAT-10.SPEC-003 (Reschedule -- Outcome & Confirm) | Client taps "Confirm Reschedule" | Booking reference, chosen new start_time, reschedule timestamp |

## Processing Logic

1. Receive the confirmed action (cancel or reschedule) with the Booking reference and, for a reschedule, the chosen new start_time.
2. Re-check eligibility for this Booking (FEAT-10.SPEC-005) against its *current* state -- if ineligible (e.g., a conflicting transition already committed, or the state has since moved to Completed/No-Show), stop and report the conflict outcome (see Outcome Definitions) rather than proceeding.
3. For a reschedule, re-validate the chosen new start_time against live availability one final time (FEAT-03, XBR-01) -- if the slot is no longer free, stop and report the slot-lost outcome.
4. Compute the window comparison (FEAT-10.SPEC-005) against the Booking's original start_time, to determine which branch the commit will produce.
5. **If cancelling:** write the Booking's state to Cancelled by Client and set the cancellation timestamp.
6. **If rescheduling outside the window (Rule 5 applies):** update the same Booking record's start_time to the chosen new time and set the reschedule timestamp; state remains unchanged (Confirmed or Awaiting Outcome, whichever it already was); the existing Deposit Transaction and Access Link continue to apply to this same record.
7. **If rescheduling inside the window (Rule 6 applies -- late reschedule):** (a) transition the original Booking's state to Rescheduled (terminal) and set the reschedule timestamp; (b) create a new Booking record for the chosen new time, carrying over the same service, duration, client, and the policy version current at that moment (a fresh acknowledgment, exactly as a new booking would, since this is functionally a late-cancellation-plus-new-booking per FEAT-09.SPEC-003 Rule 6); (c) flag the new Booking as requiring its own fresh deposit under FEAT-07's ordinary eligibility rules.
8. Hand off the recorded action (Booking reference, action type, initiator: Client, timestamp, and for a reschedule the original and new start_time) to FEAT-09.SPEC-004 for deposit outcome evaluation.
9. On a successful write, trigger FEAT-10.SPEC-006 (Cancellation/Reschedule Notification) with the outcome type and, for a late reschedule, the new Booking's reference.
10. Report success back to the triggering screen with the resulting state, for FEAT-10.SPEC-001/FEAT-10.SPEC-003 to route the client onward (to FEAT-06.SPEC-004, or into FEAT-07's deposit step for a late reschedule).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|-----------------|-------------------|
| Cancellation committed | Eligibility passes; client confirmed a cancel | Booking: state -> Cancelled by Client, cancellation timestamp set | Client is routed to FEAT-06.SPEC-004 showing the booking as Cancelled | FEAT-06.SPEC-004, FEAT-09.SPEC-004, FEAT-10.SPEC-006 |
| Reschedule committed, outside window | Eligibility passes; window comparison is outside; slot re-validation passes | Booking: start_time -> new time, reschedule timestamp set; state unchanged | Client is routed to FEAT-06.SPEC-004 showing the new time; deposit carries over | FEAT-06.SPEC-004, FEAT-09.SPEC-004, FEAT-10.SPEC-006 |
| Reschedule committed, inside window (late reschedule, compound) | Eligibility passes; window comparison is inside; slot re-validation passes | Original Booking: state -> Rescheduled, reschedule timestamp set. New Booking created with the new start_time, same service/duration/client, flagged as requiring a fresh deposit | Client is routed into FEAT-07's deposit-payment step for the new Booking | FEAT-07, FEAT-09.SPEC-004, FEAT-10.SPEC-006 |
| Eligibility conflict (a Pro-side or automated transition committed first) | Re-check at step 2 finds the Booking's current state no longer eligible | No change to any Booking record from this attempt | Client sees "This booking's details changed. Refresh to see the latest before continuing." on the triggering screen (FEAT-10.SPEC-001/FEAT-10.SPEC-003), per the Booking entity's reject-with-refresh Contention resolution | FEAT-10.SPEC-001, FEAT-10.SPEC-003 |
| Slot lost to contention (reschedule only) | Re-validation at step 3 finds the chosen new time no longer free | No change to any Booking record | Client sees "That time was just taken. Choose another." and returns to a refreshed FEAT-10.SPEC-002 | FEAT-10.SPEC-002, FEAT-10.SPEC-003 |
| Commit failure (the write itself does not save) | A transient failure during the write in steps 5-7 | No partial state change is left behind -- the original Booking remains exactly as it was before the attempt | Client sees "We couldn't cancel/reschedule this booking. Try again." with a Retry option on the triggering screen | FEAT-10.SPEC-001, FEAT-10.SPEC-003 |

## Data Model

**Reads:** Booking -- state, start_time, policy_version, service, duration, client reference. Cancellation Policy -- window_hours and computed cutoff, via FEAT-09.SPEC-002 (through FEAT-10.SPEC-005).
**Creates:** Booking -- a new record for the inside-window (late reschedule) compound outcome only, with service, duration, client, start_time (the chosen new time), and a freshly acknowledged policy_version, mirroring the fields a fresh booking (FEAT-05) would set; flagged as requiring its own deposit rather than assigning one directly (FEAT-07 owns deposit capture).
**Updates:** Booking -- state (to Cancelled by Client, or to Rescheduled for the original record in a late reschedule), start_time (for an outside-window reschedule, in place on the same record), and the cancellation/reschedule timestamp.
**Deletes:** None -- bookings are never deleted (SC-22); a cancelled or rescheduled booking remains as history with an updated state.

## Business Rules

- **Resolution of the Entity-Lifecycle Coverage Matrix's flagged discrepancy:** a client-initiated reschedule outside the cancellation window updates the existing Booking record's start_time in place -- the record's state is unchanged and no new record is created, matching FEAT-09.SPEC-003 Rule 5's "No Change... the deposit carries over" wording, which presumes one continuing record and deposit. A client-initiated reschedule inside the window is the sole case in this feature that creates a new Booking record: the original transitions to the terminal Rescheduled state and a new Booking is created for the new time, matching FEAT-09.SPEC-003 Rule 6's explicit "newly created Booking" language and its Edge Case confirming the original reaches "a terminal transition (Rescheduled)." This means this automation is a Booking Creator for the narrow late-reschedule compound case -- a fact not reflected in the dependency map's Booking "Creators: FEAT-05, FEAT-30, FEAT-21" list, which is flagged here as a carry-forward item for Stage 4/reconciliation to add FEAT-10 to that list for this one scenario, rather than silently contradicting the dependency map.
- **Consistency with feature-overview.md:** this feature's Brief (Entity-Lifecycle Coverage Matrix, Create row) states that FEAT-10.SPEC-004 creates one new Booking record, only on an inside-window (late) reschedule, and nothing on a cancellation or outside-window reschedule; this spec's step 7(b) and Data Model Creates entry are exactly that mechanism. The dependency map's Booking Creators line still omits FEAT-10 for this one scenario; that map-level delta is carried to Stage 4 (SG-01) and is not edited here.
- XBR-08: the new Booking created in a late reschedule acknowledges the cancellation policy version current at that moment, exactly as a fresh booking would -- it never inherits the original booking's bound version.
- XBR-13: this commit is the event FEAT-04 mirrors to (or removes from) the Pro's personal calendar; this automation performs no calendar write itself.
- XBR-18: a fresh manage link is not issued by this automation for an outside-window reschedule, since the same Booking record and its existing Access Link continue to apply; a late reschedule's new Booking is reached through the deposit-payment confirmation's own manage-link issuance (FEAT-08.SPEC-010), exactly as any new booking is.
- XBR-21: this commit is the event FEAT-16 writes an append-only activity event for; this automation performs no activity-log write itself.
- XBR-28: a cancellation commit is the event FEAT-20 reacts to for waitlist priority notification; this automation performs no waitlist write itself.
- The Booking entity's Contention resolution (reject-with-refresh) governs step 2's re-check: the first committed state transition always wins, and every transition is validated against the current state before this automation writes anything.

## Edge Cases

- **A client cancellation and a Pro-side action (FEAT-30) arrive for the same Booking at effectively the same time** -- Whichever commits first wins; the second arrival's re-check at step 2 finds the Booking already in a non-eligible current state and reports the eligibility-conflict outcome. Only one terminal transition is ever written for a given Booking.
- **The commit fails partway through the compound inside-window write (original transitioned to Rescheduled but the new Booking's creation fails)** -- The write is treated as a single atomic step: if any part fails, no part commits -- the original Booking is left exactly as it was (not transitioned) and no new Booking is created, so the client sees the ordinary commit-failure outcome and can retry cleanly rather than being left with an orphaned Rescheduled original and no replacement.
- **Trigger fires while a previous run is already in flight for the same Booking** -- Not possible in practice: FEAT-10.SPEC-001 and FEAT-10.SPEC-003 disable their confirm controls while a commit is in progress, and a Booking has at most one terminal cancellation/reschedule transition ever recorded, so a second commit attempt for the same Booking cannot start while the first is in flight.
- **Concurrent commit attempts for two different Bookings** -- Proceed independently; neither is delayed by the other.
- **A late reschedule's new Booking fails to reach FEAT-07's deposit step (e.g., the client closes the app before paying)** -- Governed by FEAT-03's standard slot-hold and Pro-created-deposit-request expiration rules (XBR-02) applied to the new Booking exactly as to any unpaid booking; this automation's own responsibility ends once the new Booking is created and flagged.
- **The Booking's bound policy version cannot be read at the instant of commit (the same rare inconsistency FEAT-09.SPEC-004 notes)** -- The commit is held rather than writing an outcome-less transition; the client sees the ordinary loading/error handling on the triggering screen and can retry, consistent with FEAT-09.SPEC-004's own "Outcome pending" handling once the write does proceed.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-10.SPEC-001 (Cancel Booking) | Triggered by (inbound) | "Confirm Cancellation" fires this automation |
| FEAT-10.SPEC-003 (Reschedule -- Outcome & Confirm) | Triggered by (inbound) | "Confirm Reschedule" fires this automation |
| FEAT-10.SPEC-005 (Cancellation Window & Eligibility Rule) | References (outbound) | Re-checked at commit as the authoritative eligibility gate |
| FEAT-03 (Real-Time Slot Availability Engine) | References (outbound) | Final slot re-validation for a reschedule's chosen time |
| FEAT-09.SPEC-004 (Cancellation & No-Show Outcome Evaluation) | Triggers (outbound) | Hands off the recorded action for deposit outcome evaluation |
| FEAT-07 (Deposit Payment at Booking) | Triggers (outbound) | A late reschedule's new Booking is routed into the standard deposit-payment step |
| FEAT-04.SPEC-005 (Booking-to-Calendar Sync) -- within FEAT-04 (Two-Way Calendar Sync) | Affects (outbound) | This commit is the event the Pro's personal calendar mirrors |
| FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) | Affects (outbound) | This commit is the event the append-only activity log records |
| FEAT-25.SPEC-004 (Historical Aggregate Maintenance) -- within FEAT-25 (Booking & Revenue Insights) | Affects (outbound) | A cancellation, outside-window reschedule, or the original Booking's terminal Rescheduled transition in a late reschedule is the event the rolling insights aggregates reverse or move a booking count for; the late reschedule's new Booking is counted separately, only once it reaches Confirmed |
| FEAT-20.SPEC-005 (Cancellation-Triggered Waitlist Matching) -- within FEAT-20 (Waitlist for Cancelled Slots) | Affects (outbound) | A cancellation commit is the event that frees the slot for waitlist priority |
| FEAT-10.SPEC-006 (Cancellation/Reschedule Notification) | Triggers (outbound) | A successful commit fires the client confirmation and Pro change notice |
| FEAT-06.SPEC-004 (Booking Detail via Manage Link) | Affects (outbound) | The client is routed here after a successful commit (except a late reschedule) |

## Analytics and Success Signals

- **booking_cancelled_by_client** (Booking reference, window_state: outside/inside) -- supports success-metrics.md: "Self-Service Reschedule Rate"
- **booking_rescheduled_by_client** (Booking reference, window_state: outside/inside) -- supports success-metrics.md: "Self-Service Reschedule Rate"
- **booking_update_commit_conflict** (action_type: cancel/reschedule) -- N/A -- no Stage 2 metric measures contention-loss rate specifically; retained per operational visibility into how often the Booking entity's reject-with-refresh resolution is exercised on the client side
- **booking_update_commit_failed** (action_type: cancel/reschedule) -- supports success-metrics.md: "Self-Service Reschedule Rate" (a failed commit that cannot be completed self-service is exactly the gap this metric measures against)

## Acceptance Criteria

**FEAT-10.SPEC-004-AC-01:** Given Riley confirms a cancellation on an eligible booking, when this automation commits, then the Booking's state is written to Cancelled by Client with the cancellation timestamp set, and FEAT-09.SPEC-004 is handed the action.

**FEAT-10.SPEC-004-AC-02:** Given Riley confirms a reschedule outside the cancellation window, when this automation commits, then the same Booking record's start_time is updated to the new time, its state is unchanged, and FEAT-09.SPEC-004 is handed the action.

**FEAT-10.SPEC-004-AC-03:** Given Riley confirms a reschedule inside the cancellation window, when this automation commits, then the original Booking's state is written to Rescheduled and a new Booking record is created at the new time, flagged as requiring its own fresh deposit.

**FEAT-10.SPEC-004-AC-04:** Given Riley confirms a cancel or reschedule and a Pro-side transition already committed against the same Booking moments earlier, when the eligibility re-check runs, then this attempt is rejected with the conflict outcome and no Booking record is changed by this attempt.

**FEAT-10.SPEC-004-AC-05:** Given Riley confirms a reschedule and the chosen new time is taken by another client between selection and commit, when the final slot re-validation runs, then this attempt is rejected with the slot-lost outcome and no Booking record is changed.

**FEAT-10.SPEC-004-AC-06:** Given Riley confirms a cancel or reschedule and the write itself fails to save, when the failure occurs, then the original Booking remains exactly as it was, with no partial state change.

**FEAT-10.SPEC-004-AC-07:** Given the inside-window compound write fails partway through (original transitioned but the new Booking's creation does not complete), when the failure is detected, then the entire attempt is rolled back as one unit -- the original Booking is left untransitioned and no new Booking exists.

**FEAT-10.SPEC-004-AC-08:** Given a successful cancellation commits, when the commit completes, then FEAT-10.SPEC-006 is triggered for the client confirmation and Pro change notice.

**FEAT-10.SPEC-004-AC-09:** Given a successful outside-window reschedule commits, when the commit completes, then the client is routed to FEAT-06.SPEC-004 showing the updated time.

**FEAT-10.SPEC-004-AC-10:** Given a successful inside-window reschedule commits, when the commit completes, then the client is routed into FEAT-07's deposit-payment step for the newly created Booking.

**FEAT-10.SPEC-004-AC-11:** Given a client cancellation and a Pro no-show marking are both attempted on the same Booking at effectively the same time, when both reach this automation's domain, then only the first to commit succeeds, and the second is rejected by the eligibility re-check.

**FEAT-10.SPEC-004-AC-12:** Given this automation is processing a commit for one Booking, when a separate commit attempt fires for a different Booking at the same time, then the two proceed independently and neither is delayed by the other.

**FEAT-10.SPEC-004-AC-13:** Given a late reschedule's new Booking is created but never paid, when it is left unpaid, then it is governed by FEAT-03's standard slot-hold and expiration rules (XBR-02), exactly as any other unpaid booking.

**FEAT-10.SPEC-004-AC-14:** Given Riley's booking's bound policy version cannot be read at the instant of commit, when this automation attempts the write, then the commit is held and the triggering screen shows its ordinary error/retry handling rather than writing an outcome-less transition.

**FEAT-10.SPEC-004-AC-15:** Given a new Booking is created for a late reschedule, when its policy_version is set, then it acknowledges the version current at that moment, never the original Booking's bound version.

**FEAT-10.SPEC-004-AC-16:** Given a cancellation commit succeeds, when the commit completes, then FEAT-20 and FEAT-04 each react independently to the commit (freed-slot handoff and calendar mirroring), with no calendar or waitlist write performed by this automation itself; FEAT-25.SPEC-004 likewise reacts independently to any committed cancellation or reschedule, and this automation performs no insights aggregate write.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 | 2 |
| Outcome Paths | 6 | 6 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Cancellation Window & Eligibility Rule

## Overview

**Name:** Cancellation Window & Eligibility Rule
**ID:** FEAT-10.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs whether a booking is currently eligible to be cancelled or rescheduled by its client, and computes the window countdown that determines which deposit-outcome branch applies.
**Parent Feature:** FEAT-10 -- Client-Initiated Cancel/Reschedule
**Governed Entity:** Booking (client-eligibility slice only)

## Scope and Non-Goals

**In Scope:**
- Whether a specific booking may currently be cancelled or rescheduled by the client who owns it (the eligibility gate)
- Computing and rendering the plain-language countdown to the cancellation window's cutoff, for display on FEAT-10.SPEC-001 and FEAT-10.SPEC-003
- Authorization over who may check eligibility and who may never override an ineligible result
- Boundary behavior at the exact cutoff moment, consistent with FEAT-09.SPEC-003's inclusive-boundary rule

**Non-Goals:**
- Computing the cutoff time itself from the bound policy version and window_hours -- owned by FEAT-09.SPEC-002 (Policy Versioning & Cutoff Rendering); this spec consumes that computed cutoff, never re-derives it
- Deciding which deposit-outcome branch applies once the window comparison is made -- owned by FEAT-09.SPEC-003 (Deposit Outcome Rules); this spec supplies the timing input that rule set consumes
- Evaluating and writing the actual deposit outcome once an action is recorded -- owned by FEAT-09.SPEC-004 (Cancellation & No-Show Outcome Evaluation)
- Pro-side eligibility for the Pro's own cancel/reschedule actions -- owned by Pro Booking Management (FEAT-30); XBR-09 states a Pro-made reschedule never exposes the client to the window, so this spec governs client-initiated eligibility only

## Governed Entity

**Entity:** Booking (client-eligibility slice: state, start_time, policy_version)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|--------------|
| state | enum | Pending Payment \| Confirmed \| Awaiting Outcome \| Completed \| No-Show \| Cancelled by Client \| Cancelled by Pro \| Rescheduled \| Expired (unpaid) -- this spec reads it to gate eligibility |
| start_time | date/time | The appointment's current start time, in the Pro's timezone; this spec reads it to compute the window countdown |
| policy_version | reference | The bound Cancellation Policy version; this spec reads it only to pass through to FEAT-09.SPEC-002's cutoff computation, never to interpret window_hours itself |

**Referenced (read-only):** Cancellation Policy -- window_hours and the computed cutoff, via FEAT-09.SPEC-002. This spec computes no cutoff of its own; it consumes FEAT-09.SPEC-002's rendered value and applies the eligibility gate around it.

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|--------------------|
| FEAT-10.SPEC-001 | Cancel Booking | On screen load, before offering the Cancel action; the countdown is displayed live |
| FEAT-10.SPEC-002 | Reschedule -- Select New Time | On screen load, before offering the slot list at all |
| FEAT-10.SPEC-003 | Reschedule -- Outcome & Confirm | On screen load and again at the moment of confirm, to determine which outcome branch applies and to display the countdown |
| FEAT-10.SPEC-004 | Booking Update Commit | Re-checked at the instant of commit, as the authoritative eligibility gate before any state transition is written |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|------------------------------|---------------|-----------------|-----------|
| state | No validation beyond data type -- this spec reads it to gate eligibility; it is never entered or altered by any role through this spec | Always | -- | -- | -- |
| start_time | No validation beyond data type -- this spec reads it to compute the countdown; it is never entered or altered by any role through this spec | Always | -- | -- | -- |
| policy_version | No validation beyond data type -- this spec passes it through to FEAT-09.SPEC-002; it is never entered or altered by any role through this spec | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|-------------------|-------|------------------|
| Eligibility is state-first, window-second | state, start_time, policy_version | Eligibility is checked first (state must not be Completed or No-Show, nor already Cancelled or Rescheduled); only an eligible booking's start_time and policy_version are then used to compute the window countdown -- an ineligible booking never reaches the countdown computation | "This booking can no longer be cancelled or rescheduled." (shown in place of any countdown) |
| Countdown reflects current start_time | start_time | The countdown is always computed against the booking's *current* start_time; for a booking being evaluated on FEAT-10.SPEC-003 for a prospective reschedule, the comparison uses the *original* (pre-reschedule) start_time, per FEAT-09.SPEC-003's Rule 5/Rule 6 definitions | N/A -- structural guarantee, not user-facing |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|-------------|-----------|------------------------------------------------|
| Check eligibility and view the window countdown for a booking | The Client (Riley) | Own-only -- their own booking only, verified via FEAT-06 before this feature's screens load | -- |
| Check eligibility and view the window countdown | The Pro (Talia) | Never through this spec -- the Pro's own eligibility for cancel/reschedule is governed separately by FEAT-30 | This spec is not reachable from any Pro-facing screen |
| Check eligibility and view the window countdown | Platform Operator (Support) | Never through this spec -- Support never uses or bypasses a client's access link (scope-boundaries SC-05) | No support entry point exists into this spec |
| Override an ineligible result (act on a Completed or No-Show booking anyway) | The Client (Riley) | Never | The Cancel/Reschedule actions are not shown; the Ineligible message is the only content offered in their place |
| Override an ineligible result | The Pro (Talia) | Never through this spec | Not applicable -- the Pro's own actions on a Completed/No-Show booking are governed by XBR-12 within FEAT-30, not by this spec |
| Override an ineligible result | Platform Operator (Support) | Never -- View-only per the Access Matrix | No override control exists for Support |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|------------------------|----------------|---------------------|
| eligible | Derived: true when state is one of Confirmed or Awaiting Outcome; false when state is Completed, No-Show, Cancelled by Client, Cancelled by Pro, Rescheduled, Expired (unpaid), or Pending Payment | Computed live on every screen load and again at commit | No -- always derived, never directly overridable |
| window_state | Derived: "outside" when the current moment is at or before the cutoff computed by FEAT-09.SPEC-002 against the relevant start_time; "inside" when after it | Computed live on every screen load and again at commit | No -- always derived, never directly overridable |
| countdown_display | Derived: the plain-language time remaining until the cutoff (e.g., "18 hours until the cancellation window closes"), or a plain statement that the window has already closed | Computed live on every screen load | No -- always derived from the cutoff FEAT-09.SPEC-002 renders |

## Business Rules

- XBR-08: every booking is governed by the cancellation policy version shown and acknowledged at booking; this spec reads the bound version but never edits or rebinds it.
- XBR-12: a booking already marked Completed or No-Show cannot be cancelled or rescheduled -- this is the eligibility gate's primary rule.
- The window boundary is inclusive of "outside": an action taken at exactly the cutoff moment (start_time minus window_hours, to the second) counts as outside the window, consistent with FEAT-09.SPEC-003's inclusive-boundary rule and the policy wording's "up to {window_hours} hours before" phrasing.
- A booking already in a terminal state from a prior action (Cancelled by Client, Cancelled by Pro, or Rescheduled) is never re-evaluated as eligible -- once a terminal transition has committed, this spec always returns ineligible for that same booking, consistent with the Booking entity's Contention resolution (reject-with-refresh; the first committed state transition wins).
- A Pending Payment or Expired (unpaid) booking is ineligible -- these states represent a booking that never became a confirmed appointment for the client to cancel or reschedule.

## Edge Cases

- **A booking sits exactly at the cutoff moment when eligibility is checked** -- Treated as outside the window (per the inclusive-boundary rule); the countdown display reads "0 hours" or the equivalent boundary phrasing, and the outside-window outcome branch applies if the client acts in that same instant.
- **A booking's state changes to Completed between FEAT-10.SPEC-001/002/003 loading and the client's confirm tap** -- The eligibility re-check at commit time (FEAT-10.SPEC-004) catches this; the commit is rejected and the client is shown the current, now-ineligible state, per the Booking entity's Contention resolution.
- **A client reschedules, is shown the outcome, and then reloads the same screen before confirming** -- Eligibility and the countdown are recomputed fresh on reload; a countdown that has since crossed the cutoff is reflected accurately rather than showing a stale value.
- **The booking's bound policy version cannot be read (an extremely rare data inconsistency, per FEAT-09.SPEC-004's own edge case)** -- Eligibility can still be determined from state alone, but the countdown cannot be computed; the screen shows the booking as eligible with the countdown display "Cancellation window: pending" until the read succeeds, rather than blocking the whole screen.
- **A booking is in Awaiting Outcome state (its appointment time has passed but it has not yet been marked Completed or No-Show)** -- Still eligible under this spec's rule (only Completed and No-Show gate eligibility per XBR-12); in practice the window has already closed by this point, so the inside-window outcome branch applies.

## Acceptance Criteria

**FEAT-10.SPEC-005-AC-01:** Given Riley's booking is in Confirmed state and the current moment is before the computed cutoff, when eligibility is checked, then the booking is eligible and window_state is "outside."

**FEAT-10.SPEC-005-AC-02:** Given Riley's booking is in Confirmed state and the current moment is after the computed cutoff, when eligibility is checked, then the booking is eligible and window_state is "inside."

**FEAT-10.SPEC-005-AC-03:** Given Riley's booking is in Completed state, when eligibility is checked, then the booking is ineligible and the message "This booking can no longer be cancelled or rescheduled." is shown.

**FEAT-10.SPEC-005-AC-04:** Given Riley's booking is in No-Show state, when eligibility is checked, then the booking is ineligible with the same message.

**FEAT-10.SPEC-005-AC-05:** Given Riley's booking is already Cancelled by Client, Cancelled by Pro, or Rescheduled, when eligibility is checked, then the booking is ineligible.

**FEAT-10.SPEC-005-AC-06:** Given Riley's booking is in Awaiting Outcome state, when eligibility is checked, then the booking is still eligible under this spec's rule, with window_state "inside" in practice since the appointment time has passed.

**FEAT-10.SPEC-005-AC-07:** Given Riley's booking reaches exactly the cutoff moment, when the window comparison runs, then it is treated as outside the window.

**FEAT-10.SPEC-005-AC-08:** Given Talia (the Pro) has no path into this spec, when she looks for a way to check a client's booking eligibility through it, then none exists -- her own eligibility is governed separately by FEAT-30.

**FEAT-10.SPEC-005-AC-09:** Given Support opens a Pro's account in the read-only support view, when Support looks for an entry point into this spec, then none exists, consistent with scope-boundaries SC-05.

**FEAT-10.SPEC-005-AC-10:** Given Riley looks for an override of an ineligible result, when she inspects the Cancel or Reschedule screens, then no override control is shown -- only the ineligibility message.

**FEAT-10.SPEC-005-AC-11:** Given a booking's state transitions to Completed between screen load and Riley's confirm tap, when the commit is attempted, then the re-check at commit time rejects it and the current ineligible state is shown.

**FEAT-10.SPEC-005-AC-12:** Given Riley's booking's bound policy version cannot be read at the moment of eligibility check, when the countdown is computed, then eligibility is still determined from state alone and the countdown shows "Cancellation window: pending" rather than blocking the screen.

**FEAT-10.SPEC-005-AC-13:** Given Riley is evaluating a prospective reschedule on FEAT-10.SPEC-003, when the window comparison runs, then it compares against the booking's *original* start_time, not the newly chosen time.

**FEAT-10.SPEC-005-AC-14:** Given Riley's booking is in Pending Payment or Expired (unpaid) state, when eligibility is checked, then the booking is ineligible, since it never became a confirmed appointment.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 3 | 3 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 6 | 6 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |



# Notification Spec: Cancellation/Reschedule Notification

## Overview

**Name:** Cancellation/Reschedule Notification
**ID:** FEAT-10.SPEC-006
**Type:** Notification
**Purpose:** Tells the client their cancellation or reschedule went through (with the exact deposit outcome) and tells the Pro that a client-initiated change happened, once FEAT-10.SPEC-004's commit succeeds.
**Parent Feature:** FEAT-10 -- Client-Initiated Cancel/Reschedule

## Scope and Non-Goals

**In Scope:**
- The trigger contract from FEAT-10.SPEC-004's successful commit into the two-audience message (client confirmation, Pro change notice) that product-features.md's Communications field for this feature declares
- Confirming that the exact content, channels, and delivery behavior for this two-audience message are the ones already defined and validated for a client-initiated change

**Non-Goals:**
- The exact message content, channel selection, and delivery rules (batching, retry, expiry) for the client-facing confirmation -- owned by FEAT-08.SPEC-004 (Booking Change & Refund Notice), which already defines the client-initiated cancellation and reschedule variants (outside-window refund, inside-window kept, outside-window carryover) in full; this spec is the trigger-and-audience contract into that content, not a second definition of it
- The exact message content and delivery rules for the Pro-facing change notice -- owned by FEAT-08.SPEC-005 (Pro Booking Activity Notification), which covers the Pro-facing counterpart for a client-initiated cancellation or reschedule; this spec does not redefine that content either
- The underlying deposit outcome determination -- owned by FEAT-09 (Cancellation & No-Show Policy Engine); this spec reports the outcome FEAT-09 determines, never derives it
- Sending or retrying the actual message -- owned by FEAT-08.SPEC-012 (text) and FEAT-08.SPEC-013 (email), the transactional messaging capability both FEAT-08.SPEC-004 and FEAT-08.SPEC-005 send through

**Content ownership (settled):** FEAT-08 owns all message content. This spec is the trigger-and-audience contract only: it names when the messages fire (FEAT-10.SPEC-004's successful commit) and who receives them (the client and the Pro), and it defines no wording, placeholders, or variants of its own. FEAT-08.SPEC-004 (client confirmation) and FEAT-08.SPEC-005 (Pro change notice) list this spec in their Connected Specs as the trigger-and-audience contract, so no duplicate content exists anywhere in the package.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Text | The recipient (client or Pro) has active Messaging Consent for texting (governed by FEAT-08.SPEC-011 for the client; the Pro's own notification preferences for the Pro) | A cancellation or reschedule is time-sensitive and financially material; both parties should learn of it wherever they already receive booking messages, per FEAT-08.SPEC-004/FEAT-08.SPEC-005 |
| Email | The recipient has not granted texting consent, or texting fails | Ensures the notice always reaches its recipient, per BRIEF.md's stated email fallback, mirroring FEAT-08.SPEC-004/FEAT-08.SPEC-013's fallback behavior |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Client's cancellation commits successfully | FEAT-10.SPEC-004 (Booking Update Commit) | Always, on a successfully saved client-initiated cancellation | Booking (updated state), Deposit Transaction (outcome once FEAT-09.SPEC-004 evaluates it), Cancellation Policy (window_hours, wording) |
| Client's reschedule commits successfully | FEAT-10.SPEC-004 (Booking Update Commit) | Always, on a successfully saved client-initiated reschedule (outside-window carryover or inside-window compound) | Booking (updated or new record, new start_time), Deposit Transaction (outcome), Cancellation Policy |

## Audience and Preferences

**Recipients:** The Client tied to the Booking (Access Matrix: Cancellation & No-Show Handling = Own-only), for the confirmation; the Pro (Talia) who owns the Booking (Access Matrix: Cancellation & No-Show Handling = Full), for the change notice. Both are entitled to this exact data: the client to their own booking's outcome, the Pro to the resulting state of their own booking.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Client texting consent (governs channel, not whether the confirmation sends) | Granted / Revoked | Captured at booking | FEAT-06 at booking; changed via FEAT-14 |
| Pro notification preferences (governs channel for the change notice) | In-app / text / email combinations | Set during onboarding | FEAT-27 (Pro Profile & Booking Page Settings) |

This notification carries no separate opt-out on either side: a change to the client's own paid booking, and the resulting change to the Pro's own schedule, are each transactional information neither party can decline to receive -- consistent with FEAT-08.SPEC-004's and FEAT-08.SPEC-005's treatment of the same fact.

**Quiet Hours:** N/A -- this is the direct, expected report of a change that just happened, not a discretionary interruption; it sends immediately regardless of time of day, exactly as FEAT-08.SPEC-004/FEAT-08.SPEC-005 define for this same event class. XBR-16's daytime window applies only to the discretionary pre-appointment reminder (FEAT-08.SPEC-002), never to this transactional notice.

## Content Definition

**Client confirmation (all variants):** Content, exact wording, and placeholders are FEAT-08.SPEC-004's own defined variants for a client-initiated change:
- Client cancellation, outside window (full refund) -- FEAT-08.SPEC-004's "client-initiated cancellation, outside window" text/email variant
- Client cancellation, inside window (deposit kept) -- FEAT-08.SPEC-004's "client-initiated cancellation, inside window" text/email variant
- Client reschedule, outside window (deposit carried over) -- FEAT-08.SPEC-004's "client-initiated reschedule, outside window" text/email variant
- Client reschedule, inside window (late reschedule, compound outcome) -- FEAT-08.SPEC-004's edge-case content: the original deposit is kept, reported distinctly from the new booking's own confirmation (FEAT-08.SPEC-001) for the new deposit

Each variant's CTA -- "Manage my booking" -- deep-links to FEAT-06.SPEC-004 (Booking Detail via Manage Link) for the affected booking (the original booking for a cancellation, the new booking for a late reschedule once it exists).

**Pro change notice (all variants):** Content, exact wording, and placeholders are FEAT-08.SPEC-005's (Pro Booking Activity Notification) own defined variant for a client-initiated cancellation or reschedule, reported on the Pro's dashboard and, per the Pro's own notification preferences, by text or email.

**Placeholders:** Declared and owned by FEAT-08.SPEC-004 (client confirmation) and FEAT-08.SPEC-005 (Pro change notice); this spec introduces no placeholder of its own. See those specs' Placeholders tables for the exact entity/field sources and empty-value fallbacks.

## Delivery Rules

**Batching:** None -- matching FEAT-08.SPEC-004's rule, each change event (this cancellation or this reschedule) produces its own single notice per audience at the moment FEAT-10.SPEC-004's commit succeeds.
**Deduplication:** At most one client confirmation and one Pro change notice per commit event, per FEAT-08.SPEC-004/FEAT-08.SPEC-005's deduplication rule; a rejected or failed commit attempt (FEAT-10.SPEC-004's conflict, slot-lost, or failure outcomes) never reaches this trigger and so never produces a notice.
**Retry on failure:** Governed by FEAT-08.SPEC-009: a failed text is retried once, then falls back to email, with the gap flagged on the Pro's dashboard for the client-side send; the Pro's own delivery failure is handled the same way.
**Expiry:** None -- matching FEAT-08.SPEC-004/FEAT-08.SPEC-005, a change or refund fact never becomes not-worth-delivering.

## Edge Cases

- **A late reschedule's compound outcome (the original deposit kept, plus the new booking's own deposit)** -- The client receives this notice's confirmation for the original booking's outcome, and a separate booking confirmation (FEAT-08.SPEC-001) once the new booking's deposit is paid; the two are never merged into one ambiguous message, per FEAT-08.SPEC-004's own edge case for this exact scenario.
- **FEAT-10.SPEC-004's commit is rejected by a conflicting Pro-side transition (eligibility conflict outcome)** -- No notice is triggered from this spec for the rejected attempt; whichever transition actually committed produces its own single notice through its own owning trigger (FEAT-10.SPEC-004 for the client side, FEAT-30 for the Pro side), never two conflicting notices for the same terminal state, per FEAT-08.SPEC-004's Contention-aware edge case.
- **The client's texting consent is revoked between the commit and this notice sending** -- The confirmation honors the consent state current at send time (FEAT-08.SPEC-011), routing to email if consent is no longer active.
- **An automatic refund entering "in progress" rather than completing immediately** -- Covered by FEAT-08.SPEC-004's own refund-in-progress variant and its follow-up notice once the refund completes; this spec's trigger is the same FEAT-10.SPEC-004 commit event that starts that chain via FEAT-09's refund handoff.
- **A cancellation commit succeeds but FEAT-20's waitlist notification or FEAT-04's calendar mirror has not yet completed** -- This notice is not held for either: the client and Pro notices report the booking change itself and never wait on downstream cross-feature effects that have their own independent timing.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-10.SPEC-004 (Booking Update Commit) | Triggered by (inbound) | A successful cancellation or reschedule commit fires this notification's trigger contract |
| FEAT-08.SPEC-004 (Booking Change & Refund Notice) | Content owner (outbound) | Owns the exact client-facing content, channels, and delivery rules this spec's trigger feeds; lists this spec in its Connected Specs as a trigger-and-audience contract |
| FEAT-08.SPEC-005 (Pro Booking Activity Notification) | Content owner (outbound) | Owns the exact Pro-facing content, channels, and delivery rules this spec's trigger feeds; lists this spec in its Connected Specs as a trigger-and-audience contract |
| FEAT-09 (Cancellation & No-Show Policy Engine) | References (inbound) | Supplies the deposit outcome the client-facing content reports |
| FEAT-06.SPEC-004 (Booking Detail via Manage Link) | Navigation (outbound) | The client confirmation's CTA deep-links here |
| FEAT-08.SPEC-012 / FEAT-08.SPEC-013 (Text / Email Capabilities) | Triggers (outbound, via FEAT-08.SPEC-004/005) | Perform the actual sends |

## Analytics and Success Signals

- **client_change_notice_sent** (change_type: cancel / reschedule; window_state: outside / inside) -- supports success-metrics.md: "Self-Service Reschedule Rate"
- **pro_change_notice_sent** (change_type: cancel / reschedule) -- supports success-metrics.md: "Automatic Refund Correctness" (the Pro-visible record of the outcome is part of what makes the refund path observable and trustworthy)
- **client_change_notice_cta_tapped** (destination: manage_link) -- supports success-metrics.md: "Self-Service Access Success"

## Acceptance Criteria

**FEAT-10.SPEC-006-AC-01:** Given Riley's cancellation commits successfully outside the window, when FEAT-10.SPEC-004 completes, then this trigger fires FEAT-08.SPEC-004's outside-window cancellation variant to Riley and FEAT-08.SPEC-005's cancellation variant to Talia.

**FEAT-10.SPEC-006-AC-02:** Given Riley's cancellation commits successfully inside the window, when FEAT-10.SPEC-004 completes, then this trigger fires FEAT-08.SPEC-004's inside-window (deposit kept) variant to Riley.

**FEAT-10.SPEC-006-AC-03:** Given Riley's reschedule commits successfully outside the window, when FEAT-10.SPEC-004 completes, then this trigger fires FEAT-08.SPEC-004's outside-window reschedule variant to Riley, showing the new time and the carried-over deposit.

**FEAT-10.SPEC-006-AC-04:** Given Riley's reschedule commits successfully inside the window (late reschedule, compound outcome), when FEAT-10.SPEC-004 completes, then Riley receives the notice that her original deposit is kept, reported separately from the new booking's own confirmation once its deposit is paid.

**FEAT-10.SPEC-006-AC-05:** Given a client-initiated cancellation or reschedule commits successfully, when the commit completes, then Talia receives a Pro-facing change notice reflecting the update, via FEAT-08.SPEC-005.

**FEAT-10.SPEC-006-AC-06:** Given FEAT-10.SPEC-004's commit is rejected by a conflicting Pro-side transition, when the rejection occurs, then this spec triggers no notice for the rejected attempt.

**FEAT-10.SPEC-006-AC-07:** Given Riley has revoked texting consent since booking, when her cancellation or reschedule confirmation is triggered, then it is sent by email, honoring her current consent state.

**FEAT-10.SPEC-006-AC-08:** Given a text confirmation to Riley fails to deliver, when FEAT-08.SPEC-009's retry-then-fallback runs, then Riley still receives the confirmation by email.

**FEAT-10.SPEC-006-AC-09:** Given Riley taps "Manage my booking" from her confirmation, when the tap registers, then the client_change_notice_cta_tapped event fires and she is taken to FEAT-06.SPEC-004 for the affected booking.

**FEAT-10.SPEC-006-AC-10:** Given an automatic refund from Riley's outside-window cancellation cannot complete immediately, when FEAT-09 sets the Deposit Transaction to Refund in Progress, then Riley receives FEAT-08.SPEC-004's refund-in-progress notice as this trigger's continuation.

**FEAT-10.SPEC-006-AC-11:** Given Riley's cancellation frees a slot that a waitlisted client later claims, when the cancellation notice is delivered, then it is not held or delayed waiting for FEAT-20's waitlist notification to complete first.

**FEAT-10.SPEC-006-AC-12:** Given both the client confirmation and Pro change notice are due for the same commit event, when FEAT-10.SPEC-004 completes, then both are triggered from that single event, never batched together or deduplicated against each other, since they serve two distinct recipients.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (text, email) | 2 |
| Trigger Paths | 2 (cancel commit, reschedule commit) | 2 |
| Preference States | 2 (consent granted, consent revoked) | 2 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |


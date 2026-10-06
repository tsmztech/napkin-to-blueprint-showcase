---
document_type: spec
spec_type: screen
spec_id: FEAT-16.SPEC-001
spec_name: Booking Activity Timeline
spec_slug: booking-activity-timeline
parent_feature: FEAT-16
parent_feature_name: Booking & Payment Activity Record
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 19
---

# Screen Spec: Booking Activity Timeline

## Overview

**Name:** Booking Activity Timeline
**ID:** FEAT-16.SPEC-001
**Type:** Screen
**Purpose:** The Pro (or Platform Operator Support, read-only) opens a single booking's full, ordered event history to reference when preparing to respond to a client dispute, and can act from it by starting a goodwill refund or requesting a dispute evidence summary.
**Parent Feature:** FEAT-16 -- Booking & Payment Activity Record

## Scope and Non-Goals

**In Scope:**
- Rendering the ordered event list for one Booking's Activity Events, including the policy version shown/acknowledged, appointment time, message delivery events (including gaps), cancellation/reschedule, no-show mark, deposit outcome, and any dispute event
- Distinguishing the Pro's own view from Platform Operator (Support)'s read-only, private-notes-excluded view of the same screen
- An entry point from this screen into the goodwill refund flow and into the dispute evidence download

**Non-Goals:**
- Editing, correcting, or deleting any Activity Event -- excluded per FEAT-16.SPEC-005 (Validation & Limits: "append-only and immutable once written"); this screen has no write path to an entry, ever, for any role
- Executing the goodwill refund itself -- excluded per the Internal Dependency Map; this screen only navigates to FEAT-30.SPEC-003 (Goodwill Deposit Refund), which owns the refund logic
- Assembling or generating the downloadable dispute evidence file -- excluded per the Internal Dependency Map; this screen only navigates to FEAT-16.SPEC-004 (Dispute Summary Download), which owns the assembly
- A cross-booking search, filter, or list of activity records -- excluded per feature-overview.md's Non-Goals: this feature is scoped to one booking's timeline at a time; a cross-booking view belongs to Booking & Revenue Insights (FEAT-25)

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-12.SPEC-003 (Past Bookings Browse) | Pro selects a past booking from the browse list | The selected Booking's identifier |
| FEAT-12.SPEC-002 / FEAT-12.SPEC-005 (Attention List / Attention Flag Aggregation) | Pro taps a card-issuer dispute flag on the dashboard | The disputed Booking's identifier, with the screen opening scrolled to the dispute event |
| FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) (Platform Support Read-Only Access) | Support opens a disputed booking from the support view of a Pro's account, per a help request | The Booking's identifier; the screen renders in the Support read-only variant |
| FEAT-19.SPEC-003 (Support Access Log) | Support taps an access-log entry that references a booking | The referenced Booking identifier; Support read-only variant |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full timeline for any of her own bookings, including her own private client notes surfaced elsewhere on the booking (not duplicated on this screen) | Navigate to the goodwill refund (FEAT-30.SPEC-003) and to the dispute summary download (FEAT-16.SPEC-004) when the booking is flagged disputed; no edit action exists for any role | -- |
| Platform Operator (Support) | Full timeline for the one Pro account they are actively viewing under a help request, with the Pro's private client notes excluded (XBR-24) | View-only -- no refund entry point, no download entry point, no action of any kind (SC-05) | If Support attempts to reach a refund or download control (neither is rendered for this role, so no control exists to attempt): the screen carries no such element for Support in the first place |
| The Client (Riley) | None -- this internal operational record is never shown to the Client | None | Riley's own booking view shows her outcomes (confirmation, deposit status) but has no link, deep link, or route into this screen; a direct attempt to reach it is treated as an unauthenticated request |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen; after signing in, the user lands on FEAT-12.SPEC-001 (Today's & Upcoming Schedule), not this screen |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- no in-progress work exists on this read-only screen to preserve; after re-authentication the user returns to their prior context (dashboard or support view) |

## Layout and Content

**Header:** Booking summary bar showing the client's name, service, and appointment date/time (Pro's timezone), with a back arrow (returns to the entry source: FEAT-12.SPEC-003, the attention list, or the support view). When viewed by Support, the header additionally shows a small "Support view -- read-only" label.

**Body:** A single-column, reverse-chronological (most recent first) ordered list of Activity Event entries. Each entry shows:
- A timestamp (Pro's timezone, per XBR-25)
- An actor indicator (Client, Pro, "Chairtime" for automatic entries, or "Support view" for a logged support access)
- A plain-language description of the event (e.g., "Deposit paid," "Policy shown and acknowledged: {version}," "Text reminder sent," "Marked no-show," "Deposit kept per cancellation policy")
- Where the event is a message-delivery gap (a failed text that fell back to email, per XBR-17), the entry renders plainly inline -- e.g., "Text reminder failed to send; sent by email instead" -- never hidden or smoothed over
- Where the event is the card-issuer dispute (FEAT-16.SPEC-003), the entry is visually distinguished (e.g., a flagged marker) and sits inline in chronological order like every other entry

At the top of the body, above the first (most recent) entry, a persistent banner appears only when the booking carries an active dispute: "This booking has a card-issuer dispute" with a "Download evidence summary" action (Pro only; navigates to FEAT-16.SPEC-004).

At the bottom of the body, a "Refund as goodwill" action appears (Pro only, and only while the booking's deposit has not already reached a terminal refunded/forfeited-and-undone-unavailable state per FEAT-30.SPEC-003's own eligibility rules); navigates to FEAT-30.SPEC-003.

**Footer:** None.

Support's view omits: the dispute banner's download action, the goodwill refund action, and any entry whose details field would surface the Pro's private client note (per XBR-24 and FEAT-16.SPEC-005) -- such entries render with their non-note details intact and the note portion simply absent, never a placeholder.

### Responsive Behavior

- **Compact breakpoint:** Single-column list as described, full width; the header summary bar wraps to two lines if the client name and service do not fit on one.
- **Medium size class and above:** The list remains single-column, capped at a consistent platform-wide reading width and horizontally centered; no structural change beyond width capping.
- **Dispute banner and goodwill refund action:** Remain full-width and pinned at their respective ends of the list at every size class.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to the entry source (FEAT-12.SPEC-003, attention list, or support view) | Screen closes | Animated transition back |
| Activity Event entry | Tap | Display-only -- entries are not individually interactive beyond what is already shown inline (per the Entity-Lifecycle Coverage Matrix: "An Activity Event is never opened individually") | None | None |
| "Download evidence summary" (Pro only, disputed bookings only) | Tap | Navigate to FEAT-16.SPEC-004 (Dispute Summary Download), carrying the Booking's identifier | Screen closes | Animated transition to the download flow |
| "Refund as goodwill" (Pro only) | Tap | Navigate to FEAT-30.SPEC-003 (Goodwill Deposit Refund), carrying the Booking's identifier | Screen closes | Animated transition to the refund flow |

### Accessibility Notes

- **Focus order:** Back arrow -> dispute banner (when present) -> timeline entries in displayed order (most recent first) -> "Refund as goodwill" action (when present).
- **Dynamic content announcements:** Because this is a read-only, load-once screen, no validation or save-state announcements apply; if new Activity Events are written by another process while the screen is open, no live update occurs (see States, Offline/Degraded) so nothing is announced mid-view.
- **Keyboard alternatives:** Every action on this screen (back, download, refund) is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loaded (default) | Full ordered timeline rendered as described | Screen opens for a booking with at least one Activity Event | N/A -- this is a snapshot read-only screen; the state persists until the user navigates away |
| Loading | N/A -- feature-overview.md's Non-Functional Notes state this is a small per-booking dataset that "loads instantly"; no loading state beyond an instantaneous local render is expected | -- | -- |
| Error | N/A -- this is a read-only, append-only log; there is no user-facing write path on this screen to fail, and a read failure is treated as the Offline/Degraded case below rather than a distinct error state | -- | -- |
| Gap present | Same as Loaded, with one or more entries rendering a plainly-visible gap (e.g., a failed-then-fallback message event) inline, per XBR-17 | The booking's timeline includes at least one such event | N/A -- persists for the life of the timeline; a gap is a permanent, immutable fact once recorded |
| Offline/Degraded | The most recently loaded timeline for this booking remains viewable read-only; the dispute-download and goodwill-refund actions (which require a live connection to their own flows) are disabled with the note "This action needs a connection." | Connectivity is lost after the timeline has loaded at least once | Connectivity restored -- the disabled actions re-enable; the timeline itself does not need to reload since it is a small, immutable dataset already held |

## Validation Rules

N/A -- this is a read-only screen with no user input fields; validation for the actions it navigates to (goodwill refund, dispute download) is owned entirely by FEAT-30.SPEC-003 and FEAT-16.SPEC-004 respectively.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-12.SPEC-003 (Past Bookings Browse), the attention list, or the support view (whichever was the entry source) | FEAT-12 (or FEAT-19) |
| "Download evidence summary" tap | FEAT-16.SPEC-004 (Dispute Summary Download) | -- |
| "Refund as goodwill" tap | FEAT-30.SPEC-003 (Goodwill Deposit Refund) | FEAT-30 |

## Data Model

**Creates:** None -- this screen never writes an Activity Event; all entries are written by FEAT-16.SPEC-002 (and, for support-view entries handed off by FEAT-19.SPEC-002, through FEAT-16.SPEC-002 as the sole writer).
**Reads:** Activity Event (event_type, time, actor, details) for the one Booking, ordered by time; Booking (service, start_time, client reference, state, policy_version); Message (type, channel, send time, delivery_status) for delivery-event rendering; Cancellation Policy (version, plain_language_wording) for the policy-acknowledgment entry; Deposit Transaction (status, outcome_reason, Disputed overlay) for outcome and dispute-flag rendering.
**Updates:** None.
**Deletes:** None.

## Business Rules

- No role, including the Pro, can edit or delete an entry rendered on this screen -- governed entirely by FEAT-16.SPEC-005 (Validation & Limits: append-only and immutable).
- Support's view excludes the Pro's private client notes from any entry's details, per XBR-24 and FEAT-16.SPEC-005 (Authorization Rules).
- A message-delivery gap renders inline exactly as it occurred -- it is never smoothed over, summarized away, or hidden, per XBR-17.
- The policy version and wording shown to the client at booking (FEAT-09.SPEC-002) is rendered as its own distinct entry, never merged into the "created" entry, so the exact version and acknowledgment time are independently visible.
- The "Refund as goodwill" action is available only while FEAT-30.SPEC-003's own eligibility rules permit a goodwill refund on this booking; this screen defers entirely to that spec's eligibility determination rather than re-deriving it.
- The dispute banner and its download action appear only while the Deposit Transaction carries the Disputed overlay set by FEAT-16.SPEC-003.

## Edge Cases

- **A booking has no events beyond "created"** -- The timeline shows a single entry; no empty state applies (a timeline only exists for bookings that have happened, per feature-overview.md's States field).
- **Support opens a timeline for a booking with a private Pro note attached to an event** -- The entry renders with its non-note details intact; the note content is simply absent from the entry, never replaced with a placeholder like "[hidden]" (per XBR-24).
- **The Pro taps "Refund as goodwill" on a booking whose deposit was already refunded** -- The action is not shown in this case per FEAT-30.SPEC-003's eligibility rules; if the underlying state changes between page load and tap (see concurrent-access entry below), the Pro is routed into FEAT-30.SPEC-003, which independently re-checks eligibility and refuses with its own current-state message.
- **A new Activity Event is written (by another feature acting on this booking) while the Pro has this screen open** -- No live update occurs; the screen shows a snapshot as of load time (per the Offline/Degraded state's rationale that this is a small, rarely-changing dataset). Reopening the screen shows the new entry.
- **Concurrent access: the Deposit Transaction's terminal outcome changes between this screen's load and the Pro tapping an action into FEAT-30.SPEC-003** -- No conflict resolution occurs on this screen itself, because it never writes to the Deposit Transaction; FEAT-30.SPEC-003 (per the dependency map's Contention note: reject-with-refresh, one terminal outcome per deposit) is the one that detects and resolves the conflict when the Pro's action reaches it.
- **A dispute notice arrives while the Pro is already viewing this booking's timeline** -- No live update occurs (per Offline/Degraded rationale); the dispute banner and its download action appear the next time the screen is opened.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-16.SPEC-002 (Activity Event Recording) | References (inbound) | This screen renders the entries that automation writes |
| FEAT-16.SPEC-003 (Card-Issuer Dispute Integration) | References (inbound) | The dispute banner and flagged entry reflect this spec's write |
| FEAT-16.SPEC-004 (Dispute Summary Download) | Navigation (outbound) | "Download evidence summary" navigates here |
| FEAT-16.SPEC-005 (Activity Record Immutability & Visibility Rules) | References (inbound) | Governs the absence of any edit control and the Support private-notes exclusion |
| FEAT-12.SPEC-003 (Past Bookings Browse) | Navigation (inbound) | Pro arrives from a selected past booking |
| FEAT-12.SPEC-002 / FEAT-12.SPEC-005 (Attention List / Attention Flag Aggregation) | Navigation (inbound) | Pro arrives by tapping a dispute flag |
| FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry), FEAT-19.SPEC-002 (Support View Logging), FEAT-19.SPEC-003 (Support Access Log) -- within FEAT-19 (Platform Support Read-Only Access) | Navigation (inbound) | Support arrives from the support view of a Pro's account |
| FEAT-30.SPEC-003 (Goodwill Deposit Refund) | Navigation (outbound) | "Refund as goodwill" navigates here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| activity_record_viewed | viewer role (Pro / Support), booking has an active dispute (yes/no) | The timeline finishes loading | N/A -- no success-metrics.md metric is connected to FEAT-16; this signal (named in feature-overview.md's Signals field) is retained for operational observability of how often the record is consulted, most notably around disputes |
| dispute_download_entry_tapped | -- | The Pro taps "Download evidence summary" | N/A -- no connected success-metrics.md metric; retained to observe how often the flagged record leads into evidence assembly |
| goodwill_refund_entry_tapped | -- | The Pro taps "Refund as goodwill" from this screen | N/A -- no connected success-metrics.md metric; the resulting refund outcome is measured downstream by FEAT-30.SPEC-011's own signals |

## Acceptance Criteria

**FEAT-16.SPEC-001-AC-01:** Given Talia opens a past booking from FEAT-12.SPEC-003, when the timeline finishes loading, then she sees every Activity Event for that booking in reverse-chronological order, including the policy version shown and acknowledged and its acknowledgment time.

**FEAT-16.SPEC-001-AC-02:** Given Talia is viewing a booking's timeline where a reminder text failed and fell back to email, when she looks at that entry, then it reads plainly, for example "Text reminder failed to send; sent by email instead," rather than being hidden.

**FEAT-16.SPEC-001-AC-03:** Given Talia is viewing a booking whose Deposit Transaction carries the Disputed overlay, when the screen loads, then a banner "This booking has a card-issuer dispute" appears at the top with a "Download evidence summary" action.

**FEAT-16.SPEC-001-AC-04:** Given Talia taps "Download evidence summary" on a disputed booking's timeline, when the tap registers, then she is navigated to FEAT-16.SPEC-004 with that booking's identifier carried along.

**FEAT-16.SPEC-001-AC-05:** Given Talia is viewing a booking eligible for a goodwill refund, when she taps "Refund as goodwill," then she is navigated to FEAT-30.SPEC-003 with that booking's identifier carried along.

**FEAT-16.SPEC-001-AC-06:** Given Talia is viewing any booking's timeline, when she looks for an edit or delete control on any entry, then none exists anywhere on the screen.

**FEAT-16.SPEC-001-AC-07:** Given Platform Operator Support opens a booking's timeline after Talia's help request, when the screen renders, then it shows the "Support view -- read-only" label, omits the goodwill-refund action and the dispute-download action entirely, and excludes Talia's private client notes from any entry.

**FEAT-16.SPEC-001-AC-08:** Given Riley (the Client) attempts to reach this screen, when the attempt is made, then she is treated as an unauthorized/unauthenticated user and is not shown any part of this timeline.

**FEAT-16.SPEC-001-AC-09:** Given an unauthenticated visitor reaches this screen's route directly, when the screen would otherwise load, then they are redirected to the Pro sign-in screen and land on FEAT-12.SPEC-001 after signing in, not on this timeline.

**FEAT-16.SPEC-001-AC-10:** Given Talia's session expires while this screen is open, when she next interacts with it, then the dialog "Your session has expired. Sign in to continue." appears, and after re-authenticating she returns to her prior context.

**FEAT-16.SPEC-001-AC-11:** Given Talia loses connectivity after this booking's timeline has already loaded, when she looks at the screen, then the previously loaded entries remain visible read-only and the dispute-download and goodwill-refund actions (if present) show "This action needs a connection." and are disabled.

**FEAT-16.SPEC-001-AC-12:** Given Talia is viewing a booking with only a "created" event so far, when the screen loads, then exactly that one entry is shown with no empty-state message.

**FEAT-16.SPEC-001-AC-13:** Given Talia has this screen open and another feature writes a new Activity Event to this same booking in the background, when Talia continues viewing without navigating away, then the new entry does not appear until she reopens the screen.

**FEAT-16.SPEC-001-AC-14:** Given Talia's deposit for this booking was already refunded before she opens the timeline, when the screen loads, then the "Refund as goodwill" action is not shown, per FEAT-30.SPEC-003's eligibility rules.

**FEAT-16.SPEC-001-AC-15:** Given Talia taps "Refund as goodwill" and the deposit's state changed to a terminal outcome between load and tap, when FEAT-30.SPEC-003 re-checks eligibility, then that spec refuses with its own current-state message rather than this screen silently proceeding.

**FEAT-16.SPEC-001-AC-16:** Given Support is viewing a timeline entry whose details would normally include Talia's private client note, when the entry renders, then the note content is simply absent -- no placeholder text appears in its place.

**FEAT-16.SPEC-001-AC-17:** Given Talia opens this screen and taps the back arrow, when the tap registers, then she returns to whichever entry point she arrived from (Past Bookings Browse, the attention list, or nowhere else).

**FEAT-16.SPEC-001-AC-18:** Given Talia is viewing a booking's timeline, when she taps any individual entry, then nothing happens -- entries are display-only and are never opened individually.

**FEAT-16.SPEC-001-AC-19:** Given Talia is viewing a booking's timeline that includes a cancellation, a no-show mark, and a deposit outcome, when she reads the list, then each of those three events appears as its own distinct, correctly ordered entry rather than being merged into one summary line.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 4 | 4 |
| States | 4 (loaded, gap present, offline/degraded, N/A loading/error justified) | 4 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |

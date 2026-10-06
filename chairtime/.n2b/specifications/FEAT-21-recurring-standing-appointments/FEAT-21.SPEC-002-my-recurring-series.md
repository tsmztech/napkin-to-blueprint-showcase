---
document_type: spec
spec_type: screen
spec_id: FEAT-21.SPEC-002
spec_name: My Recurring Series
spec_slug: my-recurring-series
parent_feature: FEAT-21
parent_feature_name: Recurring/Standing Appointments
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

# Screen Spec: My Recurring Series

## Overview

**Name:** My Recurring Series
**ID:** FEAT-21.SPEC-002
**Type:** Screen
**Purpose:** Lets Riley see her standing appointment series and its upcoming occurrences grouped together, and cancel the whole series or just one occurrence, with no extra UI at all when she holds no series.
**Parent Feature:** FEAT-21 -- Recurring/Standing Appointments

## Scope and Non-Goals

**In Scope:**
- Listing every Recurring Series Riley holds with this Pro, each with its upcoming generated occurrences grouped underneath it
- Showing each occurrence's own status (upcoming, awaiting deposit, needs new time)
- Cancelling one occurrence, or the whole series, from this screen
- The fully-optional empty state when Riley holds no series

**Non-Goals:**
- Setting up a new series -- owned by FEAT-21.SPEC-001 (Set Up Recurring Series); this screen only manages series that already exist.
- Editing a series' interval -- excluded per this Brief's Non-Goals: a client who wants a different cadence cancels here and sets up a fresh series through FEAT-21.SPEC-001.
- Defining what cancellation actually does to the series and its occurrences -- owned by FEAT-21.SPEC-006 (Series & Occurrence Cancellation Rules); this screen only exposes the two cancel actions and shows their result.
- Viewing a single occurrence's full booking detail (receipt, deposit breakdown) -- remains FEAT-06's/FEAT-16's responsibility per the Entity-Lifecycle Coverage Matrix; this screen shows only what identifies an occurrence within its series group.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-06.SPEC-003 (My Bookings List) | Riley opens her own bookings view and taps the "Recurring appointments" row (shown by FEAT-06.SPEC-003 only when she holds a series) | None -- this screen loads all of Riley's own series with this Pro |
| FEAT-21.SPEC-001 (Set Up Recurring Series) | Riley completes setting up a new series | The just-created series, shown first in the list |
| FEAT-21.SPEC-007 (Occurrence Generated Notification) | Client taps "View my series" in an occurrence-generated notice | The client's series reference |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Full screen, her own series with this Pro only | Cancel one occurrence, or the whole series, for her own series only | -- |
| The Pro (Talia) | No | No | Talia never reaches this screen; her Full access to series tied to her own schedule is satisfied by FEAT-21.SPEC-010 (Pro Recurring Series Management), reached from FEAT-30's Pro booking detail |
| Platform Operator (Support) | No | No | This screen is client-facing; Support's View-only access to Recurring Series is surfaced through FEAT-19's own screen, never through this feature's screens (Capability Coverage Map) |
| Unauthenticated | No | No | Reached only via a client's own access link or manage-link session (FEAT-06); an unauthenticated visitor is shown the "request a new link" prompt rather than this screen |
| Expired session | No | No | If Riley's access link session has expired, she is shown "This link has expired. Request a new one to see your bookings." (FEAT-06.SPEC-001, Access Link Request) instead of this screen |

## Layout and Content

**Header:** Screen title "Recurring appointments" with a back arrow (returns to FEAT-06.SPEC-003, My Bookings List).

**Body:** One card per Recurring Series Riley holds with this Pro, each containing:
- Series summary line: "{service_name}, every {interval} weeks" and a "Cancel series" text action
- Below the summary, a list of the series' upcoming occurrences, each row showing: the occurrence's date and time, its status badge (Upcoming / Awaiting deposit / Needs new time), and a "Cancel this one" text action scoped to that single occurrence

Cards are ordered by the series' next upcoming occurrence date, soonest first.

**Footer:** None.

### Responsive Behavior

- **Compact size class:** Cards stack full width in a single column, as described above.
- **Medium size class and above:** Cards remain single-column, capped at a consistent platform-wide content width (the design layer's decision) and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-06.SPEC-003 (My Bookings List) | Screen closes | Animated transition back to the bookings list |
| "Cancel series" (per card) | Tap | Opens a confirmation dialog, then triggers FEAT-21.SPEC-006's whole-series cancellation rule if confirmed | Dialog appears; on confirm, the card is removed from the list once cancellation completes | Confirmation dialog: "Cancel this whole series? Your next {occurrence_count} upcoming appointments will be cancelled." with "Cancel Series" and "Keep Series" options; on success, toast "Series cancelled" |
| "Cancel this one" (per occurrence row) | Tap | Opens a confirmation dialog, then triggers FEAT-21.SPEC-006's single-occurrence cancellation rule if confirmed | Dialog appears; on confirm, the occurrence row is removed once cancellation completes, series card remains | Confirmation dialog: "Cancel this appointment on {occurrence_date}? The rest of your series will continue as usual." with "Cancel Appointment" and "Keep It" options; on success, toast "Appointment cancelled" |
| Occurrence status badge | Display only | No action -- read-only status indicator | None | Not interactive |

### Accessibility Notes

- **Focus order:** Back arrow -> each series card in list order -> within a card: series summary, "Cancel series", then each occurrence row's date/status and "Cancel this one" action, top to bottom.
- **Dynamic content announcements:** A confirmation dialog's text is announced to assistive technology on open; the "Series cancelled" and "Appointment cancelled" toasts are announced on success.
- **Keyboard alternatives:** Every cancel action and dialog choice is reachable by keyboard; there are no pointer-only gestures on this screen.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (default) | No extra UI at all -- the recurring-series section is not shown on FEAT-06.SPEC-003 and this screen is not reachable, since this Brief's States field defines the empty state as fully optional | Riley holds no Recurring Series with this Pro | Riley sets up a series through FEAT-21.SPEC-001 |
| Loaded | One or more series cards shown with their occurrence groups, as described in Layout and Content | Riley holds at least one series | Riley navigates away, or her last series is cancelled (returning to Empty) |
| Loading | A brief loading indicator in place of the card list | Screen first opens while series data is being fetched | Data loads (transition to Loaded) or fails (transition to Error) |
| Error | Error banner at the top with a Retry option; no card list shown | The series data fails to load | Riley taps Retry, or navigates away |
| Cancelling | The card or row being cancelled shows a brief loading indicator; its cancel action is disabled | Riley confirms a cancel dialog | Cancellation completes (row/card removed) or fails (Error) |
| Offline/Degraded | N/A -- requires connectivity for correctness, consistent with the rest of scheduling (per this Brief's States field); a cancel attempted without connectivity shows "You'll need to be online to do this. Please check your connection and try again." and nothing is submitted | Connectivity lost while attempting to cancel | Connectivity restored and Riley retries |

## Validation Rules

Validation governed by FEAT-21.SPEC-006 (Series & Occurrence Cancellation Rules). See that spec for the exact behavior of each cancel action, including the contention outcome when the Pro acts on the same series or occurrence at the same time.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-06.SPEC-003 (My Bookings List) | FEAT-06 (Client Booking Identity) |
| Successful series cancellation | This screen, with the cancelled card removed | -- |
| Successful occurrence cancellation | This screen, with the cancelled row removed | -- |

## Data Model

**Reads:** Recurring Series -- interval, originating service and time, state, generated occurrences (Riley's own series with this Pro only); Booking (occurrence) -- service, start_time, state, for each occurrence shown in a series group.
**Creates:** None.
**Updates:** Recurring Series -- state transitioned to Ended (via FEAT-21.SPEC-006, on whole-series cancel). Booking (occurrence) -- state transitioned to Cancelled (via FEAT-21.SPEC-006).
**Deletes:** None.

## Business Rules

- Both cancel actions are governed entirely by FEAT-21.SPEC-006 -- this screen never decides what cancellation does to a series or its occurrences, only exposes the two actions and reflects the result.
- The occurrence group list pattern (Brief's Shared UI Patterns) is followed exactly: cancelling one occurrence never reads as cancelling the series -- the two actions are visually and functionally distinct, with the series card remaining after a single-occurrence cancel.
- An occurrence's status badge reflects its Booking state as read from FEAT-21.SPEC-004 (generation) and FEAT-21.SPEC-005 (deposit lifecycle): Upcoming (Pending Payment, deposit not yet requested, or Confirmed), Awaiting deposit (deposit link sent, unpaid), Needs new time (usual slot unavailable, per FEAT-21.SPEC-004's conflict-handling path).

## Edge Cases

- **Riley has no upcoming occurrences left in a series (all generated occurrences have completed or been cancelled, but the series is still Active)** -- The series card still shows with an empty occurrence list and the note "Your next appointment will appear here once it's scheduled," since the series itself remains Active and will keep generating occurrences as the horizon rolls forward.
- **The Pro cancels the same series Riley is viewing, at the same time Riley taps "Cancel series" (concurrent-edit conflict)** -- Per the dependency map's Recurring Series Contention note, the resolution is reject-with-refresh: whichever cancellation commits first wins, and the other party's screen refreshes to show the already-cancelled state before their action completes; Riley sees "This series was just cancelled." instead of the usual success toast if the Pro's cancellation committed first.
- **Riley taps "Cancel series" or "Cancel this one" twice rapidly** -- The second tap is ignored while the first cancellation is in progress (action disabled during the Cancelling state).
- **Network failure during a cancel action** -- Error banner: "Could not complete that cancellation. Check your connection and try again." with a Retry button; the card/row remains in its pre-cancellation state.
- **An occurrence Riley tries to cancel was released unpaid moments earlier by FEAT-21.SPEC-005** -- The action is refused with refresh: "This appointment is no longer active." and the row updates to reflect the release, since there is nothing left for Riley's cancel action to act on.
- **Riley reopens this screen immediately after cancelling an occurrence or series** -- The list reflects the just-completed cancellation, not a stale cached view.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-003 (My Bookings List) | Navigation (inbound/outbound) | Riley reaches this screen from the "Recurring appointments" row on her own bookings view; the back arrow returns her there |
| FEAT-21.SPEC-001 (Set Up Recurring Series) | Navigation (inbound) | A newly created series lands Riley here |
| FEAT-21.SPEC-006 (Series & Occurrence Cancellation Rules) | References (outbound) | Governs the effect and contention outcome of both cancel actions |
| FEAT-21.SPEC-004 (Occurrence Generation & Conflict Handling) | References (inbound) | Supplies each occurrence's Upcoming/Needs-new-time status |
| FEAT-21.SPEC-005 (Occurrence Deposit Request & Release) | References (inbound) | Supplies each occurrence's Awaiting-deposit status |
| FEAT-21.SPEC-010 (Pro Recurring Series Management) | References (sibling) | The Pro-side counterpart showing the same series and occurrences; a Pro cancellation there refreshes this screen per the contention rule |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| recurring_occurrence_cancelled | series reference | Riley cancels a single occurrence from this screen | N/A -- Stage 2 names no distinct signal for cancelling a single occurrence; per this Brief's Signals note, this action reuses the generic booking-cancelled signal the owning cancellation feature already emits for any Booking, rather than a new series-specific signal invented here |
| recurring_series_cancelled | interval_weeks, occurrences_cancelled_count | Riley cancels the whole series from this screen | supports success-metrics.md: "Self-Service Reschedule Rate" |
| recurring_series_list_viewed | series_count | This screen loads with one or more series | N/A -- no Stage 2 metric measures view frequency for this screen; retained so usage of the management surface is observable |

## Acceptance Criteria

**FEAT-21.SPEC-002-AC-01:** Given Riley holds no Recurring Series with Talia, when she opens FEAT-06.SPEC-003 (My Bookings List), then no recurring-series section or extra UI appears at all.

**FEAT-21.SPEC-002-AC-02:** Given Riley holds one series with two upcoming occurrences, when she opens this screen, then she sees one card showing the series' interval and both occurrences listed underneath with their status badges.

**FEAT-21.SPEC-002-AC-03:** Given Riley taps "Cancel this one" on an occurrence, when she confirms "Cancel Appointment" in the dialog, then that occurrence's Booking is cancelled per FEAT-21.SPEC-006, its row is removed, the toast "Appointment cancelled" appears, and the series card remains with its other occurrences.

**FEAT-21.SPEC-002-AC-04:** Given Riley taps "Cancel series," when she confirms "Cancel Series" in the dialog, then the series is ended and every not-yet-occurred occurrence is cancelled per FEAT-21.SPEC-006, the card is removed, and the toast "Series cancelled" appears.

**FEAT-21.SPEC-002-AC-05:** Given Riley opens the "Cancel this one" dialog and taps "Keep It" instead, then the dialog closes and no cancellation occurs.

**FEAT-21.SPEC-002-AC-06:** Given an occurrence's deposit link has been sent and is unpaid, when Riley views this screen, then that occurrence shows the "Awaiting deposit" status badge.

**FEAT-21.SPEC-002-AC-07:** Given an occurrence's usual slot is no longer available per FEAT-21.SPEC-004, when Riley views this screen, then that occurrence shows the "Needs new time" status badge.

**FEAT-21.SPEC-002-AC-08:** Given Talia cancels the same series Riley is viewing at effectively the same moment Riley taps "Cancel series," when both commit, then the first to commit wins, and the other party sees the refreshed, already-cancelled state -- for example, Riley sees "This series was just cancelled." if Talia's action committed first.

**FEAT-21.SPEC-002-AC-09:** Given Riley taps "Cancel series" and the operation fails due to a network error, then an error banner reads "Could not complete that cancellation. Check your connection and try again." and the card remains in its pre-cancellation state.

**FEAT-21.SPEC-002-AC-10:** Given Riley loses connectivity and attempts a cancel action, then she sees "You'll need to be online to do this. Please check your connection and try again." and nothing is submitted.

**FEAT-21.SPEC-002-AC-11:** Given a series has no upcoming occurrences left but remains Active, when Riley views this screen, then the series card shows with the note "Your next appointment will appear here once it's scheduled."

**FEAT-21.SPEC-002-AC-12:** Given an occurrence Riley attempts to cancel was released unpaid moments earlier, when her cancel action is evaluated, then it is refused with "This appointment is no longer active." and the row refreshes to reflect the release.

**FEAT-21.SPEC-002-AC-13:** Given Riley cancels a single occurrence, when she reopens this screen, then the emitted event reuses the generic booking-cancelled signal, and no series-specific single-occurrence signal is recorded.

**FEAT-21.SPEC-002-AC-14:** Given Talia wants to view or manage her own schedule's series, when she looks for this screen, then it is not reachable to her -- her equivalent view is FEAT-21.SPEC-010 (Pro Recurring Series Management).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 3 | 3 |
| States | 5 (empty, loaded, loading, error, cancelling) plus offline N/A | 6 |
| Business Rules | 3 | 3 |
| Edge Cases | 6 | 6 |

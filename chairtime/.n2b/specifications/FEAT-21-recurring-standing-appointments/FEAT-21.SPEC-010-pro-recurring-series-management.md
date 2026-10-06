---
document_type: spec
spec_type: screen
spec_id: FEAT-21.SPEC-010
spec_name: Pro Recurring Series Management
spec_slug: pro-recurring-series-management
parent_feature: FEAT-21
parent_feature_name: Recurring/Standing Appointments
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-28
acceptance_criteria_count: 22
---

# Screen Spec: Pro Recurring Series Management

## Overview

**Name:** Pro Recurring Series Management
**ID:** FEAT-21.SPEC-010
**Type:** Screen
**Purpose:** Lets Talia set up a standing appointment for a client while the client is at the chair, then see that client's series and upcoming occurrences on her own schedule, and cancel one occurrence or end the whole series.
**Parent Feature:** FEAT-21 -- Recurring/Standing Appointments

## Scope and Non-Goals

**In Scope:**
- Setting up a Recurring Series for a client from one of that client's existing bookings on Talia's schedule, by choosing a repeat interval of every 1 to 12 weeks
- Listing the client's Active series on Talia's schedule, each with its upcoming generated occurrences grouped underneath and each occurrence's own status
- Cancelling one upcoming occurrence, or ending the whole series, on the client's behalf
- Showing the refreshed series when the client changes the same series or occurrence at the same moment
- The empty, loading, error, and offline/degraded behavior of this connectivity-dependent screen

**Non-Goals:**
- Validating the interval and the booking-horizon ceiling -- owned by FEAT-21.SPEC-003 (Recurring Series Setup & Generation Limits); this screen only surfaces its result and error message.
- Defining what cancelling one occurrence or the whole series does, and how a simultaneous client and Pro change resolves -- owned by FEAT-21.SPEC-006 (Series & Occurrence Cancellation Rules); this screen only exposes the two actions and reflects their result.
- Generating occurrences, resolving a conflicted occurrence time, or requesting and releasing occurrence deposits -- owned by FEAT-21.SPEC-004 and FEAT-21.SPEC-005; this screen shows their results as occurrence statuses and never picks a replacement time on the client's behalf, because the pick-a-new-time flow belongs to the client (FEAT-21.SPEC-008).
- Editing a series' interval after creation -- excluded per this Brief's Non-Goals: the feature names only setup, group viewing/management, and cancellation; a different cadence means ending the series here and setting up a fresh one from a later booking.
- A pause action for a series -- excluded per this Brief's Non-Goals: no Stage 2 flow describes pausing, so only Active and Ended states exist.
- Recurring-series management inside FEAT-30's own screens -- excluded per FEAT-30's Non-Goals; FEAT-30 links to this screen instead.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-30.SPEC-001 (Cancel Booking (Pro-Initiated)) -- within FEAT-30 (Pro Booking Management), Pro booking detail outbound link | Talia opens a client's booking on her schedule and taps the "Recurring series" link ("Repeat this booking" when the booking has no series, "Manage recurring series" when it belongs to one) | Booking reference and client reference; the screen opens on the set-up form when that booking is eligible to originate a series, otherwise on the client's series |
| FEAT-21.SPEC-009 (Occurrence Deposit Lifecycle Notification) | Talia taps "View series" in the release notice for an unpaid occurrence | The released occurrence's series reference |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen, for series and bookings tied to her own schedule only | Set up a series for a client, cancel one occurrence, end the whole series | A reference to a series or booking that is not on her schedule shows "This series isn't on your schedule." with a single "Back" option; nothing about the other Pro's client is shown |
| The Client (Riley) | No | No | No control on any client-facing surface reaches this screen; Riley's own equivalent screens are FEAT-21.SPEC-001 (set up) and FEAT-21.SPEC-002 (view and cancel), which are reached through her own booking confirmation and bookings view |
| Platform Operator (Support) | No | No | This screen is Pro-facing and offers write actions; Support's view-only access to Recurring Series is surfaced through FEAT-19's own screen, never through this feature's screens |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29); no client or series data is shown, and after signing in Talia lands on FEAT-12 (Pro Daily Schedule Dashboard) |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- a chosen but unsubmitted interval is preserved and restored once she re-authenticates; no cancellation is ever completed on an expired session |

## Layout and Content

**Header:** Back arrow (returns to where Talia came from) and the title "Standing appointments" with the client's name ("{client_name}") beneath it.

**Body:** Two regions, top to bottom.

1. **Set-up region** (shown only when the entry booking is eligible, see Business Rules):
   - Booking summary line, read-only: "{service_name}, {booking_date} at {booking_time}" -- the service and time the series will repeat.
   - **Repeat interval** (numeric stepper, required): whole weeks from 1 to 12, labeled "Repeat every ___ weeks". No value is pre-selected.
   - Preview line, read-only, shown once an interval is chosen: "First repeat: {first_due_date}. It is booked automatically once that date is inside your booking window."
   - Primary "Set up standing appointment" button, disabled until an interval is chosen.
2. **Series region:** one card per Active Recurring Series that the client holds on Talia's schedule (normally zero or one). Each card contains:
   - Series summary line: "{service_name}, every {interval} weeks" and an "End series" text action.
   - A list of the series' upcoming occurrences, ordered by date, soonest first. Each row shows the occurrence's date and time, a status badge (Upcoming / Awaiting deposit / Needs new time), and a "Cancel this one" text action scoped to that single occurrence.
   - When a series has no upcoming occurrences yet, the note "The next appointment will appear here once it's scheduled."

When both regions apply, the set-up region sits above the series region. When the entry booking is not eligible and the client holds no series, the body shows the empty state (see States).

**Footer:** None.

### Responsive Behavior

- **Compact size class:** Both regions stack full width in a single column; the primary set-up button sits at the bottom of the set-up region.
- **Medium size class and above:** The column stays single, capped at a consistent platform-wide content width (the design layer's decision) and horizontally centered; no structural change beyond width capping.
- **Occurrence rows:** The date/status pair and the "Cancel this one" action share a row at every size class; at compact width the action wraps beneath the date/status pair.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to the originating screen: FEAT-30.SPEC-001 (Cancel Booking (Pro-Initiated)) booking detail, or FEAT-12 (Pro Daily Schedule Dashboard) when Talia arrived from a FEAT-21.SPEC-009 notice | Screen closes | Animated transition back |
| Repeat interval stepper | Increment/decrement or type a value | Captures the chosen interval | Stepper shows the value; the preview line appears; "Set up standing appointment" becomes enabled | Stepper and preview show the new values |
| "Set up standing appointment" button | Tap | 1. Validate the interval via FEAT-21.SPEC-003. 2. If valid, create the Recurring Series in state Active from the entry booking, with originating service and time copied from that booking. 3. Hand off to FEAT-21.SPEC-004 to generate the first occurrence(s). | Button shows a loading state; on success the set-up region is replaced by the new series card | Success: toast "Standing appointment set up -- every {interval} weeks". Failure: field-level or banner message (see States) |
| "Set up standing appointment" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |
| "End series" (per card) | Tap | Opens a confirmation dialog; on confirm, triggers FEAT-21.SPEC-006's whole-series cancellation as The Pro | Dialog appears; on confirm the card is removed once the cancellation completes | Dialog: "End this whole series? {occurrence_count} upcoming appointments for {client_name} will be cancelled and their deposits refunded in full." with "End Series" and "Keep Series". Success toast: "Series ended" |
| "Cancel this one" (per occurrence row) | Tap | Opens a confirmation dialog; on confirm, triggers FEAT-21.SPEC-006's single-occurrence cancellation as The Pro | Dialog appears; on confirm the row is removed once the cancellation completes; the series card remains | Dialog: "Cancel the {occurrence_date} appointment for {client_name}? The rest of the series continues and any deposit is refunded in full." with "Cancel Appointment" and "Keep It". Success toast: "Appointment cancelled" |
| Occurrence status badge | Display only | No action -- read-only status indicator | None | Not interactive |
| Preview line, booking summary line | Display only | No action | None | Not interactive |

### Accessibility Notes

- **Focus order:** Back arrow -> Repeat interval stepper -> "Set up standing appointment" -> each series card in order: series summary, "End series", then each occurrence row's date/status and "Cancel this one", top to bottom.
- **Dynamic content announcements:** The validation message, the set-up toast, both cancellation toasts, the refreshed-series notices, and every confirmation dialog's text are announced to assistive technology on appearance; a validation message is programmatically associated with the stepper.
- **Focus management:** After a dialog closes, focus returns to the control that opened it; after a successful set-up, focus moves to the new series card's summary line.
- **Keyboard alternatives:** The stepper is operable by arrow keys or direct numeric entry; every action and dialog choice is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | A loading indicator in place of both regions | Screen opens while the booking and the client's series are being fetched | Data loads (Set-up ready, Loaded, or Empty) or fails (Error) |
| Set-up ready | Set-up region with the stepper unset and the button disabled; series region shown below if the client also holds a series | The entry booking is eligible and the client holds no series, or holds one from another booking | Talia chooses an interval, or leaves |
| Filling | Stepper shows the chosen value, preview line shown, button enabled | Talia sets an interval | Talia submits or leaves |
| Submitting | Button shows a loading spinner; stepper disabled | Talia taps "Set up standing appointment" | Validation and creation complete or fail |
| Validation Error | Stepper error state with "Choose a repeat interval between 1 and 12 weeks." below it; button re-enabled; attempted value stays visible | FEAT-21.SPEC-003 rejects the interval | Talia corrects the value |
| Loaded | Series region with one or more cards and their occurrence groups | The client holds at least one Active series on Talia's schedule | Talia leaves, or the last series ends (returning to Empty) |
| Empty | Text "No standing appointments for {client_name}." and, when the entry booking is not eligible, one line stating why (for example "This booking has already started a standing appointment or was cancelled.") | The entry booking is not eligible and the client holds no Active series | Talia leaves; the state is never blocking |
| Cancelling | The card or row being cancelled shows a loading indicator and its action is disabled | Talia confirms a cancel dialog | Cancellation completes (row/card removed) or fails (Error) |
| Error | Error banner at the top with a Retry option; the chosen interval, if any, is preserved; regions that failed to load are not shown | A data load, series creation, or cancellation fails after passing validation | Talia taps Retry or leaves |
| Offline/Degraded | N/A -- requires connectivity for correctness, consistent with the rest of scheduling (this Brief's States field); an attempted set-up or cancellation shows "You'll need to be online to do this. Please check your connection and try again." and nothing is submitted or queued | Connectivity lost while attempting an action | Connectivity restored and Talia retries |

## Validation Rules

Validation of the repeat interval and the booking-horizon ceiling is governed by FEAT-21.SPEC-003 (Recurring Series Setup & Generation Limits). See that spec for the exact rule and its error message, "Choose a repeat interval between 1 and 12 weeks." This screen applies the rule on submit. The effect and eligibility of each cancel action is governed by FEAT-21.SPEC-006 (Series & Occurrence Cancellation Rules); its refusal messages ("This series has already been cancelled." and "This appointment is no longer active.") are shown here exactly as that spec defines them.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap (arrived from a booking) | FEAT-30.SPEC-001 (Cancel Booking (Pro-Initiated)) booking detail | FEAT-30 (Pro Booking Management) |
| Back arrow tap (arrived from a release notice) | FEAT-12 (Pro Daily Schedule Dashboard) | FEAT-12 (Pro Daily Schedule Dashboard) |
| Successful set-up | This screen, with the new series card shown | -- |
| Successful occurrence or series cancellation | This screen, with the cancelled row or card removed | -- |

## Data Model

**Creates:** Recurring Series -- interval (from the stepper), originating service and time (copied from the entry booking, never entered here), state set to Active.
**Reads:** Recurring Series -- interval, originating service and time, state, generated occurrences (series on Talia's schedule only); Booking (occurrence and entry booking) -- service, start_time, state, source, series reference; Client -- name (for display only).
**Updates:** Recurring Series -- state transitioned to Ended (via FEAT-21.SPEC-006, on "End series"). Booking (occurrence) -- state transitioned to Cancelled by Pro (via FEAT-21.SPEC-006).
**Deletes:** None.

## Business Rules

- Set-up is offered only when the entry booking is on Talia's schedule, is in Pending Payment, Confirmed, or Completed state, is not itself a generated occurrence of a series, and has never originated a series (FEAT-21.SPEC-003's one-series-per-booking rule, whatever that series' current state). Otherwise the set-up region is not shown.
- Talia may set up a series for any client on her schedule (FEAT-21.SPEC-003 Authorization Rules); the interval and horizon rules are the same as on FEAT-21.SPEC-001. Occurrence generation afterwards follows the ordinary client-facing notice and horizon rules (FEAT-21.SPEC-004, XBR-01, XBR-03), because it stands in for a booking the client would otherwise make.
- A successful set-up hands off immediately to FEAT-21.SPEC-004 for the first occurrence(s); Talia does not wait on this screen for generation, and the new series card shows "The next appointment will appear here once it's scheduled." until it completes.
- Both cancel actions are governed entirely by FEAT-21.SPEC-006. Every cancelled occurrence's deposit is refunded in full because any Pro cancellation refunds in full (XBR-09, applied by FEAT-09); the client is told through FEAT-08.SPEC-004 (Booking Change & Refund Notice), and the Pro's personal calendar mirrors each cancelled occurrence (XBR-13, FEAT-04).
- The occurrence group list pattern (this Brief's Shared UI Patterns) is followed exactly: cancelling one occurrence never reads as ending the series, the two actions are separate controls, and the series card remains after a single-occurrence cancellation.
- An occurrence's status badge follows FEAT-21.SPEC-002's mapping: Upcoming (Pending Payment before the deposit request, or Confirmed), Awaiting deposit (deposit link sent, unpaid), Needs new time (usual slot unavailable, per FEAT-21.SPEC-004).
- Contention (dependency map, Recurring Series): the Client and the Pro can both change a series or an occurrence; the resolution is reject-with-refresh, first committed change wins (FEAT-21.SPEC-006).

## Edge Cases

- **Riley cancels the same series or occurrence Talia is viewing, at the same moment Talia confirms a cancel (concurrent-edit conflict)** -- Reject-with-refresh, per FEAT-21.SPEC-006 and the dependency map's Contention note: the first committed change wins, Talia's screen refreshes to the updated series, and she sees "Riley already cancelled this. Your view has been updated." instead of the success toast.
- **The client's booking is cancelled or expires between Talia opening the screen and submitting the set-up** -- Submission is refused with refresh: "This booking is no longer active, so it can't be made recurring." and the set-up region disappears.
- **Talia opens the screen for a booking that already originated a series** -- The set-up region is not shown; the client's series (if still Active) is shown instead, or the empty state explains that the booking already started a standing appointment.
- **Talia taps a set-up or cancel control twice rapidly** -- The second tap is ignored while the first is in progress (control disabled in Submitting/Cancelling).
- **Talia chooses 0, 13 or more, or a fractional number of weeks** -- FEAT-21.SPEC-003 rejects it; the stepper shows the error and her attempted value stays visible.
- **Network failure during set-up or cancellation** -- Error banner: "Could not complete that. Check your connection and try again." with Retry; the interval stays entered and the card or row stays in its pre-action state.
- **An occurrence Talia tries to cancel was released unpaid moments earlier by FEAT-21.SPEC-005, or was already cancelled** -- Refused with refresh: "This appointment is no longer active." and the row updates to its current state.
- **Talia taps "End series" on a series that already ended** -- Refused with "This series has already been cancelled." and the card is removed on refresh.
- **The series' service was archived and generation has paused** -- The series card still shows; the note "New appointments aren't being scheduled because this service is no longer active." appears under the summary line, matching the gap FEAT-21.SPEC-004 flags on FEAT-12.
- **Talia reopens the screen right after a set-up or cancellation** -- The screen loads the current data, not a stale cached view.
- **The client holds a very long list of upcoming occurrences (a 1-week interval across a wide booking horizon)** -- The occurrence list scrolls inside the card; nothing is truncated or paginated, since the list is bounded by the booking horizon.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-30.SPEC-001 (Cancel Booking (Pro-Initiated)) -- within FEAT-30 (Pro Booking Management) | Navigation (inbound/outbound) | Talia arrives from the "Recurring series" link on a client's booking detail and returns there with the back arrow |
| FEAT-21.SPEC-003 (Recurring Series Setup & Generation Limits) | References (outbound) | Validates the interval and horizon at set-up; authorizes Talia to create a series for a client |
| FEAT-21.SPEC-004 (Occurrence Generation & Conflict Handling) | Triggers (outbound) / References (inbound) | A successful set-up hands off to it; the occurrences and statuses it produces appear here |
| FEAT-21.SPEC-005 (Occurrence Deposit Request & Release) | References (inbound) | Supplies the Awaiting deposit status and the removal of a released occurrence |
| FEAT-21.SPEC-006 (Series & Occurrence Cancellation Rules) | References (outbound) | Governs the effect, authorization, and contention outcome of both cancel actions |
| FEAT-21.SPEC-009 (Occurrence Deposit Lifecycle Notification) | Navigation (inbound) | The release notice's "View series" call to action opens this screen |
| FEAT-21.SPEC-002 (My Recurring Series) | References (inbound) | The client-facing counterpart; the same series and occurrences, shown to Riley |
| FEAT-09 (Cancellation & No-Show Policy Engine) | References (outbound) | Applies the full-refund outcome for each Pro-cancelled occurrence (XBR-09) |
| FEAT-08.SPEC-004 (Booking Change & Refund Notice) | Triggers (outbound) | Tells the client about each occurrence the Pro cancels |
| FEAT-04 (Two-Way Calendar Sync) | Triggers (outbound) | Mirrors created, moved, and cancelled occurrences to Talia's personal calendar (XBR-13) |
| FEAT-12 (Pro Daily Schedule Dashboard) | Navigation (outbound) | Back destination when Talia arrived from a release notice; also where generation gaps are flagged |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| recurring_series_created | interval_weeks, entry source (Pro at the chair) | Talia's set-up submission creates the Recurring Series | N/A -- "Self-Service Reschedule Rate" measures clients acting without the Pro, so a Pro-created series does not feed it; the event keeps the Stage 2 signal name so Pro-created and client-created series are countable together |
| recurring_occurrence_cancelled | series reference, acting role (Pro) | Talia cancels a single occurrence from this screen | N/A -- Stage 2 names no distinct signal for cancelling a single occurrence; per this Brief's Signals note, this action reuses the generic booking-cancelled signal the owning cancellation feature emits for any Booking |
| recurring_series_cancelled | interval_weeks, occurrences_cancelled_count, acting role (Pro) | Talia ends the whole series from this screen | N/A -- "Self-Service Reschedule Rate" measures client-initiated changes; a Pro-initiated series end is recorded under the Stage 2 signal name for completeness but feeds no client self-service metric |
| recurring_series_management_viewed | series_count, entry source (booking detail / release notice) | The screen loads with one or more series | N/A -- no Stage 2 metric measures view frequency for this screen; retained so usage of the Pro-side surface is observable |

## Acceptance Criteria

**FEAT-21.SPEC-010-AC-01:** Given Talia opens Riley's eligible booking in FEAT-30.SPEC-001 and taps "Repeat this booking", when this screen opens, then the set-up region shows the booking summary, an unset interval stepper, and a disabled "Set up standing appointment" button.

**FEAT-21.SPEC-010-AC-02:** Given Talia sets the interval to 3 weeks, when the value is entered, then the button becomes enabled and the preview line shows the first repeat date and that it is booked once inside her booking window.

**FEAT-21.SPEC-010-AC-03:** Given Talia submits an interval of 3 weeks, when FEAT-21.SPEC-003 validation passes, then a Recurring Series is created in state Active with the originating service and time copied from Riley's booking, the toast "Standing appointment set up -- every 3 weeks" appears, and FEAT-21.SPEC-004 is triggered to generate its first occurrence(s).

**FEAT-21.SPEC-010-AC-04:** Given Talia submits an interval of 0, 13, or 2.5 weeks, when validation runs, then the stepper shows "Choose a repeat interval between 1 and 12 weeks.", no series is created, and her attempted value stays visible.

**FEAT-21.SPEC-010-AC-05:** Given Riley's booking has already originated a series, when Talia opens this screen from that booking, then the set-up region is not shown.

**FEAT-21.SPEC-010-AC-06:** Given Riley's booking is Cancelled or Expired, when Talia opens this screen from it and Riley holds no Active series, then the Empty state shows "No standing appointments for Riley Chen." with the reason line.

**FEAT-21.SPEC-010-AC-07:** Given Riley holds one Active series with two upcoming occurrences, when Talia opens this screen, then one card shows the interval and both occurrences with their status badges, soonest first.

**FEAT-21.SPEC-010-AC-08:** Given Talia taps "Cancel this one" and confirms "Cancel Appointment", when FEAT-21.SPEC-006 commits the cancellation, then only that occurrence's Booking becomes Cancelled by Pro, its row is removed, the toast "Appointment cancelled" appears, and the series card remains.

**FEAT-21.SPEC-010-AC-09:** Given Talia taps "End series" and confirms "End Series", when FEAT-21.SPEC-006 commits, then the series becomes Ended, every not-yet-occurred occurrence becomes Cancelled by Pro, the card is removed, and the toast "Series ended" appears.

**FEAT-21.SPEC-010-AC-10:** Given Talia opens either cancel dialog and taps "Keep It" or "Keep Series", then the dialog closes and nothing is cancelled.

**FEAT-21.SPEC-010-AC-11:** Given a cancelled occurrence had a captured deposit, when Talia's cancellation commits, then the deposit is refunded in full under FEAT-09 regardless of timing, and Riley is notified through FEAT-08.SPEC-004.

**FEAT-21.SPEC-010-AC-12:** Given an occurrence's deposit link was sent and is unpaid, when Talia views the screen, then the occurrence shows the "Awaiting deposit" badge; and given its usual slot is unavailable per FEAT-21.SPEC-004, it shows "Needs new time" with no control for Talia to pick a time.

**FEAT-21.SPEC-010-AC-13:** Given Riley taps "Cancel series" on FEAT-21.SPEC-002 at the same moment Talia confirms "End series", when both reach commit, then the first commit wins and the other party's screen refreshes; if Riley's committed first, Talia sees "Riley already cancelled this. Your view has been updated."

**FEAT-21.SPEC-010-AC-14:** Given an occurrence was released unpaid by FEAT-21.SPEC-005 moments before Talia confirms cancelling it, when her action is evaluated, then it is refused with "This appointment is no longer active." and the row refreshes.

**FEAT-21.SPEC-010-AC-15:** Given a series already ended, when Talia's stale screen submits "End series", then it is refused with "This series has already been cancelled." and the card is removed on refresh.

**FEAT-21.SPEC-010-AC-16:** Given Talia loses connectivity, when she attempts a set-up or a cancellation, then she sees "You'll need to be online to do this. Please check your connection and try again." and nothing is submitted or queued.

**FEAT-21.SPEC-010-AC-17:** Given a network failure occurs during a cancellation, when the request fails, then an error banner reads "Could not complete that. Check your connection and try again." with Retry, and the row or card stays in its pre-cancellation state.

**FEAT-21.SPEC-010-AC-18:** Given the series data is loading, when Talia opens the screen, then a loading indicator replaces both regions; and given the load fails, an error banner with Retry replaces them.

**FEAT-21.SPEC-010-AC-19:** Given Talia taps "Set up standing appointment" or a confirm button twice rapidly, then the second tap is ignored while the first is in progress.

**FEAT-21.SPEC-010-AC-20:** Given Riley (the Client) or a Platform Operator (Support) looks for this screen, when they search their own surfaces, then no control reaches it -- Riley uses FEAT-21.SPEC-001 and FEAT-21.SPEC-002, and Support uses FEAT-19's view-only screen.

**FEAT-21.SPEC-010-AC-21:** Given Talia is signed out or her session has expired, when she opens this screen, then she is sent to the Pro sign-in screen or sees "Your session has expired. Sign in to continue." with a chosen interval preserved after re-authentication.

**FEAT-21.SPEC-010-AC-22:** Given Talia sets up a series, cancels one occurrence, and ends a series, then recurring_series_created, recurring_occurrence_cancelled (reusing the generic booking-cancelled signal), and recurring_series_cancelled are emitted with acting role Pro.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 9 (loading, set-up ready, filling, submitting, validation error, loaded, empty, cancelling, error) plus offline N/A | 10 |
| Business Rules | 7 | 7 |
| Edge Cases | 11 | 11 |

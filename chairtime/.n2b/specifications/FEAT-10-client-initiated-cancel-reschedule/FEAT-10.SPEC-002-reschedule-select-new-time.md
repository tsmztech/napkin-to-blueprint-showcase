---
document_type: spec
spec_type: screen
spec_id: FEAT-10.SPEC-002
spec_name: Reschedule -- Select New Time
spec_slug: reschedule-select-new-time
parent_feature: FEAT-10
parent_feature_name: Client-Initiated Cancel/Reschedule
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 15
---

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

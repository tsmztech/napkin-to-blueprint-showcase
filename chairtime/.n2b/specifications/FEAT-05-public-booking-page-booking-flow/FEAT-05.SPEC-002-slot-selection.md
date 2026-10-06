---
document_type: spec
spec_type: screen
spec_id: FEAT-05.SPEC-002
spec_name: Slot Selection
spec_slug: slot-selection
parent_feature: FEAT-05
parent_feature_name: Public Booking Page & Booking Flow
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

# Screen Spec: Slot Selection

## Overview

**Name:** Slot Selection
**ID:** FEAT-05.SPEC-002
**Type:** Screen
**Purpose:** Client picks a genuinely free time for the chosen service from the live slot list, confirmed against the live slot check at the instant of the pick; the checkout hold itself is placed later, when the client advances into the payment step (FEAT-05.SPEC-004 -> FEAT-07.SPEC-001, per FEAT-03.SPEC-002 and XBR-02).
**Parent Feature:** FEAT-05 -- Public Booking Page & Booking Flow

## Scope and Non-Goals

**In Scope:**
- Requesting and displaying the live slot list for the chosen service from FEAT-03 (Real-Time Slot Availability Engine)
- Letting the client pick a genuinely free time
- Re-checking the tapped time against the live slot list (FEAT-03) at the instant of the pick, without placing a hold
- Offering a join-waitlist path (FEAT-20.SPEC-001) when the service is fully booked
- Returning the client here with a refreshed list when a hold expires or a contested slot is lost

**Non-Goals:**
- Computing which times are genuinely free -- owned entirely by FEAT-03 (Real-Time Slot Availability Engine); this screen only displays what FEAT-03 returns and never derives availability itself
- Creating or expiring the checkout hold -- owned by FEAT-05.SPEC-006 (Slot Hold & Re-Validation at Checkout), which FEAT-05.SPEC-004 triggers when the client advances into the payment step; this screen places no hold
- The waitlist join itself (contact capture, Waitlist Entry creation) -- owned by FEAT-20.SPEC-001; this screen only offers the entry point
- Collecting the client's name, phone, or other details -- handled by FEAT-05.SPEC-003 (Client Details & Consent), the next step

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-05.SPEC-001 (Public Booking Page) | Client taps a service | Chosen service's ID, name, price, duration |
| FEAT-05.SPEC-006 (Slot Hold & Re-Validation at Checkout) | The client's checkout hold expires before payment completes | Same chosen service; slot list refreshed; plain expiry message shown |
| FEAT-05.SPEC-006 (Slot Hold & Re-Validation at Checkout) | The chosen slot is lost to a contesting client, or no longer valid, when the hold is requested on the payment step | Same chosen service; slot list refreshed; plain "just taken" or "no longer available" message shown |
| FEAT-05.SPEC-004 (Policy Acknowledgment & Deposit Checkout) | Client navigates back (via FEAT-05.SPEC-003) or the slot is lost when continuing | Same chosen service; previously viewed slot list refreshed |
| FEAT-07.SPEC-001 (Deposit Payment) | The checkout hold expires on the deposit payment screen (with or without a prior decline) | Same chosen service; slot list refreshed; plain expiry message shown |
| FEAT-20.SPEC-008 (Waitlist Opening Notification) | Client taps the claim link in a waitlist opening notice within the claim window | The matched service and opened slot; priority claim context |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Full screen | Select any genuinely free time shown | -- |
| The Pro (Talia), preview mode | Full screen, identical rendering | Select a time exactly as a client would; no real hold ever consumes actual availability against real clients (see Business Rules) | -- |
| Platform Operator (Support) | Full screen, read-only, reached only through FEAT-19's account view | View the slot list only | Time selection is not offered; consistent with XBR-24 |
| Unauthenticated | Yes -- the default and intended state for the Client role | Yes, identical to the Client row above | -- |
| Expired session | N/A -- no session exists to expire on this public flow | N/A | N/A |

## Layout and Content

**Header:** Back arrow (returns to FEAT-05.SPEC-001) with the chosen service's name, price, and duration shown as persistent context beneath it.

**Body:** A live list of available time slots for the chosen service, grouped by day, in chronological order. Each slot is a single tappable time element. If no times are available for the visible range, the list shows a plain message in place of slots, followed by a "Join the waitlist" action (see States, Empty).

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Slots list in a single column, grouped by day heading, full width.
- **Medium size class and above:** Slots list may show more times per row (a grid rather than a single column) within the same day grouping; no change to grouping or day-heading structure.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-05.SPEC-001 (Public Booking Page) | Screen closes | Standard backward transition |
| Time slot | Tap | Re-checks the tapped time against the live slot list from FEAT-03 (no hold is placed here; the checkout hold starts when the client advances into the payment step, FEAT-05.SPEC-004 -> FEAT-07.SPEC-001) | Slot shows a brief "checking this time" loading indicator | On success: navigate to FEAT-05.SPEC-003 (Client Details & Consent) with the chosen time carried. If the time is gone: plain "That time was just taken." message and refreshed list, slot removed from the list |
| Time slot (while a re-check is in flight for the same client) | Tap | No action -- debounced | None | Slot remains in its loading indicator state |
| "Join the waitlist" action (Empty state only) | Tap | Navigate to FEAT-20.SPEC-001 (Join Waitlist) carrying the chosen service | Screen closes | Standard forward transition |

### Accessibility Notes

- **Focus order:** Back arrow -> service context header -> day groupings top to bottom -> time slots within each day, chronological.
- **Announcements:** When the slot list refreshes (after an expiry or contention loss), the refreshed content and the accompanying plain message are announced to assistive technology.
- **Keyboard alternatives:** Every time slot is reachable and selectable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Loading | A neutral loading placeholder in place of the slot list | Screen first opens for the chosen service | Slot list loads successfully or a load error occurs |
| Loaded (slots available) | Slot list rendered grouped by day, as described in Layout and Content | Live slot data returns at least one available time | Client taps a slot, or the list is refreshed |
| Empty (fully booked) | A plain message: "No open times right now for this service. Check back soon." in place of the slot list, with a "Join the waitlist" action beneath it | Live slot data returns zero available times within the visible booking horizon | The Pro opens availability, the client taps "Join the waitlist" (-> FEAT-20.SPEC-001), or the client selects a different service (via back arrow) |
| Checking slot | The tapped slot shows a brief "checking this time" indicator; other slots remain visible but not selectable during this brief moment | Client taps a time slot | The re-check succeeds (navigate onward) or fails (slot taken or no longer valid) |
| Slot lost to contention | A plain message: "That time was just taken." appears briefly, the list refreshes, and the taken slot is removed | The live re-check on tap finds the time taken, or FEAT-05.SPEC-006 reports the slot lost when the hold is requested on the payment step | Client picks a different slot or leaves the screen |
| Error | A plain error message: "Couldn't load available times. Try again." with a Retry action | Live slot data fails to load | Client taps Retry and the load succeeds, or leaves the screen |
| Offline/Degraded | A plain banner: "Check your connection and try again." replaces the slot list | Connectivity is lost while loading or after load, before a slot is picked | Connectivity is restored and the load or re-check completes |

## Validation Rules

Not applicable -- this screen has no user text input, only a selection action against system-provided data. This screen re-checks the chosen slot's genuine availability against the live slot list at the instant of selection, and FEAT-05.SPEC-006 re-validates it again when the hold is requested on the payment step; this screen never trusts a previously computed slot as still valid without that re-check.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|-------------------------------------|
| Back arrow tap | FEAT-05.SPEC-001 (Public Booking Page) | -- |
| Successful slot pick (time still free) | FEAT-05.SPEC-003 (Client Details & Consent) | -- |
| "Join the waitlist" tap (Empty state) | FEAT-20.SPEC-001 (Join Waitlist) | FEAT-20 (Waitlist for Cancelled Slots) |

## Data Model

**Creates:** None -- the Booking record and checkout hold are created by FEAT-05.SPEC-006 when the client advances into the payment step (triggered from FEAT-05.SPEC-004), not by this screen.
**Reads:** Live slot list for the chosen service, computed and served by FEAT-03 (Real-Time Slot Availability Engine); Service -- name, price, duration (carried as context from FEAT-05.SPEC-001).
**Updates:** None.
**Deletes:** None.

## Business Rules

- The slot list this screen displays is live-computed by FEAT-03, never derived or cached independently by this screen (XBR-01: a time is offered only if it passes the live slot check).
- Available slots appear within roughly one second of the service selection that led here, and the list updates within roughly one second of a slot being taken by another client (Non-Functional Notes; success-metrics.md: "Slot Search Responsiveness").
- Selecting a slot does not reserve it. The checkout hold of platform parameter: `checkout-hold-timeout-minutes` (XBR-02) is placed by FEAT-05.SPEC-006, requested from FEAT-03.SPEC-002, only when the client advances into the deposit payment step; the client is never shown a slot as reserved until that hold is confirmed.
- In preview mode, the Pro walks through the identical slot-selection screen; no real hold or Booking is created against real availability and no real charge is ever taken (owned by FEAT-07's preview handling, referenced here for consistency with the feature's Shared UI Pattern).
- A fully booked service (zero available times in the booking horizon) offers a "Join the waitlist" action leading to FEAT-20.SPEC-001; this screen does not capture the waitlist entry itself.
- The Booking Page Availability Gate (FEAT-05.SPEC-008) is evaluated on every load of this screen, before any of its content renders; a paused or unavailable page never shows the slot list (XBR-06, XBR-14, XBR-27).

## Edge Cases

- **Client loses connectivity mid-selection** -- The Offline/Degraded state renders; nothing is charged and no hold is created.
- **Two clients tap the same slot at effectively the same time** -- Both pass the pick-time check; the first to advance into the payment step wins the hold, and the other sees the "slot lost to contention" state and a refreshed list on FEAT-05.SPEC-004, never a payment error (FEAT-03.SPEC-005, XBR-01).
- **Client navigates back to this screen after their checkout hold expires** -- The list refreshes and shows a plain "that hold has expired, please pick a time again" message; the client's previously entered service selection is preserved, and the client picks a new time without re-entering the service.
- **Client rapidly taps multiple different slots in succession** -- Only the first tap's re-check proceeds; subsequent taps on other slots are ignored while the first is in flight (each slot request is debounced per client, per Interactions).
- **All slots for the visible date range are taken between page load and the client's tap** -- The client's tap re-checks against the live list at the instant of selection; if the specific slot is no longer available, the contention message appears rather than a silent failure.
- **Client's device is offline when a previously viewed slot becomes stale** -- Nothing proceeds while offline; on reconnecting, the client's tap re-checks the slot against the live list before advancing.
- **Client taps "Join the waitlist" and the service gains open times before they finish** -- FEAT-20.SPEC-001 owns that case; this screen simply navigates out and the client can return through the back arrow to a refreshed live list.

## Connected Specs

| Connected Spec | Connection Type | Description |
|-----------------|-------------------|--------------|
| FEAT-05.SPEC-001 (Public Booking Page) | Navigation (inbound) | Client arrives here after picking a service |
| FEAT-05.SPEC-003 (Client Details & Consent) | Navigation (outbound) | A successful slot pick advances the client here |
| FEAT-05.SPEC-006 (Slot Hold & Re-Validation at Checkout) | References (inbound) | Reports an expired hold or a slot lost when the hold is requested on the payment step; this screen then shows the refreshed list and message |
| FEAT-05.SPEC-008 (Booking Page Availability Gate) | References (inbound) | Governs whether this screen renders on every load |
| FEAT-03.SPEC-001 (Slot Availability Computation) -- within FEAT-03 (Real-Time Slot Availability Engine) | References (inbound) | Source of the live slot list this screen displays |
| FEAT-20.SPEC-001 (Join Waitlist) -- within FEAT-20 (Waitlist for Cancelled Slots) | Navigation (outbound) | A fully booked service offers the "Join the waitlist" action from the Empty state |
| FEAT-20.SPEC-008 (Waitlist Opening Notification) -- within FEAT-20 | Navigation (inbound) | The claim link in a waitlist opening notice lands here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-------------------|
| slot_list_viewed | service ID, slot count shown | Slot list finishes loading | supports success-metrics.md: "Slot Search Responsiveness" |
| slot_selected | service ID, time-to-selection since page load | Client taps an available slot | supports success-metrics.md: "Booking Completion Speed" |
| slot_selection_lost_to_contention | service ID | The tapped slot is found taken on the live re-check, or reported taken by FEAT-05.SPEC-006 on the payment step | supports success-metrics.md: "Zero Double-Booking Confidence" |
| slot_list_empty_shown | service ID | The Empty (fully booked) state renders | supports success-metrics.md: "Slot Search Responsiveness" |
| waitlist_join_tapped | service ID | Client taps "Join the waitlist" in the Empty state | supports success-metrics.md: "Slot Search Responsiveness" |

## Acceptance Criteria

**FEAT-05.SPEC-002-AC-01:** Given Riley has picked a service on FEAT-05.SPEC-001, when the Slot Selection screen loads, then Riley sees a live list of available times grouped by day for that service, within roughly one second.

**FEAT-05.SPEC-002-AC-02:** Given Riley is viewing the slot list, when Riley taps an available time, then the time is re-checked against the live slot list and, if still free, Riley advances to FEAT-05.SPEC-003 with that time carried as context; no hold is placed yet.

**FEAT-05.SPEC-002-AC-03:** Given Riley taps a time slot, when the live re-check finds another client's hold or Booking already covers that time, then Riley sees "That time was just taken." and a refreshed list with that time removed, never a payment error.

**FEAT-05.SPEC-002-AC-04:** Given a service has zero available times in the booking horizon, when Riley opens the Slot Selection screen for it, then Riley sees "No open times right now for this service. Check back soon." and no time is selectable.

**FEAT-05.SPEC-002-AC-05:** Given Riley's checkout hold expires while she is in the payment step, when she is returned to this screen, then she sees a refreshed list and a plain message that her hold expired, and can pick a time again without re-selecting the service.

**FEAT-05.SPEC-002-AC-06:** Given Riley loses connectivity while the slot list is loading, when the load fails due to connectivity, then Riley sees "Check your connection and try again." and no pick is registered.

**FEAT-05.SPEC-002-AC-07:** Given the live slot data fails to load for a reason other than connectivity, when Riley opens this screen, then Riley sees "Couldn't load available times. Try again." with a Retry action.

**FEAT-05.SPEC-002-AC-08:** Given Riley taps the back arrow, when the navigation completes, then Riley returns to FEAT-05.SPEC-001 (Public Booking Page).

**FEAT-05.SPEC-002-AC-09:** Given Riley taps two different time slots in rapid succession, when the first tap's re-check is still in flight, then the second tap has no effect until the first resolves.

**FEAT-05.SPEC-002-AC-10:** Given Talia previews her own booking page and reaches this screen, when she taps an available time, then the identical re-check-and-advance behavior occurs with no real hold, Booking, or charge ever created through the flow.

**FEAT-05.SPEC-002-AC-11:** Given Platform Operator (Support) views this screen through the read-only account view, when Support looks for a way to select a time, then no selection control is offered.

**FEAT-05.SPEC-002-AC-12:** Given a slot becomes unavailable between page load and Riley's tap, when Riley taps that specific slot, then the tap re-validates against the live list and Riley sees the contention message rather than the stale slot being accepted.

**FEAT-05.SPEC-002-AC-13:** Given Riley's pick passes the live re-check, when FEAT-05.SPEC-003 loads, then the chosen time and service remain visible as context, per the feature's persistence pattern.

**FEAT-05.SPEC-002-AC-14:** Given a service has zero available times, when Riley taps "Join the waitlist" in the Empty state, then Riley navigates to FEAT-20.SPEC-001 (Join Waitlist) with the chosen service carried.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 4 | 4 |
| States | 7 (loading, loaded, empty, checking, lost-to-contention, error, offline) | 7 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |

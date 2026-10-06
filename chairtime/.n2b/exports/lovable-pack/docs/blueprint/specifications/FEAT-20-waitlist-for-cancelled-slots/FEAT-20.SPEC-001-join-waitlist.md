---
document_type: spec
spec_type: screen
spec_id: FEAT-20.SPEC-001
spec_name: Join Waitlist
spec_slug: join-waitlist
parent_feature: FEAT-20
parent_feature_name: Waitlist for Cancelled Slots
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 17
---

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

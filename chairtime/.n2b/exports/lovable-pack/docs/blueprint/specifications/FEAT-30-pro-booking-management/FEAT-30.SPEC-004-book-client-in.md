---
document_type: spec
spec_type: screen
spec_id: FEAT-30.SPEC-004
spec_name: Book Client In
spec_slug: book-client-in
parent_feature: FEAT-30
parent_feature_name: Pro Booking Management
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 16
---

# Screen Spec: Book Client In

## Overview

**Name:** Book Client In
**ID:** FEAT-30.SPEC-004
**Type:** Screen
**Purpose:** Talia chooses a service, a time, and an existing or new client to book the client in directly, then issues a held deposit request (link or on-screen scan code) instead of collecting payment inside this screen.
**Parent Feature:** FEAT-30 -- Pro Booking Management

## Scope and Non-Goals

**In Scope:**
- Choosing a service, a genuinely free time (with the Pro-only notice/horizon exception), and an existing or new client
- A quick existing-client lookup by phone number
- Capturing a new client's name and phone (and email, if texting is declined)
- Choosing the deposit-request delivery method (link by text/email, or an on-screen code)
- Saving the booking-in action and reflecting its success, contention, or failure

**Non-Goals:**
- Computing which times are genuinely free -- owned by FEAT-03 (Real-Time Slot Availability Engine); this screen only displays what FEAT-03 returns, with the exception FEAT-30.SPEC-006 states
- Creating the Booking record, placing the slot hold, or issuing the deposit request itself -- owned by FEAT-30.SPEC-010 (Pro-Created Booking & Deposit Request Hold) and FEAT-30.SPEC-013 (Deposit Request & Expiry Notice); this screen triggers those but does not implement them
- A full searchable client list -- owned by FEAT-13 (Client Record Management)'s Client List Search & Filter (FEAT-24); this screen offers only a quick inline lookup while booking someone in
- Collecting the client's deposit payment -- owned by FEAT-07 (Deposit Payment at Booking), reached by the client through the issued request, not inside this screen

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-12 (Pro Daily Schedule Dashboard) | Talia opens "Book client in" from the schedule | None -- form starts empty |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | All actions: choose service, time, client, delivery method, and save | -- |
| The Client (Riley) | No | No | No control on any Client-facing surface reaches this screen; a client is booked in by the Pro, never by themselves through this screen |
| Platform Operator (Support) | Full screen, read-only, reached only through FEAT-19's account view | View only | Save control is not shown, consistent with SC-05 |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29) |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- entered form data is preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Back arrow (returns to FEAT-12) with the title "Book client in" and a "Save" action button (right-aligned, disabled until service, time, and client are all set).

**Body:** A single-column flow with the following sections in order:
- Service selector (list of the Pro's Active services, per FEAT-01)
- Time selector: once a service is chosen, a live list of available time slots for that service, grouped by day, with Talia's own notice/horizon exception applied (the same slot-list mechanism as FEAT-05.SPEC-002 and FEAT-30.SPEC-002)
- Client selector: a phone-number lookup field for an existing client, or a "New client" toggle revealing Name (required), Phone (required), and Email (required only if the texting-consent toggle below is left off) fields
- Texting-consent toggle for a new client, unchecked by default (mirroring FEAT-05's own opt-in default)
- Deposit-request delivery choice: "Send a link" (by text if consented, otherwise email) or "Show a code to scan" -- a single selection, defaulting to "Send a link"

**Footer:** None -- Save is in the header.

### Responsive Behavior

- **Compact breakpoint:** Single-column flow as described, full width.
- **Medium size class and above:** Sections remain single-column, capped at a consistent platform-wide form width and horizontally centered; the slot list may show more times per row within the same day grouping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-12 | Screen closes | Standard backward transition |
| Service selector | Select | Sets the chosen service; reveals the time selector | Time selector appears, live slot list requested | Standard selection state |
| Time slot | Tap | Re-validates the candidate against FEAT-03.SPEC-004 (with the Pro-only notice/horizon exception) | Slot selected | If valid: selection shown. If contested: plain "just taken" message, list refreshed |
| Client phone lookup | Type, then select a match | Looks up an existing Client by phone-number match | Existing client's name shown as the selection | Matching client's name displayed; no match shows "No client found -- add as new" |
| "New client" toggle | Tap | Reveals Name, Phone, Email fields for entry | Existing-client lookup hidden | Standard toggle state |
| Name / Phone / Email fields | Type | Captures new client details | Field shows entered text | Standard input focus state |
| Texting-consent toggle | Tap | Sets whether the new client is offered text delivery | Toggle state changes; Email field becomes required if left off | Email field shows required indicator when consent is off |
| Deposit-request delivery choice | Select | Sets link vs. on-screen code for the request | Selected option highlighted | Standard selection state |
| Save button | Tap | 1. Validates all fields via FEAT-30.SPEC-006/FEAT-03.SPEC-004 (slot fit and notice/horizon exception). 2. Triggers FEAT-30.SPEC-010 (Pro-Created Booking & Deposit Request Hold). | Button shows loading state during save | Success: confirmation shown, then navigate to FEAT-12. Contested slot: plain "just taken" message, list refreshed, save not submitted. Failure: retry prompt shown |
| Save button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> Service selector -> Time selector (once revealed) -> Client phone lookup / New client toggle -> Name -> Phone -> Email (when shown) -> Texting-consent toggle -> Deposit-request delivery choice -> Save.
- **Announcements:** Field validation errors, the "just taken" contention message, and save success/failure are announced to assistive technology.
- **Keyboard alternatives:** Every field, toggle, and slot is reachable and selectable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Empty (default) | Service selector shown, all other sections hidden or disabled, Save disabled | Screen first opens | Talia selects a service |
| Selecting time | Live slot list shown for the chosen service | Talia selects a service | Talia selects a time, or changes the service |
| Selecting client | Phone lookup and "New client" toggle available | Talia selects a valid time | Talia selects an existing client or completes new-client fields |
| Ready to save | All required fields complete, Save enabled | Service, time, and client (existing or complete new-client entry) are all set | Talia taps Save or navigates away |
| Slot contested | A plain message "That time was just taken." appears, time selector refreshes | The selected slot is lost to another booking before Save completes | Talia picks a different time |
| Saving | Save button shows loading state, form fields disabled | Talia taps Save | Save completes (success, contested, or failure) |
| Saved | Confirmation shown ("Booking created. {client_name} will get a request to pay their deposit.") before returning to FEAT-12 | FEAT-30.SPEC-010 reports the booking and hold created successfully | Talia is navigated to FEAT-12 |
| Error | Inline error banner "Couldn't save this booking. Try again." with a Retry action | FEAT-30.SPEC-010 reports a processing failure | Talia taps Retry and it succeeds, or she backs out |
| Offline/Degraded | Banner "Check your connection and try again." replaces the Save control; entered form data remains visible and editable | Connectivity is lost while the screen is open | Connectivity is restored and Talia can save |

## Validation Rules

Slot validity and the notice/horizon exception governed by FEAT-03.SPEC-004 and FEAT-30.SPEC-006. See those specs for the exact conditions. The remaining fields on this screen are simple, screen-local validations not warranting a standalone spec:

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| Service selector | Required | On Save | "Choose a service." |
| Time selector | A candidate must pass FEAT-03.SPEC-004's fit rule | On selection and again on Save | "That time doesn't fit -- pick another." |
| Client (existing or new) | Required -- one of an existing-client match or a complete new-client entry | On Save | "Choose an existing client or add a new one." |
| New client -- Name | Required, 1-100 characters, when adding a new client | On blur | "Client name is required." |
| New client -- Phone | Required, valid reachable format, when adding a new client | On blur | "Enter a valid phone number." |
| New client -- Email | Required when the texting-consent toggle is off, optional otherwise | On blur / On submit | "Email is required when texting isn't enabled." |
| Deposit-request delivery choice | Required (defaults to "Send a link") | Always set | -- |

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|-------------------------------------|
| Back arrow tap | FEAT-12 (Pro Daily Schedule Dashboard) | -- |
| Successful save | FEAT-12 (Pro Daily Schedule Dashboard) | -- |

## Data Model

**Creates:** Client (new client path only) -- name, phone, optional email, per FEAT-30.SPEC-010's downstream creation; Booking -- created by FEAT-30.SPEC-010 on save, not by this screen directly.
**Reads:** Service -- Active services list (FEAT-01); live slot list for the chosen service (FEAT-03); Client -- existing-client lookup by phone.
**Updates:** None directly.
**Deletes:** None.

## Business Rules

- XBR-03: Talia may book a client in inside her own minimum_booking_notice or beyond her booking_horizon; the candidate slot's fit rule is never exempted.
- XBR-05: the deposit amount is computed exactly from the chosen service's rule and is never entered or altered by Talia on this screen.
- XBR-15: the deposit-request delivery choice of "Send a link" resolves to text only if the client has active texting consent; otherwise it resolves to email automatically, per FEAT-14's consent rule -- Talia's choice is "link vs. code," not "text vs. email."
- A new client entered here carries the same data-sensitivity treatment as any Client record (SC-03): visible only to Talia, never to any other Pro or client.
- The "just taken" recovery message on a contested slot matches the same wording used by any other booking path (FEAT-05, FEAT-10), per the feature's Shared UI Pattern.

## Edge Cases

- **Talia's chosen slot is taken by a client-side booking between her selection and Save** -- The save re-validates the candidate; if it is no longer available, Talia sees the plain "just taken" message and a refreshed list, never a silent failure or a payment error (XBR-01).
- **Talia looks up an existing client by phone and finds no match** -- She sees "No client found -- add as new" and can switch directly to the new-client fields with the phone number carried over.
- **Talia enters a new client's phone number that already matches an existing client for her account** -- The phone-number match resolves to the existing Client record rather than creating a duplicate (dependency map's Client Contention rule); Talia is shown the matched existing client instead.
- **Talia navigates away with fields partially filled** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Talia taps Save twice rapidly** -- The second tap is ignored while the first save is in progress.
- **Network failure during save** -- Error banner: "Couldn't save this booking. Try again." with a Retry button; entered form data is preserved.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-30.SPEC-006 (Pro Booking Action Rules) | References (inbound) | Notice/horizon exception applied to slot re-validation |
| FEAT-30.SPEC-010 (Pro-Created Booking & Deposit Request Hold) | Triggers (outbound) | Save triggers Booking and hold creation |
| FEAT-03 (Real-Time Slot Availability Engine) | References (inbound) | Source of the live slot list this screen displays |
| FEAT-01 (Service & Pricing Management) | References (inbound) | Source of the Active services list |
| FEAT-13 (Client Record Management) | References (inbound) | The quick existing-client lookup; the full searchable list belongs there |
| FEAT-30.SPEC-013 (Deposit Request & Expiry Notice) | Triggers (outbound) | Talia's delivery-method choice governs which content variant this notification sends |
| FEAT-12 (Pro Daily Schedule Dashboard) | Navigation (inbound/outbound) | Entry point and return destination |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-------------------|
| pro_book_client_in_started | () | Screen loads | supports success-metrics.md: "Pro Change Correctness" |
| pro_book_client_in_saved | client_type (existing / new), delivery_method (link / code) | Save completes successfully | supports success-metrics.md: "Pro Change Correctness" |
| pro_book_client_in_slot_lost_to_contention | () | The chosen slot is lost between selection and save | supports success-metrics.md: "Zero Double-Booking Confidence" |

## Acceptance Criteria

**FEAT-30.SPEC-004-AC-01:** Given Talia opens this screen and selects a service, when the time selector appears, then she sees a live slot list for that service with her own notice/horizon exception applied.

**FEAT-30.SPEC-004-AC-02:** Given Talia selects a service, a time, and an existing client by phone lookup, when she taps Save, then the booking is created, a deposit request is issued, and she sees confirmation before returning to FEAT-12.

**FEAT-30.SPEC-004-AC-03:** Given Talia's phone lookup finds no matching client, when she views the result, then she sees "No client found -- add as new" and can switch to new-client entry with the phone number carried over.

**FEAT-30.SPEC-004-AC-04:** Given Talia enters a new client and leaves the texting-consent toggle off, when she reaches the Email field, then it is required, and Save is blocked until it is filled.

**FEAT-30.SPEC-004-AC-05:** Given Talia enters a new client's phone number that matches an existing client's record, when the match is detected, then she is shown the existing client instead of creating a duplicate.

**FEAT-30.SPEC-004-AC-06:** Given Talia selects a time inside her own minimum_booking_notice, when the candidate is re-validated, then it is accepted, per the Pro-only exception.

**FEAT-30.SPEC-004-AC-07:** Given Talia's selected slot is taken by another booking before she saves, when the save attempt processes, then she sees "That time was just taken." and a refreshed slot list, and no booking is created.

**FEAT-30.SPEC-004-AC-08:** Given Talia chooses "Show a code to scan" as the delivery method, when the save completes, then a scannable code is shown rather than a text or email being sent.

**FEAT-30.SPEC-004-AC-09:** Given Talia chooses "Send a link" for a client without texting consent, when the deposit request is issued, then it delivers by email automatically, per XBR-15.

**FEAT-30.SPEC-004-AC-10:** Given Talia navigates away with fields partially filled, when she attempts to leave, then a confirmation dialog "You have unsaved changes. Discard?" appears with "Discard" and "Keep Editing" options.

**FEAT-30.SPEC-004-AC-11:** Given Talia taps Save twice rapidly, when the first save is in progress, then the second tap has no effect until the first resolves.

**FEAT-30.SPEC-004-AC-12:** Given the save fails due to a processing error, when the failure occurs, then Talia sees "Couldn't save this booking. Try again." with a Retry action, and her entered data is preserved.

**FEAT-30.SPEC-004-AC-13:** Given Talia loses connectivity while filling this screen, when she attempts to save, then she sees "Check your connection and try again." and nothing is submitted.

**FEAT-30.SPEC-004-AC-14:** Given Riley (the Client) has no path to this screen, when the product's screens are reviewed for a reachable control, then none exists.

**FEAT-30.SPEC-004-AC-15:** Given Talia has not yet selected a service, time, and client, when she looks at the Save button, then it is disabled.

**FEAT-30.SPEC-004-AC-16:** Given the deposit amount for the chosen service, when the booking is created, then it is computed exactly from the Service's rule and never entered or altered by Talia on this screen.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 9 | 9 |
| States | 9 (empty, selecting time, selecting client, ready, contested, saving, saved, error, offline) | 9 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

---
document_type: spec
spec_type: screen
spec_id: FEAT-06.SPEC-001
spec_name: Access Link Request
spec_slug: access-link-request
parent_feature: FEAT-06
parent_feature_name: Client Booking Identity
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Screen Spec: Access Link Request

## Overview

**Name:** Access Link Request
**ID:** FEAT-06.SPEC-001
**Type:** Screen
**Purpose:** Client enters the phone number they used at booking and requests a one-tap access link sent to that phone, so they can view or manage their bookings with this one Pro without a password.
**Parent Feature:** FEAT-06 -- Client Booking Identity

## Scope and Non-Goals

**In Scope:**
- Phone number entry and submission to request an on-demand access link
- The "no bookings found" outcome for a phone number with no matching Client record for this Pro
- The single-tap "request a new link" re-entry point used after an invalid, expired, or already-used link (FEAT-06.SPEC-002)
- Surfacing a link-delivery failure with an immediate retry

**Non-Goals:**
- Validating and redeeming a tapped link -- owned by FEAT-06.SPEC-002 (Access Link Validation & Redemption); this screen only issues the request
- Viewing the client's bookings -- owned by FEAT-06.SPEC-003 (My Bookings List), reached only after a valid link is redeemed
- Any password, PIN, or account-creation flow -- excluded per BRIEF.md's Target Users & Roles: clients "must not face a signup wall or need a password-style account"
- Pro sign-in -- the Pro has their own dashboard login (FEAT-29), entirely out of this feature's scope per product-features.md's Access field

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| Default entry | Client navigates directly to manage their booking without already holding a valid link (e.g., typed the manage-my-booking address, or opened it from memory) | None -- form starts empty |
| FEAT-06.SPEC-002 (Access Link Validation & Redemption) | The tapped link was invalid, expired, or already used | A single-tap "request a new link" prompt is shown in place of the empty form's first state |
| FEAT-20.SPEC-001 (Join Waitlist) | Client taps the access-link link on the waitlist Confirmed state | None -- form starts empty |
| FEAT-20.SPEC-009 (Waitlist Expiry Notification) | Client taps "View my waitlists" when a fresh access link could not be issued at send time | None -- form starts empty so the client can request a new link |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Full screen | Enter a phone number and request a link for their own number | -- |
| The Pro (Talia) | No | No | This mechanism is not the Pro's sign-in path (product-features.md, Access); a Pro who lands here sees the same screen as any client would, since the screen has no way to distinguish a Pro visiting by phone from a client -- the Pro's own sign-in is reached only through FEAT-29, never through this screen |
| Platform Operator (Support) | No | No | Support access never uses or bypasses client identity (scope-boundaries SC-05); support has no route into this screen at all |
| Unauthenticated | Yes | Yes | Not applicable -- this screen is itself the unauthenticated entry point; no separate unauthenticated state exists here |
| Expired session | N/A | N/A | N/A -- this screen precedes any session; there is nothing here that can expire until after a link is requested and redeemed (see FEAT-06.SPEC-003 and FEAT-06.SPEC-004 for their own expired-session handling) |

## Layout and Content

**Header:** Screen title "Manage your booking" with the Pro's display name shown below it (e.g., "with Talia").

**Body:** A single-column form with:
- Phone number input (required, telephone-formatted), with helper text "Enter the phone number you used when booking"
- "Send me a link" action button, directly below the input
- A result region below the button that shows nothing until the form is submitted, then shows exactly one of: a confirmation message, the "no bookings found" message, a delivery-failure message with a retry action, or (when arriving from FEAT-06.SPEC-002) the "request a new link" prompt pre-populating this same result region above an otherwise empty phone field

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Single-column form as described above, full width, result region directly below the button.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Phone number input | Type | Captures the entered digits | Field shows entered text | Standard input focus state |
| Phone number input | Blur (empty or malformed) | Triggers field-level validation (this screen's own basic format check) | Error state on field | "Enter a valid phone number" below the field |
| "Send me a link" button | Tap | 1. Validates phone format. 2. If valid, checks the request rate limit governed by FEAT-06.SPEC-007. 3. If under the limit, matches the phone number to a Client record for this Pro, governed by FEAT-06.SPEC-008. 4. If matched, generates a single-use, 30-minute on-demand Access Link inline on this screen. 5. Triggers FEAT-06.SPEC-006 (Access Link Delivery) to send the link. | Button shows a brief loading state while the link is generated and sent | Success: result region shows "Check your phone for a link to view your bookings." No match: result region shows the plain "no bookings found" message. Rate-limited: result region shows the rate-limit message. Delivery failure: result region shows the failure message with a retry action. |
| "Send me a link" button (while loading) | Tap | No action -- ignored while a request is already in flight | None | Button remains in its loading state |
| "Try again" (delivery-failure retry) | Tap | Re-attempts delivery of the same still-valid Access Link via FEAT-06.SPEC-006, or, if it has since expired, generates a fresh one following the same steps as the original request | Button shows a brief loading state | Success: same confirmation message as above. Repeated failure: failure message remains with the retry action still available. |
| "Request a new link" (shown after an invalid/expired/used link) | Tap | Clears the prior prompt and returns the form to its normal empty state, ready for a fresh phone number entry | Result region clears, phone field is focused and empty | Standard empty-form appearance |

### Accessibility Notes

- **Focus order:** Phone number input -> "Send me a link" button -> result region (when populated).
- **Validation and result announcements:** The field's error state and the result region's message (confirmation, no-bookings, rate-limit, or delivery-failure) are announced to assistive technology as they appear, and the result message is programmatically associated with the form.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (default) | Phone field empty, button enabled, no result region content | Screen first opens with no incoming re-request context | Client begins typing or the screen is entered via FEAT-06.SPEC-002 with a re-request prompt |
| Filling | Phone field contains client input | Client types in the field | Client taps "Send me a link" or navigates away |
| Sending | Button shows loading state, form disabled | Client taps "Send me a link" or "Try again" | Request completes with any outcome below |
| Link Sent | Result region shows "Check your phone for a link to view your bookings." | Phone matched a Client record and delivery was attempted successfully | Client leaves the screen (typically to check their phone) |
| No Bookings Found | Result region shows the plain "no bookings found" message; phone field remains editable for another attempt | Phone number does not match any Client record for this Pro | Client edits the phone number and resubmits |
| Rate Limited | Result region shows a plain message that no more links can be requested for this number right now and to try again shortly | The per-phone-number request rate limit (FEAT-06.SPEC-007) is exceeded | The rate-limit window (FEAT-06.SPEC-007) elapses and the client resubmits |
| Delivery Failed | Result region shows "We couldn't send your link. Try again." with a "Try again" action | Text or email delivery of the generated link fails (FEAT-06.SPEC-006) | Client taps "Try again" |
| Re-Request Prompt | Result region shows "That link no longer works. Request a new one." with a single-tap "Request a new link" action; phone field is empty below it | Client arrives from FEAT-06.SPEC-002 after an invalid, expired, or already-used link | Client taps "Request a new link", returning to the Empty state |
| Offline/Degraded | Banner "You're offline. Connect to the internet to request a link." at the top; the phone field remains visible but the "Send me a link" button is disabled while offline | Connectivity is lost while this screen is open, or the screen is opened without connectivity | Connectivity is restored -- the button re-enables and the banner clears |

## Validation Rules

**Option B -- Inline (simple validation, not shared beyond this screen):**

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| Phone number | Required, must be a valid reachable phone number format | On blur and on submit | "Enter a valid phone number" |

Rate-limit and phone-to-Client matching rules are not defined here -- see FEAT-06.SPEC-007 (Access Link Lifecycle & Scope Rules) for the rate limit and FEAT-06.SPEC-008 (Client Identity & Privacy Isolation Rule) for matching.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Client taps the delivered link (outside this screen, on their phone) | FEAT-06.SPEC-002 (Access Link Validation & Redemption) | -- |

## Data Model

**Creates:** Access Link -- scope set to "all of this client's bookings with this Pro," expiry set to 30 minutes from creation, state set to Issued. Created only when the phone number matches a Client record (FEAT-06.SPEC-008) and the rate limit permits it (FEAT-06.SPEC-007).
**Reads:** Client -- phone field, matched against the entered number for this Pro only (FEAT-06.SPEC-008).
**Updates:** None.
**Deletes:** None.

## Business Rules

- Rate limiting on link requests per phone number is governed by FEAT-06.SPEC-007 -- this screen enforces it but does not redefine it.
- Phone-to-Client matching and the cross-client/cross-Pro privacy isolation boundary are governed by FEAT-06.SPEC-008 -- a phone number with no match never reveals whether it exists for another Pro or another client.
- Every generated Access Link is delivered by FEAT-06.SPEC-006, which decides the channel (text with active consent, otherwise email).
- XBR-18: Access links open only that one client's bookings with that one Pro; on-demand links are single-use for 30 minutes.

## Edge Cases

- **Client resubmits before a previously issued, still-valid link expires** -- A new Access Link is generated and delivered as normal; the prior link remains valid independently until it is used or its own 30-minute expiry is reached (governed by FEAT-06.SPEC-007). The client is not warned about the earlier link.
- **Client double-taps "Send me a link"** -- The second tap is ignored while the first request is in flight (button in loading state).
- **Client enters a phone number that matches a Client record for a different Pro entirely** -- Treated identically to no match at all: the plain "no bookings found" message, with no hint that the number exists for another Pro (FEAT-06.SPEC-008).
- **Client navigates away while the request is in flight** -- The request completes in the background; if the client returns to this screen, it reloads in the Empty state rather than showing a stale result.
- **No Access Link screen ever updates an existing Access Link, Client, or Booking record** -- this is a creation-only screen (it reads Client to match, and creates a new Access Link), so no concurrent-edit conflict entry applies here; the Access Link entity's own single-use contention is handled at redemption time by FEAT-06.SPEC-002.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-007 (Access Link Lifecycle & Scope Rules) | References (outbound) | Rate-limit check at request time and the 30-minute expiry set on the created link |
| FEAT-06.SPEC-008 (Client Identity & Privacy Isolation Rule) | References (outbound) | Phone-to-Client matching and the privacy isolation boundary |
| FEAT-06.SPEC-006 (Access Link Delivery) | Triggers (outbound) | A successful match triggers delivery of the generated link |
| FEAT-06.SPEC-002 (Access Link Validation & Redemption) | Navigation (inbound) | Routes back here with a re-request prompt after an invalid, expired, or already-used link |
| FEAT-05.SPEC-003 (Client Details & Consent) | Related (no navigation) | FEAT-05.SPEC-003 performs the returning-client lookup in place (name pre-fill) and does not navigate here; that in-place lookup applies the same identity-match rule as FEAT-06.SPEC-008 |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| access_link_requested | outcome (matched / no_match / rate_limited), entry source (default / re_request_prompt / from_waitlist) | "Send me a link" completes processing | supports success-metrics.md: "Self-Service Access Success" |
| access_link_delivery_failed | retry_attempted (yes / no) | Text or email delivery of the generated link fails | supports success-metrics.md: "Self-Service Access Success" |

## Acceptance Criteria

**FEAT-06.SPEC-001-AC-01:** Given Riley is on the Access Link Request screen, when she enters the phone number she used at booking and taps "Send me a link", then the number is matched to her Client record, an Access Link is generated, delivery is triggered, and the result region shows "Check your phone for a link to view your bookings."

**FEAT-06.SPEC-001-AC-02:** Given Riley is on the Access Link Request screen, when she taps "Send me a link" with the phone field empty, then the field shows the error "Enter a valid phone number" and no request is sent.

**FEAT-06.SPEC-001-AC-03:** Given Riley enters a phone number with no matching Client record for this Pro, when she taps "Send me a link", then the result region shows a plain "no bookings found" message with no hint about whether the number exists elsewhere.

**FEAT-06.SPEC-001-AC-04:** Given Riley has already requested the maximum number of links for her phone number within the current rate-limit window (FEAT-06.SPEC-007), when she taps "Send me a link" again, then the result region shows the rate-limited message and no new link is generated.

**FEAT-06.SPEC-001-AC-05:** Given Riley's requested link fails to deliver, when the failure is detected, then the result region shows "We couldn't send your link. Try again." with a "Try again" action.

**FEAT-06.SPEC-001-AC-06:** Given Riley taps "Try again" after a delivery failure and the original link is still valid, when the retry is processed, then the same Access Link is re-delivered rather than a new one being generated.

**FEAT-06.SPEC-001-AC-07:** Given Riley arrives at this screen from FEAT-06.SPEC-002 after tapping an expired link, when the screen loads, then it shows the "That link no longer works. Request a new one." prompt with a single-tap "Request a new link" action.

**FEAT-06.SPEC-001-AC-08:** Given Riley is viewing the re-request prompt, when she taps "Request a new link", then the screen returns to its empty state with the phone field cleared and focused.

**FEAT-06.SPEC-001-AC-09:** Given Riley is on this screen with no connectivity, when the screen detects she is offline, then the banner "You're offline. Connect to the internet to request a link." appears and the "Send me a link" button is disabled.

**FEAT-06.SPEC-001-AC-10:** Given Riley loses connectivity while the offline banner is shown, when connectivity is restored, then the banner clears and the "Send me a link" button re-enables.

**FEAT-06.SPEC-001-AC-11:** Given Riley taps "Send me a link" and the request is still processing, when she taps the button again immediately, then the second tap is ignored and the button remains in its loading state.

**FEAT-06.SPEC-001-AC-12:** Given Riley arrives at this screen from the waitlist Confirmed state (FEAT-20.SPEC-001) or a waitlist expiry notification (FEAT-20.SPEC-009), when the screen loads, then the form starts empty and she enters her phone number and taps "Send me a link".

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 7 | 7 |
| States | 5 (no bookings found, rate limited, delivery failed, re-request prompt, offline) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |

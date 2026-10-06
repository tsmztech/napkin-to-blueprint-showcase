---
document_type: spec
spec_type: screen
spec_id: FEAT-05.SPEC-001
spec_name: Public Booking Page (Landing & Service List)
spec_slug: public-booking-page-landing-service-list
parent_feature: FEAT-05
parent_feature_name: Public Booking Page & Booking Flow
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Screen Spec: Public Booking Page (Landing & Service List)

## Overview

**Name:** Public Booking Page (Landing & Service List)
**ID:** FEAT-05.SPEC-001
**Type:** Screen
**Purpose:** Entry point a client reaches from the Pro's Instagram bio link, showing the Pro's public profile and service list with prices, durations, and the deposit rule in plain words, from which the client picks a service to begin booking.
**Parent Feature:** FEAT-05 -- Public Booking Page & Booking Flow

## Scope and Non-Goals

**In Scope:**
- Displaying the Pro's public profile (display name, photo, intro, general area)
- Listing the Pro's active services with name, price, duration, and deposit rule in plain language
- Letting the client pick a service to begin the booking flow
- Rendering the identical screen for a Pro previewing their own page

**Non-Goals:**
- Deciding whether the normal flow, a paused message, or an unavailable message renders here -- governed by FEAT-05.SPEC-008 (Booking Page Availability Gate); this spec covers only the normal-flow rendering
- Computing the exact deposit amount and cancellation cut-off for a specific booking -- that is FEAT-05.SPEC-004 and FEAT-05.SPEC-009's responsibility once a service and time are chosen; this screen shows only the service-level deposit rule (e.g., "50% deposit required")
- Listing time slots -- handled by FEAT-05.SPEC-002 (Slot Selection) once a service is picked
- Editing services, prices, or the Pro's profile -- owned by FEAT-01 (Service & Pricing Management) and FEAT-27 (Pro Profile & Booking Page Settings); this screen only displays what those features have set

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| External (Instagram bio link) | Client taps the Pro's booking link | None -- page loads for this Pro's booking_link_name |
| External (forwarded former link name) | Client taps a link using a name the Pro renamed away from within the last 12 months (platform parameter: `booking-link-forward-window-months`) (XBR-27) | Forwarded transparently to the current booking_link_name; client sees no difference |
| FEAT-15 (Pro Onboarding & Setup Wizard) | Pro taps "preview" during first-time setup | Preview mode flag -- identical screens render, no real payment is taken |
| FEAT-27.SPEC-001 (Profile & Booking Page Settings) (Pro Profile & Booking Page Settings) | Pro taps "preview" from settings | Preview mode flag -- identical screens render, no real payment is taken |
| FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) | Support taps the Booking Page Preview entry during an active support session | Preview mode flag -- identical screens render read-only, no real payment is taken |
| FEAT-20.SPEC-009 (Waitlist Expiry Notification) | Client taps "Join the waitlist again" in a waitlist expiry notice | None -- page loads for this Pro's booking_link_name |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Full screen -- this is the primary, intended audience | Select a service to begin booking | -- |
| The Pro (Talia), preview mode | Full screen, identical to what a client sees | Walk through service selection exactly as a client would, but no deposit is ever charged | -- |
| Platform Operator (Support) | Full screen, reached only through the read-only account view (FEAT-19) after a Pro's help request, never through the live public link itself | View only -- cannot select a service or proceed into the booking flow | Attempting to select a service is not offered; the support view is display-only, consistent with XBR-24 |
| Unauthenticated | Yes -- this is the default and intended state for the Client. No sign-in exists or is required for this role (BRIEF.md: clients "must not face a signup wall or need a password-style account") | Yes, identical to the Client row above | -- |
| Expired session | N/A -- clients never hold a session on this page to expire; each visit is independent, and previously entered flow data on a return visit is handled by the persistence rule in Business Rules, not by session state | N/A | N/A |

## Layout and Content

**Header:** The Pro's profile block -- photo (or a placeholder if none is set), display_name, intro (if provided), and general_area. When rendered in preview mode, a persistent banner reads "Preview -- this is what your clients see. No payment will be taken." above the profile block, visible to the Pro only.

**Body:** A vertically stacked list of the Pro's Active services, in display_order, below the profile block. Each service item shows:
- Service name
- Price (in the Pro's account currency)
- Duration
- Deposit rule in plain language (e.g., "$25 deposit required" or "50% deposit required"), derived from the service's deposit_rule

Each service item is a single tappable element.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint (phone width, the primary target -- this page is designed mobile-first for an in-app social-media browser):** Profile block and service list both full width, single column, as described above.
- **Medium size class and above:** Content remains single-column and is capped at a comfortable reading width, horizontally centered; no structural change beyond width capping, since the product's entire audience for this screen is expected at phone width (BRIEF.md, Scale & Non-Functional Expectations).

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Service item | Tap | Navigate to FEAT-05.SPEC-002 (Slot Selection) with the chosen service's ID and duration | Screen transitions to Slot Selection | Standard forward transition; the chosen service's name and price remain visible as context on the next screen (Shared UI Pattern: persistence across steps) |
| Profile photo, intro, general_area | -- | Display-only, non-interactive | None | -- |
| Preview banner (Pro only) | -- | Display-only, non-interactive | None | -- |

### Accessibility Notes

- **Focus order:** Profile block (photo, then display name, then intro, then general area) -> service list, top to bottom, in display_order.
- **Announcements:** On initial load, the page title (the Pro's display name) is announced to assistive technology. If the service list is empty or fails to load, the resulting state message (see States) is announced when it appears.
- **Keyboard alternatives:** Every service item is reachable and selectable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Loading | Profile block and service list show a neutral loading placeholder | Page first requested | Data loads successfully or a load error occurs |
| Loaded (normal) | Profile and full active service list rendered as described in Layout and Content | Data loads successfully and at least one Active service exists | Client taps a service, or the Pro edits services elsewhere and the page is reloaded |
| Empty (no active services) | Profile block renders normally; in place of the service list, a plain message: "This pro hasn't added any services yet. Check back soon." No service is selectable. | Pro Account exists, availability gate (FEAT-05.SPEC-008) allows the normal flow, but zero services are in Active status | A service becomes Active and the page is reloaded |
| Error | A plain error message: "Something went wrong loading this page. Try again." with a Retry action | The profile or service data fails to load | Client taps Retry and the load succeeds, or the client leaves the page |
| Offline/Degraded | A plain banner: "Check your connection and try again." replaces the body content; the profile header remains visible if already loaded | Connectivity is lost while loading or after load, before a service is selected | Connectivity is restored and the page (or the in-progress load) completes |

## Validation Rules

Not applicable -- this screen has no user input, only a selection action. Field validation for downstream steps is governed by FEAT-05.SPEC-007 (Booking Details Field Validation).

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|-------------------------------------|
| Service item tap | FEAT-05.SPEC-002 (Slot Selection) | -- |

## Data Model

**Creates:** None.
**Reads:** Pro Account -- display_name, photo, intro, general_area (public profile fields only; studio_address is never shown here per its confirmation-only disclosure rule). Service -- name, price, duration, deposit_rule, display_order, status (Active services only, in display_order).
**Updates:** None.
**Deletes:** None.

## Business Rules

- This screen renders only when FEAT-05.SPEC-008 (Booking Page Availability Gate) determines the normal flow applies; the gate's paused and unavailable messages replace this screen's content entirely rather than layering on top of it.
- A renamed booking link (booking_link_name) keeps forwarding transparently from its previous name for at least 12 months (platform parameter: `booking-link-forward-window-months`) (XBR-27); the client never sees or needs to know a rename happened.
- Every previously entered value from a later step in this flow (time, name, phone, email, note, acknowledgment) persists if the client returns to this screen and re-advances, per the feature's Shared UI Pattern -- this screen itself holds no such state, since it is always the entry point.
- The deposit rule shown here is the service-level rule (fixed amount or percentage); the exact deposit amount and cancellation cut-off for a specific booking are computed later by FEAT-05.SPEC-009 and shown on FEAT-05.SPEC-004.
- Preview mode (FEAT-15, FEAT-27) renders this screen identically to what a client sees, with a Pro-only banner overlay and no real payment ever taken through the flow it leads into.

## Edge Cases

- **A service is archived by the Pro while a client is viewing this page** -- The page shows the list as loaded; if the client selects that service on a stale render, FEAT-05.SPEC-002's slot request detects the service is no longer Active and the client sees a plain "this service is no longer available" message with a refreshed service list (this screen only reads Service data and creates no record itself, so no concurrent-edit conflict arises on this screen; the conflict is caught downstream).
- **All of a Pro's services are archived, leaving none Active** -- The Empty state renders; no service is selectable.
- **Client double-taps a service item rapidly** -- The second tap is ignored while navigation to FEAT-05.SPEC-002 is already in progress.
- **Client navigates directly to this URL after previously abandoning a booking mid-flow** -- The page loads fresh with no pre-selected service; any previously entered client details from a later step are preserved per the feature's persistence rule only if the client resumes from where they left off, not by re-entering here.
- **Client's in-app browser caches a stale version of the service list** -- The page re-fetches the current Active service list on load rather than trusting a cached render, so an out-of-date price is never shown.

## Connected Specs

| Connected Spec | Connection Type | Description |
|-----------------|-------------------|--------------|
| FEAT-05.SPEC-002 (Slot Selection) | Navigation (outbound) | Client's service selection navigates here with the chosen service's ID and duration |
| FEAT-05.SPEC-008 (Booking Page Availability Gate) | References (inbound) | This screen renders only when the gate resolves to the normal flow |
| FEAT-15 (Pro Onboarding & Setup Wizard) | Navigation (inbound) | Pro reaches this screen in preview mode during first-time setup |
| FEAT-27 (Pro Profile & Booking Page Settings) | Navigation (inbound) | Pro reaches this screen in preview mode from settings |
| FEAT-01 (Service & Pricing Management) | References (inbound) | Source of the Service records this screen displays |
| FEAT-19 (Platform Support Read-Only Access) | Navigation (inbound) | Support reaches a read-only rendering of this screen through the account view after a help request |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-------------------|
| booking_page_viewed | entry source (bio link / forwarded link / preview), service count shown | Page finishes loading in the normal flow | supports success-metrics.md: "Booking Completion Speed" |
| service_selected | service ID, position in list | Client taps a service item | supports success-metrics.md: "Booking Completion Speed" |
| booking_page_load_failed | reason category (error / empty / offline) | The Error, Empty, or Offline/Degraded state renders | supports success-metrics.md: "Booking Completion Speed" (a failed or empty load directly threatens the under-one-minute benchmark) |

## Acceptance Criteria

**FEAT-05.SPEC-001-AC-01:** Given Riley taps the Pro's Instagram bio link, when the page loads, then Riley sees the Pro's display name, photo, intro, and general area, followed by a list of the Pro's active services showing name, price, duration, and deposit rule in plain words.

**FEAT-05.SPEC-001-AC-02:** Given Riley is viewing the service list, when Riley taps a service, then Riley is taken to FEAT-05.SPEC-002 (Slot Selection) for that service.

**FEAT-05.SPEC-001-AC-03:** Given Talia previews her own booking page from FEAT-27, when the page loads, then Talia sees the identical screen a client would see, with an added preview banner reading "Preview -- this is what your clients see. No payment will be taken."

**FEAT-05.SPEC-001-AC-04:** Given a Pro has no Active services, when Riley opens the booking page, then Riley sees the message "This pro hasn't added any services yet. Check back soon." and no service is selectable.

**FEAT-05.SPEC-001-AC-05:** Given the page data fails to load, when Riley opens the booking link, then Riley sees "Something went wrong loading this page. Try again." with a Retry action.

**FEAT-05.SPEC-001-AC-06:** Given Riley loses connectivity while the page is loading, when the load attempt fails due to connectivity, then Riley sees "Check your connection and try again." and can retry once connectivity returns.

**FEAT-05.SPEC-001-AC-07:** Given a Pro renamed her booking link within the last 12 months, when Riley taps a bio link still using the old name, then Riley is forwarded transparently to the current page with no visible difference.

**FEAT-05.SPEC-001-AC-08:** Given Platform Operator (Support) opens the read-only account view after a help request, when the booking page rendering appears, then Support can view the profile and service list but has no service-selection control available.

**FEAT-05.SPEC-001-AC-09:** Given Riley double-taps a service item, when the first tap has already begun navigation, then the second tap has no additional effect.

**FEAT-05.SPEC-001-AC-10:** Given a service Riley is about to select is archived by the Pro moments earlier, when Riley taps it, then FEAT-05.SPEC-002 reports the service is no longer available and Riley sees a refreshed service list rather than an error.

**FEAT-05.SPEC-001-AC-11:** Given FEAT-05.SPEC-008 determines the Pro's account is paused, when Riley opens the booking link, then this screen's normal content does not render at all -- the gate's own message renders instead.

**FEAT-05.SPEC-001-AC-12:** Given Riley opens the booking page on a phone-width in-app browser, when the page renders, then the profile block and service list are both fully readable and every service item is reliably tappable without horizontal scrolling.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 3 | 3 |
| States | 5 (loading, loaded, empty, error, offline) | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |

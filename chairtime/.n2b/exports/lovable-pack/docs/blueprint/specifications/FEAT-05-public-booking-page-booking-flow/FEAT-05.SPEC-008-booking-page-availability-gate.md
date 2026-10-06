---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-05.SPEC-008
spec_name: Booking Page Availability Gate
spec_slug: booking-page-availability-gate
parent_feature: FEAT-05
parent_feature_name: Public Booking Page & Booking Flow
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 6
acceptance_criteria_count: 13
---

# Logic/Rule Spec: Booking Page Availability Gate

## Overview

**Name:** Booking Page Availability Gate
**ID:** FEAT-05.SPEC-008
**Type:** Logic/Rule
**Purpose:** Determines, on every load of the booking link, whether to render the normal booking flow, a plain "not accepting bookings" message, or a plain "this booking page isn't available" message.
**Parent Feature:** FEAT-05 -- Public Booking Page & Booking Flow
**Governed Entity:** Pro Account (availability-gating fields: status, booking_link_name) and Payout Account status (read-only input to this gate)

## Scope and Non-Goals

**In Scope:**
- Resolving the requested booking_link_name to a Pro Account, including forwarding a renamed link for at least 12 months (platform parameter: `booking-link-forward-window-months`)
- Evaluating the Pro Account's status (Active, Paused, Closing, Closed) to decide whether bookings may be taken
- Evaluating the Payout Account's status to decide whether the link can go live and accept deposits
- Defining the exact messages shown for the paused and unavailable outcomes

**Non-Goals:**
- Setting or changing the Pro Account's pause state itself -- owned by FEAT-27 (Pro Profile & Booking Page Settings); this spec only reads the current state to decide what renders
- Setting or changing the Payout Account's status -- owned by FEAT-28 (Payout Account Connection & Payout Visibility); this spec only reads the current status
- Deciding go-live readiness during onboarding (whether every required setup step is complete) -- owned by FEAT-15's go-live rule (XBR-26); this spec governs what an already-live-or-not link shows on each visit, not the one-time go-live decision itself
- Rendering the normal flow's actual content once the gate allows it -- owned by FEAT-05.SPEC-001 through FEAT-05.SPEC-005, each of which renders only after this gate resolves to "normal flow"

## Governed Entity

**Entity:** Pro Account (availability-gating fields) and Payout Account (status only)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| status | enum (Active \| Paused \| Closing \| Closed) | The Pro Account's current status; Paused covers both subscription lapse and a Pro-chosen pause, each optionally with a pause message and end date |
| booking_link_name | text | The public link segment a visitor's request is resolved against, including any of the Pro's previous names still within their 12-month forwarding window |
| payout_account_status (read-only, from Payout Account) | enum (Not Connected \| Verification Pending \| Active \| Action Required \| Disconnected) | Whether the Pro's payout account is active enough to accept new deposits |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|---------------------|
| FEAT-05.SPEC-001 | Public Booking Page (Landing & Service List) | On every page load, before any of the screen's own content renders |
| FEAT-05.SPEC-002 | Slot Selection | On every page load, as a continuation of the same gated flow |
| FEAT-05.SPEC-003 | Client Details & Consent | On every page load, as a continuation of the same gated flow |
| FEAT-05.SPEC-004 | Policy Acknowledgment & Deposit Checkout | On every page load, as a continuation of the same gated flow -- also re-checked when the client taps Acknowledge & continue (before the checkout hold and navigation to FEAT-07.SPEC-001), since a pause or payout change could occur mid-flow |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| booking_link_name | Must resolve to an existing Pro Account, either directly or via a still-forwarding previous name (within 12 months of the rename, platform parameter: `booking-link-forward-window-months`) | Always | On every page load | Not applicable -- an unresolved link renders the "page isn't available" outcome, not a form-field error | Yes (blocks the entire flow) |
| status | Must be Active for the normal flow to render | Always | On every page load and re-checked at payment time | Not applicable -- a non-Active status renders the paused or unavailable outcome, not a form-field error | Yes (blocks the entire flow) |
| payout_account_status (read-only) | Must be Active for the normal flow to render | Always | On every page load and re-checked at payment time | Not applicable -- a non-Active payout status renders the unavailable outcome, not a form-field error | Yes (blocks the entire flow) |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|-----------------|
| Outcome precedence | status, payout_account_status, booking_link_name | Evaluated in order: (1) does booking_link_name resolve at all -- if not, "page isn't available"; (2) is status Closed or Closing -- if so, "page isn't available"; (3) is status Paused -- if so, "not accepting bookings" with any pause message the Pro set; (4) is payout_account_status not Active -- if so, "not accepting bookings" (the client experience is identical to a Pro-chosen pause, since neither case should ever expose deposit collection); (5) otherwise, render the normal flow | See individual outcome messages below |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View the normal booking flow | The Client (Riley) | Only when the gate resolves to "normal flow" | If Paused or payout-inactive: "This pro isn't taking new bookings right now." (plus any Pro-set pause message and end date, if set); if link unresolved or account Closed/Closing: "This booking page isn't available." |
| View the normal booking flow in preview mode | The Pro (Talia) | Always, regardless of the gate's outcome for real clients -- preview always shows the normal flow so the Pro can review it even while paused or before go-live | Not applicable -- preview mode is never denied by this gate; a Pro who wants to see how a real client experiences a pause views it explicitly labeled as such in FEAT-27, not through this gate |
| Take a deposit through the flow this gate protects | The Client (Riley) | Only when the gate resolves to "normal flow" (which already requires payout_account_status Active) | No deposit collection is ever offered when the gate does not resolve to "normal flow" (XBR-06) |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Resolved Pro Account | Direct match on the current booking_link_name, or the Pro Account whose forwarding table still lists the requested name within its 12-month window | On every page load | No -- this is a system lookup, not a user choice |
| Gate outcome (normal / paused / unavailable) | Derived from the Cross-Field Rule's precedence order above | On every page load, and re-derived when the client taps Acknowledge & continue on FEAT-05.SPEC-004 | No |

## Business Rules

- No deposit can be taken, and the booking link cannot go live, unless the Pro's payout account is active (XBR-06); this gate is the enforcement point for that rule on every visit, not only at go-live.
- A paused account (subscription lapse after the 7-day grace, platform parameter: `subscription-payment-failure-grace-period-days`, or a Pro-chosen pause; the combined Paused state is resolved by FEAT-27.SPEC-009, Pause State Precedence Rule, which this gate reads) takes no new bookings or deposits, while existing bookings keep their reminders, refunds, and client self-service unchanged (XBR-14) -- this gate affects only new-booking entry, never an existing Booking's own lifecycle.
- A renamed booking link keeps forwarding from the old name for at least 12 months (platform parameter: `booking-link-forward-window-months`); a closed, paused-to-closure, or mistyped link shows the plain "this booking page isn't available" message, never another Pro's page (XBR-27).
- The gate is re-evaluated when the client taps Acknowledge & continue on FEAT-05.SPEC-004 (before the checkout hold is placed and the client reaches FEAT-07.SPEC-001), not only on initial page load, so a pause or payout change that occurs mid-flow is caught before any hold or charge (consistent with FEAT-05.SPEC-004's Business Rules).
- Preview mode (reached only by the signed-in Pro through FEAT-15 or FEAT-27) always renders the normal flow regardless of this gate's outcome for real clients, since its purpose is for the Pro to review the client-facing screens.

## Edge Cases

- **A visitor uses a booking_link_name that was renamed more than 12 months ago** -- The forwarding window has closed; the link no longer resolves, and the visitor sees "This booking page isn't available."
- **A Pro pauses their account while a client is mid-flow on FEAT-05.SPEC-002 or FEAT-05.SPEC-003** -- The gate's re-check on FEAT-05.SPEC-004's Acknowledge & continue catches the new Paused status before any hold or charge; the client sees "This pro isn't taking new bookings right now." instead of reaching payment.
- **A Pro's payout account moves from Active to Action Required while a client is mid-flow** -- The same re-check on Acknowledge & continue catches this; the client is refused before any hold or charge, with the same "not accepting bookings" experience as a Pro-chosen pause (the client is never shown a payout-specific technical message).
- **A Pro sets a pause message and end date, then removes the pause before the end date arrives** -- The very next page load reflects the Active status with no lingering pause message.
- **The Pro previews the page while genuinely paused** -- Preview mode still renders the normal flow, per the Authorization Rules; the Pro sees the paused experience only by explicitly viewing it as such in FEAT-27, not by this gate substituting it into preview.
- **Two mistyped variations of a link both fail to resolve** -- Both show the identical generic "This booking page isn't available." message; the gate never reveals whether a name was once valid, was never valid, or belongs to a since-closed account.
- **A closed account's former link is reused as a rename target by a different, unrelated Pro** -- Not possible under this gate: booking_link_name is unique across all Pro Accounts at any given time (Pro Account field definition), so no ambiguity between a closed Pro's old name and a different Pro's chosen name can arise.

## Acceptance Criteria

**FEAT-05.SPEC-008-AC-01:** Given a Pro Account is Active with an active payout account, when Riley opens the booking link, then the gate resolves to "normal flow" and FEAT-05.SPEC-001 renders normally.

**FEAT-05.SPEC-008-AC-02:** Given a Pro Account is Paused (subscription lapse or Pro-chosen), when Riley opens the booking link, then Riley sees "This pro isn't taking new bookings right now." with any pause message and end date the Pro set.

**FEAT-05.SPEC-008-AC-03:** Given a Pro Account's payout account status is not Active, when Riley opens the booking link, then Riley sees the "not accepting bookings" experience, with no mention of payout status specifically.

**FEAT-05.SPEC-008-AC-04:** Given a Pro Account's status is Closed or Closing, when Riley opens the booking link, then Riley sees "This booking page isn't available."

**FEAT-05.SPEC-008-AC-05:** Given a visitor uses a booking_link_name that does not resolve to any Pro Account, when the page loads, then the visitor sees "This booking page isn't available."

**FEAT-05.SPEC-008-AC-06:** Given a Pro renamed her booking link within the last 12 months, when Riley uses the old name, then the request resolves and forwards transparently to the current page.

**FEAT-05.SPEC-008-AC-07:** Given a Pro renamed her booking link more than 12 months ago, when a visitor uses the old name, then the request no longer resolves and the visitor sees "This booking page isn't available."

**FEAT-05.SPEC-008-AC-08:** Given Talia previews her own booking page while her account is genuinely Paused, when she opens the preview, then she sees the normal flow, not the paused message.

**FEAT-05.SPEC-008-AC-09:** Given a Pro's account transitions to Paused while Riley is mid-flow on FEAT-05.SPEC-003, when Riley reaches FEAT-05.SPEC-004 and taps Acknowledge & continue, then the gate's re-check refuses to advance (no hold is placed) and Riley sees "This pro isn't taking new bookings right now."

**FEAT-05.SPEC-008-AC-10:** Given a Pro's payout account moves out of Active status while Riley is mid-flow, when Riley taps Acknowledge & continue, then the gate's re-check refuses to advance with the same "not accepting bookings" experience as a pause.

**FEAT-05.SPEC-008-AC-11:** Given a Pro removes a pause before its stated end date, when the page is next loaded, then the gate resolves to "normal flow" with no lingering pause message.

**FEAT-05.SPEC-008-AC-12:** Given a Pro Account is Closing (within its 30-day cooling-off period after an account-closure request), when Riley opens the booking link, then Riley sees "This booking page isn't available."

**FEAT-05.SPEC-008-AC-13:** Given two different mistyped link variations, when each is requested, then both show the identical generic "This booking page isn't available." message with no distinguishing detail.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 3 | 3 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 3 | 3 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |

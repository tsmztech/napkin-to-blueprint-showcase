# FEAT-05 — Public Booking Page & Booking Flow

This chapter covers Public Booking Page & Booking Flow (FEAT-05), a Core-tier feature. It carries 9 specifications carrying 121 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-05.SPEC-001 | Public Booking Page (Landing & Service List) | screen | 12 |
| FEAT-05.SPEC-002 | Slot Selection | screen | 14 |
| FEAT-05.SPEC-003 | Client Details & Consent | screen | 13 |
| FEAT-05.SPEC-004 | Policy Acknowledgment & Deposit Checkout | screen | 15 |
| FEAT-05.SPEC-005 | Booking Confirmation | screen | 12 |
| FEAT-05.SPEC-006 | Slot Hold & Re-Validation at Checkout | automation | 12 |
| FEAT-05.SPEC-007 | Booking Details Field Validation | logic-rule | 16 |
| FEAT-05.SPEC-008 | Booking Page Availability Gate | logic-rule | 13 |
| FEAT-05.SPEC-009 | Policy Acknowledgment Capture & Integrity Check | logic-rule | 14 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Public Booking Page & Booking Flow

## Summary

**Feature:** Public Booking Page & Booking Flow
**ID:** FEAT-05
**Description:** The single, mobile-first page a client reaches from the Pro's Instagram bio link -- showing the Pro's name, services with prices and durations, and the deposit rule in plain words -- where a client picks a service, a genuinely free time, enters their name and phone, opts into texts, and pays the deposit, all in one continuous flow.
**Priority:** Core
**Phase:** MVP
**Type:** User-Facing
**Rationale:** This is the literal product described in BRIEF.md's Vision: "a client opens the pro's link... picks a service and a genuinely free time... pays a card deposit." It is the entire reason the product exists and the founder's stated one-minute benchmark. [RESEARCH-INFORMED: added the policy-disclosure gap -- clients disputing deposit and cancellation charges they did not understand at booking is a recurring complaint, from BBB complaint records and forum-derived complaint summaries (2 source types, MEDIUM confidence)]

**Key Capabilities:**
- View a Pro's services, prices, durations, and deposit rule in plain language
- Pick a service and a genuinely free time slot
- Enter name and phone, and opt in to text messages
- Explicitly acknowledge the deposit and cancellation policy, shown in plain words with the exact amounts and cut-off time for this booking, before paying
- Provide an email address when declining texts, so confirmations and reminders can still arrive by email; optionally add a short note for the Pro
- Complete deposit payment and receive an immediate on-screen confirmation

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-05.SPEC-001 | Public Booking Page (Landing & Service List) | Screen | The Client, The Pro (preview), Platform Operator (Support) | Entry point showing the Pro's public profile and service list with prices, durations, and deposit rule in plain words |
| FEAT-05.SPEC-002 | Slot Selection | Screen | The Client, The Pro (preview), Platform Operator (Support) | Client picks a genuinely free time for the chosen service from the live slot list |
| FEAT-05.SPEC-003 | Client Details & Consent | Screen | The Client, The Pro (preview), Platform Operator (Support) | Client enters name and phone, opts into texts, provides email if declining, and adds an optional note |
| FEAT-05.SPEC-004 | Policy Acknowledgment & Deposit Checkout | Screen | The Client, The Pro (preview), Platform Operator (Support) | Client sees this booking's exact deposit and cancellation terms, acknowledges them, and continues into the deposit payment step owned by FEAT-07.SPEC-001 (card entry and Pay live there) |
| FEAT-05.SPEC-005 | Booking Confirmation | Screen | The Client, The Pro (preview), Platform Operator (Support) | Client sees an immediate on-screen confirmation of the completed booking |
| FEAT-05.SPEC-006 | Slot Hold & Re-Validation at Checkout | Automation | The Client, The Pro | Places a short checkout hold and creates the Pending Payment Booking when the client taps Acknowledge & continue on SPEC-004 (not at slot pick), re-validates it before charging, and resolves expiry or contention outcomes |
| FEAT-05.SPEC-007 | Booking Details Field Validation | Logic/Rule | The Client | Validation rules governing name, phone, email, opt-in, and note fields captured in the flow |
| FEAT-05.SPEC-008 | Booking Page Availability Gate | Logic/Rule | The Client, The Pro | Determines whether the page shows the normal flow, a "not accepting bookings" message, or a "page isn't available" message |
| FEAT-05.SPEC-009 | Policy Acknowledgment Capture & Integrity Check | Logic/Rule | The Client, The Pro | Computes this booking's exact deposit amount and cancellation cut-off, records the acknowledged policy version and wording on the Booking, and re-validates that version at payment time |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| View a Pro's services, prices, durations, and deposit rule in plain language | FEAT-05.SPEC-001 | Primary purpose of the landing/service-list screen | Phase 2 (Explicit) |
| Pick a service and a genuinely free time slot | FEAT-05.SPEC-001, FEAT-05.SPEC-002, FEAT-05.SPEC-006 | Service pick on the landing screen; time pick on the Slot Selection screen (no hold is placed at slot pick); the checkout hold is placed and re-validated by the checkout automation when the client taps Acknowledge & continue | Phase 2 (Explicit) / Phase 4 (Trigger-Response) |
| Enter name and phone, and opt in to text messages | FEAT-05.SPEC-003, FEAT-05.SPEC-007 | Primary purpose of the Client Details screen; field rules enforced by the validation spec | Phase 2 (Explicit) / Phase 5 (Rule Discovery) |
| Explicitly acknowledge the deposit and cancellation policy, with exact amounts and cut-off, before paying | FEAT-05.SPEC-004, FEAT-05.SPEC-009 | Primary purpose of the checkout screen; exact amount/cut-off computation and version capture governed by the policy-acknowledgment rule | Phase 2 (Explicit) / Phase 5 (Rule Discovery) |
| Provide an email address when declining texts; optional note for the Pro | FEAT-05.SPEC-003, FEAT-05.SPEC-007 | Client Details screen collects both fields; validation spec enforces the conditional-required email and note length/hint | Phase 2 (Explicit) / Phase 5 (Rule Discovery) |
| Complete deposit payment and receive an immediate on-screen confirmation | FEAT-05.SPEC-004, FEAT-05.SPEC-005 | Checkout screen navigates to FEAT-07.SPEC-001, which owns card entry and capture and returns to the confirmation screen; confirmation screen displays the result immediately | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-05.SPEC-006 | Slot Hold & Re-Validation at Checkout | Phase 4 (Trigger-Response) | XBR-01 (first payer wins a contested slot) and XBR-02 (checkout hold of a few minutes) imply a system process -- placing a hold, re-validating it at payment, and resolving expiry/contention -- that the feature description never states directly |
| FEAT-05.SPEC-007 | Booking Details Field Validation | Phase 5 (Rule Discovery) | The Validation & Limits field names five-plus distinct rules on the captured details (name length, phone format, conditional-required email, actively-checked opt-in, note length/hint) -- past the inline threshold, so it becomes a standalone Logic/Rule shared across the details and checkout screens |
| FEAT-05.SPEC-008 | Booking Page Availability Gate | Phase 6 (Negative/Failure Analysis) | The Alternate flow (paused Pro account), the States field's Empty case, and the Access field's unauthorized-visitor line (mistyped/closed link) all describe conditions where the normal flow must not render -- a conditional-display rule shared by every screen in the feature, elaborating XBR-06, XBR-14, and XBR-27 |
| FEAT-05.SPEC-009 | Policy Acknowledgment Capture & Integrity Check | Phase 5 (Rule Discovery) | The [AUDIT-ADDED] disclosure requirement and the Cancellation Policy entity's contention rule ("if the version changes between acknowledgment and payment, the client is refused with refresh") both describe derivation and cross-entity-consistency logic too complex to leave inline in the checkout screen |

## Entity-Lifecycle Coverage Matrix

**Entity: Booking**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-05.SPEC-006 | A Booking record is created in the Pending Payment state when the client taps Acknowledge & continue on SPEC-004 and the checkout hold is placed (aligned to FEAT-03.SPEC-002); slot pick creates neither the hold nor the Booking, fixing the service, start time, duration, client reference, and agreed price/deposit | Booking is also created by FEAT-30 and FEAT-21; this row covers only the client-initiated path |
| Read (single) | FEAT-05.SPEC-004, FEAT-05.SPEC-005 | Checkout screen reads the in-progress terms (the Booking itself exists only once the client advances into payment); Confirmation screen reads the completed booking to display its details | -- |
| Read (list) | N/A | Not applicable to this feature -- a client's own booking list is reached through Client Booking Identity (FEAT-06), and the Pro's schedule is FEAT-12 | -- |
| Update | N/A -- Owned by FEAT-07 | The Pending Payment -> Confirmed transition is written by Deposit Payment at Booking (FEAT-07) once the deposit succeeds; this feature only supplies the Pending Payment record and observes the result | -- |
| Delete/Archive | N/A | Bookings are never deleted -- kept for the life of the account (SC-22); recorded as a product-wide non-goal, not a gap specific to this feature | -- |
| State Transition | FEAT-05.SPEC-006 (Pending Payment -> Expired (unpaid)); N/A -- Owned by FEAT-07 (Pending Payment -> Confirmed) | Hold expiry without payment moves the booking to Expired (unpaid) (written per FEAT-03) and releases the slot; a successful deposit's Confirmed transition is written by FEAT-07 | -- |

**Entity: Client**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-05.SPEC-003 | A new phone number entered on the Client Details screen creates a Client record scoped to this Pro | Phone-number match within a Pro resolves to a single record rather than a duplicate (dependency map's Client contention rule) |
| Read (single) | FEAT-05.SPEC-003 | The screen requests a phone-based identity lookup from Client Booking Identity (FEAT-06); a match pre-fills the client's name | Cross-feature inbound -- see Cross-Feature Touchpoints |
| Read (list) | N/A | Not applicable -- this feature never lists clients | -- |
| Update | N/A -- Owned by FEAT-06 | The dependency map attributes a returning client's own updates to their email and consent to Client Booking Identity (FEAT-06), even though a returning client reaches that update path through this feature's screens | Flagged, not resolved, per the instruction to elaborate the dependency map's ownership lines rather than re-derive them |
| Delete/Archive | N/A -- Owned by FEAT-13 | Client deletion (hard delete of contact details and notes; de-identified financial history retained) is Client Record Management's (FEAT-13) responsibility | -- |
| State Transition | N/A | The Client entity carries no lifecycle state field distinct from its Messaging Consent and Booking history | -- |

**Entity: Messaging Consent**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-05.SPEC-003 | Actively checking the texting opt-in creates a Granted consent record with the exact wording shown and a timestamp | If the client declines texts, no consent record is created for that channel at all -- the client instead supplies an email for the fallback channel |
| Read (single) | N/A | Not applicable to this feature -- consulted before every message by Automated Booking Messaging (FEAT-08) and viewed by the Pro via FEAT-12 | -- |
| Read (list) | N/A | Not applicable to this feature | -- |
| Update | N/A -- Owned by FEAT-14 / FEAT-06 | Revocation (STOP or opt-out link) is Messaging Consent Management's (FEAT-14) responsibility; re-grant on a later visit is Client Booking Identity's (FEAT-06) | -- |
| Delete/Archive | N/A -- Owned by FEAT-13 | Deleted only as part of Client deletion; evidence retained where law requires | -- |
| State Transition | FEAT-05.SPEC-003 | Sets the initial Granted state at booking time; later transitions (Revoked, Re-granted) belong to FEAT-14 and FEAT-06 | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Service | FEAT-05.SPEC-001, FEAT-05.SPEC-002, FEAT-05.SPEC-004 | Name, price, duration, and deposit rule shown on the landing page and used to compute the exact deposit at checkout |
| Cancellation Policy | FEAT-05.SPEC-004, FEAT-05.SPEC-009 | Current plain-language wording and version shown for acknowledgment; version re-checked against what was acknowledged before payment |
| Pro Account | FEAT-05.SPEC-001, FEAT-05.SPEC-008 | Public profile fields (name, photo, intro, general area) shown on the landing page; status, pause state, and booking-link validity read by the availability gate |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Client picks a service | Load the live slot list for that service | Cross-feature (Inbound from FEAT-03) | FEAT-05.SPEC-002 |
| Client picks a free time | Re-check the time against the live slot list; no hold is placed and no Booking is created | Inline in triggering screen | FEAT-05.SPEC-002 |
| Client taps Acknowledge & continue on the policy acknowledgment | Place the short checkout hold on the slot, create the Booking in Pending Payment state, and navigate to FEAT-07.SPEC-001 for card entry | Standalone Automation | FEAT-05.SPEC-006 |
| Client submits payment on FEAT-07.SPEC-001 | Re-validate the held slot is still valid before charging | Standalone Automation | FEAT-05.SPEC-006 |
| Slot hold expires before payment completes | Booking moves to Expired (unpaid); client sees "that hold has expired, please pick a time again" and returns to the live slot list, never charged | Standalone Automation (elaboration) / inline in Slot Selection | FEAT-05.SPEC-006 / FEAT-05.SPEC-002 |
| Two clients contest the same slot | First to complete payment wins; the other sees a plain "just taken" message, never a payment error | Standalone Automation (elaboration of XBR-01) | FEAT-05.SPEC-006 |
| Client enters name, phone, opt-in, email, note | Validate every field per its rule (length, format, conditional-required email, actively-checked opt-in, note length/hint) | Standalone Logic/Rule | FEAT-05.SPEC-007 |
| Client checks the policy acknowledgment box | Record the exact policy version, wording, and timestamp on the Booking | Standalone Logic/Rule | FEAT-05.SPEC-009 |
| Client taps Acknowledge & continue | Re-verify the acknowledged policy version still matches the Pro's current version; if changed, refuse and require re-acknowledgment of the refreshed wording | Standalone Logic/Rule | FEAT-05.SPEC-009 |
| Client (or Pro previewing) loads the booking link | Check Pro Account status and payout account status to decide whether to render the normal flow, a "not accepting bookings" message, or a "this booking page isn't available" message | Standalone Logic/Rule | FEAT-05.SPEC-008 |
| Client enters a phone number matching an existing record | Recognize the returning client and pre-fill their name | Cross-feature (Inbound from FEAT-06) | FEAT-05.SPEC-003 |
| Deposit payment succeeds | FEAT-07.SPEC-001 returns the client to the on-screen confirmation, shown immediately | Cross-feature (Inbound from FEAT-07) | FEAT-05.SPEC-005 |
| Deposit payment succeeds | Write an Activity Event, trigger the booking confirmation message, and update the Pro's connected calendar and schedule | Cross-feature (FEAT-16, FEAT-08, FEAT-04, FEAT-12 responsibilities) | Owned by those features |
| Payment fails or is declined | FEAT-07.SPEC-001 shows the specific decline reason; the slot stays held; retry with a different card without re-entering name, phone, service, or the policy acknowledgment | Cross-feature (FEAT-07.SPEC-001; relies on SPEC-006's hold retention) | FEAT-05.SPEC-006 / FEAT-07.SPEC-001 |
| Client loses connection mid-flow | Show a plain "check your connection and try again" message; nothing charged | Inline in each screen's Offline/Degraded state | FEAT-05.SPEC-002 / SPEC-003 / SPEC-004 |
| A chosen service has no remaining free times | Offer a "Join the waitlist" action that navigates to FEAT-20.SPEC-001 | Cross-feature (Outbound to FEAT-20) | FEAT-05.SPEC-002 |
| Booking confirmation is shown | Offer an optional "Make this a standing appointment" action that navigates to FEAT-21.SPEC-001 | Cross-feature (Outbound to FEAT-21) | FEAT-05.SPEC-005 |

The Communications field names one message (the immediate booking confirmation), and it is explicitly delegated: "Triggers the immediate booking confirmation handled by Automated Booking Messaging (FEAT-08)." Delivery rules, channel selection, and content are owned entirely by FEAT-08's Notification spec; this feature's own on-screen confirmation is a same-screen success display with no delivery rules of its own, so it stays inline (SPEC-005) rather than becoming a Notification spec here -- `notification_count: 0` is intentional. Likewise, payment processing is owned by FEAT-07, transactional messaging by FEAT-08, and file storage for the profile photo by FEAT-27 (this feature only displays the stored photo) -- all three category-level dependencies from assumptions-constraints.md are satisfied by other features' Integration specs, so `integration_count: 0` is intentional, not an omission.

## Shared Context

**Shared Entities:**
- Booking -- created in Pending Payment state by SPEC-006 when the client taps Acknowledge & continue and the checkout hold is placed (not at slot pick); read by SPEC-004 and SPEC-005; its policy_version and acknowledgment fields are written by SPEC-009. Fields relevant to this feature: service, start_time, duration, client, price_agreed, deposit_amount, policy_version, state, source.
- Client -- created or matched by SPEC-003 (via FEAT-06's phone lookup); validated by SPEC-007. Fields relevant to this feature: name, phone, email, booking_notes.
- Messaging Consent -- created by SPEC-003 when the client opts in. Fields relevant to this feature: channel, state, timestamp, exact consent wording shown.

**Shared UI Patterns:**
- Continuous single flow -- SPEC-001 through SPEC-005 form one uninterrupted sequence at phone width inside an in-app browser; every previously entered value (service, time, name, phone, email, note, acknowledgment) persists across steps and survives an error, per the feature's States field ("a failed step... keeps all previously entered information intact"). Spec Writers for all five screens should describe this persistence consistently rather than re-deriving it per screen.
- Plain-language money and policy display -- SPEC-001 (deposit rule) and SPEC-004 (exact deposit amount and cut-off time) share one formatting convention for presenting currency and cancellation terms in plain words, computed by SPEC-009.
- Preview mode -- SPEC-001 through SPEC-005 all support a Pro-facing preview rendering (reached from FEAT-15 or FEAT-27) that walks the identical screens a client would see, without taking a real payment.

**Shared Validation:**
- SPEC-007 defines all client-entered field rules (name, phone, email, opt-in, note); SPEC-003 references it rather than duplicating the rules.
- SPEC-009 defines the policy-acknowledgment computation and integrity check; SPEC-004 references it rather than duplicating the rule.

## Internal Dependency Map

```
SPEC-001 (Public Booking Page) -> [rendered under] -> SPEC-008 (Booking Page Availability Gate)
SPEC-001 (Public Booking Page) -> [Client picks a service] -> SPEC-002 (Slot Selection)
SPEC-002 (Slot Selection) -> [Client picks a free time; no hold placed] -> SPEC-003 (Client Details & Consent)
SPEC-002 (Slot Selection) -> [fully booked service, Client taps join waitlist] -> FEAT-20.SPEC-001 (Join Waitlist, cross-feature)
SPEC-003 (Client Details & Consent) -> [validates entries using] -> SPEC-007 (Booking Details Field Validation)
SPEC-003 (Client Details & Consent) -> [Client taps Continue] -> SPEC-004 (Policy Acknowledgment & Deposit Checkout)
SPEC-004 (Policy Acknowledgment & Deposit Checkout) -> [Client ticks policy agreement] -> SPEC-009 (Policy Acknowledgment Capture & Integrity Check)
SPEC-004 (Policy Acknowledgment & Deposit Checkout) -> [Client taps Acknowledge & continue] -> SPEC-006 (Slot Hold & Re-Validation at Checkout) -> [hold placed, Pending Payment Booking created] -> FEAT-07.SPEC-001 (Deposit Payment, card entry, cross-feature)
FEAT-07.SPEC-001 (Deposit Payment) -> [Client submits payment] -> SPEC-006 (re-validates hold before charging)
FEAT-07.SPEC-001 (Deposit Payment) -> [payment succeeds] -> SPEC-005 (Booking Confirmation)
FEAT-07.SPEC-001 (Deposit Payment) -> [payment declined] -> retry inline on FEAT-07.SPEC-001 (hold preserved by SPEC-006)
SPEC-006 (Slot Hold & Re-Validation at Checkout) -> [hold expires unpaid] -> SPEC-002 (Slot Selection, refreshed list)
SPEC-005 (Booking Confirmation) -> [Client taps Make this a standing appointment] -> FEAT-21.SPEC-001 (Set Up Recurring Series, cross-feature)
FEAT-06 (Client Booking Identity) -> [phone number recognized] -> SPEC-003 (Client Details & Consent, name pre-filled)
```

**Default Entry:** SPEC-001 (Public Booking Page) -- the screen a client reaches directly from the Pro's Instagram bio link.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-05.SPEC-002 | Inbound | FEAT-03 (Real-Time Slot Availability Engine) | Live slot list for the chosen service | Client picks a service |
| FEAT-05.SPEC-006 | Outbound | FEAT-03 (Real-Time Slot Availability Engine) | Requests the checkout hold (FEAT-03.SPEC-002) and re-validates the slot before payment | Client taps Acknowledge & continue / submits payment |
| FEAT-05.SPEC-003 | Inbound | FEAT-06 (Client Booking Identity) | Recognizes a returning client by phone number and pre-fills their name | Client enters a phone number matching an existing record |
| FEAT-05.SPEC-004 | Outbound | FEAT-07 (Deposit Payment at Booking) | Navigates to FEAT-07.SPEC-001, which owns card entry and the Pay action | Client taps Acknowledge & continue |
| FEAT-05.SPEC-004 | Inbound | FEAT-07 (Deposit Payment at Booking) | Client backing out of the payment screen returns to the acknowledgment with the hold still active | Client taps back on FEAT-07.SPEC-001 |
| FEAT-05.SPEC-005 | Inbound | FEAT-07 (Deposit Payment at Booking) | FEAT-07.SPEC-001 returns to the confirmation on payment success | Deposit payment succeeds |
| FEAT-05.SPEC-008 | Inbound | FEAT-27 (Pro Profile & Booking Page Settings) | Pro's pause state, optional pause message, and booking-link status feed the availability gate | Page load / Pro toggles pause or renames the link |
| FEAT-05.SPEC-008 | Inbound | FEAT-28 (Payout Account Connection & Payout Visibility) | Payout account status determines whether the link can go live and accept deposits (XBR-06) | Page load |
| FEAT-05.SPEC-005 | Outbound | FEAT-08 (Automated Booking Messaging) | Triggers the immediate booking confirmation message | Deposit payment succeeds |
| FEAT-05.SPEC-005 | Outbound | FEAT-12 (Pro Daily Schedule Dashboard) | The confirmed booking becomes visible on the Pro's schedule | Deposit payment succeeds |
| FEAT-05.SPEC-002 | Outbound | FEAT-20 (Waitlist for Cancelled Slots) | A fully booked service offers a "Join the waitlist" action navigating to FEAT-20.SPEC-001 | Client taps join waitlist on a fully booked service |
| FEAT-05.SPEC-005 | Outbound | FEAT-21 (Recurring/Standing Appointments) | An optional "Make this a standing appointment" action navigates to FEAT-21.SPEC-001 | Client taps the action on the confirmation |
| FEAT-05.SPEC-001 | Inbound | FEAT-15 (Pro Onboarding & Setup Wizard) | Pro previews the booking page exactly as a client will see it before the link goes live | Pro taps preview during first-time setup |
| FEAT-05.SPEC-001 | Inbound | FEAT-27 (Pro Profile & Booking Page Settings) | Pro previews the booking page from settings | Pro taps preview |

## Non-Functional Notes

**Data volumes / growth:** This is the product's highest-traffic surface -- every one of a Pro's 20-40 bookings a week and their eventual multi-year booking history begins here (ASMP-22); the page and slot list must stay equally responsive as a Pro's history accumulates over years (SC-22).

**Responsiveness:** Available slots appear within roughly one second of a service selection, and the full flow -- landing through paid confirmation -- completes in under one minute (ASMP-21; success metrics: Booking Completion Speed, Slot Search Responsiveness). The flow requires a live connection for anything that books or pays and says so plainly when connectivity is missing, per the product-wide offline stance (ASMP-27). Every screen is readable and fully operable at phone width inside an in-app social-media browser, with scaling text, sufficient contrast, reliably tappable controls, and full screen-reader support (ASMP-28).

**Data sensitivity / privacy:** This feature captures personal data -- client name, phone, optional email, and an optional note -- visible only to the client themselves and their one Pro, never to any other pro or client (ASMP-23; SC-03). The policy acknowledgment and its timestamp form dispute evidence and must be retained with the Booking. Service names, prices, and the cancellation policy wording are intentionally public.

**Compliance flags:** US SMS-consent rules govern the texting opt-in captured here -- it must be an explicit, never-pre-checked action, with the exact wording shown kept as evidence (ASMP-24). No health-data regime applies: the optional note is explicitly hinted not to contain medical information, since health intake is out of scope (SC-08).

## Non-Goals

- **Health or medical intake forms, and custom intake questionnaires** -- Excluded per SC-08: the only free text this feature captures is a short optional note to the Pro, hinted not to contain medical information; structured intake forms are explicitly out of scope for v1.
- **Handling or storing card data, or card entry itself, within this feature** -- Excluded per SC-11: the checkout screen collects a policy acknowledgment and navigates to FEAT-07.SPEC-001, which owns card entry and alone ever touches card data.
- **Client accounts with passwords** -- Excluded per SC-04 and BRIEF.md's Target Users & Roles ("must not face a signup wall or need a password-style account"): identity here is the client's phone number with this one Pro, established through FEAT-06, never a login.
- **Card-reader hardware or in-person balance payment inside this feature** -- Excluded per SC-16: at MVP this feature only ever takes the deposit; the balance is settled off-platform between the Pro and client by whatever means the Pro already uses.
- **Instagram integration beyond serving as the link's destination** -- Excluded per SC-06: this feature is what the bio link points to and nothing more; no Instagram-side integration exists.
- **Capturing a waitlist entry or setting up a recurring series** -- SPEC-002 only offers the entry point to FEAT-20.SPEC-001 (Join Waitlist) and SPEC-005 only offers the entry point to FEAT-21.SPEC-001 (Set Up Recurring Series); contact capture, entry creation, and series rules belong entirely to those features.



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



# Screen Spec: Client Details & Consent

## Overview

**Name:** Client Details & Consent
**ID:** FEAT-05.SPEC-003
**Type:** Screen
**Purpose:** Client enters their name and phone, opts into text messages, provides an email if declining texts, and adds an optional note for the Pro.
**Parent Feature:** FEAT-05 -- Public Booking Page & Booking Flow

## Scope and Non-Goals

**In Scope:**
- Capturing name, phone, texting opt-in, conditionally-required email, and an optional note
- Recognizing a returning client by phone number, in place on this screen, and pre-filling their name (no navigation to FEAT-06 screens)
- Creating or matching the Client record and creating the Messaging Consent record on continue
- Advancing to the policy acknowledgment and checkout step

**Non-Goals:**
- Field-level validation rules (format, length, conditional requirements) -- defined in full by FEAT-05.SPEC-007 (Booking Details Field Validation); this screen enforces those rules but does not define them
- Updating a returning client's own email or consent after this booking -- owned by FEAT-06 (Client Booking Identity), which the returning client reaches through a manage link, not through this screen on a later visit
- Collecting or displaying the deposit and cancellation policy -- handled entirely by FEAT-05.SPEC-004, the next step
- Health or medical intake -- excluded per scope-boundaries.md SC-08; the optional note is hinted not to contain medical information and no structured intake question is offered

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-05.SPEC-002 (Slot Selection) | Client picks a time that passes the live re-check | Chosen service, chosen time (not yet held) |
| FEAT-05.SPEC-004 (Policy Acknowledgment & Deposit Checkout) | Client navigates back before paying | Same service and chosen time; previously entered name, phone, opt-in, email, and note preserved |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Full screen | Enter all fields and continue | -- |
| The Pro (Talia), preview mode | Full screen, identical rendering | Fill and continue exactly as a client would; no real Client or Messaging Consent record is created against a real client (preview data is discarded, per Business Rules) | -- |
| Platform Operator (Support) | Full screen, read-only, reached only through FEAT-19's account view | View field layout only -- never a real client's entered data mid-flow, since support only reaches this rendering outside a live session | Form fields are not editable; consistent with XBR-24 |
| Unauthenticated | Yes -- the default and intended state for the Client role | Yes, identical to the Client row above | -- |
| Expired session | N/A -- no session exists to expire on this public flow; no checkout hold exists yet on this step; the hold (platform parameter: `checkout-hold-timeout-minutes`) starts only when the client advances into the payment step | N/A | N/A |

## Layout and Content

**Header:** Back arrow (returns to FEAT-05.SPEC-002) with the chosen service and chosen time shown as persistent context.

**Body:** A single-column form with the following fields in order:
- Name (text input, required)
- Phone (text input, required)
- Text message opt-in (checkbox, never pre-checked, with the exact consent wording shown beside it)
- Email (text input, required only when the opt-in checkbox is unchecked; optional otherwise)
- Note to the Pro (multi-line text input, optional, with a visible hint: "e.g., \"first full set\" -- please don't include medical information")

**Footer:** "Continue" action button, full width, advances to FEAT-05.SPEC-004.

### Responsive Behavior

- **Compact breakpoint:** Single-column form as described, full width, Continue in the footer.
- **Medium size class and above:** Form remains single-column, capped at a comfortable form width and horizontally centered; no structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-05.SPEC-002 (Slot Selection), entries preserved | Screen closes | Standard backward transition |
| Phone input | Blur, matching an existing Client record for this Pro | Look up the returning client in place, applying the same phone-based identity-match rule FEAT-06 (Client Booking Identity) uses; no navigation occurs | Name field auto-fills with the matched client's name | Name field shows the pre-filled value, editable by the client |
| Name input | Type / blur | Captures text input; validated per FEAT-05.SPEC-007 | Field shows entered text or error state | Standard input feedback; error message per FEAT-05.SPEC-007 on invalid blur |
| Phone input | Type / blur | Captures text input; validated per FEAT-05.SPEC-007 | Field shows entered text or error state | Standard input feedback; error message per FEAT-05.SPEC-007 on invalid blur |
| Text opt-in checkbox | Tap | Toggles opt-in state; when checked, the Email field becomes optional; when unchecked, Email becomes required | Email field's required indicator updates immediately | Checkbox shows checked/unchecked state with the exact consent wording remaining visible |
| Email input | Type / blur | Captures text input; validated per FEAT-05.SPEC-007 (conditionally required) | Field shows entered text or error state | Standard input feedback; error message per FEAT-05.SPEC-007 |
| Note input | Type | Captures text input; length-checked per FEAT-05.SPEC-007 | Field shows entered text and remaining character count | Standard input feedback |
| Continue button | Tap | 1. Validate all fields via FEAT-05.SPEC-007. 2. If valid, create or match the Client record and create the Messaging Consent record (consent capture is handled by FEAT-14.SPEC-003). 3. Navigate to FEAT-05.SPEC-004. | Button shows loading state during processing | Success: navigate to FEAT-05.SPEC-004. Failure: inline field-level error messages, focus moves to the first invalid field |
| Continue button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> Name -> Phone -> Text opt-in checkbox -> Email -> Note -> Continue.
- **Validation announcements:** When a field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Consent wording:** The exact texting consent wording is always presented adjacent to, and read together with, the opt-in checkbox by assistive technology -- it is never conveyed by a separate disconnected label.
- **Keyboard alternatives:** Every action on this screen, including the opt-in checkbox, is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Empty (default) | All fields empty except any auto-filled name from a phone match; Continue enabled | Screen first opens for this booking attempt | Client begins typing or continues |
| Filling | Form fields contain client input, Continue enabled | Client types in any field | Client taps Continue or navigates away |
| Validating | Continue button shows a loading indicator | Client taps Continue | Validation completes (pass or fail) |
| Validation Error | Failed fields highlighted with error messages below them, per FEAT-05.SPEC-007 | Validation fails | Client corrects the field(s) and re-triggers validation |
| Processing | Continue button shows a loading indicator, form disabled | Validation passes, Client/Consent creation begins | Processing completes or fails |
| Error | Error banner: "Something went wrong saving your details. Try again." with a Retry option | Client or Consent creation fails after validation passes | Client taps Retry, or navigates away with entered data preserved |
| Offline/Degraded | Banner: "Check your connection and try again." at the top; form remains fillable but Continue is disabled until connectivity returns; nothing is queued, since this step requires a live connection (per the product-wide offline stance) | Connectivity lost while the screen is open | Connectivity restored -- Continue re-enables |

## Validation Rules

Validation governed by FEAT-05.SPEC-007 (Booking Details Field Validation). See that spec for all field-level and cross-field rules (name, phone, conditionally required email, actively-checked opt-in, note length/hint). This screen applies validation on field blur and on Continue.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|-------------------------------------|
| Back arrow tap | FEAT-05.SPEC-002 (Slot Selection) | -- |
| Successful Continue | FEAT-05.SPEC-004 (Policy Acknowledgment & Deposit Checkout) | -- |

## Data Model

**Creates:** Client record -- name, phone, email (if provided), booking_notes (the optional note) set from form input, scoped to this Pro. Messaging Consent record -- channel: text, state: Granted, timestamp, and the exact consent wording shown, created only if the opt-in checkbox is checked.
**Reads:** An existing Client record, via FEAT-06's phone-based lookup, to pre-fill name for a returning client.
**Updates:** None -- a returning client's own later updates to email or consent are owned by FEAT-06, not this screen.
**Deletes:** None.

## Business Rules

- A phone number matching an existing Client record within this Pro resolves to that single record rather than creating a duplicate (dependency map's Client contention note); the matched client's name is pre-filled and editable. The lookup runs in place on this screen and applies the same phone-based identity-match rule FEAT-06 uses (including for FEAT-06.SPEC-001's access-link requests); it does not navigate to any FEAT-06 screen.
- The Booking Page Availability Gate (FEAT-05.SPEC-008) is evaluated on every load of this screen, so a paused or unavailable page never shows this form (XBR-06, XBR-14, XBR-27).
- Creating the Messaging Consent record on Continue triggers FEAT-14.SPEC-003 (Consent Capture at Booking), which records the consent state, exact wording, and timestamp under FEAT-14's rules (XBR-15).
- If the client declines texts (opt-in left unchecked), no Messaging Consent record is created for the text channel at all; the client must instead supply an email so confirmations and reminders can arrive by the email fallback (XBR-15).
- The texting opt-in is never pre-checked -- it must be an explicit, affirmative action, with the exact wording shown kept as evidence (US SMS-consent rules, ASMP-24).
- Every value entered on this screen persists if the client navigates back to FEAT-05.SPEC-002 or forward to FEAT-05.SPEC-004 and returns, or if a failed step is retried, per the feature's Shared UI Pattern.
- In preview mode, the Pro's entries on this screen are discarded on completion and never create a real Client or Messaging Consent record.

## Edge Cases

- **Client navigates away and returns with unsaved changes** -- No confirmation dialog is needed; entered values are preserved automatically per the feature's persistence rule, so nothing is at risk of being discarded.
- **Client taps Continue twice rapidly** -- The second tap is ignored while the first Continue action is in progress (button in loading state).
- **A second, unrelated booking attempt with the same phone number arrives at effectively the same time (two first bookings for the same phone close together)** -- Both attempts resolve to a single merged Client record rather than creating a duplicate; field values from whichever Continue action completes second are last-write-wins on the shared Client record's contact fields, per the dependency map's Client contention note.
- **Client enters a phone number, then changes it before continuing** -- Any name pre-fill from the original phone's match is cleared, and a fresh lookup runs against the new number on blur.
- **All optional fields (email when texting is opted in, note) left empty** -- Continue proceeds; email and note are stored as empty.
- **Network failure during Client/Consent creation, after validation passes** -- Error banner: "Something went wrong saving your details. Try again." with Retry; entered form data is preserved.
- **Client loses connectivity while filling the form** -- The Offline/Degraded state renders; Continue is disabled until connectivity returns, since this step requires a live connection.

## Connected Specs

| Connected Spec | Connection Type | Description |
|-----------------|-------------------|--------------|
| FEAT-05.SPEC-002 (Slot Selection) | Navigation (inbound) | Client arrives here after picking a time that passes the live re-check |
| FEAT-05.SPEC-004 (Policy Acknowledgment & Deposit Checkout) | Navigation (outbound) | Successful Continue advances the client here |
| FEAT-05.SPEC-007 (Booking Details Field Validation) | References (inbound) | Validation and consent rules applied to form fields |
| FEAT-05.SPEC-008 (Booking Page Availability Gate) | References (inbound) | Governs whether this screen renders on every load |
| FEAT-06 (Client Booking Identity) | References (outbound) | Source of the phone-based identity-match rule applied by the in-place lookup; no navigation to any FEAT-06 screen (FEAT-06.SPEC-001 is not an entry from this screen) |
| FEAT-14.SPEC-003 (Consent Capture at Booking) -- within FEAT-14 (Messaging Consent Management) | Triggers (outbound) | Continue's creation of the Messaging Consent record triggers consent capture |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-------------------|
| client_details_submitted | returning client (yes/no), texting opted in (yes/no), note provided (yes/no) | Continue succeeds and the client advances | supports success-metrics.md: "Booking Completion Speed" |
| returning_client_recognized | -- | Phone lookup matches an existing Client record and pre-fills the name | supports success-metrics.md: "Self-Service Access Success" |
| client_details_validation_failed | field(s) that failed | Continue is blocked by a validation error | supports success-metrics.md: "Booking Completion Speed" |

## Acceptance Criteria

**FEAT-05.SPEC-003-AC-01:** Given Riley has just picked a time on FEAT-05.SPEC-002, when the Client Details & Consent screen loads, then Riley sees empty Name, Phone, opt-in, Email, and Note fields with the chosen service and time shown as context.

**FEAT-05.SPEC-003-AC-02:** Given Riley enters a phone number matching an existing Client record with this Pro, when the phone field loses focus, then the Name field auto-fills with the matched client's name in place, with no navigation away from this screen.

**FEAT-05.SPEC-003-AC-03:** Given Riley fills in valid Name and Phone, checks the texting opt-in, and taps Continue, then the Messaging Consent record is created with state Granted and the exact wording shown (triggering FEAT-14.SPEC-003 consent capture), and Riley advances to FEAT-05.SPEC-004.

**FEAT-05.SPEC-003-AC-04:** Given Riley leaves the texting opt-in unchecked, when Riley taps Continue without an email entered, then the Email field shows a required-field error and Continue does not proceed.

**FEAT-05.SPEC-003-AC-05:** Given Riley leaves the texting opt-in unchecked and provides a valid email, when Riley taps Continue, then no Messaging Consent record is created for the text channel, and Riley advances with the email stored for the fallback channel.

**FEAT-05.SPEC-003-AC-06:** Given Riley is on this screen, when Riley looks at the opt-in checkbox on first load, then it is unchecked by default -- never pre-checked.

**FEAT-05.SPEC-003-AC-07:** Given Riley enters a note exceeding the length limit defined by FEAT-05.SPEC-007, when the note field loses focus, then the field shows the corresponding error message from that spec.

**FEAT-05.SPEC-003-AC-08:** Given Riley taps Continue twice in rapid succession, when the first tap has already begun processing, then the second tap has no additional effect.

**FEAT-05.SPEC-003-AC-09:** Given Riley loses connectivity while filling the form, when the Offline/Degraded state renders, then Continue is disabled and Riley sees "Check your connection and try again."

**FEAT-05.SPEC-003-AC-10:** Given Riley navigates back to FEAT-05.SPEC-002 from this screen and returns, when the screen reloads, then all previously entered field values are preserved.

**FEAT-05.SPEC-003-AC-11:** Given two first-time bookings with the same phone number arrive close together for the same Pro, when both complete Continue, then the Client record is merged rather than duplicated, per the dependency map's Client contention note.

**FEAT-05.SPEC-003-AC-12:** Given Talia previews her own booking page and reaches this screen, when she fills in the form and taps Continue, then the identical flow advances her with no real Client or Messaging Consent record created.

**FEAT-05.SPEC-003-AC-13:** Given Riley's Client/Consent creation fails due to a processing error after validation passes, when the failure occurs, then Riley sees "Something went wrong saving your details. Try again." with entered data preserved.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 7 (empty, filling, validating, validation error, processing, error, offline) | 7 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |



# Screen Spec: Policy Acknowledgment & Deposit Checkout

## Overview

**Name:** Policy Acknowledgment & Deposit Checkout
**ID:** FEAT-05.SPEC-004
**Type:** Screen
**Purpose:** Client sees this booking's exact deposit and cancellation terms, explicitly acknowledges them, and continues into the deposit payment step (FEAT-07.SPEC-001), where the checkout hold starts and the deposit is paid.
**Parent Feature:** FEAT-05 -- Public Booking Page & Booking Flow

## Scope and Non-Goals

**In Scope:**
- Displaying the booking summary (service, time, price) and the exact deposit amount and cancellation cut-off computed for this specific booking
- Capturing the client's explicit acknowledgment of the deposit and cancellation policy
- On "Acknowledge & continue": re-checking the acknowledged policy version and the booking page's availability, triggering the checkout hold and the Pending Payment Booking (FEAT-05.SPEC-006), and navigating the client into the payment step (FEAT-07.SPEC-001)
- Showing the outcome of that hand-off when it fails -- policy changed, slot lost, or page no longer available -- and the corresponding next step

**Non-Goals:**
- Computing the exact deposit amount and cancellation cut-off, or capturing the acknowledged policy version -- owned by FEAT-05.SPEC-009 (Policy Acknowledgment Capture & Integrity Check); this screen displays what that spec computes and calls it to record the acknowledgment
- Card entry, the Pay action, payment processing, declines, retries, and the payment success/hold-expired states -- owned by FEAT-07.SPEC-001 (Deposit Payment), which this screen navigates to; this screen never renders card fields or touches card data (scope-boundaries.md SC-11)
- Placing and expiring the checkout hold, and creating the Pending Payment Booking -- owned by FEAT-05.SPEC-006, aligned to FEAT-03.SPEC-002 (the hold starts when the client advances into the payment step); this screen only triggers it
- Rendering the on-screen confirmation itself -- handled by FEAT-05.SPEC-005 once payment succeeds, reached from FEAT-07.SPEC-001

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-05.SPEC-003 (Client Details & Consent) | Client taps Continue after entering valid details | Chosen service, chosen time (not yet held), entered name/phone/opt-in/email/note |
| FEAT-07.SPEC-001 (Deposit Payment) | Client taps the back arrow on the payment screen | The in-progress Booking and its still-active checkout hold; the prior acknowledgment preserved |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Full screen | Acknowledge the policy and continue to payment | -- |
| The Pro (Talia), preview mode | Full screen, identical rendering, including the exact deposit amount and cut-off her own settings would produce | Walk through acknowledgment and continue to the simulated payment step (FEAT-07.SPEC-001) with no real hold, Booking, or card charge (Business Rules) | -- |
| Platform Operator (Support) | Full screen, read-only, reached only through FEAT-19's account view | View the policy display only -- never a real client's in-progress checkout | Acknowledgment checkbox and continue control are not offered; consistent with XBR-24 |
| Unauthenticated | Yes -- the default and intended state for the Client role | Yes, identical to the Client row above | -- |
| Expired session | N/A -- no session exists to expire; on first arrival no checkout hold exists yet, and once the client has continued, the hold's own timeout governs how long they have to complete payment (see Business Rules, Edge Cases) | N/A | N/A |

## Layout and Content

**Header:** Back arrow (returns to FEAT-05.SPEC-003, all entries preserved) with a booking summary: service name, date and time, and full price.

**Body:**
- Deposit and cancellation policy block, in plain language, stating: the exact deposit amount due now for this booking, the exact date and time the cancellation window closes for this appointment, and what happens to the deposit if the client cancels or no-shows inside versus outside that window.
- Policy acknowledgment checkbox, never pre-checked, labeled with a plain-language statement that checking it means agreeing to the terms shown above.

**Footer:** "Acknowledge & continue" action button, full width, disabled until the acknowledgment checkbox is checked. Card entry and the Pay action (labeled with the deposit amount) live on FEAT-07.SPEC-001, the next screen.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described, full width, action button in the footer.
- **Medium size class and above:** Content remains single-column, capped at a comfortable form width and horizontally centered; no structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-05.SPEC-003 (Client Details & Consent), entries preserved | Screen closes | Standard backward transition |
| Policy acknowledgment checkbox | Tap | Triggers FEAT-05.SPEC-009 to record the intent to acknowledge; toggles checkbox state | "Acknowledge & continue" enables when checked | Checkbox shows checked/unchecked state |
| Acknowledge & continue button | Tap (checkbox checked) | 1. Re-validate the acknowledged policy version still matches the current version, via FEAT-05.SPEC-009. 2. Re-check the booking page's availability gate (FEAT-05.SPEC-008). 3. If unchanged, trigger FEAT-05.SPEC-006 to request the checkout hold from FEAT-03.SPEC-002 and create the Pending Payment Booking. 4. If the hold is placed, navigate to FEAT-07.SPEC-001 (Deposit Payment). | Button shows loading state | Success: navigate to FEAT-07.SPEC-001 with the slot held. Policy changed: refused with the refreshed wording shown and re-acknowledgment required. Slot lost: plain message and return to FEAT-05.SPEC-002. Page unavailable: the gate's plain "not accepting bookings" message |
| Acknowledge & continue button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> booking summary -> policy block -> acknowledgment checkbox -> Acknowledge & continue button.
- **Announcements:** The exact deposit amount and cancellation cut-off are announced as part of the policy block on load. A policy-changed refusal or a slot-lost message is announced immediately when it appears.
- **Keyboard alternatives:** The acknowledgment checkbox and Acknowledge & continue button are reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Empty | N/A -- this screen never renders a collection that can be empty; it always shows a single in-progress booking's summary and policy terms | N/A | N/A |
| Loading | Neutral loading placeholder in place of the booking summary and policy block while this screen's own on-open data (booking summary, exact deposit/cut-off from FEAT-05.SPEC-009, current Cancellation Policy wording) is fetched | Screen first opens, before that data has finished loading | Data loads successfully (-> Unacknowledged) or fails (-> Error) |
| Error | Blocking message: "Couldn't load your booking summary. Try again." with a Retry action in place of the booking summary and policy block; acknowledgment checkbox and continue button are not offered until this resolves | This screen's own on-open data (booking summary, exact deposit/cut-off, current Cancellation Policy wording) fails to load | Client taps Retry and the data loads successfully, or navigates away |
| Unacknowledged | Policy block shown, checkbox unchecked, continue button disabled | Screen first opens | Client checks the acknowledgment checkbox |
| Acknowledged | Checkbox checked, continue button enabled | Client checks the acknowledgment checkbox | Client taps Acknowledge & continue, or unchecks the box |
| Starting checkout | Continue button shows a loading indicator, form disabled | Client taps Acknowledge & continue | The hold is placed (navigate to FEAT-07.SPEC-001) or the hand-off fails (policy changed, slot lost, page unavailable) |
| Policy changed | Blocking message: "The cancellation policy has changed since you agreed to it. Please review the updated terms." with the refreshed wording shown and the checkbox reset to unchecked | FEAT-05.SPEC-009 detects the acknowledged version no longer matches the current version when the client continues | Client re-acknowledges the current wording |
| Slot lost | Plain message: "That time was just taken." or "That time is no longer available." and return to FEAT-05.SPEC-002 | FEAT-05.SPEC-006 reports the hold could not be placed (contested or no longer valid) | Client picks a new time on FEAT-05.SPEC-002 |
| Offline/Degraded | Banner "Check your connection and try again." replaces the continue control; the policy block remains visible if already loaded; continue is disabled | Connectivity is lost while loading or before the hand-off completes | Connectivity is restored and the pending action (load or continue) completes |

## Validation Rules

Validation governed by FEAT-05.SPEC-009 (Policy Acknowledgment Capture & Integrity Check) for the acknowledgment step. This screen requires the acknowledgment checkbox to be checked before Acknowledge & continue enables; card-level validation is owned entirely by the payment-processing capability behind FEAT-07.SPEC-001.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|-------------------------------------|
| Back arrow tap | FEAT-05.SPEC-003 (Client Details & Consent) | -- |
| Acknowledge & continue (hold placed) | FEAT-07.SPEC-001 (Deposit Payment) | FEAT-07 (Deposit Payment at Booking) |
| Slot lost when continuing | FEAT-05.SPEC-002 (Slot Selection), refreshed list | -- |
| Service archived when continuing | FEAT-05.SPEC-001 (Public Booking Page), refreshed service list | -- |

Successful payment navigates onward to FEAT-05.SPEC-005 (Booking Confirmation) from FEAT-07.SPEC-001, not from this screen.

## Data Model

**Creates:** None directly -- the Pending Payment Booking and its checkout hold are created by FEAT-05.SPEC-006 when the client continues, and the Deposit Transaction is created by FEAT-07 on payment.
**Reads:** Service -- price, deposit_rule (for the exact deposit computation, performed by FEAT-05.SPEC-009). Cancellation Policy -- current plain_language_wording, window_hours, version (for display and re-validation). The in-progress booking details (service, chosen time), to display its terms.
**Updates:** The recorded policy_version, acknowledged wording, and acknowledgment timestamp, written by FEAT-05.SPEC-009 when the client checks the acknowledgment box (carried onto the Booking when FEAT-05.SPEC-006 creates it).
**Deletes:** None.

## Business Rules

- The deposit is computed once, exactly, from the service's rule in the Pro's account currency, cannot be altered by the client, and is charged once per booking (XBR-05); the amount shown here is the same amount charged on FEAT-07.SPEC-001.
- No deposit can be taken, and this screen is never reached, unless the Pro's payout account is active (XBR-06) -- enforced by FEAT-05.SPEC-008, on load and again when the client taps Acknowledge & continue.
- Every booking is governed by the cancellation policy version shown and acknowledged at booking; policy edits never change existing bookings once confirmed (XBR-08).
- If the current policy version changes between the client's acknowledgment and the Acknowledge & continue tap, the client is refused and must review and re-acknowledge the refreshed wording (FEAT-05.SPEC-009's integrity check, dependency map's Cancellation Policy contention note).
- If the service is archived before the client continues, the client is refused with a refresh back to the service list (dependency map's Service contention note).
- The checkout hold starts only when the client advances into the payment step by tapping Acknowledge & continue (FEAT-03.SPEC-002, XBR-02); earlier steps hold nothing. Once placed, a declined payment or a return to this screen via the payment screen's back arrow does not lose the hold within its window, and the acknowledgment persists (Shared UI Pattern persistence).
- In preview mode, no real hold, Booking, or card charge is ever created; the Pro continues to a simulated payment step.
- Confirming consent at booking submission triggers FEAT-14.SPEC-003 (Consent Capture at Booking), which records the consent decision under FEAT-14's rules (XBR-15).

## Edge Cases

- **The client returns here from the payment screen's back arrow** -- The acknowledgment and entries are preserved and the existing hold remains active; tapping Acknowledge & continue again reuses the existing Pending Payment Booking and hold rather than placing a second one (FEAT-05.SPEC-006).
- **The acknowledged policy version changes between acknowledgment and the continue tap** -- The hand-off is refused with the "Policy changed" state; the client must review and re-acknowledge the current wording before continuing (FEAT-05.SPEC-009). No hold is placed.
- **The chosen slot is taken or no longer valid by the time the client continues** -- FEAT-05.SPEC-006's hold request is refused before any Booking is created; the client sees the "Slot lost" state and returns to FEAT-05.SPEC-002, and is never sent to the payment step for a slot they do not hold.
- **The chosen service is archived by the Pro before the client continues** -- The hand-off is refused with a refresh back to FEAT-05.SPEC-001's service list; nothing is held.
- **A pause or payout change occurs while the client is on this screen** -- The availability gate (FEAT-05.SPEC-008) re-check on continue shows the plain "not accepting bookings" message; no hold is placed.
- **Client loses connectivity while continuing** -- The Offline/Degraded state renders; nothing is held or charged until the action completes on a live connection, and the client is never shown an ambiguous state.
- **Client taps Acknowledge & continue twice rapidly** -- The second tap is ignored while the first hand-off is in progress.
- **Client unchecks the acknowledgment box after checking it** -- The continue button disables again immediately; no data is lost, and re-checking re-enables it.
- **This screen's own on-open data fails to load** -- The booking summary, the exact deposit/cut-off computed by FEAT-05.SPEC-009, or the current Cancellation Policy wording cannot be fetched; the Error state renders with "Couldn't load your booking summary. Try again." and a Retry action; the acknowledgment checkbox and continue button are withheld until the data loads successfully, so the client is never asked to acknowledge or advance against stale or missing terms.

## Connected Specs

| Connected Spec | Connection Type | Description |
|-----------------|-------------------|--------------|
| FEAT-05.SPEC-003 (Client Details & Consent) | Navigation (inbound) | Client arrives here after entering valid details |
| FEAT-07.SPEC-001 (Deposit Payment) -- within FEAT-07 (Deposit Payment at Booking) | Navigation (outbound) / Navigation (inbound) | Acknowledge & continue navigates here once the hold is placed; this screen owns card entry and payment; its back arrow returns here |
| FEAT-05.SPEC-005 (Booking Confirmation) | References (outbound) | Reached from FEAT-07.SPEC-001 after successful payment, not directly from this screen |
| FEAT-05.SPEC-002 (Slot Selection) | Navigation (outbound) | A lost slot returns the client here with a refreshed list |
| FEAT-05.SPEC-006 (Slot Hold & Re-Validation at Checkout) | Triggers (outbound) | Acknowledge & continue triggers the checkout hold and Pending Payment Booking |
| FEAT-05.SPEC-009 (Policy Acknowledgment Capture & Integrity Check) | References (outbound) | Computes the exact deposit/cut-off shown, and records and re-validates the acknowledgment |
| FEAT-05.SPEC-008 (Booking Page Availability Gate) | References (inbound) | Governs whether this screen renders, on load and again on Acknowledge & continue |
| FEAT-14.SPEC-003 (Consent Capture at Booking) -- within FEAT-14 (Messaging Consent Management) | Triggers (outbound) | Booking submission into checkout triggers consent capture for the client's opt-in/opt-out choice |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-------------------|
| policy_acknowledged | -- | Client checks the acknowledgment checkbox | supports success-metrics.md: "Policy Clarity at Booking" |
| checkout_started | deposit amount, service ID | Client taps Acknowledge & continue and the hold is placed | supports success-metrics.md: "Deposit Capture Rate" |
| policy_version_mismatch_at_continue | -- | FEAT-05.SPEC-009 detects a version change between acknowledgment and the continue tap | supports success-metrics.md: "Policy Clarity at Booking" |

## Acceptance Criteria

**FEAT-05.SPEC-004-AC-01:** Given Riley arrives on this screen after entering valid details, when the screen loads, then Riley sees the booking summary and the exact deposit amount and cancellation cut-off time computed for this specific booking.

**FEAT-05.SPEC-004-AC-02:** Given Riley is on this screen, when Riley looks at the acknowledgment checkbox on first load, then it is unchecked and the Acknowledge & continue button is disabled.

**FEAT-05.SPEC-004-AC-03:** Given Riley checks the acknowledgment checkbox, when the check registers, then the Acknowledge & continue button enables and FEAT-05.SPEC-009 records the acknowledged policy version, wording, and timestamp.

**FEAT-05.SPEC-004-AC-04:** Given Riley has acknowledged the policy and the hand-off checks pass, when Riley taps Acknowledge & continue, then the checkout hold is placed (via FEAT-05.SPEC-006) and Riley navigates to FEAT-07.SPEC-001 (Deposit Payment), where card entry and payment happen.

**FEAT-05.SPEC-004-AC-05:** Given Riley is on this screen, when Riley looks for card fields or a Pay button, then none are offered here; card entry is owned by FEAT-07.SPEC-001.

**FEAT-05.SPEC-004-AC-06:** Given the cancellation policy version changes after Riley acknowledged it but before she taps Acknowledge & continue, when she taps it, then the hand-off is refused, no hold is placed, she sees "The cancellation policy has changed since you agreed to it. Please review the updated terms." with the refreshed wording, and must re-acknowledge before continuing.

**FEAT-05.SPEC-004-AC-07:** Given another client holds Riley's chosen time first, when Riley taps Acknowledge & continue, then no Booking is created, Riley sees "That time was just taken." and is returned to FEAT-05.SPEC-002 with a refreshed list, never sent to the payment step.

**FEAT-05.SPEC-004-AC-08:** Given Riley's chosen time no longer passes the live slot rules when she taps Acknowledge & continue, when the hold request is refused, then she sees "That time is no longer available." and is returned to FEAT-05.SPEC-002 with a refreshed list.

**FEAT-05.SPEC-004-AC-09:** Given the chosen service is archived by the Pro before Riley continues, when Riley taps Acknowledge & continue, then Riley is refused with a refresh back to FEAT-05.SPEC-001's service list and nothing is held.

**FEAT-05.SPEC-004-AC-10:** Given Riley taps Acknowledge & continue twice rapidly, when the first hand-off is already in progress, then the second tap has no additional effect.

**FEAT-05.SPEC-004-AC-11:** Given Riley loses connectivity while continuing, when the Offline/Degraded state renders, then Riley sees "Check your connection and try again." and no hold is placed until the action completes.

**FEAT-05.SPEC-004-AC-12:** Given Riley returns to this screen from FEAT-07.SPEC-001's back arrow while her hold is still active, when she taps Acknowledge & continue again, then the existing hold and Pending Payment Booking are reused and no second hold is placed.

**FEAT-05.SPEC-004-AC-13:** Given Talia previews her own booking page and reaches this screen, when she acknowledges the policy and continues, then she reaches a simulated payment step with no real hold, Booking, or card charge created.

**FEAT-05.SPEC-004-AC-14:** Given Riley unchecks the acknowledgment box after checking it, when the uncheck registers, then the Acknowledge & continue button disables again without losing any other entered data.

**FEAT-05.SPEC-004-AC-15:** Given this screen's own on-open data (booking summary, exact deposit/cut-off, or current Cancellation Policy wording) fails to load, when the screen attempts to render, then Riley sees "Couldn't load your booking summary. Try again." with a Retry action, and neither the acknowledgment checkbox nor the continue button is offered until the data loads successfully.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 4 | 4 |
| States | 9 (empty, loading, error, unacknowledged, acknowledged, starting checkout, policy changed, slot lost, offline) | 9 |
| Business Rules | 8 | 8 |
| Edge Cases | 9 | 9 |



# Screen Spec: Booking Confirmation

## Overview

**Name:** Booking Confirmation
**ID:** FEAT-05.SPEC-005
**Type:** Screen
**Purpose:** Client sees an immediate on-screen confirmation of the completed, paid booking.
**Parent Feature:** FEAT-05 -- Public Booking Page & Booking Flow

## Scope and Non-Goals

**In Scope:**
- Displaying the confirmed booking's details immediately after successful deposit payment
- Reading the completed Booking record to render this confirmation, including when reached on a fresh load after a payment success whose navigation initially failed
- Offering the client a "Make this a standing appointment" action that leads to FEAT-21.SPEC-001 (Set Up Recurring Series)

**Non-Goals:**
- Sending the confirmation message (text or email) -- entirely owned by FEAT-08 (Automated Booking Messaging), triggered by the payment success, not by this screen
- Setting up the recurring series itself -- owned by FEAT-21.SPEC-001; this screen only offers the entry point
- Recording the Activity Event for the new Booking -- owned by FEAT-16.SPEC-002 (Activity Event Recording), triggered by the same confirmed Booking this screen displays
- Any post-confirmation self-service (viewing, cancelling, or rescheduling later) -- owned by FEAT-06 (Client Booking Identity) and FEAT-10 (Client-Initiated Cancel/Reschedule), reached through the confirmation message's manage link, not from this screen directly
- Updating the Pro's schedule or connected calendar -- owned by FEAT-12 (Pro Daily Schedule Dashboard) and FEAT-04 (Two-Way Calendar Sync) respectively, both triggered by the same payment success this screen displays the result of

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-07.SPEC-001 (Deposit Payment) | Deposit payment succeeds and the Booking reaches Confirmed (the payment screen hands off here automatically) | The now-Confirmed Booking's full details |
| FEAT-05.SPEC-004 (Policy Acknowledgment & Deposit Checkout) | A fresh load of the flow after payment succeeded but the confirmation navigation initially failed | The Confirmed Booking, read directly rather than carried in navigation state |
| FEAT-21.SPEC-001 (Set Up Recurring Series) | Client backs out of the recurring set-up without creating a series | The confirmed Booking reference |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Full screen | No further action required; may optionally tap "Make this a standing appointment" | -- |
| The Pro (Talia), preview mode | Full screen, identical rendering, showing a simulated confirmed booking | No further action; the recurring action is shown but no series is created (no real Booking exists) | -- |
| Platform Operator (Support) | Full screen, read-only, reached only through FEAT-19's account view showing a past confirmation rendering for troubleshooting | View only | Not applicable -- this screen has no actions to restrict |
| Unauthenticated | Yes -- the default and intended state for the Client role | Yes, identical to the Client row above | -- |
| Expired session | N/A -- no session exists to expire; the screen renders from the completed Booking record itself, not from in-flow session state | N/A | N/A |

## Layout and Content

**Header:** A success indicator (e.g., a checkmark treatment) with the heading "You're booked!"

**Body:** The confirmed booking's details:
- Service name
- Date and time
- Deposit amount paid
- Balance due at the appointment (price minus deposit)
- The Pro's studio address (shown here for the first time in the flow, per its confirmation-only disclosure rule)
- A short line noting that a confirmation has been sent to the client's chosen channel (text or email, matching what was selected on FEAT-05.SPEC-003)

**Footer:** A secondary "Make this a standing appointment" action offering to turn the just-confirmed booking into a recurring series. It is optional and never blocks reading the confirmation; the client's ongoing interaction with their booking otherwise happens through the confirmation message's manage link, outside this screen.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described, full width.
- **Medium size class and above:** Content remains single-column, capped at a comfortable reading width and horizontally centered; no structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Success heading, booking details, studio address | -- | Display-only, non-interactive | None | -- |
| "Make this a standing appointment" | Tap | Navigate to FEAT-21.SPEC-001 (Set Up Recurring Series) carrying the just-confirmed Booking's reference, service, and start time | Screen closes | Standard forward transition |

### Accessibility Notes

- **Focus order:** Success heading -> service and time -> deposit and balance -> studio address -> confirmation-sent notice -> "Make this a standing appointment" action.
- **Announcements:** The success heading and the fact that a confirmation was sent are announced to assistive technology as soon as the screen renders.
- **Keyboard alternatives:** The "Make this a standing appointment" action is reachable and activatable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Empty | N/A -- this screen only ever renders a single completed Booking, never a collection that can be empty | N/A | N/A |
| Confirmed | Full booking details rendered as described in Layout and Content | Deposit payment succeeded and the Booking is Confirmed | Client leaves the page (terminal state within this flow) |
| Loading | A neutral loading placeholder while the completed Booking is read | Screen is reached via a fresh load after payment success (navigation retry case) | Booking data loads successfully |
| Error | A plain message: "We're confirming your booking -- check your text or email for confirmation, or contact the pro directly if you don't see it shortly." | The completed Booking cannot be read on a fresh-load attempt, even though payment succeeded | Client leaves the page; the confirmation message (FEAT-08) still arrives independently of this screen's own load outcome |
| Offline/Degraded | A plain banner: "Check your connection to see your booking details." in place of the booking details; the success heading remains visible if already rendered | Connectivity is lost after payment succeeded but before this screen's data finishes loading | Connectivity is restored and the booking details load |

## Validation Rules

Not applicable -- this screen has no user input.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|-------------------------------------|
| "Make this a standing appointment" tap | FEAT-21.SPEC-001 (Set Up Recurring Series) | FEAT-21 (Recurring/Standing Appointments) |

Managing the booking afterward happens outside this flow, through the confirmation message's manage link (FEAT-06, FEAT-08).

## Data Model

**Creates:** None.
**Reads:** Booking -- service, start_time, duration, price_agreed, deposit_amount, balance_due, state (Confirmed). Pro Account -- studio_address (shown here for the first time, per its confirmation-only disclosure rule). Client -- the chosen consent channel (text or email), to state which channel the confirmation was sent to.
**Updates:** None.
**Deletes:** None.

## Business Rules

- The recurring offer is optional and appears only for a real Confirmed Booking; declining it (simply leaving the screen) changes nothing about the booking, and the series set-up and its rules belong entirely to FEAT-21.SPEC-001.
- This screen renders only once the Booking has actually transitioned to Confirmed by FEAT-07's successful deposit capture; it never shows a confirmed state on an unconfirmed booking.
- The immediate booking confirmation message is triggered by the deposit payment succeeding, not by this screen rendering -- the message (owned entirely by FEAT-08) and this on-screen display are two independent, parallel effects of the same payment-success event, so a failure of one never blocks or delays the other.
- The studio address is shown here for the first time in the client's flow, consistent with its confirmation-only disclosure rule (it is never shown earlier, since it may be a home address).
- The full flow from FEAT-05.SPEC-001 through this screen completes in under one minute for the under-one-minute benchmark (Non-Functional Notes; success-metrics.md: "Booking Completion Speed").
- In preview mode, this screen shows a simulated confirmed booking; no real Booking, Deposit Transaction, or confirmation message is created.

## Edge Cases

- **Payment succeeded but the direct navigation to this screen failed** -- The client reaches this screen via a fresh load of the flow, which reads the already-Confirmed Booking directly rather than relying on carried navigation state, so the client sees their confirmation correctly rather than being asked to pay again.
- **The completed Booking cannot be read even on a fresh load (a rare data-access failure)** -- The Error state renders, reassuring the client that their confirmation message (already triggered independently by the successful payment) will still arrive.
- **Client loses connectivity immediately after payment succeeds, before this screen finishes loading** -- The Offline/Degraded state renders; the booking is still correctly Confirmed on the server regardless of this screen's own load outcome, and the confirmation message still arrives.
- **Client refreshes or reopens this screen later** -- The screen re-reads the Booking and renders the same confirmed details again; there is no time limit on viewing this screen directly (though it is not the ongoing way to check a booking -- that is the manage link).

## Connected Specs

| Connected Spec | Connection Type | Description |
|-----------------|-------------------|--------------|
| FEAT-07.SPEC-001 (Deposit Payment) | Navigation (inbound) | Client arrives here automatically after successful deposit payment |
| FEAT-05.SPEC-004 (Policy Acknowledgment & Deposit Checkout) | Navigation (inbound) | A fresh load of the flow after a successful payment resolves to this screen |
| FEAT-21.SPEC-001 (Set Up Recurring Series) -- within FEAT-21 (Recurring/Standing Appointments) | Navigation (outbound) | The "Make this a standing appointment" action leads here |
| FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) | Triggers (outbound) | The newly Confirmed Booking this screen displays triggers activity event recording, in parallel with this display |
| FEAT-08 (Automated Booking Messaging) | References (outbound) | Triggers the immediate booking confirmation message, in parallel with this screen's own display |
| FEAT-12 (Pro Daily Schedule Dashboard) | References (outbound) | The confirmed booking becomes visible on the Pro's schedule as a parallel effect of the same payment success |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-------------------|
| booking_confirmation_viewed | total elapsed time from FEAT-05.SPEC-001, confirmation channel (text/email) | This screen renders in the Confirmed state | supports success-metrics.md: "Booking Completion Speed" |
| booking_confirmation_load_failed | reason (error / offline) | The Error or Offline/Degraded state renders | supports success-metrics.md: "Deposit Capture Rate" (a lost confirmation display, even after a successful charge, is a trust risk the metric's clean-outcome bar is meant to catch) |

## Acceptance Criteria

**FEAT-05.SPEC-005-AC-01:** Given Riley's deposit payment succeeds on FEAT-07.SPEC-001, when she advances to this screen, then she sees "You're booked!" along with the service, date and time, deposit paid, balance due, and the studio address.

**FEAT-05.SPEC-005-AC-02:** Given Riley reaches this screen, when it renders, then Riley sees a line confirming that a confirmation has been sent to the channel she chose (text or email).

**FEAT-05.SPEC-005-AC-03:** Given Riley's payment succeeded but the direct navigation to this screen failed, when Riley reopens the flow, then she sees the same confirmed booking details, correctly reflecting her Confirmed booking rather than being asked to pay again.

**FEAT-05.SPEC-005-AC-04:** Given the completed Booking cannot be read on a fresh-load attempt, when this screen is reached that way, then Riley sees a reassuring message that her confirmation is on its way by text or email.

**FEAT-05.SPEC-005-AC-05:** Given Riley loses connectivity immediately after payment succeeds, when this screen attempts to load, then she sees "Check your connection to see your booking details." while her booking remains correctly Confirmed on the server.

**FEAT-05.SPEC-005-AC-06:** Given Talia previews her own booking page through to this screen, when the simulated payment completes, then she sees the identical confirmation screen with no real Booking or confirmation message created.

**FEAT-05.SPEC-005-AC-07:** Given Riley reaches this screen, when the studio address is displayed, then it is the first point in the flow where that address has been shown to her.

**FEAT-05.SPEC-005-AC-08:** Given Riley's booking is confirmed, when the confirmation message and this screen's own display both fire from the same payment-success event, then a delay or failure of the confirmation message never blocks this screen from rendering, and vice versa.

**FEAT-05.SPEC-005-AC-09:** Given Riley completes her booking end to end from FEAT-05.SPEC-001 to this screen, when she reaches this screen, then the total elapsed time is recorded to validate the under-one-minute benchmark.

**FEAT-05.SPEC-005-AC-10:** Given Riley refreshes this screen later the same day, when it reloads, then it re-reads and re-renders the same confirmed booking details.

**FEAT-05.SPEC-005-AC-11:** Given Riley is viewing her confirmed booking, when she taps "Make this a standing appointment", then she navigates to FEAT-21.SPEC-001 (Set Up Recurring Series) with the confirmed Booking's reference, service, and start time carried.

**FEAT-05.SPEC-005-AC-12:** Given Riley's booking is confirmed, when the Booking reaches Confirmed, then FEAT-16.SPEC-002 is triggered to record the activity event in parallel with this screen's display, and a delay or failure of that recording never blocks this screen.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 2 (display-only content, recurring offer) | 2 |
| States | 5 (empty, confirmed, loading, error, offline) | 5 |
| Business Rules | 6 | 6 |
| Edge Cases | 4 | 4 |



# Automation Spec: Slot Hold & Re-Validation at Checkout

## Overview

**Name:** Slot Hold & Re-Validation at Checkout
**ID:** FEAT-05.SPEC-006
**Type:** Automation
**Purpose:** Creates the client's Booking in Pending Payment state and places a checkout hold on the chosen slot when the client advances into the deposit payment step (FEAT-05.SPEC-004 -> FEAT-07.SPEC-001), aligned to FEAT-03.SPEC-002; re-validates the hold immediately before charging, and resolves expiry or contention outcomes by returning the client to a fresh slot list.
**Parent Feature:** FEAT-05 -- Public Booking Page & Booking Flow

## Scope and Non-Goals

**In Scope:**
- Creating the Booking record in Pending Payment state the instant the client advances into the deposit payment step, in step with placing a checkout hold on the chosen slot (via FEAT-03.SPEC-002) -- not at slot pick
- Re-validating the held slot immediately before the deposit charge is attempted
- Transitioning the Booking to Expired (unpaid) when the underlying hold expires unpaid
- Resolving a contested slot in favor of the first client to complete payment, per XBR-01

**Non-Goals:**
- Computing slot availability or owning the Slot Hold entity itself, its timeout, or its contention resolution mechanics -- entirely owned by FEAT-03 (Real-Time Slot Availability Engine, specifically FEAT-03.SPEC-002, FEAT-03.SPEC-003, and FEAT-03.SPEC-005); this spec triggers and reacts to those mechanics but does not re-implement them
- Processing the deposit payment itself -- owned by FEAT-07 (Deposit Payment at Booking); this spec only ensures the slot is still valid immediately before that hand-off
- Placing or expiring the Pro-created deposit-request hold, or holds created through a client reschedule (FEAT-10) or a Pro-side booking (FEAT-30) -- this spec covers only the client-initiated new-booking path through FEAT-05 (Entity-Lifecycle Coverage Matrix's note that Booking is also created by other features via other paths)
- Transitioning the Booking from Pending Payment to Confirmed -- owned by FEAT-07 once the deposit succeeds

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Client advances into the deposit payment step | FEAT-05.SPEC-004 (Policy Acknowledgment & Deposit Checkout) | Fires when the client taps Acknowledge & continue, after policy acknowledgment passes its own integrity check (FEAT-05.SPEC-009) and the availability gate (FEAT-05.SPEC-008) re-check; timing per FEAT-03.SPEC-002's trigger ("advances into the deposit payment step") | Service ID, chosen start time and duration, the client's entered details and Client reference, the acknowledged policy version |
| Client submits payment | FEAT-07.SPEC-001 (Deposit Payment) | Fires when the client taps Pay on the payment screen, before the charge is attempted (the pre-charge hold-still-active check) | The Booking's held slot reference |
| Checkout hold expires unpaid | FEAT-03.SPEC-003 (Slot Hold Expiration) | Fires when the checkout Slot Hold owning this Booking reaches its timeout without a completed payment | The expired hold's owning Booking reference |

## Processing Logic

1. On an advance-into-payment trigger from FEAT-05.SPEC-004: if this client's in-progress checkout already has an Active hold and Pending Payment Booking for the same slot (the client came back from the payment screen), reuse them and skip to step 4; otherwise request a checkout Slot Hold from FEAT-03.SPEC-002 for the chosen service, start time, and duration.
2. If the hold is created successfully, create the Booking record in Pending Payment state, fixing service, start_time, duration, client reference, price_agreed and deposit_amount (computed from the Service's current rule), the acknowledged policy version, wording and timestamp (from FEAT-05.SPEC-009), and source: "client link".
3. If the hold creation is refused (the slot no longer validates, or it is contested and lost per FEAT-03.SPEC-005), take no Booking-creation action and signal the triggering screen (FEAT-05.SPEC-004) with the specific reason (no-longer-valid or contested); that screen returns the client to FEAT-05.SPEC-002.
4. Signal FEAT-05.SPEC-004 that the hold is placed so the client is navigated to FEAT-07.SPEC-001 (Deposit Payment).
5. On a payment-submission trigger from FEAT-07.SPEC-001: re-request validation of the Booking's held slot from FEAT-03 (confirming the hold is still Active and has not expired or been lost).
6. If the hold is still valid, allow FEAT-07 to collect and capture the deposit against this Booking; if it is no longer valid (expired or lost in the moment between reaching the payment screen and tapping Pay), take no payment hand-off and signal FEAT-07.SPEC-001 with the specific reason (its Hold Expired state).
7. On a hold-expiration trigger from FEAT-03.SPEC-003: confirm the owning Booking is still Pending Payment (not already Confirmed by a payment that completed in the same instant); if still Pending Payment, transition the Booking to Expired (unpaid).
8. Signal the client, if still on a screen within the flow (the payment screen, FEAT-07.SPEC-001, or FEAT-05.SPEC-004 after a back navigation), that their held time expired, and return them to a refreshed slot list (FEAT-05.SPEC-002).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Hold placed, Booking created | Slot passes re-validation and hold creation succeeds when the client advances into the payment step | New Booking created (Pending Payment) | Client advances to FEAT-07.SPEC-001 with the slot reserved | FEAT-05.SPEC-004, FEAT-07.SPEC-001 |
| Existing hold reused | The client returns to FEAT-05.SPEC-004 from the payment screen and continues again while the hold is still Active | None -- no second hold or Booking | Client advances to FEAT-07.SPEC-001 again | FEAT-05.SPEC-004, FEAT-07.SPEC-001 |
| Slot no longer valid at continue time | FEAT-03.SPEC-002 rejects the candidate slot as no longer meeting timing rules | No Booking created | Client sees "That time is no longer available." and a refreshed slot list | FEAT-05.SPEC-004, FEAT-05.SPEC-002 |
| Slot contested at continue time | Another client's hold-creation attempt for the same slot commits first | No Booking created for the losing client | Client sees "That time was just taken." and a refreshed slot list | FEAT-05.SPEC-004, FEAT-05.SPEC-002, FEAT-03.SPEC-005 |
| Slot re-validated, payment proceeds | Hold is still Active at payment-submission time | None yet -- hand-off to FEAT-07 begins | Client sees the payment step proceed normally | FEAT-07.SPEC-001, FEAT-07 |
| Slot lost before payment | Hold has expired or was lost to contention by the time payment is submitted | Booking already transitioned to Expired (unpaid) by the hold-expiration path, or no Booking to charge | Client sees "Your held time expired. Pick a new time to continue." on the payment screen and returns to FEAT-05.SPEC-002 | FEAT-07.SPEC-001, FEAT-05.SPEC-002 |
| Booking expired unpaid | The underlying checkout hold expires with payment never completed | Booking transitioned from Pending Payment to Expired (unpaid); the slot is released | Client, if still present, sees the expiry message and a refreshed list; if already left, nothing further happens | FEAT-05.SPEC-002, FEAT-03.SPEC-001 |
| Race won by payment | Payment completes at effectively the same moment the hold's timeout is reached | Booking transitions to Confirmed via FEAT-07 instead of Expired (unpaid) | Client sees their booking confirmed, not an expiry message | FEAT-07, FEAT-05.SPEC-005 |

## Data Model

**Reads:** Service -- price, deposit_rule, duration, buffer_override (to compute price_agreed and deposit_amount). Slot Hold (via FEAT-03) -- state, expiry timestamp. The in-progress checkout details (Client reference, acknowledged policy version, wording, timestamp from FEAT-05.SPEC-009).
**Creates:** Booking -- service, start_time, duration, client reference, price_agreed, deposit_amount, policy_version and acknowledgment fields (as captured by FEAT-05.SPEC-009), state: Pending Payment, source: "client link".
**Updates:** Booking -- state transitioned to Expired (unpaid) on unpaid hold expiry.
**Deletes:** None -- an Expired (unpaid) Booking is retained as history, per SC-22; only the underlying Slot Hold (owned by FEAT-03) is deleted on expiry.

## Business Rules

- A time is offered, held, or booked only if it passes the live slot check owned by FEAT-03; this automation never assumes a slot computed a moment earlier is still valid without re-validating it (XBR-01).
- The hold and the Pending Payment Booking begin only when the client advances into the deposit payment step, exactly as FEAT-03.SPEC-002 specifies (XBR-02 authority); picking a slot on FEAT-05.SPEC-002 places no hold and creates no Booking.
- The checkout hold's timeout is fixed and short: platform parameter: `checkout-hold-timeout-minutes` (same marker as defined in FEAT-03.SPEC-002 -- reused verbatim).
- The first client to complete payment wins a contested slot; the other client sees a plain "just taken" message, never a payment error (XBR-01).
- The deposit amount fixed on Booking creation (price_agreed, deposit_amount) is computed once from the Service's rule at that moment and never recalculated afterward, even if the Pro edits the service before payment completes (XBR-04, XBR-05).
- An Expired (unpaid) Booking is never charged and is retained as history rather than deleted, consistent with the product-wide no-deletion policy for bookings (SC-22).

## Edge Cases

- **Client abandons the flow before advancing into the payment step** -- No hold or Booking exists, so nothing needs to expire or be cleaned up.
- **Client abandons the flow after a hold and Booking are created but never reaches payment** -- The Booking remains Pending Payment until the hold's timeout elapses, then both the hold (via FEAT-03.SPEC-003) and this spec's own expiration path resolve to Expired (unpaid); no separate abandonment signal is needed.
- **Payment completes in the same instant the hold's timeout elapses** -- The completed-payment transition to Confirmed (FEAT-07) takes precedence; Step 7 detects the Booking is no longer Pending Payment and takes no expiration action.
- **Client's device is offline when their hold expires** -- The Booking still expires server-side on schedule; the client sees the expiry message on their next successful interaction, never a silently-accepted late payment against an expired hold.
- **The chosen service is archived between hold creation and payment submission** -- Step 5's re-validation surfaces this as a slot-no-longer-valid condition; the client is refused with a refresh back to the service list (FEAT-05.SPEC-004's Business Rules), not charged; before the hold is placed, the same archived service is caught on Acknowledge & continue and nothing is held.
- **Concurrent trigger firing (two clients advance into payment for the same slot at effectively the same time)** -- Exactly one hold-creation-and-Booking-creation attempt succeeds per FEAT-03.SPEC-005's first-committed-wins rule; the other client's attempt produces no Booking at all, so no orphaned Pending Payment record is ever created for the losing attempt.
- **Trigger fires while a previous run is in flight for the same client** -- The triggering screen (FEAT-05.SPEC-004 or FEAT-07.SPEC-001) debounces its own submit control while an attempt is in progress, so a second concurrent run for the same client's same Booking cannot start; a second run for a different Booking proceeds independently.

## Connected Specs

| Connected Spec | Connection Type | Description |
|-----------------|-------------------|--------------|
| FEAT-05.SPEC-004 (Policy Acknowledgment & Deposit Checkout) | Triggered by (inbound) / Affects (outbound) | Acknowledge & continue triggers hold-and-Booking creation; the outcome (hold placed, or slot lost) is reported back to this screen |
| FEAT-05.SPEC-002 (Slot Selection) | Affects (outbound) | An expired hold or a lost slot returns the client here with a refreshed list and message; this screen no longer triggers anything on slot pick |
| FEAT-07.SPEC-001 (Deposit Payment) | Triggered by (inbound) / Affects (outbound) | The Pay tap triggers pre-payment re-validation; a lost or expired hold is reported back to the payment screen |
| FEAT-03.SPEC-002 (Slot Hold Creation & Checkout Reservation) | Triggers (outbound) | This spec requests hold creation from FEAT-03's own automation |
| FEAT-03.SPEC-003 (Slot Hold Expiration) | Triggered by (inbound) | This spec's Booking-expiration step fires in response to FEAT-03's own hold-expiration event |
| FEAT-03.SPEC-005 (Slot Contention Resolution Rules) | References (outbound) | Governs the outcome when a candidate slot is contested |
| FEAT-07 (Deposit Payment at Booking) | Triggers (outbound) | Hands off to deposit collection once the slot is re-validated at payment time |

## Analytics and Success Signals

- **checkout_booking_created** (service ID, source: client link) -- supports success-metrics.md: "Booking Completion Speed"; the same event, tallied by its source property against a Pro's total bookings over time, is the basis for supports success-metrics.md: "DM-to-Link Migration"
- **checkout_slot_rejected** (reason: no_longer_valid / contested) -- supports success-metrics.md: "Zero Double-Booking Confidence"
- **checkout_slot_revalidated_at_payment** (result: valid / expired / contested) -- supports success-metrics.md: "Zero Double-Booking Confidence"
- **checkout_booking_expired_unpaid** (service ID) -- supports success-metrics.md: "Deposit Capture Rate" (an expired hold is a clean, non-ambiguous non-capture outcome the metric expects, not a lost or stuck state)

## Acceptance Criteria

**FEAT-05.SPEC-006-AC-01:** Given Riley has acknowledged the policy on FEAT-05.SPEC-004, when she taps Acknowledge & continue, then a checkout Slot Hold is placed and a Booking is created in Pending Payment state, and Riley advances to FEAT-07.SPEC-001; picking the slot earlier on FEAT-05.SPEC-002 created neither.

**FEAT-05.SPEC-006-AC-02:** Given Riley's chosen slot no longer passes the live validation rules at the instant she taps Acknowledge & continue, when the hold-creation request runs, then no Booking is created and Riley sees "That time is no longer available." with a refreshed slot list.

**FEAT-05.SPEC-006-AC-03:** Given Riley and another client advance into payment for the same slot at effectively the same time, when hold creation resolves, then exactly one of them ends up with a Pending Payment Booking, and the other sees "That time was just taken."

**FEAT-05.SPEC-006-AC-04:** Given Riley taps Pay on FEAT-07.SPEC-001 with her hold still Active, when the pre-payment re-validation runs, then the slot is confirmed still valid and the hand-off to FEAT-07 proceeds.

**FEAT-05.SPEC-006-AC-05:** Given Riley's checkout hold has expired by the time she taps Pay, when the pre-payment re-validation runs, then no payment hand-off occurs and Riley sees the payment screen's hold-expired message and is returned to FEAT-05.SPEC-002 with a refreshed list.

**FEAT-05.SPEC-006-AC-06:** Given Riley's checkout hold reaches its fixed timeout while she is still mid-flow with no payment completed, when the expiration event fires, then her Booking transitions from Pending Payment to Expired (unpaid) and the slot is released.

**FEAT-05.SPEC-006-AC-07:** Given Riley's payment completes at effectively the same moment her hold's timeout is reached, when both processes evaluate, then her Booking transitions to Confirmed and the expiration path takes no action against it.

**FEAT-05.SPEC-006-AC-08:** Given the chosen service is archived by the Pro between Riley advancing into payment and her payment submission, when the pre-payment re-validation runs, then Riley is refused with a refresh back to the service list, never charged.

**FEAT-05.SPEC-006-AC-09:** Given Riley abandons the flow after a Booking is created but never pays, when the hold's fixed timeout elapses, then her Booking is expired and no charge is ever attempted against it.

**FEAT-05.SPEC-006-AC-10:** Given two clients' checkout attempts for different slots resolve at the same moment, when both are processed, then each is handled independently with no interference between them.

**FEAT-05.SPEC-006-AC-11:** Given Riley's Booking is created in Pending Payment state, when the price_agreed and deposit_amount are set, then they are computed once from the Service's rule at that moment and are never recalculated even if the Pro edits the service before payment completes.

**FEAT-05.SPEC-006-AC-12:** Given Riley returns to FEAT-05.SPEC-004 from FEAT-07.SPEC-001 while her hold is still Active and continues again, when the trigger fires, then the existing hold and Pending Payment Booking are reused and no second hold is placed.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 | 3 |
| Outcome Paths | 8 | 8 |
| Business Rules | 6 | 6 |
| Edge Cases | 7 | 7 |



# Logic/Rule Spec: Booking Details Field Validation

## Overview

**Name:** Booking Details Field Validation
**ID:** FEAT-05.SPEC-007
**Type:** Logic/Rule
**Purpose:** Defines all validation rules, conditional requirements, and authorization rules for the name, phone, texting opt-in, email, and note fields captured on the Client Details & Consent screen.
**Parent Feature:** FEAT-05 -- Public Booking Page & Booking Flow
**Governed Entity:** Client record (booking-time fields: name, phone, email, booking_notes) and the texting opt-in captured alongside it, which governs whether a Messaging Consent record is created

## Scope and Non-Goals

**In Scope:**
- Per-field validation rules for name, phone, email, and booking_notes as captured during the booking flow
- The conditional-required rule linking the texting opt-in to the email field
- The rule that the texting opt-in must be an explicit, never-pre-checked action
- Authorization rules for who may submit and read these fields during the flow
- Default values for these fields at booking time

**Non-Goals:**
- Validation of a client's email or consent when updated on a later visit -- owned by FEAT-06 (Client Booking Identity), which governs the client's own self-service updates, not this booking-time capture
- Validation of the Pro's private_note field on the Client entity -- that field is Pro-only and never captured or shown in this flow; it belongs to FEAT-13 (Client Record Management)
- The phone-based identity lookup and returning-client matching logic itself -- owned by FEAT-06; this spec only validates the phone field's format, not what happens once a match is found
- Deriving or capturing the policy acknowledgment -- owned by FEAT-05.SPEC-009, a distinct governed concern from the contact and consent fields this spec covers

## Governed Entity

**Entity:** Client (booking-time fields) and the texting opt-in
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| name | text | Client's name, as entered on this booking |
| phone | text | Client's phone number -- the identity key for this Client within this Pro |
| email | text | Client's email address, required only when texting is declined |
| booking_notes | text | The client's optional note to the Pro for this booking |
| texting_opt_in | boolean (ephemeral -- drives Messaging Consent creation, not a Client field itself) | Whether the client actively consents to receive texts from this Pro |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|---------------------|
| FEAT-05.SPEC-003 | Client Details & Consent | On field blur (name, phone, email, booking_notes) and on Continue submit (all fields, including the cross-field email-requirement rule); authorization checked on screen entry and on submit |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| name | Required, non-empty, 1-100 characters | Always | On blur | "Please enter your name" / "Name must be 100 characters or fewer" | Yes |
| phone | Required, non-empty, valid reachable phone format | Always | On blur | "Please enter your phone number" / "Please enter a valid phone number" | Yes |
| texting_opt_in | No format validation -- a boolean toggle; must never be pre-checked on initial render (see Business Rules) | Always | On render (initial state check, not a validation error) | Not applicable -- this is a UI-state rule with no error condition | No |
| email | Required, valid email format | Only when texting_opt_in is false (texts declined) | On blur (format) and on submit (conditional requirement) | "Please enter your email so we can send confirmations and reminders" (when required and empty) / "Please enter a valid email address" (when provided but malformed) | Yes |
| email | Valid email format when provided, even if not required | texting_opt_in is true (texts accepted) but client still enters an email | On blur | "Please enter a valid email address" | Yes |
| booking_notes | Optional; maximum 300 characters | Always | On blur (length) | "Your note must be 300 characters or fewer" | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|-----------------|
| Conditional email requirement | texting_opt_in, email | Email is required only when texting_opt_in is false; toggling texting_opt_in to true clears the required indicator on email without clearing any value already entered | "Please enter your email so we can send confirmations and reminders" (shown only when texting_opt_in is false and email is empty at submit) |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Submit booking details (name, phone, opt-in, email, note) | The Client (Riley) | Always, for their own in-progress booking | -- |
| Read own entered (in-progress, unsubmitted) details | The Client (Riley) | Always -- limited to the client's own device session for the booking they are actively entering | -- |
| Access another client's entered (in-progress, unsubmitted) details from a different device or session | The Client (Riley) | Never | Not applicable -- no mechanism exists to view another client's unsubmitted entry from a different session; the question does not arise |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| name | Pre-filled from a matched existing Client record when the entered phone number matches one for this Pro (via FEAT-06's lookup); otherwise empty | On phone field blur, if a match is found | Yes -- the client may edit the pre-filled name before continuing |
| texting_opt_in | Unchecked (false) | On screen first load, always | Yes -- the client may check it |
| email | Empty | On screen first load | Yes |
| booking_notes | Empty | On screen first load | Yes |

## Business Rules

- The texting opt-in must be an explicit, affirmative action -- it is never pre-checked, and the exact wording shown beside it at the moment of checking is retained as compliance evidence (US SMS-consent rules, ASMP-24; XBR-15).
- If texting_opt_in is false at submit time, no Messaging Consent record is created for the text channel; the client must instead have a valid email on file so confirmations and reminders can still arrive by the email fallback (XBR-15).
- A phone number matching an existing Client record within this Pro is treated as the same client (dependency map's Client contention note) -- the phone field's validation is format-only; identity resolution itself is FEAT-06's responsibility, referenced here, not duplicated.
- The booking_notes field's 300-character limit and no-medical-information hint are enforced identically regardless of which service or Pro is being booked -- these are product-wide field rules, not per-Pro configuration (scope-boundaries.md SC-08).
- All rules in this spec apply identically whether the client is new or returning -- the product definition establishes no returning-client exemption from field validation.

## Edge Cases

- **Phone entered with international formatting (e.g., +1-555-123-4567)** -- Passes validation; the accepted format allows digits, spaces, dashes, parentheses, and a leading plus.
- **Name at exactly 100 characters** -- Passes validation; 101 characters shows the length error.
- **Booking_notes at exactly 300 characters** -- Passes validation; 301 characters shows the length error.
- **Email left as whitespace only** -- Treated as empty for the purposes of the conditional-required rule; if texting_opt_in is false, the required-field error is shown, not a format error.
- **Client checks texting_opt_in, enters an email anyway, then unchecks it before submitting** -- The email requirement is re-evaluated at submit time based on the final state of texting_opt_in; the already-entered email satisfies the requirement without needing to be re-typed.
- **Client enters a valid email while texting_opt_in remains true** -- No error; an optional, valid email is accepted even though it is not required, and is stored for the client's record.
- **Phone number changed after a name was pre-filled from a match** -- The pre-filled name is not automatically cleared by this spec's rules; FEAT-05.SPEC-003 re-runs the lookup against the new number and updates the pre-fill if a different match (or no match) results.
- **Platform Operator (Support) views this form outside a live client session** -- No submission is possible; the "never" authorization row applies, and no error state is reached because the action is not offered at all.

## Acceptance Criteria

**FEAT-05.SPEC-007-AC-01:** Given Riley leaves the name field empty and moves to the next field, then the name field shows "Please enter your name."

**FEAT-05.SPEC-007-AC-02:** Given Riley enters a name of exactly 100 characters, then no error is shown; given she enters 101 characters, then "Name must be 100 characters or fewer" is shown.

**FEAT-05.SPEC-007-AC-03:** Given Riley leaves the phone field empty and moves to the next field, then the phone field shows "Please enter your phone number."

**FEAT-05.SPEC-007-AC-04:** Given Riley enters a phone number in an invalid format, then the phone field shows "Please enter a valid phone number"; given she enters "+1-555-123-4567", then no error is shown.

**FEAT-05.SPEC-007-AC-05:** Given Riley is on the Client Details & Consent screen, when it first renders, then the texting opt-in checkbox is unchecked.

**FEAT-05.SPEC-007-AC-06:** Given Riley leaves the texting opt-in unchecked and the email field empty, when she taps Continue, then the email field shows "Please enter your email so we can send confirmations and reminders" and submission is blocked.

**FEAT-05.SPEC-007-AC-07:** Given Riley leaves the texting opt-in unchecked and enters a validly formatted email, when she taps Continue, then no email error is shown and submission proceeds.

**FEAT-05.SPEC-007-AC-08:** Given Riley checks the texting opt-in, when she taps Continue with the email field empty, then no email error is shown, since email is not required in this branch.

**FEAT-05.SPEC-007-AC-09:** Given Riley checks the texting opt-in but still enters an email in an invalid format, when the field loses focus, then "Please enter a valid email address" is shown, since format validation applies whenever a value is provided.

**FEAT-05.SPEC-007-AC-10:** Given Riley enters a booking note of exactly 300 characters, then no error is shown; given she enters 301 characters, then "Your note must be 300 characters or fewer" is shown.

**FEAT-05.SPEC-007-AC-11:** Given Riley enters a phone number matching an existing Client record with this Pro, when the phone field loses focus, then the name field is pre-filled with the matched client's name, which Riley may still edit.

**FEAT-05.SPEC-007-AC-12:** Given Riley (the Client) submits her own booking details, when submission runs, then it is allowed without restriction.

**FEAT-05.SPEC-007-AC-13:** Given Riley is filling in her booking details, when she reviews the form before submitting, then she can see and edit every value she has entered in her own session.

**FEAT-05.SPEC-007-AC-14:** Given Riley's entered, unsubmitted details exist only in her own device session, when the question of another client accessing them from a different session arises, then no mechanism exists for it -- the scenario does not occur.

**FEAT-05.SPEC-007-AC-15:** Given Riley checks the opt-in, enters an email, then unchecks the opt-in before tapping Continue, when submission runs, then the previously entered email is used to satisfy the now-active email requirement without Riley re-typing it.

**FEAT-05.SPEC-007-AC-16:** Given Riley's entered phone number matches an existing client but she changes it before continuing, when the phone field loses focus again, then the lookup re-runs against the new number and the name pre-fill updates accordingly.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 6 | 6 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 3 | 3 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 8 | 8 |



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



# Logic/Rule Spec: Policy Acknowledgment Capture & Integrity Check

## Overview

**Name:** Policy Acknowledgment Capture & Integrity Check
**ID:** FEAT-05.SPEC-009
**Type:** Logic/Rule
**Purpose:** Computes this booking's exact deposit amount and cancellation cut-off, captures the acknowledged cancellation policy version and wording into the in-progress checkout (carried onto the Booking when FEAT-05.SPEC-006 creates it), and re-validates that version still matches at payment time.
**Parent Feature:** FEAT-05 -- Public Booking Page & Booking Flow
**Governed Entity:** Booking (policy acknowledgment fields: price_agreed, deposit_amount, policy_version, acknowledgment timestamp, acknowledged wording)

## Scope and Non-Goals

**In Scope:**
- Computing the exact deposit amount for this booking from the Service's deposit_rule and the Pro's account currency
- Computing the exact cancellation cut-off time for this booking from the current Cancellation Policy's window_hours and the booking's start_time
- Recording the acknowledged Cancellation Policy version, its exact plain-language wording, and the acknowledgment timestamp into the in-progress checkout the moment the client checks the acknowledgment box; FEAT-05.SPEC-006 carries them onto the Booking when it creates it
- Re-validating, when the client advances into the payment step (Acknowledge & continue on FEAT-05.SPEC-004, before the checkout hold and hand-off to FEAT-07.SPEC-001), that the acknowledged version still matches the Pro's current Cancellation Policy version

**Non-Goals:**
- Defining or versioning the Cancellation Policy itself -- owned by FEAT-09 (Cancellation & No-Show Policy Engine); this spec only reads the current version and records which one was shown
- Displaying the policy block or the acknowledgment checkbox -- owned by FEAT-05.SPEC-004, which calls this spec's computation and capture logic but owns the screen presentation
- Defining or editing the Service's deposit_rule -- owned by FEAT-01 (Service & Pricing Management); this spec only reads it to compute the exact amount
- Determining the deposit's eventual outcome (refunded, kept, forfeited) once the appointment's cancellation window opens or closes -- owned by FEAT-09 and FEAT-11; this spec only captures what was agreed at booking time, not what happens afterward

## Governed Entity

**Entity:** Booking (policy acknowledgment fields)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| price_agreed | number | The service price fixed at booking time, in the Pro's account currency |
| deposit_amount | number | The exact deposit computed for this booking, fixed at booking time |
| policy_version | reference | The specific Cancellation Policy version shown and acknowledged for this booking |
| acknowledgment_timestamp (part of policy_version field per the dependency map: "with acknowledgment time") | date/time | The exact moment the client checked the acknowledgment box |
| acknowledged_wording (recorded alongside policy_version for dispute evidence, per Non-Functional Notes) | text | The exact plain-language wording shown at the moment of acknowledgment |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|---------------------|
| FEAT-05.SPEC-004 | Policy Acknowledgment & Deposit Checkout | Computation runs on screen load (to display the amount and cut-off); capture runs the instant the acknowledgment checkbox is checked; the integrity re-check runs when the client taps Acknowledge & continue, before the checkout hold is requested and the client is navigated to FEAT-07.SPEC-001 |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| deposit_amount | Must be computed, not entered -- derived exactly once from the Service's current deposit_rule and price at the moment the client reaches this screen; never editable by the client | Always | On screen load (computation), re-verified not to have changed at payment time as part of the Service's own contention rule (owned by FEAT-01, referenced here) | Not applicable -- this is a system-derived value with no client-facing error state | Yes (computation must complete before Pay is offered) |
| policy_version | Must reference an existing Cancellation Policy version at the moment of acknowledgment | Always | When the acknowledgment checkbox is checked | Not applicable -- if no Cancellation Policy exists at all, the booking page itself would not be live (XBR-26 requires the policy to be set before go-live) | Yes |
| acknowledgment_timestamp | No validation beyond data type -- always system-derived, never user-entered | Always | -- | -- | -- |
| acknowledged_wording | No validation beyond data type -- captured verbatim from the current wording at the moment of acknowledgment, never user-entered | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|-----------------|
| Version integrity at continue | policy_version (as recorded at acknowledgment), the Pro's current Cancellation Policy version | The recorded policy_version must still equal the Pro's current Cancellation Policy version at the instant the client taps Acknowledge & continue into the payment step; if the Pro's current version has advanced since acknowledgment, the check fails | "The cancellation policy has changed since you agreed to it. Please review the updated terms." |
| Deposit-price consistency | deposit_amount, price_agreed, Service.deposit_rule | deposit_amount must equal the value produced by applying the Service's current deposit_rule to price_agreed at the moment of computation (fixed form: the flat amount; percentage form: price_agreed x percentage / 100); the two are never computed independently or allowed to diverge | Not applicable -- this is an internal computation consistency rule with no client-facing error path of its own |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Trigger the deposit/cut-off computation and acknowledgment capture | The Client (Riley) | Only for their own in-progress Booking | -- |
| Trigger the computation and capture in preview mode | The Pro (Talia) | Always, but the result is never persisted to a real Booking | -- |
| Read the recorded acknowledgment (version, wording, timestamp) on a Booking | The Pro (Talia) | Always, for her own bookings (dispute evidence) | -- |
| Edit or delete a recorded acknowledgment once captured | The Client (Riley), The Pro (Talia) | Never -- the recorded acknowledgment is immutable once written, forming dispute evidence (Non-Functional Notes) | Not applicable -- no edit or delete control exists for this data anywhere in the product |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| price_agreed | Copied from the Service's current price at the moment the client reaches FEAT-05.SPEC-004 | On computation (screen load) | No |
| deposit_amount | Fixed form: the Service's deposit_rule flat amount. Percentage form: price_agreed x (deposit_rule percentage / 100), in the Pro's account currency | On computation (screen load) | No |
| Cancellation cut-off (displayed, not itself a stored Booking field -- derived for display from policy_version and the Booking's start_time) | Booking.start_time minus the acknowledged Cancellation Policy's window_hours | On computation (screen load) and whenever the display refreshes after a version change | No |
| policy_version | The Pro's current Cancellation Policy version at the moment the client checks the acknowledgment checkbox | On acknowledgment (checkbox checked) | No |
| acknowledged_wording | The Cancellation Policy's plain_language_wording at that same moment, captured verbatim | On acknowledgment (checkbox checked) | No |
| acknowledgment_timestamp | The current date and time | On acknowledgment (checkbox checked) | No |

## Business Rules

- The deposit is computed once, exactly, from the service's rule in the Pro's account currency, cannot be altered by the client, and is charged once per booking (XBR-05).
- Every booking is governed by the cancellation policy version shown and acknowledged at booking; policy edits never change existing bookings once the acknowledgment is captured (XBR-08).
- If the Pro's current Cancellation Policy version changes between the client's acknowledgment and the moment the client taps Acknowledge & continue, the client is refused with refresh and asked to acknowledge the current wording -- existing bookings keep their own already-recorded version regardless (dependency map's Cancellation Policy contention note).
- The recorded acknowledgment (version, wording, timestamp) forms dispute evidence and must be retained with the Booking for as long as the Booking exists (Non-Functional Notes; SC-22).
- This spec's derivations run identically in preview mode, so the Pro reviews the exact computation her own settings would produce -- but the result is discarded rather than persisted to a real Booking.

## Edge Cases

- **The Pro edits the Cancellation Policy's wording without changing the substantive window_hours, creating a new version anyway (per FEAT-09's every-edit-creates-a-new-version rule)** -- The integrity check still fails against the acknowledged version, since versions are compared by identity, not by substantive difference; the client must re-acknowledge even a wording-only change.
- **The client acknowledges, the Pro edits the policy, and the client re-acknowledges the new version, all before payment** -- The second acknowledgment overwrites the first with the new version, wording, and timestamp; only the most recent acknowledgment is carried onto the Booking when FEAT-05.SPEC-006 creates it.
- **The client continues at the exact same version that was acknowledged, with no changes in between** -- The integrity check passes and the client advances to the payment step without any re-acknowledgment prompt.
- **The Service's deposit_rule is edited by the Pro after the client reaches FEAT-05.SPEC-004 but before payment** -- Per the Service entity's contention rule (dependency map), the client pays the amount shown when they acknowledged the policy; a service archived (not merely edited) before payment instead refuses the client with a refresh, per FEAT-05.SPEC-004's Business Rules.
- **The computed cancellation cut-off falls at a boundary moment (e.g., exactly at the window_hours mark)** -- The cut-off is computed as an exact timestamp (start_time minus window_hours to the minute); a cancellation exactly at that timestamp is evaluated by FEAT-09's own boundary rule, not re-derived here -- this spec only displays and records the computed value.
- **Two different clients acknowledge the same current policy version for two different bookings at the same time** -- Each Booking records its own independent copy of the version, wording, and timestamp; there is no shared or contended record between them.

## Acceptance Criteria

**FEAT-05.SPEC-009-AC-01:** Given Riley reaches FEAT-05.SPEC-004 for a fixed-amount-deposit service, when the screen loads, then the exact deposit amount shown equals the Service's fixed deposit_rule amount.

**FEAT-05.SPEC-009-AC-02:** Given Riley reaches FEAT-05.SPEC-004 for a percentage-deposit service, when the screen loads, then the exact deposit amount shown equals price_agreed multiplied by the deposit percentage, in the Pro's account currency.

**FEAT-05.SPEC-009-AC-03:** Given Riley reaches FEAT-05.SPEC-004, when the cancellation cut-off is displayed, then it equals the booking's start time minus the current Cancellation Policy's window_hours, shown as an exact date and time.

**FEAT-05.SPEC-009-AC-04:** Given Riley checks the acknowledgment checkbox, when the check registers, then the in-progress checkout's policy_version, acknowledged_wording, and acknowledgment_timestamp (carried onto the Booking when FEAT-05.SPEC-006 creates it) are set to the current Cancellation Policy version, its exact wording, and the current moment.

**FEAT-05.SPEC-009-AC-05:** Given Riley has acknowledged one policy version, when the Pro edits the Cancellation Policy before Riley continues, then Riley's Acknowledge & continue attempt fails the integrity check and she sees "The cancellation policy has changed since you agreed to it. Please review the updated terms."

**FEAT-05.SPEC-009-AC-06:** Given Riley's Acknowledge & continue attempt fails the integrity check, when she reviews and re-checks the acknowledgment box against the refreshed wording, then the in-progress checkout's policy_version, acknowledged_wording, and acknowledgment_timestamp are overwritten with the new version's values.

**FEAT-05.SPEC-009-AC-07:** Given Riley acknowledges the current policy version and no change occurs before she continues, when she taps Acknowledge & continue, then the integrity check passes and she advances to the payment step without a re-acknowledgment prompt.

**FEAT-05.SPEC-009-AC-08:** Given a completed Booking's policy_version, acknowledged_wording, and acknowledgment_timestamp, when Talia (the Pro) views the Booking's record, then all three values are visible exactly as recorded.

**FEAT-05.SPEC-009-AC-09:** Given a completed Booking's recorded acknowledgment, when any role attempts to edit or delete it, then no such control exists anywhere in the product.

**FEAT-05.SPEC-009-AC-10:** Given Talia previews her own booking page and reaches FEAT-05.SPEC-004, when she checks the acknowledgment checkbox, then the identical computation and capture logic runs, but no real Booking record is persisted.

**FEAT-05.SPEC-009-AC-11:** Given the Pro edits only the Cancellation Policy's wording (not the window_hours) after Riley acknowledges, when Riley taps Acknowledge & continue, then the integrity check still fails, since versions are compared by identity, not by substantive difference.

**FEAT-05.SPEC-009-AC-12:** Given the Service's deposit_rule is edited by the Pro after Riley reaches FEAT-05.SPEC-004 but the service is not archived, when Riley continues to payment, then she is charged the amount shown when she acknowledged the policy, not the newly edited amount.

**FEAT-05.SPEC-009-AC-13:** Given two different clients acknowledge the same current Cancellation Policy version at the same time for two different bookings, when both acknowledgments are recorded, then each Booking holds its own independent copy of the version, wording, and timestamp.

**FEAT-05.SPEC-009-AC-14:** Given a Booking has an acknowledgment recorded, when the Booking is later referenced for a deposit dispute (FEAT-16), then the recorded version, wording, and timestamp remain available and unchanged from the moment of acknowledgment.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 4 | 4 |
| Defaults/Derivations | 6 | 6 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |


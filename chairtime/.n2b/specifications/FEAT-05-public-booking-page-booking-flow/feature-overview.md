---
document_type: feature-overview
feature_number: FEAT-05
feature_name: Public Booking Page & Booking Flow
feature_slug: public-booking-page-booking-flow
priority_tier: Core
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 9
screen_count: 5
automation_count: 1
logic_rule_count: 3
integration_count: 0
notification_count: 0
---

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

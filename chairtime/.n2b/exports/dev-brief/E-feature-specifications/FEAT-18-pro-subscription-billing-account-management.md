# FEAT-18 — Pro Subscription Billing & Account Management

This chapter covers Pro Subscription Billing & Account Management (FEAT-18), a Important-tier feature. It carries 7 specifications carrying 90 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-18.SPEC-001 | Subscribe Screen | screen | 11 |
| FEAT-18.SPEC-002 | Billing & Subscription Management Screen | screen | 13 |
| FEAT-18.SPEC-003 | Subscription Renewal & Payment-Failure Processing | automation | 11 |
| FEAT-18.SPEC-004 | Subscription-Lapse Account Pause Trigger | automation | 9 |
| FEAT-18.SPEC-005 | Subscription Billing Rules | logic-rule | 19 |
| FEAT-18.SPEC-006 | Subscription Billing Integration | integration | 15 |
| FEAT-18.SPEC-007 | Subscription Billing Notifications | notification | 12 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Pro Subscription Billing & Account Management

## Summary

**Feature:** Pro Subscription Billing & Account Management
**ID:** FEAT-18
**Description:** The Pro's own flat monthly subscription to Chairtime -- one price tier, card-based, cancel anytime -- including seeing their current plan status and updating their payment method.
**Priority:** Important
**Phase:** MVP
**Type:** Lifecycle
**Rationale:** BRIEF.md's Business Context is explicit: "revenue comes from a flat monthly subscription paid by each pro... cancel anytime, with one price tier in v1... no per-booking cut." Ranked Important rather than Core because it is the business's monetization mechanism rather than part of the client-facing booking loop the founder's headline promise describes; it must still ship at MVP because the product has no revenue model without it. [RESEARCH-INFORMED: added the market's dominant trust complaint -- unpredictable, layered fees on top of the advertised subscription are reported across three competitors, and no profiled competitor offers a flat, all-inclusive price with no per-booking or per-new-client charge, so the one price is shown up front with every feature included and no add-ons, from BBB, Capterra and Reddit-derived sources (3 products, HIGH confidence)]

**Key Capabilities:**
- Subscribe during onboarding with a card
- View current plan status and next billing date
- Update the payment method on file
- Cancel the subscription at any time, effective at the end of the current billing period
- See the single all-inclusive price and a plain statement that Chairtime takes nothing from deposits, balances, or tips

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-18.SPEC-001 | Subscribe Screen | Screen | The Pro | Pro enters payment details during onboarding, sees the single all-inclusive price and zero-fee statement, and starts the subscription |
| FEAT-18.SPEC-002 | Billing & Subscription Management Screen | Screen | The Pro, Platform Operator (Support) | Pro's ongoing view of plan status, next billing date, and the zero-fee statement, with actions to update the payment method or cancel; Support's view-only entry point for billing status |
| FEAT-18.SPEC-003 | Subscription Renewal & Payment-Failure Processing | Automation | The Pro | Runs the monthly renewal against the payment-processing capability, branches on success/failure, and starts the 7-day grace-period clock on failure |
| FEAT-18.SPEC-004 | Subscription-Lapse Account Pause Trigger | Automation | The Pro | Triggers the Pro Account's system-imposed pause when the grace period expires unresolved, and lifts it the moment billing is restored |
| FEAT-18.SPEC-005 | Subscription Billing Rules | Logic/Rule | The Pro, Platform Operator (Support) | Governs the single price tier, the 7-day grace threshold, cancellation-at-period-end, the 30-day price-change notice rule, and contention/authority resolution between the Pro and the processor |
| FEAT-18.SPEC-006 | Subscription Billing Integration | Integration | The Pro | Outbound subscribe/update-payment-method/cancel calls to, and inbound renewal-outcome and dispute events from, the payment-processing capability |
| FEAT-18.SPEC-007 | Subscription Billing Notifications | Notification | The Pro | Sends the payment-failure grace notice, renewal receipt, cancellation confirmation, and price-change notice |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Subscribe during onboarding with a card | FEAT-18.SPEC-001, FEAT-18.SPEC-006 | The Screen collects the card entry moment; the Integration spec carries it to the payment-processing capability and creates the Subscription record on success | Phase 2 (Explicit) |
| View current plan status and next billing date | FEAT-18.SPEC-002 | The Billing screen's primary display, kept current by SPEC-003's renewal outcomes | Phase 2 (Explicit) |
| Update the payment method on file | FEAT-18.SPEC-002, FEAT-18.SPEC-006 | The Billing screen's update action hands the new card to the Integration spec, which confirms the change with the processor | Phase 2 (Explicit) |
| Cancel the subscription at any time, effective at the end of the current billing period | FEAT-18.SPEC-002, FEAT-18.SPEC-005, FEAT-18.SPEC-006 | The Billing screen's cancel action is governed by the period-end rule (SPEC-005) and executed through the Integration spec | Phase 2 (Explicit) |
| See the single all-inclusive price and a plain statement that Chairtime takes nothing from deposits, balances, or tips | FEAT-18.SPEC-001, FEAT-18.SPEC-002 | Both screens display the fixed price and reference the standing zero-fee rule owned by FEAT-28.SPEC-004 | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-18.SPEC-003 | Subscription Renewal & Payment-Failure Processing | Phase 4 (Trigger-Response, time-based triggers) | The Primary Flows & Alternates field's renewal-failure alternate is a recurring, time-based side-effect with branching outcomes (success vs. grace period) -- too consequential to leave inline in a screen |
| FEAT-18.SPEC-004 | Subscription-Lapse Account Pause Trigger | Phase 4 (Trigger-Response, cross-feature effect) | The Interactions field and XBR-14 require a lapse to pause the Pro Account for new bookings while leaving existing bookings untouched; this cross-entity, cross-feature effect (Coordination Note 2) is a standalone automation, not a bare side-effect of SPEC-003 |
| FEAT-18.SPEC-005 | Subscription Billing Rules | Phase 5 (Rule Discovery) | The Validation & Limits field names four interacting, non-trivial rules (single tier, fixed 7-day grace, period-end cancellation, 30-day price-change notice) plus the dependency map's Contention resolution -- past the inline-validation threshold and shared across both screens |
| FEAT-18.SPEC-006 | Subscription Billing Integration | Phase 4 (External Dependencies lens) | ASMP-31 names billing the Pro's own monthly subscription as a payment-processing capability dependency this feature must specify |
| FEAT-18.SPEC-007 | Subscription Billing Notifications | Phase 4 (Notification surfacing) | The Communications field names four messages with real delivery rules (channel, audience, a grace deadline, or a 30-day lead time), which the Phase 4 disposition rule requires as standalone Notification specs rather than inline confirmations |

## Entity-Lifecycle Coverage Matrix

**Entity: Subscription**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-18.SPEC-001, FEAT-18.SPEC-006 | Created the moment the processor confirms the Pro's first successful charge during onboarding | Exactly one Subscription per Pro Account |
| Read (single) | FEAT-18.SPEC-002 | The Billing screen displays the Pro's own status and next billing date; Platform Operator (Support) reads the same record view-only | Also read by FEAT-19 (view-only) per the dependency map |
| Read (list) | N/A | Exactly one Subscription per Pro Account, so no list view is meaningful | -- |
| Update | FEAT-18.SPEC-003, FEAT-18.SPEC-006 | Renewal outcomes and the price field are written by FEAT-18.SPEC-003 (price is written by SPEC-003 alone, system-only, on an announced price change's effective date, per FEAT-18.SPEC-005); payment-method changes and cancellation are written by FEAT-18.SPEC-006 from processor-confirmed events. FEAT-18.SPEC-006 is not a Subscription.price writer after creation | Contention: the processor's recorded outcome is authoritative; a stale Pro screen is refused with refresh (dependency map, Contention) |
| Delete/Archive | N/A -- explicit non-goal | The Subscription record is never independently deleted by this feature: cancellation is a state transition (Cancelled, active to period end), not a deletion; the record is later retained under FEAT-29's account-closure retention rather than removed here | See Non-Goals |
| State Transition | FEAT-18.SPEC-003, FEAT-18.SPEC-004, FEAT-18.SPEC-005 | Active -> Payment Failed (7-day grace) -> Active (recovered) or Cancelled (active to period end) -> lapsed/paused; cancellation is last-write-wins against reactivation before period end (dependency map, Contention) | -- |

**Referenced Entities (read-only, with one exception noted):**

| Entity | Read By | Context |
|--------|---------|---------|
| Pro Account | FEAT-18.SPEC-001, FEAT-18.SPEC-002 | Onboarding and billing screens read the Pro's identity/account context; this feature also *writes* one field it does not own -- FEAT-18.SPEC-004 sets/clears the Pro Account's system-imposed Paused state on subscription lapse/recovery, per the dependency map's Pro Account Contention line ("FEAT-18 can set a Paused state automatically on a failed renewal"). Full Pro Account CRUD (profile, other pause reasons, closure) is owned by FEAT-27/FEAT-29, not this feature |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Pro submits card details on the Subscribe screen | Charge the first period, create the Subscription record on success | Standalone Integration, then inline success handling | FEAT-18.SPEC-006, inline in FEAT-18.SPEC-001 |
| Pro submits card details successfully | Show success confirmation, report an active subscription to FEAT-15's go-live check (XBR-26) | Inline in triggering screen | FEAT-18.SPEC-001 |
| Monthly billing cycle reaches its renewal date | Charge the Pro's payment method on file | Standalone Automation | FEAT-18.SPEC-003 |
| Renewal charge succeeds | Update next billing date; send renewal receipt | Standalone Automation, then Standalone Notification | FEAT-18.SPEC-003, FEAT-18.SPEC-007 |
| Renewal charge fails | Set status to Payment Failed, start the 7-day grace clock; send payment-failure notice with the grace deadline | Standalone Automation, then Standalone Notification | FEAT-18.SPEC-003, FEAT-18.SPEC-007 |
| Grace period expires with payment still unresolved | Trigger the Pro Account's system-imposed pause (new bookings stop; existing bookings, reminders, refunds and client self-service unchanged per XBR-11/XBR-14) | Standalone Automation, cross-feature (Pro Account pause state owned by FEAT-27) | FEAT-18.SPEC-004 |
| Pro updates the payment method while paused (or while a renewal retry is in progress) | Confirm the new card with the processor; on success, clear the grace/pause state and resume normal billing | Standalone Automation, then Standalone Integration | FEAT-18.SPEC-004, FEAT-18.SPEC-006 |
| Pro updates the payment method | Confirm the new reference with the processor | Standalone Integration | FEAT-18.SPEC-006 |
| Payment-method update fails at the processor | Show a clear failure reason and a retry action | Inline in triggering screen | FEAT-18.SPEC-002 |
| Pro cancels the subscription | Mark Cancelled (active through the already-paid period); send cancellation confirmation; at period end, pause the account (not delete) | Standalone Logic/Rule (timing), Standalone Notification, Standalone Automation (pause) | FEAT-18.SPEC-005, FEAT-18.SPEC-007, FEAT-18.SPEC-004 |
| Cancellation is invoked by FEAT-29 as part of account closure | Cancel the subscription the same way, usable by that path | Cross-feature (FEAT-29 owns account closure, XBR-20) | FEAT-18.SPEC-006 |
| A future price change is decided (product-level) | Notify the Pro at least 30 days before it applies | Standalone Notification, timing governed by Logic/Rule | FEAT-18.SPEC-007, FEAT-18.SPEC-005 |
| Pro opens the Billing screen with no connectivity | Show the last known plan status, read-only | Inline in triggering screen | FEAT-18.SPEC-002 |
| Pro attempts to pay or update the card with no connectivity | Block the action and say plainly that a live connection is required | Inline in triggering screen | FEAT-18.SPEC-002 |
| Payment-failure pause surfaces to the Pro outside this feature's own screens | Add an entry to the Pro's dashboard attention list | Cross-feature (owned by FEAT-12.SPEC-005 Attention Flag Aggregation) | FEAT-12 responsibility |

## Shared Context

**Shared Entities:**
- Subscription -- created by SPEC-001/SPEC-006 and updated by SPEC-003 (renewal outcomes and price, the latter on an announced price change's effective date) and SPEC-006 (payment method, cancellation), its state governed by SPEC-004 and SPEC-005, displayed by SPEC-002, read view-only by FEAT-19. Fields: status (Active | Payment Failed (7-day grace) | Cancelled (active to period end)), billing_cycle / next_billing_date, payment_method_reference, price.
- Pro Account (partial write) -- its Paused status is set/cleared exclusively by SPEC-004 on subscription lapse/recovery; all other Pro Account fields are owned by FEAT-27/FEAT-29.

**Shared UI Patterns:**
- Price & zero-fee statement block -- the same fixed price display and "Chairtime takes nothing from deposits, balances, or tips" statement (referencing FEAT-28.SPEC-004) appears on both SPEC-001 and SPEC-002; Spec Writers for both screens should describe it identically rather than as two components.
- Status banner -- SPEC-002 shows one consistent status treatment (Active / Payment Failed with grace deadline / Cancelled through period end) that both SPEC-003 and SPEC-004 keep current.

**Shared Validation:**
- SPEC-005 defines the single-tier price, the 7-day grace threshold, period-end cancellation timing, the 30-day price-change lead time, and the processor-authoritative contention rule once. SPEC-001, SPEC-002, SPEC-003, SPEC-004, and SPEC-006 all reference it rather than duplicating any of these rules.

## Internal Dependency Map

```
SPEC-001 (Subscribe Screen) -> [Pro submits card] -> SPEC-006 (Subscription Billing Integration) -> [processor confirms] -> SPEC-001 (success, reports go-live signal)
SPEC-001 (Subscribe Screen) -> [displays price/statement per] -> SPEC-005 (Subscription Billing Rules)
SPEC-003 (Subscription Renewal & Payment-Failure Processing) -> [monthly cycle] -> SPEC-006 (Subscription Billing Integration) -> [outcome] -> SPEC-003
SPEC-003 (Subscription Renewal & Payment-Failure Processing) -> [renewal succeeds] -> SPEC-007 (Subscription Billing Notifications)
SPEC-003 (Subscription Renewal & Payment-Failure Processing) -> [renewal fails] -> SPEC-007 (Subscription Billing Notifications)
SPEC-003 (Subscription Renewal & Payment-Failure Processing) -> [grace expires unresolved] -> SPEC-004 (Subscription-Lapse Account Pause Trigger)
SPEC-004 (Subscription-Lapse Account Pause Trigger) -> [billing restored] -> SPEC-004 (lifts pause)
SPEC-002 (Billing & Subscription Management Screen) -> [Pro updates payment method] -> SPEC-006 (Subscription Billing Integration) -> [confirmed] -> SPEC-004 (clears grace/pause if applicable) -> SPEC-002
SPEC-002 (Billing & Subscription Management Screen) -> [Pro cancels] -> SPEC-005 (Subscription Billing Rules, timing) -> SPEC-007 (Subscription Billing Notifications)
SPEC-002 (Billing & Subscription Management Screen) -> [validates actions against] -> SPEC-005 (Subscription Billing Rules)
```

**Default Entry:** SPEC-002 (Billing & Subscription Management Screen) for ongoing use, reached from Pro Sign-In & Account Lifecycle's account settings; SPEC-001 (Subscribe Screen) only on first use, reached exclusively from the onboarding wizard's subscription step.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-18.SPEC-001 | Inbound | FEAT-15 (Pro Onboarding & Setup Wizard) | Setup wizard's subscription step hands the Pro into the subscribe screen | Pro continues setup |
| FEAT-18.SPEC-001 | Outbound | FEAT-15 (Pro Onboarding & Setup Wizard) | An active subscription is reported back to the go-live gate (XBR-26) | Pro completes the subscribe step |
| FEAT-18.SPEC-002 | Inbound | FEAT-29 (Pro Sign-In & Account Lifecycle) | Pro opens billing from account settings | Pro opens billing |
| FEAT-18.SPEC-006 | Outbound | FEAT-29 (Pro Sign-In & Account Lifecycle) | Cancel capability is used by account closure (XBR-20) | Pro closes their account |
| FEAT-18.SPEC-004 | Outbound | FEAT-27 (Pro Profile & Booking Page Settings) | Triggers/lifts the Pro Account's system-imposed pause; FEAT-27 owns the pause state itself and its Pro-facing "resume bookings" toggle cannot clear a system-imposed pause | Grace period expires unresolved, or billing is restored |
| FEAT-18.SPEC-004 | Outbound | FEAT-05 (Public Booking Page & Booking Flow) | A subscription lapse (via the Pro Account pause) stops new bookings while existing bookings, reminders, refunds and client self-service continue unchanged (XBR-14, XBR-11) | Grace period expires unresolved |
| FEAT-18.SPEC-002 | Outbound | FEAT-19 (Platform Support Read-Only Access) | Support's view-only read of plan status for billing support questions | Support opens a Pro account during a help request |
| FEAT-18.SPEC-001 / SPEC-002 | Inbound | FEAT-28 (Payout Account Connection & Payout Visibility) | Price and zero-fee statement reference the standing zero-Chairtime-fee rule (FEAT-28.SPEC-004, XBR-07) rather than redefining it | Screen renders the price/statement block |
| FEAT-18.SPEC-007 | Outbound | FEAT-08 (Automated Booking & Messaging) | All four notifications are sent through the text/email delivery capability (FEAT-08.SPEC-012 text, FEAT-08.SPEC-013 email fallback) | Each notification trigger fires |
| FEAT-18.SPEC-004 | Outbound | FEAT-12 (Pro Daily Schedule Dashboard) | A payment-failure pause surfaces as a dashboard attention item (FEAT-12.SPEC-005) | Grace period starts or expires |

## Non-Functional Notes

**Data volumes / growth:** Exactly one Subscription record per Pro Account, so volume simply tracks the number of pros (a few hundred in year one per assumptions-constraints.md's product-wide scale); no growth concern specific to this feature (Data Notes field names only status, billing cycle, and a payment-method reference as captured fields).

**Responsiveness:** Plan status is a small, instant read with no meaningful loading delay (States field: "Loading: N/A -- plan status is a small, instant read"); per ASMP-27, paying or updating the payment method needs a live connection and says so plainly when missing, while the last known plan status stays viewable read-only offline.

**Data sensitivity / privacy:** Financial data classification -- this feature stores only billing status, billing cycle, and a payment-method reference; the payment-processing capability holds the actual card data and never this product's own code (Data Notes field; SC-11). Support sees plan status only, never payment details (Access field).

**Compliance flags:** N/A -- no compliance regime is named for this feature specifically; the payment-processing capability (ASMP-31) itself owns whatever financial-services and card-data obligations attach to billing the subscription, keeping that scope out of this product's own code per the hard boundary in SC-11.

## Non-Goals

- **Handling or storing card data within the product itself** -- Excluded per SC-11: card data is never stored or handled by this product's own code; the payment-processing capability owns it, and this feature holds only a payment-method reference.
- **Multiple price tiers or plan selection** -- Excluded per the Validation & Limits field: one price tier only in v1, with no plan-selection screen or logic offered to the Pro.
- **Support acting on a Pro's subscription** -- Excluded per SC-05: Platform Operator (Support) has view-only access to plan status and can never edit, cancel, or update payment details on the Pro's behalf.
- **Per-seat or multi-staff billing** -- Excluded per SC-01: the product is strictly single-operator, so there is no concept of additional users or per-additional-user pricing to bill for.
- **Closing the account and deleting its data** -- Excluded per the Validation & Limits field: account closure and data deletion are owned by Pro Sign-In & Account Lifecycle (FEAT-29); this feature only makes its cancel capability usable by that flow.
- **Independent deletion of the Subscription record** -- Intentional lifecycle decision surfaced by the CRUD matrix: cancellation is a state transition, never a record deletion; the record is retained under FEAT-29's account-closure retention posture (SC-22) rather than removed by this feature.



# Screen Spec: Subscribe Screen

## Overview

**Name:** Subscribe Screen
**ID:** FEAT-18.SPEC-001
**Type:** Screen
**Purpose:** Talia enters her card details during onboarding, sees the single all-inclusive price and the zero-fee statement, and starts her Chairtime subscription.
**Parent Feature:** FEAT-18 -- Pro Subscription Billing & Account Management

## Scope and Non-Goals

**In Scope:**
- Displaying the single all-inclusive subscription price and the plain zero-fee statement
- Collecting card details and starting the first subscription charge through the payment-processing capability
- Showing the immediate outcome (success or decline) and, on success, reporting the go-live signal to onboarding

**Non-Goals:**
- Choosing among multiple price tiers or plans -- excluded per product-features.md FEAT-18 (Validation & Limits): "One price tier only in v1 -- no plan selection is offered."
- Ongoing plan management (viewing status, changing payment method, cancelling) -- owned by FEAT-18.SPEC-002 (Billing & Subscription Management Screen); this screen exists only for the first-time subscribe moment during onboarding
- Handling or storing the card data itself -- excluded per scope-boundaries.md SC-11: card data is never stored or handled by the product's own code; the payment-processing capability owns it entirely
- Deciding the onboarding step sequence around this screen -- owned by FEAT-15 (Pro Onboarding & Setup Wizard); this spec begins where FEAT-15 hands the Pro into the subscription step

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-15 (Pro Onboarding & Setup Wizard, subscription step) | Pro completes the prior onboarding step and continues | The Pro's account reference; no pre-filled payment data |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Enter card details and submit the subscription charge | -- |
| The Client (Riley) | No | No | Clients never reach a Pro onboarding screen; there is no client-facing entry point into it |
| Platform Operator (Support) | No | No | Support's view-only access to billing status (per the Access Matrix, Subscription & Billing = View) is scoped to the ongoing Billing & Subscription Management Screen (FEAT-18.SPEC-002) once a subscription exists; there is nothing to view before the first subscription is created, so this onboarding screen is not part of Support's view |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29); onboarding cannot resume mid-flow without a signed-in Pro |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- entered card details are never preserved (card data is never held by the product, per SC-11); the Pro re-enters payment details after signing back in |

## Layout and Content

**Header:** Screen title "Subscribe to Chairtime" with the onboarding step progress indicator (consistent with the rest of FEAT-15's setup steps) above it.

**Body, top section -- Price and zero-fee statement block:** A single price card showing the subscription's monthly price (platform parameter: `subscription-price`) and the line "Billed monthly. Cancel anytime." Directly below it, in plain language: "Chairtime takes nothing from your deposits, balances, or tips -- 100% goes to you," referencing the standing zero-fee rule (FEAT-28.SPEC-004, XBR-07). This block is described identically to its sibling on FEAT-18.SPEC-002 per the Feature Breakdown Brief's Shared Context. **Display-only:** the entire block (price card and zero-fee statement) is static text with no tap targets, inputs, or state changes of its own; it carries no entry in the Interactions table below.

**Body, middle section -- Payment form:** A single-column form with:
- Card details entry (card number, expiry, security code, postal code) -- handled entirely by the payment-processing capability's own input surface (FEAT-18.SPEC-006); the product never stores or reads the raw values
- Name on card (text input, required)

**Footer:** A single "Start subscription" action button, full width, disabled until the card fields are complete.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described, full width; the price/zero-fee block stacks above the form; "Start subscription" remains full width at the bottom.
- **Medium size class and above:** Content remains single-column, capped at a consistent platform-wide form width (the design layer's decision) and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Card details entry | Type | Captures card input via the payment-processing capability's own surface | Field shows entered values (masked per card-field convention) | Standard input focus state |
| Name on card input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Name on card input | Blur (empty) | Triggers field validation via FEAT-18.SPEC-005 | Error state on field | "Name on card is required" below field |
| "Start subscription" button | Tap | 1. Validate name-on-card field. 2. If valid, submit the charge through FEAT-18.SPEC-006 (Subscription Billing Integration). | Button shows a "Processing payment, do not close this page" loading state | Success: confirmation message and automatic continuation into onboarding. Failure: inline decline message with retry. |
| "Start subscription" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Card details entry (as ordered by the payment-processing capability's own surface) -> Name on card -> Start subscription.
- **Validation announcements:** When the name-on-card field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Processing and outcome feedback:** The "Processing payment" state is announced when it begins; the success confirmation or decline message is announced immediately when the outcome arrives.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Filling | Form empty or partially filled, "Start subscription" disabled until required fields are complete | Screen first opens | All required fields complete |
| Ready | "Start subscription" enabled | Card fields and name on card are complete | Pro taps "Start subscription" |
| Processing | Button shows "Processing payment, do not close this page"; form fields disabled | Pro taps "Start subscription" | The payment-processing capability confirms success or reports a decline |
| Success | Confirmation shown ("You're subscribed") and the screen automatically continues into the next onboarding step | The processor confirms the first charge succeeded | Pro is navigated onward by FEAT-15 |
| Declined | Inline decline message from FEAT-18.SPEC-006 with a retry action; form fields (except card details, which must be re-entered) remain as entered | The processor reports a decline | Pro edits card details and retries |
| Offline/Degraded | N/A -- starting a subscription requires a live connection to the payment-processing capability by nature; a connection drop mid-submission is treated as the Declined/failure path (FEAT-18.SPEC-006), never an ambiguous charge | -- | -- |

## Validation Rules

Validation governed by FEAT-18.SPEC-005 (Subscription Billing Rules). See that spec for the single-tier price rule. This screen additionally applies simple inline validation on the name-on-card field on blur and on submit.

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| Name on card | Required, non-empty | On blur, on submit | "Name on card is required" |

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Successful subscription start | Next onboarding step | FEAT-15 (Pro Onboarding & Setup Wizard) |
| Declined payment, Pro abandons retry | Onboarding step remains on this screen | -- |

## Data Model

**Creates:** Subscription record -- created the moment the payment-processing capability confirms the first successful charge (fields: status set to Active, billing_cycle / next_billing_date set to one billing period from today, payment_method_reference set from the processor's confirmation, price set to platform parameter: `subscription-price`). The screen itself never holds card data -- only the outcome and the resulting reference (FEAT-18.SPEC-006).
**Reads:** None -- this is a first-time creation screen; no existing Subscription exists yet.
**Updates:** None.
**Deletes:** None.

## Business Rules

- Single price tier only -- no plan selection is offered on this screen, per FEAT-18.SPEC-005.
- The charge is submitted through FEAT-18.SPEC-006 (Subscription Billing Integration); this screen never handles card data directly (SC-11).
- On success, this screen reports an active subscription to FEAT-15's go-live check (XBR-26) -- the booking link cannot go live without it.
- The price and zero-fee statement shown here must match FEAT-18.SPEC-002's presentation of the same information, per the Feature Breakdown Brief's Shared UI Patterns.

## Edge Cases

- **Pro navigates away mid-entry without submitting** -- Onboarding preserves the Pro's place at this step; no partial card data is retained anywhere (card data is never held by the product), so the Pro re-enters card details on return.
- **Pro taps "Start subscription" twice rapidly** -- Second tap is ignored while the first submission is in progress (button in loading/Processing state).
- **Connection drops mid-submission** -- Treated as a failure by FEAT-18.SPEC-006: the Pro sees the same decline/failure message with retry, and no ambiguous charge state is left ("processing payment, do not close this page" resolves to a definite outcome, never a stuck state).
- **Payment succeeds but the confirmation fails to render** -- The Subscription record is still correctly created by the payment-processing capability's confirmed outcome (FEAT-18.SPEC-006); on next load of this step or the onboarding flow, the Pro sees the success state and onboarding continues -- never double-charged and never left unsure whether the subscription started.
- **No concurrent-edit conflict applies** -- This screen creates the Subscription for the first time; no prior version can exist to conflict with, so the dependency map's Subscription Contention note (processor-authoritative outcome, reject-with-refresh) does not apply until FEAT-18.SPEC-002 exists for this Pro.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-18.SPEC-006 (Subscription Billing Integration) | Triggers (outbound) | "Start subscription" submits the first charge through this integration; its confirmed outcome creates the Subscription record |
| FEAT-18.SPEC-005 (Subscription Billing Rules) | References (inbound) | Single-tier price and validation rules applied on this screen |
| FEAT-18.SPEC-002 (Billing & Subscription Management Screen) | References (inbound) | Shares the identical price/zero-fee statement block per the Brief's Shared UI Patterns |
| FEAT-28.SPEC-004 (standing zero-fee rule) | References (inbound) | The zero-fee statement cites this feature's owning rule rather than redefining it |
| FEAT-15 (Pro Onboarding & Setup Wizard) | Navigation (inbound and outbound) | Entry from the prior onboarding step; success continues to the next step and reports the go-live signal (XBR-26) |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| subscription_started | onboarding entry point | The payment-processing capability confirms the first successful charge | supports success-metrics.md: "Subscription Retention" |
| subscription_start_declined | decline reason category | The payment-processing capability reports a decline | N/A -- no Stage 2 metric measures decline frequency directly; retained so onboarding friction at this step is observable |

## Acceptance Criteria

**FEAT-18.SPEC-001-AC-01:** Given Talia is on the Subscribe Screen during onboarding, when she enters valid card details and her name on card and taps "Start subscription", then the charge is submitted through FEAT-18.SPEC-006, and on success she sees a confirmation and onboarding continues to the next step.

**FEAT-18.SPEC-001-AC-02:** Given Talia is on the Subscribe Screen, when she leaves the name-on-card field empty and taps "Start subscription", then the field shows the error "Name on card is required" and no charge is submitted.

**FEAT-18.SPEC-001-AC-03:** Given Talia submits valid card details, when the payment-processing capability declines the charge, then she sees the decline reason inline from FEAT-18.SPEC-006 and can retry with the same or a different card.

**FEAT-18.SPEC-001-AC-04:** Given Talia's subscription charge succeeds, when the Subscription record is created, then FEAT-18.SPEC-001 reports an active subscription to FEAT-15's go-live check (XBR-26).

**FEAT-18.SPEC-001-AC-05:** Given Talia taps "Start subscription" while a submission is already in progress, when she taps it again, then the second tap is ignored and the button remains in its Processing state.

**FEAT-18.SPEC-001-AC-06:** Given Talia's connection drops while her payment is processing, when the drop is detected, then she sees the same failure/decline message with a retry action, and no ambiguous or double charge occurs.

**FEAT-18.SPEC-001-AC-07:** Given Talia's payment succeeds but the confirmation fails to render on screen, when she reloads or returns to this step, then she sees the success state and onboarding continues, with no duplicate charge.

**FEAT-18.SPEC-001-AC-08:** Given Talia is on the Subscribe Screen, when she views the price block, then she sees the single all-inclusive monthly price and the statement "Chairtime takes nothing from your deposits, balances, or tips -- 100% goes to you."

**FEAT-18.SPEC-001-AC-09:** Given Talia navigates away from this step without submitting, when she returns to it later in onboarding, then no card data was retained and she must re-enter it.

**FEAT-18.SPEC-001-AC-10:** Given a support operator opens a Pro's account before that Pro has a Subscription, when they look for this screen, then it is not part of their view -- Support's billing visibility begins at FEAT-18.SPEC-002 once a subscription exists.

**FEAT-18.SPEC-001-AC-11:** Given Talia's session expires while she is filling in the Subscribe Screen, when she is prompted to sign in again, then no card data is preserved and she must re-enter her payment details from a fresh state.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 5 (filling, ready, processing, success, declined; offline N/A) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Screen Spec: Billing & Subscription Management Screen

## Overview

**Name:** Billing & Subscription Management Screen
**ID:** FEAT-18.SPEC-002
**Type:** Screen
**Purpose:** Talia's ongoing view of her subscription's plan status and next billing date, with actions to update her payment method or cancel; Support's read-only entry point into the same status for billing support questions.
**Parent Feature:** FEAT-18 -- Pro Subscription Billing & Account Management

## Scope and Non-Goals

**In Scope:**
- Displaying current plan status, next billing date, and the price/zero-fee statement block
- Updating the payment method on file
- Cancelling the subscription (effective at the end of the current billing period)
- Support's view-only read of the same status

**Non-Goals:**
- Choosing among multiple price tiers or plans -- excluded per product-features.md FEAT-18 (Validation & Limits): one price tier only in v1
- The first-time subscribe flow -- owned by FEAT-18.SPEC-001 (Subscribe Screen); this screen exists only for a Pro who already has a Subscription record
- Support editing, cancelling, or changing payment details on the Pro's behalf -- excluded per scope-boundaries.md SC-05: Platform Operator (Support) has view-only access and can never act on a Pro's subscription
- Closing the Pro's account or deleting its data -- owned by FEAT-29 (Pro Sign-In & Account Lifecycle); this screen's cancel action only cancels the subscription, per the Feature Breakdown Brief's Side-Effect Inventory

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) (Pro Sign-In & Account Lifecycle, account settings) | Pro opens billing from account settings | The Pro's account reference |
| FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) (Platform Support Read-Only Access) | Support opens a Pro's account during a help request | The Pro account reference being supported; screen renders in Support's view-only mode |
| FEAT-18.SPEC-007 (Subscription Billing Notifications) | Pro taps a billing notification's CTA (payment-failure grace notice, renewal receipt, cancellation confirmation, price-change notice) | The Pro's account reference; screen opens directly to this view |
| FEAT-27.SPEC-004 (Pause Bookings) | Talia taps "Resolve it in Billing" on the Pause Bookings screen while a subscription lapse pause is in force | The Pro's account reference |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Update payment method, cancel subscription | -- |
| The Client (Riley) | No | No | Clients have no visibility into this at all, per the Access Matrix (Subscription & Billing = None); there is no client-facing entry point |
| Platform Operator (Support) | Plan status and next billing date only (price, status, billing cycle) | No actions -- the update-payment-method and cancel controls are not shown | Attempting to reach an action control directly is not possible -- the interface offers no edit controls in Support's view by design, consistent with FEAT-19's read-only mode; Support never sees payment-method reference details beyond that a method is on file |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29) |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- no unsaved billing action was in progress that needs preserving, since actions on this screen (payment update, cancel) complete atomically through the payment-processing capability rather than as a draft |

## Layout and Content

**Header:** Screen title "Billing & Subscription" with a back arrow (returns to account settings, or closes the Support view for FEAT-19).

**Body, top section -- Status banner:** One consistent status treatment, per the Feature Breakdown Brief's Shared Context, showing exactly one of: "Active -- next billing date {date}"; "Payment failed -- update your payment method by {grace deadline} to keep your booking link live"; "Cancelled -- active through {period end date}, then your account pauses." **Display-only:** the banner itself is static text with no tap target of its own -- it reflects state driven by FEAT-18.SPEC-003/FEAT-18.SPEC-004 and carries no entry in the Interactions table below (the "Update payment method" and "Cancel subscription" controls it sits above are separate elements, listed in Interactions).

**Body, middle section -- Price and zero-fee statement block:** Identical to FEAT-18.SPEC-001's price card: the subscription's monthly price (platform parameter: `subscription-price`), "Billed monthly. Cancel anytime," and "Chairtime takes nothing from your deposits, balances, or tips -- 100% goes to you" (referencing FEAT-28.SPEC-004, XBR-07). **Display-only:** static text with no tap targets, inputs, or state changes of its own; no entry in the Interactions table below.

**Body, lower section -- Payment method:** A masked reference to the card on file (e.g., "Card ending in {last 4}," sourced from the payment-processing capability, never stored by the product) with an "Update payment method" action button below it. Not shown in Support's view. **Display-only (the masked reference text itself):** the "Card ending in {last 4}" text is static display, not tappable; only the "Update payment method" button beneath it is interactive and is listed separately in the Interactions table below.

**Footer:** A "Cancel subscription" text-style action, visually de-emphasized relative to "Update payment method." Not shown in Support's view.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described, full width, sections stacked in the order above.
- **Medium size class and above:** Content remains single-column, capped at a consistent platform-wide form width (the design layer's decision) and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to account settings (FEAT-29) or close Support's view (FEAT-19) | Screen closes | Standard transition |
| "Update payment method" button | Tap | Opens the payment-processing capability's own card-entry surface (FEAT-18.SPEC-006) | Button shows loading state while the update is confirmed | Success: "Payment method updated" and the masked reference refreshes. Failure: clear failure reason with a retry action (inline, per Side-Effect Inventory) |
| "Cancel subscription" action | Tap | Opens a confirmation dialog describing the exact effect per FEAT-18.SPEC-005 (active through the paid period, then the account pauses) | Dialog appears | Dialog text states the exact period-end date |
| Cancel confirmation dialog -- "Confirm cancel" | Tap | Submits the cancellation through FEAT-18.SPEC-006 | Status banner updates to "Cancelled -- active through {date}" | Cancellation confirmation notification fires (FEAT-18.SPEC-007) |
| Cancel confirmation dialog -- "Keep subscription" | Tap | Dismisses the dialog, no change | Dialog closes | Status banner unchanged |

### Accessibility Notes

- **Focus order:** Back arrow -> Status banner (announced) -> Price/zero-fee block -> Payment method reference -> Update payment method -> Cancel subscription.
- **Status announcements:** The status banner's content is announced to assistive technology whenever it changes (e.g., after a successful renewal, a payment failure, or a cancellation).
- **Action feedback:** "Payment method updated" and cancellation confirmation are announced on success; failure messages move focus to the relevant control.
- **Keyboard alternatives:** Every action, including the cancel confirmation dialog, is reachable and dismissible by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Active | Status banner shows "Active -- next billing date {date}"; full actions available (Pro view) | Subscription status is Active | Renewal fails (FEAT-18.SPEC-003) or Pro cancels |
| Payment Failed (grace) | Status banner shows the grace deadline; "Update payment method" is visually emphasized | FEAT-18.SPEC-003 sets status to Payment Failed | Payment method is updated and confirmed (returns to Active) or grace period expires (FEAT-18.SPEC-004 pauses the account) |
| Cancelled (active to period end) | Status banner shows "Cancelled -- active through {date}, then your account pauses"; "Update payment method" remains available (a cancelled-but-still-active subscription can still need a valid card through period end); "Cancel subscription" is replaced with no further cancel action (already cancelled) | Pro confirms cancellation | Period ends and the account pauses (FEAT-18.SPEC-004), or -- per the dependency map's Contention note -- a reactivation attempt is refused (see Edge Cases; last-write-wins against reactivation is not offered as a Pro-facing control in v1) |
| Updating payment method | "Update payment method" shows loading state | Pro submits a new card | The processor confirms or rejects the update |
| Payment method update failed | Inline failure message below the payment method section with a retry action | The processor rejects the update | Pro retries or leaves the screen |
| Support view (read-only) | Status banner and price/zero-fee block only; no payment method reference, no action controls | Support opens the screen via FEAT-19 | Support closes the view |
| Offline/Degraded | Banner "You're offline -- showing your last known plan status" above the status banner; the last-loaded status remains viewable read-only; "Update payment method" and "Cancel subscription" are disabled with the message "A live connection is required to change your billing" | Connectivity lost while screen is open, or screen opened while offline | Connectivity restored -- actions re-enable and the screen refreshes to the current status |

## Validation Rules

Validation governed by FEAT-18.SPEC-005 (Subscription Billing Rules). See that spec for the single-tier price, cancellation-timing, and grace-period rules. No additional field-level validation exists on this screen -- payment method entry is handled entirely by the payment-processing capability's own surface (FEAT-18.SPEC-006).

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap (Pro) | Account settings | FEAT-29 (Pro Sign-In & Account Lifecycle) |
| Back arrow tap (Support) | Closes the read-only view | FEAT-19 (Platform Support Read-Only Access) |

## Data Model

**Creates:** None.
**Reads:** Subscription -- status, billing_cycle / next_billing_date, payment_method_reference (masked), price. Displayed identically for the Pro's full view and Support's status-only view (Support never sees payment_method_reference).
**Updates:** Subscription -- payment_method_reference (via FEAT-18.SPEC-006, on successful payment method update); status (set to Cancelled, active to period end, via FEAT-18.SPEC-006 and FEAT-18.SPEC-005's timing rule, on cancel confirmation).
**Deletes:** None -- cancellation is a state transition, never a record deletion, per the Feature Breakdown Brief's Entity-Lifecycle Coverage Matrix.

## Business Rules

- The single price tier and zero-fee statement shown here must match FEAT-18.SPEC-001's presentation, per the Feature Breakdown Brief's Shared UI Patterns.
- Cancellation timing (active through the already-paid period, then pause) is governed by FEAT-18.SPEC-005 -- this screen never states or implies an immediate cutoff.
- Support's access is strictly view-only per SC-05 -- no action control is ever rendered in Support's view, regardless of subscription state.
- XBR-14: when the account is paused (from either subscription lapse or a Pro-chosen pause), this screen's own display and actions are unaffected -- the Pro can still view billing status and update the payment method to resolve a lapse.
- The processor's recorded outcome is authoritative for the Subscription's status, per the dependency map's Contention note -- if this screen's loaded status has gone stale relative to a processor-confirmed change, the next action is refused with refresh (see Edge Cases).

## Edge Cases

- **Payment method update fails at the processor** -- Inline failure message with the exact decline reason from FEAT-18.SPEC-006, and a retry action; the previous payment method reference remains on file and unaffected.
- **Pro opens this screen with no connectivity** -- The last known plan status is shown read-only with the offline banner; nothing is charged or changed while offline.
- **Pro attempts to pay or update the card with no connectivity** -- The action is blocked and the screen states plainly: "A live connection is required to change your billing."
- **Subscription status changed by the processor (e.g., a renewal completed or failed) while this screen is open** -- The screen is not live-updating by default; if the Pro attempts an action (update payment method, cancel) against a status that has since changed, the action is refused with a dialog: "Your billing status changed. Refreshing to show the latest." and the screen reloads the current status before the Pro retries. Resolution: reject-with-refresh, per the dependency map's Contention note for the Subscription entity (the processor's recorded outcome is authoritative).
- **Pro attempts to cancel a subscription that is already Cancelled (active to period end)** -- The cancel action is not offered a second time; the status banner already reflects the cancellation and its period-end date.
- **Pro wants to undo a cancellation before period end** -- Not offered as a control on this screen in v1: per the dependency map's Contention note, "cancellation is last-write-wins against reactivation before period end," meaning this screen does not expose a reactivation path; the Pro would need to subscribe again after the account pauses.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-18.SPEC-006 (Subscription Billing Integration) | Triggers (outbound) | Update-payment-method and cancel actions submit through this integration |
| FEAT-18.SPEC-005 (Subscription Billing Rules) | References (inbound) | Cancellation timing, grace threshold, and single-tier price rules applied on this screen |
| FEAT-18.SPEC-003 (Subscription Renewal & Payment-Failure Processing) | References (inbound) | Renewal outcomes keep the status banner current |
| FEAT-18.SPEC-004 (Subscription-Lapse Account Pause Trigger) | References (inbound) | Grace/pause state shown in the status banner; clears when billing is restored via this screen's payment-method update |
| FEAT-18.SPEC-007 (Subscription Billing Notifications) | Navigation (inbound) | Every billing notification's CTA deep-links here |
| FEAT-29 (Pro Sign-In & Account Lifecycle) | Navigation (inbound and outbound) | Entry from account settings; back arrow returns there |
| FEAT-19 (Platform Support Read-Only Access) | Navigation (inbound) | Support's entry point for billing status during a help request |
| FEAT-28.SPEC-004 (standing zero-fee rule) | References (inbound) | The zero-fee statement cites this feature's owning rule rather than redefining it |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| billing_screen_viewed | viewer role (pro / support) | The screen is opened | supports success-metrics.md: "Subscription Retention" |
| payment_method_updated | outcome (success / failure) | Pro submits a payment method update and the processor responds | supports success-metrics.md: "Subscription Retention" |
| subscription_cancel_confirmed | days remaining in current period at cancellation | Pro confirms cancellation | supports success-metrics.md: "Subscription Retention" |

## Acceptance Criteria

**FEAT-18.SPEC-002-AC-01:** Given Talia's subscription is Active, when she opens the Billing & Subscription Management Screen, then she sees "Active -- next billing date {date}" and the price/zero-fee statement block.

**FEAT-18.SPEC-002-AC-02:** Given Talia's subscription is in Payment Failed (grace), when she opens this screen, then she sees the grace deadline and an emphasized "Update payment method" action.

**FEAT-18.SPEC-002-AC-03:** Given Talia taps "Update payment method" and enters a valid new card, when the processor confirms it, then she sees "Payment method updated" and the masked card reference refreshes.

**FEAT-18.SPEC-002-AC-04:** Given Talia taps "Update payment method" and the processor rejects the new card, when the rejection is returned, then she sees the exact decline reason inline with a retry action and her previous payment method remains on file.

**FEAT-18.SPEC-002-AC-05:** Given Talia taps "Cancel subscription", when the confirmation dialog appears, then it states the exact date her subscription remains active through before the account pauses.

**FEAT-18.SPEC-002-AC-06:** Given Talia confirms cancellation, when the action completes, then the status banner updates to "Cancelled -- active through {date}, then your account pauses" and she receives the cancellation confirmation notification (FEAT-18.SPEC-007).

**FEAT-18.SPEC-002-AC-07:** Given a support operator opens a Pro's account via FEAT-19, when they view this screen, then they see only status, next billing date, and the price/zero-fee block, with no payment method reference and no action controls.

**FEAT-18.SPEC-002-AC-08:** Given Talia opens this screen with no connectivity, when the screen loads, then she sees her last known plan status read-only with the offline banner, and "Update payment method" and "Cancel subscription" are disabled.

**FEAT-18.SPEC-002-AC-09:** Given Talia attempts to update her payment method with no connectivity, when she taps the action, then it is blocked with the message "A live connection is required to change your billing."

**FEAT-18.SPEC-002-AC-10:** Given Talia's subscription status changed at the processor while this screen was open, when she attempts an action against the stale status, then the action is refused with "Your billing status changed. Refreshing to show the latest." and the screen reloads the current status.

**FEAT-18.SPEC-002-AC-11:** Given Talia's subscription is already Cancelled (active to period end), when she views this screen, then no cancel action is offered a second time.

**FEAT-18.SPEC-002-AC-12:** Given Talia's account is paused from a subscription lapse, when she opens this screen, then she can still view billing status and update her payment method to resolve the lapse, per XBR-14.

**FEAT-18.SPEC-002-AC-13:** Given Talia's session expires while viewing this screen, when she is prompted to sign in again, then no partial billing action is left in an ambiguous state -- any action she takes only completes atomically after re-authentication.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 7 (active, payment failed, cancelled, updating, update failed, support view, offline) | 7 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Automation Spec: Subscription Renewal & Payment-Failure Processing

## Overview

**Name:** Subscription Renewal & Payment-Failure Processing
**ID:** FEAT-18.SPEC-003
**Type:** Automation
**Purpose:** Runs the monthly renewal charge against the payment-processing capability, branches on success or failure, updates the billing date on success, and starts the 7-day grace-period clock on failure.
**Parent Feature:** FEAT-18 -- Pro Subscription Billing & Account Management

## Scope and Non-Goals

**In Scope:**
- Firing the monthly renewal charge on the subscription's billing cycle
- Branching on the processor's outcome: success or failure
- Updating the Subscription's next_billing_date on success
- Setting the Subscription's status to Payment Failed and starting the grace-period clock on failure
- Updating the Subscription's price field to the new value of platform parameter: `subscription-price` on an announced price change's effective date

**Non-Goals:**
- Pausing the Pro Account when the grace period expires unresolved -- owned by FEAT-18.SPEC-004 (Subscription-Lapse Account Pause Trigger); this spec only starts the clock
- Sending the renewal receipt or payment-failure notice -- owned by FEAT-18.SPEC-007 (Subscription Billing Notifications); this spec triggers those notifications but does not define their content
- The processor charge mechanics themselves -- owned by FEAT-18.SPEC-006 (Subscription Billing Integration); this spec consumes that integration's inbound renewal-outcome events
- Cancellation processing -- owned by FEAT-18.SPEC-005 (timing) and FEAT-18.SPEC-006 (execution); a subscription already in Cancelled (active to period end) status does not renew, per Validation & Limits in product-features.md FEAT-18

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Monthly billing cycle reaches its renewal date | System (schedule-based, driven by the Subscription's own next_billing_date field) | Fires once per Subscription when its next_billing_date arrives, only if the Subscription's status is Active | Subscription reference, payment_method_reference, price (platform parameter: `subscription-price`) |
| Renewal outcome received | FEAT-18.SPEC-006 (Subscription Billing Integration) | Fires when the payment-processing capability reports the renewal charge's result (an external-event trigger) | Outcome (succeeded / failed), failure reason category (on failure), processor's confirmed charge date |
| Price-change effective date arrives | System (schedule-based, driven by the effective_date carried by FEAT-18.SPEC-007's price-change notice, timing governed by FEAT-18.SPEC-005) | Fires once per Subscription on the effective_date of an announced price change, which always falls at least platform parameter: `subscription-price-change-notice-days` after the notice was sent | Subscription reference, new value of platform parameter: `subscription-price`, effective_date |

## Processing Logic

1. On the Subscription's next_billing_date, if its status is Active, submit the renewal charge for its price (platform parameter: `subscription-price`) against its payment_method_reference through FEAT-18.SPEC-006.
2. Receive the renewal outcome event from FEAT-18.SPEC-006.
3. If the outcome is success: set the Subscription's next_billing_date forward by one billing cycle; leave status as Active; trigger the renewal receipt notification (FEAT-18.SPEC-007).
4. If the outcome is failure: set the Subscription's status to Payment Failed (7-day grace); record the grace-period start as the failure's confirmed date; compute the grace deadline as the start date plus platform parameter: `subscription-payment-failure-grace-period-days`; trigger the payment-failure grace notice (FEAT-18.SPEC-007) carrying that deadline.
5. If a Subscription in Payment Failed status has its payment method updated and confirmed before the grace deadline (via FEAT-18.SPEC-002 and FEAT-18.SPEC-006), retry the renewal charge; on success, follow step 3; on continued failure, the grace deadline is not extended -- the clock started at the original failure date continues.
6. On the effective_date of an announced price change (carried by the price-change notice FEAT-18.SPEC-007 already sent, per FEAT-18.SPEC-005's timing rule), update the Subscription's price field to the new platform parameter: `subscription-price` value. This update runs independently of step 1: if the effective_date coincides with a renewal date, the price is updated first, so that renewal's charge (step 1) is submitted at the new price.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Renewal succeeds | The processor confirms the charge | Subscription.next_billing_date advances one billing cycle; status remains Active | Status banner continues to show "Active -- next billing date {new date}"; renewal receipt notification sent | FEAT-18.SPEC-002, FEAT-18.SPEC-007 |
| Renewal fails, grace starts | The processor reports a decline or failure | Subscription.status set to Payment Failed (7-day grace); grace deadline recorded | Status banner shows the grace deadline with an emphasized "Update payment method" action; payment-failure grace notice sent | FEAT-18.SPEC-002, FEAT-18.SPEC-007 |
| Retry during grace succeeds | The Pro updates the payment method and the retried charge succeeds before the grace deadline | Subscription.status returns to Active; next_billing_date advances one billing cycle; grace state cleared | Status banner returns to "Active -- next billing date {date}"; renewal receipt sent | FEAT-18.SPEC-002, FEAT-18.SPEC-004 (clears any pause that had not yet been triggered), FEAT-18.SPEC-007 |
| Retry during grace fails again | The Pro's updated payment method is also declined, or no update is made | Subscription remains Payment Failed; grace deadline unchanged (not extended) | Status banner continues to show the original grace deadline | FEAT-18.SPEC-002 |
| Grace period expires unresolved | No successful charge occurs by the grace deadline | Subscription remains Payment Failed | Handed off to FEAT-18.SPEC-004, which pauses the Pro Account | FEAT-18.SPEC-004 |
| No renewal needed | Subscription status is not Active on its billing date (e.g., already Cancelled active-to-period-end) | None -- the automation does not fire a charge | No feedback -- the cancellation's own timing (FEAT-18.SPEC-005) governs what happens at period end instead | FEAT-18.SPEC-005 |
| Price-change effective date reached | The effective_date carried by an already-sent price-change notice arrives | Subscription.price is updated to the new platform parameter: `subscription-price` value | No direct feedback on the write itself -- the Pro was already informed by the price-change notice at least platform parameter: `subscription-price-change-notice-days` in advance; the Billing screen (FEAT-18.SPEC-002) and any renewal charge from this date forward reflect the new price | FEAT-18.SPEC-002, FEAT-18.SPEC-005, FEAT-18.SPEC-007 |

## Data Model

**Reads:** Subscription -- status, next_billing_date, payment_method_reference, price.
**Creates:** None.
**Updates:** Subscription -- status (Active <-> Payment Failed), next_billing_date (advanced on success), price (updated to the new value of platform parameter: `subscription-price` on an announced price change's effective date).
**Deletes:** None.

## Business Rules

- Only a Subscription in Active status renews on its billing date; a Cancelled (active to period end) subscription never renews, per product-features.md FEAT-18 (Validation & Limits).
- The grace period is fixed at platform parameter: `subscription-payment-failure-grace-period-days`, per FEAT-18.SPEC-005.
- The grace deadline is set once, at the original failure's confirmed date, and is never extended by a subsequent failed retry within the same grace window.
- XBR-11 and XBR-14: this automation never itself cancels bookings or affects existing bookings -- a renewal failure only changes the Subscription's own status; the Pro Account pause (which stops new bookings) is a separate, standalone effect owned by FEAT-18.SPEC-004.
- The payment-processing capability's recorded outcome is authoritative, per the dependency map's Contention note for the Subscription entity -- this automation never infers success or failure locally.
- A price-change effective date is a system-only write, parallel to the grace-status transition: the Subscription's price field is updated to the new platform parameter: `subscription-price` value exactly once, on the effective_date already disclosed by FEAT-18.SPEC-007's price-change notice, and is never triggered by any Pro action or screen input.

## Edge Cases

- **The Pro cancels the subscription on the same day a renewal would have fired** -- Cancellation (FEAT-18.SPEC-005) takes effect for billing purposes immediately: the Subscription's status is Cancelled (active to period end) before this automation's renewal check runs, so no renewal charge fires and the "No renewal needed" outcome applies.
- **The renewal outcome event arrives twice (duplicate delivery from FEAT-18.SPEC-006)** -- The second delivery is a no-op: a Subscription already advanced to the next billing cycle, or already in Payment Failed with its grace deadline already recorded, is not changed again, and no duplicate notification fires.
- **Concurrent trigger firing (the scheduled renewal and a Pro-initiated payment-method-update retry overlap for the same Subscription)** -- Only one outcome is applied: the payment-processing capability processes one charge at a time per Subscription, and this automation applies whichever confirmed outcome arrives first; the other attempt, if it was in fact a duplicate submission, is reconciled to the same confirmed outcome rather than double-charging.
- **A trigger fires while a previous run is still in flight for the same Subscription** -- A second renewal attempt cannot start for a Subscription whose prior renewal charge has not yet returned an outcome; the automation waits for the in-flight outcome before evaluating whether another attempt is needed.
- **The payment method is updated but the retry charge itself times out** -- Treated as a failure outcome (per FEAT-18.SPEC-006's degradation behavior): the Subscription remains Payment Failed with its original grace deadline unchanged, and the Pro sees the standard failure state, not a false success.
- **A price-change effective date coincides with a scheduled renewal date** -- The price update (step 6) is applied before the renewal check (step 1) runs, so the renewal charge submitted that day uses the new platform parameter: `subscription-price` value, and the resulting renewal receipt (FEAT-18.SPEC-007) reflects the updated price rather than the prior one.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-18.SPEC-006 (Subscription Billing Integration) | Triggered by (inbound) | The renewal-outcome inbound event fires this automation's outcome branching |
| FEAT-18.SPEC-006 (Subscription Billing Integration) | Triggers (outbound) | This automation submits the renewal (and retry) charge requests |
| FEAT-18.SPEC-002 (Billing & Subscription Management Screen) | Affects (outbound) | Status banner and next billing date reflect this automation's outcomes |
| FEAT-18.SPEC-004 (Subscription-Lapse Account Pause Trigger) | Affects (outbound) | An unresolved grace expiry hands off to this automation |
| FEAT-18.SPEC-005 (Subscription Billing Rules) | References (inbound) | Grace-period length, renewal-eligibility rules, and the price-change notice lead time that sets each effective_date |
| FEAT-18.SPEC-007 (Subscription Billing Notifications) | Triggers (outbound) | Renewal success and failure each trigger their respective notification |

## Analytics and Success Signals

- **subscription_renewed** (renewal number in sequence) -- supports success-metrics.md: "Subscription Retention"
- **subscription_payment_failed** (grace deadline) -- supports success-metrics.md: "Subscription Retention"
- **subscription_payment_recovered** (days into grace period when recovered) -- supports success-metrics.md: "Subscription Retention"

## Acceptance Criteria

**FEAT-18.SPEC-003-AC-01:** Given Talia's subscription is Active and reaches its billing date, when the renewal charge succeeds, then next_billing_date advances one cycle, status remains Active, and she receives the renewal receipt (FEAT-18.SPEC-007).

**FEAT-18.SPEC-003-AC-02:** Given Talia's subscription is Active and reaches its billing date, when the renewal charge fails, then status is set to Payment Failed with a grace deadline platform parameter: `subscription-payment-failure-grace-period-days` from the failure date, and she receives the payment-failure grace notice (FEAT-18.SPEC-007).

**FEAT-18.SPEC-003-AC-03:** Given Talia's subscription is in Payment Failed (grace), when she updates her payment method and the retried charge succeeds before the grace deadline, then status returns to Active, next_billing_date advances, and she receives a renewal receipt.

**FEAT-18.SPEC-003-AC-04:** Given Talia's subscription is in Payment Failed (grace) and her retried payment method is also declined, when the retry fails, then the Subscription remains Payment Failed and the original grace deadline is unchanged.

**FEAT-18.SPEC-003-AC-05:** Given Talia's subscription is in Payment Failed (grace) and no successful charge occurs by the grace deadline, when the deadline passes, then this automation hands off to FEAT-18.SPEC-004, which pauses the Pro Account.

**FEAT-18.SPEC-003-AC-06:** Given Talia's subscription is Cancelled (active to period end), when its former billing date arrives, then no renewal charge fires.

**FEAT-18.SPEC-003-AC-07:** Given the payment-processing capability delivers the same renewal-outcome event twice, when the second delivery arrives, then the Subscription's state does not change again and no duplicate notification fires.

**FEAT-18.SPEC-003-AC-08:** Given a scheduled renewal and a Pro-initiated retry could overlap for the same Subscription, when both are in flight, then only one confirmed outcome is applied and no duplicate charge occurs.

**FEAT-18.SPEC-003-AC-09:** Given a renewal charge for Talia's subscription is already in flight, when the schedule would otherwise fire a second attempt for the same Subscription, then the second attempt waits for the in-flight outcome rather than starting concurrently.

**FEAT-18.SPEC-003-AC-10:** Given Talia updates her payment method during grace and the retry charge itself times out, when the timeout is reported, then the Subscription remains Payment Failed with its original grace deadline, and she is not shown a false success.

**FEAT-18.SPEC-003-AC-11:** Given a price change's effective_date arrives for Talia's subscription, when this automation processes it, then Subscription.price is updated to the new platform parameter: `subscription-price` value, and any renewal charge submitted on or after that date uses the new price.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 (scheduled renewal, renewal outcome received, price-change effective date) | 3 |
| Outcome Paths | 7 | 7 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |



# Automation Spec: Subscription-Lapse Account Pause Trigger

## Overview

**Name:** Subscription-Lapse Account Pause Trigger
**ID:** FEAT-18.SPEC-004
**Type:** Automation
**Purpose:** Triggers the Pro Account's system-imposed pause when the grace period expires unresolved, and lifts it the moment billing is restored.
**Parent Feature:** FEAT-18 -- Pro Subscription Billing & Account Management

## Scope and Non-Goals

**In Scope:**
- Setting the Pro Account's system-imposed Paused state when the grace period expires unresolved
- Clearing that system-imposed pause the moment billing is restored (payment method confirmed and the retry charge succeeds)
- Coordinating with the Pro Account's own pause mechanism so a system-imposed pause is distinguished from a Pro-chosen one

**Non-Goals:**
- Deciding whether a renewal succeeded or failed, or starting the grace clock -- owned by FEAT-18.SPEC-003 (Subscription Renewal & Payment-Failure Processing); this spec only acts on that automation's "grace expires unresolved" outcome
- Any other Pro Account field (profile, other pause reasons, closure) -- owned by FEAT-27 (Pro Profile & Booking Page Settings) and FEAT-29 (Pro Sign-In & Account Lifecycle), per the Feature Breakdown Brief's Referenced Entities note: this feature writes only the Paused state, and only for this one reason
- What a paused account looks like to clients on the public booking page -- owned by FEAT-05 (Public Booking Page & Booking Flow), which reads the Pro Account's Paused state; this spec only sets or clears that state
- Affecting existing bookings, reminders, refunds, or client self-service -- explicitly excluded per XBR-14 and XBR-11: a subscription lapse pauses new bookings only, and this automation never touches Booking, Deposit Transaction, or Messaging Consent records

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Grace period expires unresolved | FEAT-18.SPEC-003 (Subscription Renewal & Payment-Failure Processing) | Fires when a Subscription remains Payment Failed at its recorded grace deadline with no successful charge | Subscription reference, Pro Account reference, grace deadline that passed |
| Billing restored | FEAT-18.SPEC-003 (Subscription Renewal & Payment-Failure Processing) | Fires when a Subscription that was Payment Failed (whether or not the account was already paused) returns to Active via a successful retry | Subscription reference, Pro Account reference |

## Processing Logic

1. On "grace period expires unresolved," read the Pro Account's current status.
2. If the Pro Account is not already Paused for any reason, set its status to Paused with the system-imposed subscription-lapse reason recorded (distinct from a Pro-chosen pause, per the dependency map's Contention note: "a system-imposed subscription pause cannot be cleared by the Pro's 'resume bookings' toggle until billing is restored").
3. If the Pro Account is already Paused for a Pro-chosen reason, record that a subscription lapse also applies, so that clearing the Pro-chosen pause alone does not resume bookings while billing remains unresolved.
4. On "billing restored," read the Pro Account's current status.
5. If the Pro Account's Paused state was set (or is jointly held) for the system-imposed subscription-lapse reason, clear that reason. If no other pause reason remains, set status back to Active. If a Pro-chosen pause reason still applies independently, the account remains Paused for that reason alone.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Account paused (lapse only) | Grace expires unresolved and the account was Active | Pro Account.status set to Paused, reason = subscription lapse | Dashboard attention item added (FEAT-12.SPEC-005); public booking page (FEAT-05) stops offering new bookings | FEAT-12, FEAT-05 |
| Account already paused, lapse reason added | Grace expires unresolved while the account is already Paused for a Pro-chosen reason | Pro Account records the subscription-lapse reason alongside the existing pause reason | No visible change to the booking page (already paused); the Pro's "resume bookings" toggle becomes unable to clear the pause by itself until billing is restored | FEAT-27 |
| Pause lifted (lapse only) | Billing restored and the subscription-lapse reason was the only pause reason | Pro Account.status set back to Active | Public booking page resumes offering new bookings; dashboard attention item clears | FEAT-12, FEAT-05 |
| Lapse reason cleared, Pro-chosen pause remains | Billing restored while a Pro-chosen pause reason also applies | Subscription-lapse reason removed; Pro Account remains Paused for the Pro-chosen reason | No visible change to the booking page (still paused, now only for the Pro's own reason); the Pro's "resume bookings" toggle now works | FEAT-27 |
| No action needed | Billing restored but no subscription-lapse pause reason was ever recorded (grace never expired) | None | None | -- |

## Data Model

**Reads:** Pro Account -- status, pause reason(s); Subscription -- status.
**Creates:** None.
**Updates:** Pro Account -- status (Active <-> Paused), the subscription-lapse pause reason component specifically (this feature never writes any other Pro Account field, per the dependency map's Referenced Entities note).
**Deletes:** None.

## Business Rules

- XBR-14: a paused account (from this trigger or a Pro-chosen pause) takes no new bookings or deposits, while existing bookings keep their reminders, refunds, and client self-service unchanged.
- XBR-11: this automation never cancels or silently changes any existing Booking; any conflict this pause creates for an already-scheduled booking is out of scope here (there is none -- existing bookings are simply unaffected).
- Per the dependency map's Pro Account Contention note: a system-imposed subscription pause cannot be cleared by the Pro's own "resume bookings" toggle (owned by FEAT-27) until billing is restored -- only this automation clears it.
- Governed by FEAT-18.SPEC-005 (Subscription Billing Rules): the 7-day grace threshold (platform parameter: `subscription-payment-failure-grace-period-days`) whose expiry fires this automation, and the rule that a lapse pauses the account rather than deleting it, are defined there; this automation enforces them at grace expiry and does not redefine them.
- This automation is the sole writer of the subscription-lapse pause reason; all other Pro Account fields remain exclusively owned by FEAT-27 and FEAT-29.

## Edge Cases

- **Grace expires while the Pro has simultaneously initiated a payment-method update that has not yet been confirmed** -- The pause is triggered as soon as the grace deadline passes with no confirmed successful charge; if the in-flight update then succeeds moments later, the "billing restored" outcome fires immediately after and lifts the pause -- the Pro Account may be Paused for a brief window but never left paused once billing is genuinely restored.
- **Concurrent trigger firing (grace expiry and billing restoration are reported at effectively the same time, e.g., a very late retry)** -- FEAT-18.SPEC-003 reports at most one outcome per Subscription at a time (per its own concurrency handling), so this automation never receives both triggers simultaneously for the same Subscription; whichever outcome the payment-processing capability actually confirmed is the one applied.
- **Trigger fires while a previous run is in flight for the same Pro Account** -- A second pause/lift evaluation for the same Pro Account waits for the prior one to complete before reading and writing status, so the two writes cannot race and leave an inconsistent combined pause-reason state.
- **The Pro manually pauses their own account (via FEAT-27) while a subscription-lapse pause is already in effect** -- Both reasons are recorded jointly (per Processing Logic step 3); billing being restored later clears only the subscription-lapse reason, leaving the account Paused for the Pro's own reason until the Pro clears it themselves via FEAT-27.
- **Billing is restored for a Pro Account that was never actually paused (grace resolved before the deadline via FEAT-18.SPEC-003's retry path)** -- No pause was ever set by this automation, so "billing restored" is a no-op here; the Subscription itself simply returns to Active per FEAT-18.SPEC-003, without this automation ever having acted.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-18.SPEC-003 (Subscription Renewal & Payment-Failure Processing) | Triggered by (inbound) | "Grace expires unresolved" and "billing restored" outcomes fire this automation |
| FEAT-27 (Pro Profile & Booking Page Settings) | Affects (outbound) | Sets/clears the Pro Account's system-imposed pause; FEAT-27 owns the pause state field itself and the Pro-facing "resume bookings" toggle, which cannot clear this pause alone |
| FEAT-05 (Public Booking Page & Booking Flow) | Affects (outbound) | A pause set here stops the booking page from offering new bookings; existing bookings, reminders, refunds and client self-service continue unchanged (XBR-14, XBR-11) |
| FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) | A payment-failure pause surfaces as a dashboard attention item (FEAT-12.SPEC-005) |
| FEAT-18.SPEC-005 (Subscription Billing Rules) | References (inbound) | Enforces the grace-threshold expiry and pause-not-delete rules defined there (this spec appears in that rule's Enforced By table) |
| FEAT-18.SPEC-002 (Billing & Subscription Management Screen) | Affects (outbound) | The screen's status banner and grace messaging reflect this automation's state |

## Analytics and Success Signals

- **pro_account_paused_subscription_lapse** (days grace was open before pause) -- supports success-metrics.md: "Subscription Retention"
- **pro_account_pause_lifted_billing_restored** (days paused before restoration) -- supports success-metrics.md: "Subscription Retention"

## Acceptance Criteria

**FEAT-18.SPEC-004-AC-01:** Given Talia's subscription grace period expires with no successful charge, when this automation fires, then her Pro Account's status is set to Paused with the subscription-lapse reason recorded.

**FEAT-18.SPEC-004-AC-02:** Given Talia's Pro Account is paused only for a subscription lapse, when billing is restored (a successful retry), then her Pro Account's status returns to Active.

**FEAT-18.SPEC-004-AC-03:** Given Talia's Pro Account is already Paused for a Pro-chosen reason, when her grace period also expires unresolved, then the subscription-lapse reason is recorded alongside the existing pause, and her own "resume bookings" toggle cannot clear the pause until billing is restored.

**FEAT-18.SPEC-004-AC-04:** Given Talia's Pro Account is paused for both a Pro-chosen reason and a subscription lapse, when billing is restored, then only the subscription-lapse reason is cleared and the account remains Paused for her own reason until she clears it via FEAT-27.

**FEAT-18.SPEC-004-AC-05:** Given Talia's Pro Account is paused from a subscription lapse, when a client visits her public booking page, then no new bookings can be started, while any of Talia's existing bookings keep their reminders, refunds, and client self-service unchanged, per XBR-14.

**FEAT-18.SPEC-004-AC-06:** Given Talia's Pro Account is paused from a subscription lapse, when the pause is set, then a dashboard attention item appears per FEAT-12.SPEC-005.

**FEAT-18.SPEC-004-AC-07:** Given a pause-evaluation run is already in flight for Talia's Pro Account, when a second trigger fires for the same account, then the second evaluation waits for the first to complete before reading or writing status.

**FEAT-18.SPEC-004-AC-08:** Given Talia's subscription grace resolves before its deadline via a successful retry, when the Subscription returns to Active, then this automation never sets a pause, since the grace never actually expired.

**FEAT-18.SPEC-004-AC-09:** Given Talia's Pro Account is briefly paused because a late retry had not yet confirmed when the grace deadline passed, when that retry's success is confirmed moments later, then the pause is lifted immediately and the account is not left paused once billing is genuinely restored.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (grace expires unresolved, billing restored) | 2 |
| Outcome Paths | 5 | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |



# Logic/Rule Spec: Subscription Billing Rules

## Overview

**Name:** Subscription Billing Rules
**ID:** FEAT-18.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs the single price tier, the 7-day grace threshold (platform parameter: `subscription-payment-failure-grace-period-days`), cancellation-at-period-end timing, the 30-day price-change notice rule (platform parameter: `subscription-price-change-notice-days`), and contention/authority resolution between the Pro and the payment-processing capability for the Subscription entity.
**Parent Feature:** FEAT-18 -- Pro Subscription Billing & Account Management
**Governed Entity:** Subscription

## Scope and Non-Goals

**In Scope:**
- Field validation for every Subscription field
- The single-tier price rule (no plan selection)
- The 7-day payment-failure grace threshold
- Cancellation-at-period-end timing
- The 30-day price-change notice rule (platform parameter: `subscription-price-change-notice-days`)
- Authorization rules for every action on the Subscription record, per role
- Contention/authority resolution between the Pro and the payment-processing capability

**Non-Goals:**
- The renewal charge mechanics and grace-clock processing itself -- owned by FEAT-18.SPEC-003 (Subscription Renewal & Payment-Failure Processing); this spec defines the threshold values and timing rules that automation applies
- The Pro Account pause mechanics -- owned by FEAT-18.SPEC-004 (Subscription-Lapse Account Pause Trigger); this spec governs only the Subscription entity itself, not the Pro Account
- The screens' layout and interaction mechanics -- owned by FEAT-18.SPEC-001 and FEAT-18.SPEC-002; this spec is their shared rule source, referenced by ID rather than duplicated
- Support acting on a subscription -- excluded per scope-boundaries.md SC-05: Platform Operator (Support) has view-only access and can never edit, cancel, or update payment details on the Pro's behalf

## Governed Entity

**Entity:** Subscription
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| status | enum | Active \| Payment Failed (7-day grace) \| Cancelled (active to period end) |
| billing_cycle | derived | The recurring monthly interval this Subscription bills on |
| next_billing_date | date | The date the next renewal charge is due |
| payment_method_reference | text | A reference to the card on file; the payment-processing capability holds the actual card data |
| price | number | The single all-inclusive subscription price (platform parameter: `subscription-price`) |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-18.SPEC-001 | Subscribe Screen | On submit, applying the single-tier price; authorization on screen entry |
| FEAT-18.SPEC-002 | Billing & Subscription Management Screen | On cancel confirmation and payment-method update; authorization on screen entry and on each action |
| FEAT-18.SPEC-003 | Subscription Renewal & Payment-Failure Processing | On each renewal cycle, applying the grace threshold; on a price change's effective date, writing the new value of platform parameter: `subscription-price` into the Subscription's price field |
| FEAT-18.SPEC-004 | Subscription-Lapse Account Pause Trigger | Reads the grace threshold's expiry to decide when to pause |
| FEAT-18.SPEC-006 | Subscription Billing Integration | Authority resolution when the processor's outcome and the Pro's local view disagree |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| status | Must be one of: Active, Payment Failed (7-day grace), Cancelled (active to period end) | Always | On every write | No validation beyond data type -- status transitions are governed by Business Rules below, not by field-level format checks | No |
| billing_cycle | No validation beyond data type -- derived from next_billing_date and never directly entered | Always | -- | -- | -- |
| next_billing_date | Must be a valid future date at the time it is set | On write (create, renewal advance) | On write | "Could not schedule the next billing date. Please try again." | Yes |
| payment_method_reference | Must be a reference confirmed by the payment-processing capability; never a raw card value | Always | On write (create, payment-method update) | "This card could not be confirmed. Please check your details and try again." | Yes |
| price | Must equal the single platform-set tier (platform parameter: `subscription-price`) | Always | On write | No validation beyond data type -- there is no Pro-facing price input to reject; the value is set by the product, never entered | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Grace status requires a failure basis | status, next_billing_date | status can only be Payment Failed (7-day grace) following a recorded renewal failure against the existing next_billing_date -- it cannot be set directly by any user action | N/A -- this is a system-only transition; no user-facing input can trigger it directly, so no error message applies |
| Cancellation preserves the paid period | status, next_billing_date | When status is set to Cancelled (active to period end), next_billing_date is not advanced further -- it marks the date the account pauses, not a future renewal | N/A -- enforced structurally by FEAT-18.SPEC-006's cancellation processing, not by user input validation |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create Subscription (first subscribe) | The Pro | Only once per Pro Account -- a Pro Account already holding an active or cancelled-pending Subscription cannot create a second one | The Subscribe Screen (FEAT-18.SPEC-001) is not reached a second time; onboarding routes a Pro who already has a Subscription straight past the subscription step |
| Create Subscription | Platform Operator (Support) | Never | No create control exists in Support's view; Support only ever views an existing Subscription, never originates one |
| Create Subscription | The Client | Never | Clients have no subscription-related capability at all, per the Access Matrix (Subscription & Billing = None); no client-facing screen exposes this action |
| View Subscription (status, next billing date, price) | The Pro | Always, own Subscription only | -- |
| View Subscription (status, next billing date, price -- no payment_method_reference) | Platform Operator (Support) | Always, scoped to the one Pro Account being supported at a time | -- |
| View Subscription | The Client | Never | Clients have no visibility into this at all, per the Access Matrix (Subscription & Billing = None); no client-facing screen or link exposes this entity |
| Update payment method | The Pro | Always, own Subscription only | -- |
| Update payment method | Platform Operator (Support) | Never | No control is rendered in Support's view (FEAT-18.SPEC-002); this is a structural absence, not a permission check that could be bypassed |
| Update payment method | The Client | Never | No client-facing screen or capability exists for this action, per the Access Matrix (Subscription & Billing = None) |
| Cancel Subscription (Pro-initiated) | The Pro | Always, own Subscription only, and only when status is Active or Payment Failed (not already Cancelled) | Cancel action is not offered a second time once already Cancelled (active to period end) |
| Cancel Subscription (invoked by account closure) | The Pro, via FEAT-29 | Only as part of the Pro's own account closure flow (XBR-20) | N/A -- this path is always Pro-authorized because it is the Pro's own closure action; there is no separate denial case |
| Cancel Subscription | Platform Operator (Support) | Never | No cancel control exists in Support's view; excluded per scope-boundaries.md SC-05 |
| Cancel Subscription | The Client | Never | No client-facing screen or capability exists for this action, per the Access Matrix (Subscription & Billing = None) |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| billing_cycle | Derived as "monthly," fixed by the single-tier definition | On create | No |
| next_billing_date | Set to one billing cycle (one month) from the date the first charge is confirmed | On create | No -- always processor-confirmed, never Pro-entered |
| price | Defaults to platform parameter: `subscription-price` on create; re-written to the new value of platform parameter: `subscription-price` on an announced price change's effective date, executed by FEAT-18.SPEC-003 | On create; and again on each announced price change's effective date | No -- one price tier only, per product-features.md FEAT-18 (Validation & Limits); the value is never Pro-entered on create or on a later price change, only ever platform-set |
| status | Defaults to Active | On create (immediately following a confirmed first charge) | No |

## Business Rules

- **Single price tier:** Exactly one subscription price applies to every Subscription (platform parameter: `subscription-price`); no plan selection exists anywhere in the product, per product-features.md FEAT-18 (Validation & Limits).
- **Grace threshold:** A renewal failure starts a grace period of platform parameter: `subscription-payment-failure-grace-period-days`, measured from the failure's confirmed date; the deadline is fixed at that point and is not extended by a subsequent failed retry within the same window (FEAT-18.SPEC-003).
- **Cancellation timing:** Cancellation takes effect at the end of the already-paid billing period, never an immediate mid-period cutoff -- a cancelled subscription remains Active in every functional sense (booking link stays live, billing already collected) through that date, then the Pro Account pauses (FEAT-18.SPEC-004).
- **Price-change notice:** Any future price change is announced to the Pro at least platform parameter: `subscription-price-change-notice-days` before it applies to their Subscription, per FEAT-18.SPEC-007's price-change notice.
- **Price-change effective-date update:** On the effective_date carried by that notice, FEAT-18.SPEC-003 writes the new platform parameter: `subscription-price` value into the Subscription's price field, exactly once per announced change; this write is a system-only transition, never triggered by a Pro action, and follows the same pattern as the grace-status transition above -- the rule fixes the timing, FEAT-18.SPEC-003 executes the write.
- **Processor authority (contention):** The payment-processing capability's recorded outcome for a charge, refund, or payment-method confirmation is authoritative for the Subscription's status and payment_method_reference; a stale Pro-facing screen showing a status that has since changed at the processor is refused with refresh (FEAT-18.SPEC-002's Edge Cases), never silently overwritten by a locally computed guess.
- **Reactivation authority (contention):** Cancellation is last-write-wins against a reactivation attempted before period end -- if both a cancel and a reactivation were somehow requested close together, whichever the processor confirms last stands; in practice this product offers no Pro-facing reactivation control before period end (FEAT-18.SPEC-002's Edge Cases), so this rule protects against any future or support-invoked path rather than a control the Pro sees today.
- **Cross-feature scope (XBR-14, XBR-11):** These rules govern only the Subscription entity; they never cancel, modify, or flag any Booking, Deposit Transaction, or Messaging Consent record. The Pro Account's pause state is governed separately by FEAT-27/FEAT-18.SPEC-004.
- **Cross-feature scope (XBR-20):** Account closure (owned by FEAT-29) invokes this Subscription's cancellation the same way a Pro-initiated cancel does -- it does not bypass the period-end timing rule.

## Edge Cases

- **Grace deadline computed at exactly the boundary (a retry confirmed on the exact calendar day the grace period ends)** -- The retry is honored if the payment-processing capability confirms it before the deadline moment passes; a confirmation arriving after that moment is treated as "grace expired unresolved," per FEAT-18.SPEC-003/FEAT-18.SPEC-004.
- **Cancellation requested on the exact day of the next scheduled renewal** -- The renewal does not fire once cancellation is confirmed (FEAT-18.SPEC-003's "No renewal needed" outcome); the Subscription is Cancelled (active to period end) through the already-paid period that predates this renewal date, and the account pauses at that existing period's end rather than being charged again first.
- **Price-change notice window overlaps a Pro's own grace period** -- The two timers are independent: a price change and a payment-failure grace period can both be in progress for the same Subscription at once, and each notice (FEAT-18.SPEC-007) is sent on its own schedule without one suppressing the other.
- **A Subscription's payment_method_reference is confirmed by the processor while the Pro's screen is mid-submission on a separate action (e.g., cancel)** -- The processor-authority rule governs: whichever confirmed outcome lands first is applied, and the other in-flight action is refused with refresh if it was evaluated against a status that has since changed.
- **Support views a Subscription whose payment_method_reference the Pro just updated** -- Support's view never includes payment_method_reference at all (per Authorization Rules), so no stale-value scenario applies to Support; they see only status, next billing date, and price, refreshed on each open.

## Acceptance Criteria

**FEAT-18.SPEC-005-AC-01:** Given Talia is subscribing for the first time, when her Subscription is created, then its price is set to the single tier (platform parameter: `subscription-price`) and no plan-selection option is ever offered.

**FEAT-18.SPEC-005-AC-02:** Given Talia's Subscription already exists, when she completes onboarding again for any reason, then she is not routed back through the Subscribe Screen a second time.

**FEAT-18.SPEC-005-AC-03:** Given Talia's renewal charge fails, when the failure is confirmed, then her Subscription's grace deadline is set to the failure date plus the fixed grace-period threshold.

**FEAT-18.SPEC-005-AC-04:** Given Talia's subscription is in Payment Failed (grace) and a retry also fails before the deadline, when the failed retry is confirmed, then the original grace deadline remains unchanged.

**FEAT-18.SPEC-005-AC-05:** Given Talia cancels her subscription, when the cancellation is confirmed, then her Subscription remains functionally active (booking link stays live) through the end of her already-paid billing period, never cutting off immediately.

**FEAT-18.SPEC-005-AC-06:** Given a price change is decided for the product, when the change is announced, then Talia is notified at least the fixed price-change notice window before it applies to her Subscription.

**FEAT-18.SPEC-005-AC-07:** Given Talia's Billing screen shows a status that has since changed at the payment-processing capability, when she attempts an action against it, then the action is refused with refresh rather than trusting the stale local status.

**FEAT-18.SPEC-005-AC-08:** Given Talia (The Pro) views her own Subscription, when she opens the Billing screen, then she sees the full status, next billing date, price, and payment method reference.

**FEAT-18.SPEC-005-AC-09:** Given a support operator (Platform Operator) views a Pro's Subscription, when they open the Billing screen in Support's view, then they see status, next billing date, and price only -- never the payment method reference.

**FEAT-18.SPEC-005-AC-10:** Given a support operator attempts to change a Pro's payment method or cancel their subscription, when they look for a control to do so, then none is shown -- Support's view offers no edit path at all.

**FEAT-18.SPEC-005-AC-11:** Given a Client attempts to reach any subscription-related view, when they try, then no such view or link exists for the Client role.

**FEAT-18.SPEC-005-AC-12:** Given Talia's subscription is already Cancelled (active to period end), when she looks for a cancel action, then it is not offered a second time.

**FEAT-18.SPEC-005-AC-13:** Given Talia closes her account via FEAT-29 (account closure, XBR-20), when the closure flow invokes this Subscription's cancellation, then the same period-end timing rule applies -- there is no immediate cutoff bypass for account closure.

**FEAT-18.SPEC-005-AC-14:** Given Talia's renewal fails on the exact calendar day her account would otherwise cross the grace deadline, when her retry is confirmed just before that deadline moment passes, then the retry is honored and the Subscription returns to Active.

**FEAT-18.SPEC-005-AC-15:** Given Talia's subscription is cancelled effective on the same date a renewal would otherwise have fired, when that date arrives, then no renewal charge fires and the account pauses at the already-paid period's end.

**FEAT-18.SPEC-005-AC-16:** Given Talia has both a price-change notice pending and an active payment-failure grace period at the same time, when each notice's own schedule arrives, then both are delivered independently without one suppressing the other.

**FEAT-18.SPEC-005-AC-17:** Given a support operator or a Client attempts to originate a new Subscription, when they look for a way to do so, then no create path exists for either role -- only the Pro can create a Subscription, and only once.

**FEAT-18.SPEC-005-AC-18:** Given a Client attempts to update a payment method or cancel a subscription, when they look for a way to do so, then no client-facing screen or capability exists for either action.

**FEAT-18.SPEC-005-AC-19:** Given a price change's effective_date arrives for Talia's subscription, when the date is reached, then her Subscription's price field is updated to the new value of platform parameter: `subscription-price` by FEAT-18.SPEC-003, as governed by this rule's timing.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 13 | 13 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 9 | 9 |
| Edge Cases | 5 | 5 |



# Integration Spec: Subscription Billing Integration

## Overview

**Name:** Subscription Billing Integration
**ID:** FEAT-18.SPEC-006
**Type:** Integration
**Purpose:** Carries outbound subscribe, update-payment-method, and cancel requests to the payment-processing capability, and receives inbound renewal-outcome and dispute events, so the Pro's subscription billing is handled without the product ever holding card data.
**Parent Feature:** FEAT-18 -- Pro Subscription Billing & Account Management

## Scope and Non-Goals

**In Scope:**
- Submitting the first subscribe charge, payment-method updates, and cancellation requests to the payment-processing capability
- Receiving renewal-outcome events (success, failure) and reflecting them on the Subscription
- Receiving card-issuer dispute notices related to the subscription charge itself
- User-facing behavior when the capability is slow, unavailable, or rejects a request
- Disclosure to the Pro about what data is shared with the capability

**Non-Goals:**
- Choosing the payment-processing vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate, so this spec stays vendor-neutral throughout
- The grace-period clock and pause-trigger logic that reacts to a renewal failure -- owned by FEAT-18.SPEC-003 and FEAT-18.SPEC-004; this spec delivers the raw outcome event and stops there
- Client deposit, balance, or payout processing -- a separate relationship with the same capability category, owned by FEAT-07.SPEC-005 (deposits) and FEAT-28.SPEC-006 (payout accounts); this spec covers only the Pro's own subscription billing
- Writing the Subscription's price field after creation -- FEAT-18.SPEC-003 is the sole Subscription.price writer after creation (on an announced price change's effective date, per FEAT-18.SPEC-005); this spec only sets the price on the initial create from the first-charge confirmation and never re-writes it
- The screens' layout and content that surface this integration's outcomes -- owned by FEAT-18.SPEC-001 and FEAT-18.SPEC-002; this spec defines only the capability contract they consume

## Capability Category

**Category:** Payment processing
**Dependency Source:** ASMP-31 -- "Payment-processing capability... required to... bill the pro's own monthly subscription" (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Payment processing — Pro subscription billing" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-18, FEAT-15)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Talia subscribes with a card during onboarding and her first charge is confirmed | Subscribe during onboarding with a card | FEAT-18.SPEC-001 (Subscribe Screen) |
| Talia updates the payment method on file | Update the payment method on file | FEAT-18.SPEC-002 (Billing & Subscription Management Screen) |
| Talia cancels her subscription, effective at the end of the current billing period | Cancel the subscription at any time, effective at the end of the current billing period | FEAT-18.SPEC-002 (Billing & Subscription Management Screen) |
| Talia's monthly subscription renews automatically without her needing to re-enter payment details | Subscribe during onboarding with a card | FEAT-18.SPEC-003 (Subscription Renewal & Payment-Failure Processing) |
| Account closure cancels Talia's subscription the same way, without a separate billing flow | Cancel the subscription at any time (invoked by account closure, XBR-20) | FEAT-29 (Pro Sign-In & Account Lifecycle) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| First subscribe charge request | Subscription -- price (platform parameter: `subscription-price`); Pro Account -- reference (never bank or identity details, which the Pro provides directly to the capability's own surface) | Talia submits the Subscribe Screen (FEAT-18.SPEC-001) | The capability must know what to charge and which account to attribute it to |
| Card details (entered directly into the capability's own surface, never touching the product's own code) | Not a product entity -- entered by the Pro directly on the capability's surface | Talia enters payment details on FEAT-18.SPEC-001 or the payment-method update on FEAT-18.SPEC-002 | The capability handles all card data per SC-11; the product never receives or transmits the raw card values |
| Renewal charge request | Subscription -- price, payment_method_reference | Each monthly billing-cycle date, for an Active Subscription (FEAT-18.SPEC-003) | The capability must know what to charge and against which stored payment method |
| Payment-method update request | Subscription -- payment_method_reference (the new card is entered directly into the capability's surface, as above) | Talia submits "Update payment method" (FEAT-18.SPEC-002) | The capability must confirm and store the new card, replacing the reference on file |
| Cancellation request | Subscription -- reference | Talia confirms cancel (FEAT-18.SPEC-002), or FEAT-29 invokes cancellation as part of account closure (XBR-20) | The capability must stop billing this Subscription at the end of the current period |

Bank details, identity details, the Pro's studio address, and every other Pro Account and Booking field never leave the product through this integration.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| First-charge confirmation | The capability confirms the first subscribe charge succeeded | Subscription -- created with status Active, next_billing_date, payment_method_reference, price |
| First-charge decline | The capability declines the first subscribe charge | No Subscription created; decline reason category surfaced to FEAT-18.SPEC-001 |
| Renewal outcome (success) | The capability confirms a monthly renewal charge succeeded | Subscription -- next_billing_date advanced (via FEAT-18.SPEC-003) |
| Renewal outcome (failure) | The capability reports a monthly renewal charge failed or was declined | Subscription -- status set to Payment Failed (via FEAT-18.SPEC-003); failure reason category recorded |
| Payment-method update confirmation | The capability confirms a new card was captured and stored | Subscription -- payment_method_reference updated |
| Payment-method update rejection | The capability rejects the new card | Subscription -- payment_method_reference unchanged; rejection reason category surfaced to FEAT-18.SPEC-002 |
| Cancellation confirmation | The capability confirms the subscription will stop billing at period end | Subscription -- status set to Cancelled (active to period end), via FEAT-18.SPEC-005's timing rule |
| Card-issuer dispute notice on a subscription charge | A dispute is raised against a subscription charge | Subscription -- no status field change (subscriptions carry no Disputed overlay in v1); the dispute is recorded for the Pro's own account activity and surfaced to Support (FEAT-19), consistent with XBR-22's dispute-evidence pattern for other financial records |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| First-charge succeeded | The capability confirms Talia's first subscribe charge | Subscription created (status Active) | Confirmation shown on FEAT-18.SPEC-001; onboarding continues and reports the go-live signal (XBR-26) | FEAT-18.SPEC-001, FEAT-15.SPEC-005 |
| First-charge declined | The capability declines Talia's first subscribe charge | No Subscription created | FEAT-18.SPEC-001 shows the decline reason inline with a retry action | FEAT-18.SPEC-001 |
| Renewal outcome received | Monthly renewal charge resolves, success or failure | Handled by FEAT-18.SPEC-003 (advance billing date or set Payment Failed) | Status banner (FEAT-18.SPEC-002) updates; renewal receipt or payment-failure notice fires (FEAT-18.SPEC-007) | FEAT-18.SPEC-003, FEAT-18.SPEC-002, FEAT-18.SPEC-007 |
| Payment-method update confirmed | The capability confirms a new card | payment_method_reference updated | FEAT-18.SPEC-002 shows "Payment method updated"; if the Subscription was in grace, a renewal retry follows (FEAT-18.SPEC-003) | FEAT-18.SPEC-002, FEAT-18.SPEC-003 |
| Payment-method update rejected | The capability rejects a new card | No change | FEAT-18.SPEC-002 shows the rejection reason inline with retry | FEAT-18.SPEC-002 |
| Cancellation confirmed | The capability confirms the subscription will stop billing at period end | Subscription status set to Cancelled (active to period end) | FEAT-18.SPEC-002's status banner updates; cancellation confirmation notification fires | FEAT-18.SPEC-002, FEAT-18.SPEC-007 |
| Subscription status changed | The Subscription's status changes to or from Active as a result of any inbound event above (first-charge success, renewal outcome, payment-method-update recovery, cancellation) | No additional data change beyond the event's own | No direct Pro-facing feedback; the change is reported to FEAT-15.SPEC-005 (Go-Live Evaluation & Booking Link Activation), which re-evaluates the go-live check as an external-event trigger | FEAT-15.SPEC-005 |
| Subscription charge disputed | A card issuer raises a dispute against a subscription charge | Dispute recorded in the Pro's own account activity | The Pro sees the dispute noted in their account activity; Support can view it during a help request | FEAT-19 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-18.SPEC-001 (Subscribe Screen) | "Start subscription" shows "Processing payment, do not close this page"; after 10 seconds a note appears: "Still working -- this is taking longer than usual." | The button is disabled with "Subscription setup is temporarily unavailable. Try again in a few minutes." Nothing is charged and onboarding stays on this step. | The decline reason is shown inline in plain language: "This card was declined: {reason}. Try another card." No Subscription is created. |
| FEAT-18.SPEC-002 (Billing & Subscription Management Screen -- update payment method) | "Update payment method" shows a progress state; after 10 seconds: "Still working -- this is taking longer than usual." | The action is disabled with "Payment updates are temporarily unavailable. Your current payment method is unchanged. Try again in a few minutes." | The rejection reason is shown inline: "This card could not be confirmed: {reason}. Try another card." Previous payment method remains on file. |
| FEAT-18.SPEC-002 (Billing & Subscription Management Screen -- cancel) | The cancel confirmation shows a progress state after the Pro confirms; after 10 seconds: "Still working -- this is taking longer than usual." | The confirm action is disabled with "Cancellation could not be completed right now. Your subscription is unchanged. Try again in a few minutes." | N/A -- a cancellation request against an existing, confirmed Subscription is not a rejectable operation from the capability's side; there is no decline case for stopping future billing. |
| FEAT-18.SPEC-003 (Subscription Renewal & Payment-Failure Processing) | N/A -- this is a background automation with no user-facing waiting state; a slow renewal charge simply resolves later, and the Subscription remains in its prior state until an outcome arrives | The renewal attempt is treated as inconclusive and retried on the next scheduled attempt within the automation's own retry handling; the Subscription is not moved to Payment Failed on a down-capability alone -- only on a genuine decline outcome | N/A -- "rejects" for a renewal is the same as the failure outcome already defined in FEAT-18.SPEC-003 (sets Payment Failed and starts the grace clock) |

## Consent and Disclosure

- **First subscription disclosure** -- Before Talia's first charge is submitted on FEAT-18.SPEC-001, the screen states plainly: "To start your subscription, your subscription price is charged monthly through an external payment service, which handles your card details directly. Chairtime never sees or stores your card number." This is descriptive text on the screen itself (not a separate consent gate), since entering payment details on the capability's own surface is itself the affirmative action.
- **Payment-method update disclosure** -- The "Update payment method" flow on FEAT-18.SPEC-002 carries the same statement: card details are entered directly with the external payment service and never reach the product's own code.
- **What is never shared** -- The Pro's studio address, sign-in credentials, client data, and every Booking and Client field stay entirely inside the product; only the Subscription's price and a payment-method reference (never the card itself) are exchanged with the capability, as detailed in Data Exchanged.

## Edge Cases

- **Renewal-outcome event arrives for a Subscription that was since cancelled** -- The event is recorded against the Subscription's history but produces no status change and no user feedback: a Cancelled (active to period end) Subscription is not moved back to Active or Payment Failed by a late-arriving renewal event for a charge that should not have been attempted (per FEAT-18.SPEC-003's "No renewal needed" rule).
- **The same renewal-outcome event is delivered twice** -- The second delivery changes nothing: a Subscription already advanced to Active with its new billing date, or already recorded as Payment Failed with its grace deadline set, is not changed again, and no duplicate notification fires.
- **Events arrive out of order (a payment-method-update confirmation arrives after a renewal-failure event for the same billing cycle)** -- The Subscription reflects the most recent event by its own event time, not arrival time: if the payment-method update's event time is later than the renewal failure's, the subsequent renewal retry (FEAT-18.SPEC-003) is what resolves the grace state, not the update confirmation alone.
- **Capability goes down mid-cancellation submission** -- If the cancellation was not confirmed, the Subscription's status is unchanged and FEAT-18.SPEC-002 shows the capability-down message; no half-cancelled state exists.
- **A dispute notice arrives for a subscription charge that has already been superseded by a later renewal** -- The dispute is recorded against the specific charge it names, in the Pro's account activity, without affecting the Subscription's current status; Chairtime never rules on the dispute (SC-17), consistent with how FEAT-16 handles deposit disputes.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-18.SPEC-001 (Subscribe Screen) | Triggered by (inbound) | "Start subscription" initiates the first-charge request |
| FEAT-18.SPEC-001 (Subscribe Screen) | Affects (outbound) | First-charge outcome and degradation states surface here |
| FEAT-18.SPEC-002 (Billing & Subscription Management Screen) | Triggered by (inbound) | Payment-method-update and cancel actions initiate requests |
| FEAT-18.SPEC-002 (Billing & Subscription Management Screen) | Affects (outbound) | Update, cancel, and degradation outcomes surface here |
| FEAT-18.SPEC-003 (Subscription Renewal & Payment-Failure Processing) | Triggers (outbound) | Renewal-outcome inbound events fire this automation |
| FEAT-18.SPEC-003 (Subscription Renewal & Payment-Failure Processing) | Triggered by (inbound) | The automation submits renewal (and retry) charge requests through this integration |
| FEAT-18.SPEC-005 (Subscription Billing Rules) | References (inbound) | Processor-authority contention rule governs how inbound events are resolved against local state |
| FEAT-18.SPEC-007 (Subscription Billing Notifications) | Triggers (outbound) | Renewal outcomes and cancellation confirmation lead to their respective notifications |
| FEAT-15.SPEC-005 (Go-Live Evaluation & Booking Link Activation) -- within FEAT-15 (Pro Onboarding & Setup Wizard) | Triggers (outbound) | First-charge success reports the go-live signal (XBR-26); every Subscription status change to or from Active (the "Subscription status changed" inbound event) is an external-event trigger for that automation |
| FEAT-29 (Pro Sign-In & Account Lifecycle) | Triggered by (inbound) | Account closure (XBR-20) invokes cancellation through this integration |
| FEAT-19 (Platform Support Read-Only Access) | Affects (outbound) | A subscription-charge dispute notice is surfaced to Support during a help request |

## Analytics and Success Signals

- **subscription_first_charge_outcome** (outcome: succeeded / declined) -- supports success-metrics.md: "Subscription Retention"
- **subscription_payment_method_update_outcome** (outcome: succeeded / rejected) -- supports success-metrics.md: "Subscription Retention"
- **subscription_cancelled** (invoked via: self-service / account closure; fires when the payment-processing capability confirms the cancellation) -- supports success-metrics.md: "Subscription Retention"
- **subscription_capability_degradation_shown** (condition: slow / down / rejected; screen: spec ID) -- N/A -- no Stage 2 metric measures degradation frequency directly; retained so the product's tolerance for capability trouble on billing is observable

## Acceptance Criteria

**FEAT-18.SPEC-006-AC-01:** Given Talia submits her card details on the Subscribe Screen, when the capability confirms the first charge, then a Subscription is created with status Active and the go-live signal is reported to FEAT-15 (XBR-26).

**FEAT-18.SPEC-006-AC-02:** Given Talia submits her card details on the Subscribe Screen, when the capability declines the charge, then no Subscription is created and the decline reason is shown inline with a retry action.

**FEAT-18.SPEC-006-AC-03:** Given Talia's subscription reaches its monthly billing date, when the capability confirms the renewal charge, then the outcome is delivered to FEAT-18.SPEC-003 and her next billing date advances.

**FEAT-18.SPEC-006-AC-04:** Given Talia updates her payment method, when the capability confirms the new card, then her payment_method_reference is updated and she sees "Payment method updated."

**FEAT-18.SPEC-006-AC-05:** Given Talia updates her payment method, when the capability rejects the new card, then her previous payment method remains on file and she sees the rejection reason inline with retry.

**FEAT-18.SPEC-006-AC-06:** Given Talia confirms cancellation, when the capability confirms the subscription will stop billing at period end, then her Subscription's status is set to Cancelled (active to period end).

**FEAT-18.SPEC-006-AC-07:** Given a card issuer disputes one of Talia's subscription charges, when the dispute notice arrives, then it is recorded in her account activity and made visible to Support during a help request, without Chairtime ruling on the dispute.

**FEAT-18.SPEC-006-AC-08:** Given Talia taps "Start subscription" while the capability is unavailable, when the request cannot be sent, then the button is disabled with "Subscription setup is temporarily unavailable. Try again in a few minutes." and nothing is charged.

**FEAT-18.SPEC-006-AC-09:** Given Talia attempts to cancel while the capability is unavailable, when the confirm action is submitted, then it is disabled with "Cancellation could not be completed right now. Your subscription is unchanged. Try again in a few minutes."

**FEAT-18.SPEC-006-AC-10:** Given Talia has never subscribed before, when she reaches the payment step, then she sees the disclosure that her card details are handled directly by an external payment service and never stored by Chairtime.

**FEAT-18.SPEC-006-AC-11:** Given a renewal-outcome event arrives for a Subscription that was already cancelled, when the event is processed, then no status change occurs and no user feedback fires.

**FEAT-18.SPEC-006-AC-12:** Given the same renewal-outcome event is delivered twice, when the second delivery arrives, then the Subscription's state does not change again and no duplicate notification fires.

**FEAT-18.SPEC-006-AC-13:** Given a payment-method-update confirmation and a renewal-failure event arrive out of order for the same billing cycle, when both are processed, then the Subscription reflects the most recent event by its own event time, not arrival order.

**FEAT-18.SPEC-006-AC-14:** Given the capability goes down mid-cancellation submission before confirming, when Talia checks her subscription afterward, then its status is unchanged -- no half-cancelled state exists.

**FEAT-18.SPEC-006-AC-15:** Given Talia's Subscription status changes to or from Active (for example, first-charge success or a renewal failure), when the change is recorded, then the change is reported to FEAT-15.SPEC-005 as an external-event trigger and no Pro-facing message is produced by the report itself.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 5 | 5 |
| Inbound Events | 8 | 8 |
| Degradation Paths | 9 (4 screens/automations x 3 conditions = 12 cells; 3 N/A cells excluded from denominator per justified exclusion) | 9 |
| Consent and Disclosure | 3 | 3 |
| Edge Cases | 5 | 5 |



# Notification Spec: Subscription Billing Notifications

## Overview

**Name:** Subscription Billing Notifications
**ID:** FEAT-18.SPEC-007
**Type:** Notification
**Purpose:** Sends Talia the payment-failure grace notice, renewal receipt, cancellation confirmation, and price-change notice, so she is never surprised by her own billing.
**Parent Feature:** FEAT-18 -- Pro Subscription Billing & Account Management

## Scope and Non-Goals

**In Scope:**
- The payment-failure grace notice, with the exact grace deadline
- The renewal receipt, sent on every successful monthly charge
- The cancellation confirmation, with the exact period-end date
- The price-change notice, sent at least 30 days (platform parameter: `subscription-price-change-notice-days`) before a price change applies

**Non-Goals:**
- Deciding when a renewal succeeds, fails, or a grace period expires -- owned by FEAT-18.SPEC-003 (Subscription Renewal & Payment-Failure Processing); this spec begins where that automation's outcome fires
- The Pro Account pause itself -- owned by FEAT-18.SPEC-004; a pause is not separately notified by this spec beyond what the payment-failure grace notice already covers, since the grace notice is the Pro's warning before any pause occurs
- Notifying clients about anything related to the Pro's own billing -- excluded per the Access Matrix: clients have no visibility into Subscription & Billing at all; nothing in this spec is ever sent to a Client
- The underlying text/email delivery mechanics (send, retry, delivery-status reporting) -- owned by FEAT-08.SPEC-012 (text) and FEAT-08.SPEC-013 (email), per the dependency map's External Touchpoints table; this spec defines only the content, audience, and delivery rules specific to these four messages

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always, for all four notices | Talia checks Chairtime reactively throughout her working day; an in-app notice is always available on her next open, matching her behavioral context |
| Text | Always, when Talia's notification preferences (FEAT-27) include text delivery for Pro notifications | Talia works with her phone in hand between clients; a billing issue -- especially the payment-failure grace notice -- needs to reach her promptly even when she is not inside the app |
| Email | Always, as the fallback when text is not enabled in her preferences, and always in addition to text for the renewal receipt and cancellation confirmation (financial records worth keeping in an inbox) | Per BRIEF.md's stated fallback pattern (ASMP-32) and because a receipt or cancellation confirmation is the kind of record a Pro may want to search for later, unlike a transient reminder |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Renewal charge fails | FEAT-18.SPEC-003 (Subscription Renewal & Payment-Failure Processing) | Fires when a renewal charge is confirmed failed and the grace period starts | Subscription reference, grace deadline, price |
| Renewal charge succeeds | FEAT-18.SPEC-003 (Subscription Renewal & Payment-Failure Processing) | Fires on every confirmed successful renewal charge | Subscription reference, price charged, new next_billing_date |
| Cancellation confirmed | FEAT-18.SPEC-006 (Subscription Billing Integration) | Fires when the payment-processing capability confirms the subscription will stop billing at period end | Subscription reference, period-end date |
| Price change decided | Product-level decision (FEAT-18.SPEC-005's price-change notice rule) | Fires when a future price change is decided and must be announced at least the fixed notice window ahead | Subscription reference, current price, new price, effective date |

## Audience and Preferences

**Recipients:** The Pro (Talia) -- the sole recipient of every notice in this spec, traced to the Access Matrix (Subscription & Billing = Full for the Pro, None for the Client, View for Platform Operator Support). Support never receives these notices; Support's visibility into billing is the read-only screen (FEAT-18.SPEC-002), not a notification feed.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Pro notification channels (text / email / in-app) | Any combination, in-app always on | Text + in-app (email as fallback if texting is unavailable) | FEAT-27 (Pro Profile & Booking Page Settings, notification preferences) |

There is no separate off switch for these four billing notices: per FEAT-18.SPEC-005's business rules, a payment failure, a successful renewal, a cancellation, and a price change are each consequential enough to Talia's business that the in-app copy is always delivered regardless of her channel preferences; only the choice of text vs. email as the secondary channel is hers to set.

**Quiet Hours:** N/A -- these are billing-status notices tied to financial events (a failed charge, a successful charge, a cancellation, a price change), not time-sensitive reminders bound to a daily schedule; XBR-16's quiet-hours window governs client-facing appointment reminders, not the Pro's own billing notices, so none of the four messages in this spec is held for quiet hours.

## Content Definition

**Payment-Failure Grace Notice:**
- **In-app title:** Payment failed -- update by {grace_deadline}
- **In-app body:** Your subscription payment didn't go through. Update your payment method by {grace_deadline} to keep your booking link live.
- **Text:** "Chairtime: your subscription payment failed. Update your payment method by {grace_deadline} to keep taking bookings: {manage_link}"
- **Email subject:** Action needed: update your Chairtime payment method by {grace_deadline}
- **Email body:**
  Hi {pro_first_name},

  Your latest subscription payment didn't go through. To keep your booking link live and taking new bookings, update your payment method by {grace_deadline}.

  Your existing bookings, reminders, and client access are unaffected either way.
- **CTA:** Update payment method -- deep-links to FEAT-18.SPEC-002 (Billing & Subscription Management Screen)

**Renewal Receipt:**
- **In-app title:** Subscription renewed
- **In-app body:** Your {price} monthly payment went through. Next billing date: {next_billing_date}.
- **Email subject:** Your Chairtime subscription receipt
- **Email body:**
  Hi {pro_first_name},

  This confirms your Chairtime subscription renewed successfully.

  Amount charged: {price}
  Next billing date: {next_billing_date}

  Chairtime takes nothing from your deposits, balances, or tips -- this is the only charge on your account.
- **CTA:** View billing -- deep-links to FEAT-18.SPEC-002 (Billing & Subscription Management Screen)

**Cancellation Confirmation:**
- **In-app title:** Subscription cancelled
- **In-app body:** Your subscription is cancelled. It stays active through {period_end_date}, then your booking link pauses.
- **Text:** "Chairtime: your subscription is cancelled, active through {period_end_date}. Your booking link pauses after that."
- **Email subject:** Your Chairtime subscription is cancelled
- **Email body:**
  Hi {pro_first_name},

  This confirms your Chairtime subscription is cancelled. It remains active through {period_end_date} -- you keep full use of your booking link until then.

  After {period_end_date}, your booking link will pause to new bookings. You can resubscribe at any time.
- **CTA:** View billing -- deep-links to FEAT-18.SPEC-002 (Billing & Subscription Management Screen)

**Price-Change Notice:**
- **In-app title:** Your subscription price is changing on {effective_date}
- **In-app body:** Starting {effective_date}, your monthly price changes from {current_price} to {new_price}.
- **Text:** "Chairtime: your subscription price changes from {current_price} to {new_price} starting {effective_date}."
- **Email subject:** Your Chairtime subscription price is changing
- **Email body:**
  Hi {pro_first_name},

  We're writing to let you know your Chairtime subscription price will change from {current_price} to {new_price}, starting {effective_date}.

  This is your only notice before the change applies -- at least {subscription-price-change-notice-days} days ahead, as always. Everything else about your subscription stays the same.
- **CTA:** View billing -- deep-links to FEAT-18.SPEC-002 (Billing & Subscription Management Screen)

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {pro_first_name} | Pro Account -- display_name (first token) | Talia | Greeting renders as "Hi," |
| {grace_deadline} | Subscription -- derived, failure date plus platform parameter: `subscription-payment-failure-grace-period-days` | March 14 | Never empty -- computed at the moment the grace notice fires, per FEAT-18.SPEC-003 |
| {price} | Subscription -- price (platform parameter: `subscription-price`) | $39/month (illustrative only) | Never empty -- one price for every Pro |
| {next_billing_date} | Subscription -- next_billing_date | April 14 | Never empty -- set by the renewal automation before this notice fires |
| {period_end_date} | Subscription -- derived, the date the already-paid period ends | May 1 | Never empty -- computed by FEAT-18.SPEC-005's cancellation-timing rule before this notice fires |
| {manage_link} | Access reference into FEAT-18.SPEC-002, delivered via FEAT-08.SPEC-012/FEAT-08.SPEC-013 | (rendered as a tappable link) | Never empty -- generated at send time |
| {current_price} / {new_price} | Subscription -- price (before) / product-level decided new price (both platform parameter: `subscription-price`, before and after the change) | $39/month / $45/month | Never empty -- both values are fixed at the moment the price change is decided |
| {effective_date} | Product-level decision -- the date the new price applies | May 1 | Never empty -- set when the price change is decided, at least platform parameter: `subscription-price-change-notice-days` in advance |

## Delivery Rules

**Batching:** None of these four notices are ever batched together or with any other notification -- each is a distinct, individually consequential financial event and is delivered as its own message the moment its trigger fires.
**Deduplication:** At most one instance of each notice per triggering event. A renewal-outcome event delivered twice (per FEAT-18.SPEC-006's Edge Cases) produces no second notice, since the underlying Subscription state does not change on the duplicate delivery. A price-change notice is sent exactly once per decided change, keyed to the specific effective_date.
**Retry on failure:** Text delivery failure is retried per platform parameter: `message-delivery-retry-count`, then falls back to email as the delivery of record, consistent with FEAT-08.SPEC-009's product-wide retry-and-fallback rule; the in-app notice always stands regardless of text/email outcome and is never itself retried (it is delivered the next time Talia opens the product).
**Expiry:** None of these four notices expire undelivered in the ordinary sense -- the in-app notice remains visible until Talia views it, and the underlying billing state it describes (grace deadline, cancellation date, price-change date) is also always visible on FEAT-18.SPEC-002, so a delayed text or email never leaves Talia with no way to learn the same information.

## Edge Cases

- **The underlying Subscription changes state before an already-queued notice is delivered (e.g., Talia updates her payment method moments after the payment-failure notice fires, before the text is sent)** -- The already-triggered payment-failure grace notice is still delivered as queued (it accurately reported the failure that just happened); no separate "never mind" message is sent, but the subsequent renewal-recovered outcome (FEAT-18.SPEC-003) triggers no confusing follow-up beyond the ordinary path -- if the retry succeeds, no renewal receipt is sent for a retry succeeding during grace in this spec's inventory (the recovered state is reflected on FEAT-18.SPEC-002 directly), avoiding a contradictory pair of messages.
- **Talia's notification preferences (FEAT-27) are set to no channels beyond in-app between trigger and delivery** -- Per Audience and Preferences, the in-app copy always delivers regardless of her channel preference; only text vs. email as the secondary channel is affected, so she never receives zero notice.
- **A price-change notice and a payment-failure grace notice are both pending for the same Subscription at the same time** -- Each is delivered independently, per FEAT-18.SPEC-005's business rule that these two timers do not interact; Talia may receive both notices in the same period without either being suppressed.
- **The renewal receipt's amount does not match what Talia expects because a price change took effect since her last renewal** -- The receipt always reflects the price actually charged, which is the Subscription's own price field as updated by FEAT-18.SPEC-003 on the price change's effective_date (to the new platform parameter: `subscription-price` value); because that effective_date always falls at least the required notice window after this notice was sent, the new amount on the receipt was never a surprise by the time it was first charged.
- **Cancellation confirmation is triggered by account closure (FEAT-29, XBR-20) rather than Talia's own action on FEAT-18.SPEC-002** -- The same cancellation confirmation content and delivery rules apply regardless of which flow invoked the cancellation; the notice's CTA still deep-links to FEAT-18.SPEC-002, even though Talia is mid-closure, since the billing screen remains the accurate source of her subscription's final status.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-18.SPEC-003 (Subscription Renewal & Payment-Failure Processing) | Triggered by (inbound) | Renewal failure and renewal success each fire their respective notice |
| FEAT-18.SPEC-006 (Subscription Billing Integration) | Triggered by (inbound) | Cancellation confirmation fires the cancellation notice |
| FEAT-18.SPEC-005 (Subscription Billing Rules) | Triggered by (inbound) | The price-change notice rule fires the price-change notice |
| FEAT-18.SPEC-002 (Billing & Subscription Management Screen) | Navigation (outbound) | Every notice's CTA deep-links here |
| FEAT-27 (Pro Profile & Booking Page Settings) | References (inbound) | The Pro notification channel preference governs the secondary channel |
| FEAT-08.SPEC-012 (Transactional Text Messaging Capability) | Triggers (outbound) | Carries the text variant of each notice |
| FEAT-08.SPEC-013 (Transactional Email Capability) | Triggers (outbound) | Carries the email variant of each notice, and the fallback when text is not enabled |

## Analytics and Success Signals

- **billing_notification_delivered** (notice_type: payment_failure_grace / renewal_receipt / cancellation_confirmation / price_change; channel) -- supports success-metrics.md: "Subscription Retention"
- **billing_notification_cta_tapped** (notice_type; destination: FEAT-18.SPEC-002) -- supports success-metrics.md: "Subscription Retention"
- **payment_failure_grace_notice_to_recovery_time** (days between notice delivery and payment method resolution, where resolved) -- supports success-metrics.md: "Subscription Retention"

## Acceptance Criteria

**FEAT-18.SPEC-007-AC-01:** Given Talia's renewal charge fails, when the grace period starts, then she receives the payment-failure grace notice in-app and on her preferred secondary channel, stating the exact grace deadline.

**FEAT-18.SPEC-007-AC-02:** Given Talia's renewal charge succeeds, when the charge is confirmed, then she receives the renewal receipt in-app and by email, stating the exact amount charged and her next billing date.

**FEAT-18.SPEC-007-AC-03:** Given Talia confirms cancellation, when the cancellation is confirmed by the payment-processing capability, then she receives the cancellation confirmation stating the exact date her subscription remains active through.

**FEAT-18.SPEC-007-AC-04:** Given a price change is decided for Talia's subscription, when the notice fires, then she receives it at least the fixed notice window before the new price applies, stating the current price, new price, and effective date.

**FEAT-18.SPEC-007-AC-05:** Given Talia has set her notification channels to email only, when a payment failure occurs, then she still receives the in-app notice and the email notice, with no text sent.

**FEAT-18.SPEC-007-AC-06:** Given Talia taps the CTA on any of these four notices, when she taps "View billing" or "Update payment method", then she lands on FEAT-18.SPEC-002 (Billing & Subscription Management Screen).

**FEAT-18.SPEC-007-AC-07:** Given a renewal-outcome event is delivered twice for the same charge, when the second delivery arrives, then no duplicate notice is sent.

**FEAT-18.SPEC-007-AC-08:** Given Talia's text delivery of the payment-failure grace notice fails, when the retry attempts are exhausted, then the notice falls back to email as the delivery of record, and the in-app notice still stands regardless.

**FEAT-18.SPEC-007-AC-09:** Given Talia has both a pending price-change notice and an active payment-failure grace period, when each notice's own trigger fires, then both are delivered independently without either being suppressed.

**FEAT-18.SPEC-007-AC-10:** Given Talia's account closure (FEAT-29) invokes cancellation rather than her own action on FEAT-18.SPEC-002, when the cancellation is confirmed, then she still receives the same cancellation confirmation content with a CTA back to FEAT-18.SPEC-002.

**FEAT-18.SPEC-007-AC-11:** Given a support operator is viewing a Pro's account, when any of these four notices fires, then it is never sent to Support -- these notices are delivered only to the Pro.

**FEAT-18.SPEC-007-AC-12:** Given Talia's subscription renewal fails at a moment that falls late in the day, when the payment-failure grace notice fires, then it is delivered immediately with no quiet-hours hold, since these billing notices are exempt from the appointment-reminder quiet-hours window (XBR-16 governs client reminders, not Pro billing notices).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 3 (in-app, text, email) | 3 |
| Trigger Paths | 4 | 4 |
| Preference States | 2 (default text+in-app, email-only fallback) | 2 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |


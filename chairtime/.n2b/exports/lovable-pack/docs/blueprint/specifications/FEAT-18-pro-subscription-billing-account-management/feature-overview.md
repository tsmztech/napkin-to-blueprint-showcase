---
document_type: feature-overview
feature_number: FEAT-18
feature_name: Pro Subscription Billing & Account Management
feature_slug: pro-subscription-billing-account-management
priority_tier: Important
feature_type: Lifecycle
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 7
screen_count: 2
automation_count: 2
logic_rule_count: 1
integration_count: 1
notification_count: 1
---

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

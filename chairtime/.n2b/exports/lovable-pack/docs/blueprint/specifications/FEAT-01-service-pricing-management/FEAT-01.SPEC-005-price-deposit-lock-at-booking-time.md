---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-01.SPEC-005
spec_name: Price & Deposit Lock at Booking Time
spec_slug: price-deposit-lock-at-booking-time
parent_feature: FEAT-01
parent_feature_name: Service & Pricing Management
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-26
rule_count: 9
acceptance_criteria_count: 14
---

# Logic/Rule Spec: Price & Deposit Lock at Booking Time

## Overview

**Name:** Price & Deposit Lock at Booking Time
**ID:** FEAT-01.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs that editing or archiving a service never changes the price, duration, or deposit already agreed on a confirmed booking -- this feature's elaboration of XBR-04.
**Parent Feature:** FEAT-01 -- Service & Pricing Management
**Governed Entity:** Service (specifically, the propagation behavior of each Service field toward Bookings that already reference it)

## Scope and Non-Goals

**In Scope:**
- Defining, per Service field, whether an edit to it retroactively affects a Booking that already references that service
- Defining that Booking.price_agreed, Booking.duration, and Booking.deposit_amount are captured once, at booking time, and never recalculated afterward
- Defining that archiving a service never alters, cancels, or hides any existing Booking that references it
- Authorization over the one action this spec exists to forbid: manually overriding a confirmed booking's locked values

**Non-Goals:**
- Field-format validation of Service's own fields (required, length, range, the minimum chargeable deposit) -- fully owned by FEAT-01.SPEC-004; this spec assumes those rules already passed
- Counting or warning about upcoming bookings before an archive is confirmed -- that check is FEAT-01.SPEC-006 (Archive Impact Check); this spec defines what happens to those bookings' data once the archive completes, not whether the Pro is warned first
- Computing the deposit amount from a service's deposit rule -- that computation belongs to the payment-processing capability behind FEAT-07; this spec only asserts that, once computed and captured on a Booking, it is never recalculated by a later Service edit
- Partial refunds or tiered cancellation schedules -- excluded per scope-boundaries.md SC-18: this spec concerns price/deposit immutability, not refund percentages, which stay a binary rule owned by FEAT-09

## Governed Entity

**Entity:** Service (this spec's rules concern each field's propagation behavior toward referencing Bookings; the Service field list itself is defined in FEAT-01.SPEC-004 and is not repeated here except as the subject of each propagation rule)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| name | text | The service's client-facing name |
| price | number | The fixed price of the service, in the Pro Account's currency |
| duration | number | The service's fixed length, in minutes |
| deposit_rule | enum + number | Fixed amount or percentage deposit rule |
| buffer_override | number (optional) | Owned by FEAT-02; not addressed by this feature |
| display_order | number | The service's position on the booking page |
| status | enum (Active \| Archived) | The service's current lifecycle state |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-01.SPEC-003 | Edit Service | On every field save -- the save never touches any existing Booking's own copied fields |
| FEAT-01.SPEC-003 / FEAT-01.SPEC-006 | Edit Service / Archive Impact Check | On archive confirmation -- the status change never alters, cancels, or hides any existing Booking |
| FEAT-05 (Public Booking Page & Booking Flow), FEAT-07 (Deposit Payment at Booking) | -- (cross-feature) | At the moment a Booking is created and confirmed, Service's then-current price, duration, and deposit rule are read once and copied onto the Booking; this spec's guarantee begins from that moment |
| FEAT-12 (Pro Daily Schedule Dashboard), FEAT-16 (Booking & Payment Activity Record), FEAT-30 (Pro Booking Management) | -- (cross-feature) | Every surface that displays a Booking's price, duration, or deposit reads the Booking's own locked copy, never the Service's current live value |

## Field Validation Rules

{This spec governs propagation behavior, not field format -- format rules for every Service field live in FEAT-01.SPEC-004. For each field, this table states whether an edit propagates retroactively to an existing Booking.}

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| name | Does not propagate a booking-time lock -- a service rename is reflected wherever its name is shown, including on existing bookings, since the product definition locks only price, duration, and deposit (XBR-04) | Always | Not applicable -- there is no invalid state to block | N/A | No |
| price | Locked -- editing price never changes Booking.price_agreed on any Booking already at Confirmed or later in its lifecycle | Always, from the moment a Booking reaches Confirmed | Enforced structurally: no save path exists that writes to a Booking's price_agreed from a Service edit | N/A -- there is no user-facing error; the lock is a structural guarantee, not a validation the Pro can trigger or fail | Yes (as a structural block, not a form error) |
| duration | Locked -- editing duration never changes Booking.duration on any Booking already at Confirmed or later | Always, from the moment a Booking reaches Confirmed | Enforced structurally, as above | N/A | Yes (structural) |
| deposit_rule | Locked -- editing the deposit rule never recomputes Booking.deposit_amount on any Booking already at Confirmed or later | Always, from the moment a Booking reaches Confirmed | Enforced structurally, as above | N/A | Yes (structural) |
| buffer_override | No propagation concern for this feature -- owned entirely by FEAT-02; this spec makes no claim about its effect on availability computation | Always | -- | -- | -- |
| display_order | No propagation concern -- display-only ordering field, never referenced by any Booking | Always | -- | -- | -- |
| status | Locked in the honoring sense -- setting status to Archived never cancels, hides, or alters any existing Booking that references the service; the archived service continues to display correctly on every past and upcoming Booking that references it | Always | Enforced structurally at archive time (FEAT-01.SPEC-003/FEAT-01.SPEC-006) | N/A | Yes (structural) |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Simultaneous edit has no compounding effect | price, deposit_rule, duration | Editing more than one locked field in the same save has no greater retroactive effect than editing one -- all three remain independently locked on any Confirmed-or-later Booking regardless of how many are changed together | N/A -- structural guarantee, no error state |
| Archive plus prior edit | status, price, duration, deposit_rule | A service that was edited and later archived carries both changes forward identically: existing Bookings keep the values that were locked in at their own booking time, regardless of how many edits or an eventual archive followed | N/A -- structural guarantee, no error state |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Manually override a Confirmed-or-later booking's locked price, duration, or deposit | Nobody (Never) -- not even the Pro | N/A -- no condition grants this | No control exists anywhere in the product to alter these fields on a Booking once Confirmed; every surface that displays them (FEAT-12, FEAT-16, FEAT-30) renders them read-only |
| Edit a service's price, duration, or deposit rule going forward (future bookings only) | The Pro | Always, for their own account's services | -- |
| Edit a service's price, duration, or deposit rule going forward | Platform Operator (Support) | Never | No edit control is rendered for this role (see FEAT-01.SPEC-004) |
| Edit a service's price, duration, or deposit rule going forward | The Client | Never | Not reachable, as defined in FEAT-01.SPEC-001/002/003 |
| Archive a service that has upcoming bookings | The Pro | Allowed after acknowledging the impact warning from FEAT-01.SPEC-006 (or immediately, if no upcoming bookings exist) | -- |
| Archive a service that has upcoming bookings | Platform Operator (Support) | Never | No Archive control is rendered for this role |
| Archive a service that has upcoming bookings | The Client | Never | Not reachable, as defined above |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Booking.price_agreed | Copied once from Service.price at the moment the Booking is confirmed | On Booking confirmation only (owned by FEAT-05/FEAT-07) | No -- never recalculated by a later Service edit; no Pro or Client override exists |
| Booking.duration | Copied once from Service.duration at the moment the Booking is confirmed | On Booking confirmation only | No -- never recalculated by a later Service edit |
| Booking.deposit_amount | Computed once from Service.price and Service.deposit_rule at the moment the Booking is confirmed (computation owned by FEAT-07) | On Booking confirmation only | No -- never recalculated by a later Service edit |

## Business Rules

- XBR-04 (owned by FEAT-01): service edits and archiving apply to future bookings only; confirmed bookings keep the price, duration, and deposit agreed at booking. This spec is that rule's full elaboration for the Service side.
- XBR-11: setup changes -- including a Service edit or archive -- never silently cancel a confirmed booking; any resulting conflict (for example, an archived service with upcoming bookings) is flagged on the Pro's booking management surface (FEAT-30) and resolved only by an explicit Pro choice there. This spec never auto-resolves such a conflict itself.
- This spec's guarantee begins at Confirmed and holds for every later Booking state (Awaiting Outcome, Completed, No-Show, Cancelled by Client, Cancelled by Pro, Rescheduled) -- once locked, a Booking's price, duration, and deposit are never unlocked by any later state transition either.
- A Booking that is Rescheduled (FEAT-10, FEAT-30) is still the same Booking record with the same locked price and deposit; only its time (and, per FEAT-30, potentially other fields those features own) changes -- rescheduling is never a mechanism for re-pricing.
- FEAT-01.SPEC-006's impact check runs before an archive is confirmed, but the lock this spec defines applies regardless of whether that check ran or what it found -- the lock is unconditional on every Confirmed-or-later Booking, not contingent on the Pro having been warned.

## Edge Cases

- **Client is mid-checkout (Booking state Pending Payment, not yet Confirmed) when the Pro edits the service's price** -- This spec's lock applies from Confirmed onward, not before. A Pending Payment attempt reflects the Service's values at the moment the payment step reads them (owned by FEAT-05/FEAT-07); if the price changes before payment completes, the client's outcome for that not-yet-confirmed attempt is defined by those features, not by this spec's guarantee.
- **Pro archives a service while a client is mid-checkout for it** -- Per the dependency map's Service Contention note: if the service is archived before the client's payment completes, the client is refused with a refresh back to the service list; no Booking was ever confirmed for that attempt, so this spec's lock never applied to it.
- **Service archived while a referencing Booking is later Completed or marked No-Show** -- The Booking's locked price, duration, and deposit remain exactly as captured at booking time; archiving has no effect on already-locked data regardless of the Booking's subsequent state.
- **Two Pro edits to the same service's price in quick succession, both before any new booking is confirmed against either version** -- Whichever edit is saved last (per the dependency map's Service Contention note: last-write-wins between the Pro's own sessions) is the value read the next time a Booking is confirmed; this has no bearing on any Booking already locked before either edit.
- **A goodwill refund (FEAT-30) is issued against a locked deposit** -- The refund changes the Deposit Transaction's outcome (owned by FEAT-09/FEAT-30), never the Booking's locked deposit_amount figure itself, which remains the historical record of what was agreed.
- **Support views a booking's locked price during a help request** -- Support sees the same read-only locked value the Pro sees; Support has no path, and never will, to alter it (Access Matrix: Booking & Payment = View for Support).

## Acceptance Criteria

**FEAT-01.SPEC-005-AC-01:** Given Talia has a Confirmed booking for a service priced at $80, when she later edits that service's price to $100, then the existing booking's price_agreed remains $80.

**FEAT-01.SPEC-005-AC-02:** Given Talia has a Confirmed booking for a 60-minute service, when she later edits that service's duration to 90 minutes, then the existing booking's duration remains 60 minutes.

**FEAT-01.SPEC-005-AC-03:** Given Talia has a Confirmed booking whose deposit was computed under a 20% deposit rule, when she later changes the service's deposit rule to a fixed amount, then the existing booking's deposit_amount is unchanged.

**FEAT-01.SPEC-005-AC-04:** Given Talia renames a service from "Classic Set" to "Signature Set," when the rename saves, then every existing booking referencing that service now shows "Signature Set" as its service name, since name is not a locked field.

**FEAT-01.SPEC-005-AC-05:** Given Talia archives a service that has three upcoming Confirmed bookings, when the archive completes, then all three bookings remain Confirmed with their original price, duration, and deposit intact, and none is cancelled or hidden from the Pro's schedule.

**FEAT-01.SPEC-005-AC-06:** Given Talia (the Pro) looks for any way to manually change a Confirmed booking's locked price on her dashboard, then no such control exists anywhere in the product -- the field renders read-only on FEAT-12, FEAT-16, and FEAT-30.

**FEAT-01.SPEC-005-AC-07:** Given Riley (the Client) has a Confirmed booking, when the Pro edits or archives the underlying service, then Riley's booking confirmation and reminder continue to show the original price and deposit she agreed to, unchanged.

**FEAT-01.SPEC-005-AC-08:** Given a client is mid-checkout for a service (Pending Payment, not yet Confirmed) and the Pro edits its price before payment completes, then this spec's lock does not yet apply to that attempt, since no Booking has reached Confirmed.

**FEAT-01.SPEC-005-AC-09:** Given a client is mid-checkout for a service and the Pro archives it before payment completes, then the client's payment attempt is refused and they are returned to the service list, per the dependency map's Service Contention note.

**FEAT-01.SPEC-005-AC-10:** Given Talia edits both the price and the deposit rule of a service in the same save, when an existing Confirmed booking already references it, then that booking's price_agreed and deposit_amount both remain exactly as they were at booking time.

**FEAT-01.SPEC-005-AC-11:** Given a booking referencing an archived service later reaches Completed, when Talia views its history, then the price, duration, and deposit shown are exactly what was locked in at booking time, unaffected by the archive.

**FEAT-01.SPEC-005-AC-12:** Given Talia rebooks a rescheduled appointment (FEAT-10/FEAT-30) that keeps the same booking record, then its already-locked price and deposit are unchanged by the reschedule; only its time changes.

**FEAT-01.SPEC-005-AC-13:** Given Platform Operator (Support) views a Confirmed booking's locked price during a help request, when Support looks for any way to edit it, then no edit control is rendered, consistent with Support's View-only access to Booking & Payment.

**FEAT-01.SPEC-005-AC-14:** Given Talia edits a service's price twice from two different devices in quick succession, when both saves complete, then the later-completing save's price is the value used for any Booking confirmed afterward, per the dependency map's last-write-wins resolution -- and this has no effect on any booking already locked before either edit.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 7 | 7 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 6 | 6 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

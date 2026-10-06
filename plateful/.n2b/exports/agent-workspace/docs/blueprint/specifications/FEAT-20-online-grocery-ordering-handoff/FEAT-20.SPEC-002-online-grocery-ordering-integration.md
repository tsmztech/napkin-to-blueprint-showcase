---
document_type: spec
spec_type: integration
spec_id: FEAT-20.SPEC-002
spec_name: Online Grocery Ordering Integration
spec_slug: online-grocery-ordering-integration
parent_feature: FEAT-20
parent_feature_name: Online Grocery Ordering Handoff
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 15
---

# Integration Spec: Online Grocery Ordering Integration

## Overview

**Name:** Online Grocery Ordering Integration
**ID:** FEAT-20.SPEC-002
**Type:** Integration
**Purpose:** The product sends the household's current grocery list to an online grocery-ordering capability for fulfillment and receives back acceptance, rejection, or per-item availability.
**Parent Feature:** FEAT-20 -- Online Grocery Ordering Handoff

## Scope and Non-Goals

**In Scope:**
- Submitting the household's current unticked Grocery List Items to the online-ordering capability when the household confirms a handoff
- Receiving the capability's outcome (full acceptance, partial acceptance with unavailable items, or rejection) and passing it to FEAT-20.SPEC-001 for display
- User-facing behavior when the capability is slow, unavailable, or rejects a request
- Disclosure to the household about what grocery list data is shared with the capability

**Non-Goals:**
- Choosing the online-ordering vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate, and Stage 2-3 documents name only the category-level capability (feature-overview.md's Non-Goals, functional-language-only)
- Selling or fulfilling grocery orders within the product -- excluded per scope-boundaries.md (SC-07): the product plans and lists groceries; it hands the list off to an external capability rather than selling or shipping ingredients itself
- Delivery tracking or order-status updates after the capability accepts the list -- excluded per feature-overview.md's Non-Goals: once the capability has accepted the list, further order lifecycle happens entirely within that external capability
- The screen mechanics of the Handoff Screen -- owned by FEAT-20.SPEC-001; this spec defines only the capability behavior that screen surfaces
- Collecting a delivery address or payment details on the household's behalf -- feature-overview.md's Connected Entities name only Grocery List and Grocery List Item as data this feature reads; the Household entity carries no delivery-address field in the dependency map, so any address or payment collection needed for fulfillment happens entirely within the online-ordering capability, outside this product's boundary

## Capability Category

**Category:** Online grocery ordering
**Dependency Source:** ASMP-37 -- "Online grocery-ordering and family-calendar capabilities (Later) -- Required only for Online Grocery Ordering Handoff (FEAT-20) and Family Calendar Sync (FEAT-21); the core product works fully without them." (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Online grocery ordering (ASMP-37, Later)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-20)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision; BRIEF.md's Ecosystem & Integrations section names only the category, and feature-overview.md's Rationale notes named retailers are explicitly left to Stage 4.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Maya or Sam sends the household's current unticked Grocery List items to the online-ordering capability for fulfillment | Hand off the list | FEAT-20.SPEC-001 (Grocery Handoff Screen) |
| The household sees whether the online-ordering capability accepted, partially accepted, or rejected the handoff, with any unavailable items called out | See handoff status | FEAT-20.SPEC-001 (Grocery Handoff Screen) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Grocery list items to fulfill | Grocery List Item -- ingredient_name, quantity_and_unit, aisle (unticked items only, as of the confirm-time snapshot) | Household confirms the handoff on FEAT-20.SPEC-001 | The capability needs the exact items, quantities, and groupings to source and fulfill the order; quantity_and_unit already carries each item's own unit as recorded on the Grocery List Item, so the capability interprets quantities per item with no separate household-level unit or currency field needed |
| Handoff attempt reference | An opaque, single-use reference generated for this attempt (not a persisted entity field) | Household confirms the handoff | Lets the capability's response be matched back to this specific attempt, since no handoff entity is stored on the product side to look the attempt up later |

Household member identities and contact details, delivery address, payment information, dietary rules and allergy data, household-level unit-system and currency settings, and every Grocery List Item beyond the fields above (e.g., ticked state, origin, "added by") never leave the product -- this feature's declared dependencies (feature-overview.md's Referenced Entities, product-features.md's FEAT-20 Connected Entities) name only Grocery List and Grocery List Item, so no Household-entity field is exchanged with the external capability. Any pricing the capability shows back to the household is presented in whatever terms the capability itself determines; this feature neither supplies nor governs that presentation.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Full acceptance | The capability accepts every submitted item | Nothing persisted -- FEAT-20.SPEC-001's transient result state is set to Success (full), since this feature persists no handoff entity (feature-overview.md's Entity-Lifecycle Coverage Matrix) |
| Partial acceptance with unavailable item names | The capability accepts the list but marks one or more submitted items unavailable | Nothing persisted -- FEAT-20.SPEC-001's transient result state is set to Success (partial), carrying the unavailable item names for display |
| Rejection | The capability declines the submitted list outright | Nothing persisted -- FEAT-20.SPEC-001's transient result state is set to Failure |
| Failure / timeout | The request cannot be completed (capability unreachable, or no response within its normal response window) | Nothing persisted -- FEAT-20.SPEC-001's transient result state is set to Failure |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Handoff accepted (full) | The capability accepts every item in the submitted snapshot | None persisted; transient result set to Success (full) | FEAT-20.SPEC-001 shows "Your list is on its way" with no unavailable-items list | FEAT-20.SPEC-001 |
| Handoff accepted (partial) | The capability accepts the list but marks one or more submitted items unavailable | None persisted; transient result set to Success (partial), carrying the unavailable item names | FEAT-20.SPEC-001 shows "Your list is on its way" plus "Still need these from the store:" listing each unavailable item by name | FEAT-20.SPEC-001 |
| Handoff rejected | The capability declines the submitted list outright | None persisted; transient result set to Failure | FEAT-20.SPEC-001 shows "We couldn't hand off your list to online ordering. Nothing has changed -- your grocery list is exactly as it was." with Retry and Return to In-App Shopping | FEAT-20.SPEC-001 |
| Handoff call fails or times out | The capability is unreachable, or does not respond within its normal response window | None persisted; transient result set to Failure | FEAT-20.SPEC-001 shows the same failure message and options as a rejection | FEAT-20.SPEC-001 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-20.SPEC-001 (Grocery Handoff Screen) | The Confirming state's progress indicator continues past its normal duration; after a brief wait, a note appears: "Still working -- this can take a moment." The Grocery List itself remains unaffected and fully usable elsewhere. | The request is not sent. The screen shows "Online grocery ordering isn't available right now. Your grocery list is safe -- try again later." with Retry and Return to In-App Shopping; the Grocery List is unchanged. | The screen shows "We couldn't hand off your list to online ordering. Nothing has changed -- your grocery list is exactly as it was." with Retry and Return to In-App Shopping; the Grocery List is unchanged. |

## Consent and Disclosure

- **First-handoff disclosure** -- The first time any household member confirms a handoff, a notice appears before the request is sent: "Handing off your list shares your grocery list items, quantities, and aisle groupings with an external online-ordering service so it can fulfill your order. Your other household data stays private." with "Continue" and "Cancel" options. Shown once per household; afterwards a "How this data is shared" link on FEAT-20.SPEC-001 reopens the same wording on demand.
- **What is never shared** -- Household member identities and contact details, delivery address, payment information, dietary rules and allergy data, and every Grocery List Item field beyond ingredient name, quantity/unit, and aisle. This boundary is stated in the first-handoff disclosure notice and in the "How this data is shared" link's content.

## Edge Cases

- **A response arrives for an attempt the household has already left or retried** -- It is discarded; only the currently open attempt (if any) reflects a result, since no handoff attempt is persisted for later lookup.
- **The same acceptance event is delivered twice for one handoff attempt** -- The second delivery changes nothing further; the screen already reflects the result and no duplicate confirmation is shown.
- **A rejection and an earlier partial-acceptance event both concern the same attempt (out-of-order arrival)** -- Exactly one outcome event is expected per attempt, since each handoff is a single request/response pair; if two nonetheless arrive, the first to reach the still-open FEAT-20.SPEC-001 is the one displayed, and the later one is discarded.
- **The capability goes down after a request has been sent but before any response is confirmed** -- The screen shows the capability-down message from Degradation Behavior; no local record reflects a half-completed handoff, since no handoff entity is ever persisted on either outcome.
- **Some Grocery List Items are already ticked at the moment the household confirms the handoff** -- Only unticked items are included in the submitted snapshot; ticked items are treated as already resolved and are excluded from what leaves the product.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-20.SPEC-001 (Grocery Handoff Screen) | Triggered by (inbound) | Confirm Handoff and Retry both initiate a request through this integration |
| FEAT-20.SPEC-001 (Grocery Handoff Screen) | Affects (outbound) | Success, Failure, and degradation states surface here |
| FEAT-20.SPEC-003 (Handoff Eligibility & Authorization Rules) | References (inbound) | This integration refuses to initiate a request for an ineligible household or member rather than re-deriving either eligibility check itself |

## Analytics and Success Signals

- N/A -- analytics events for this capability's behaviors (grocery_handoff_initiated, grocery_handoff_succeeded, grocery_handoff_failed) are emitted by FEAT-20.SPEC-001, the screen that owns the triggering interaction and the household-facing outcome, per feature-overview.md's Side-Effect Inventory; this integration spec defines the request/response contract those events describe but does not itself emit analytics, to avoid double counting. No metric in success-metrics.md names Online Grocery Ordering Handoff as its Connected Feature.

## Acceptance Criteria

**FEAT-20.SPEC-002-AC-01:** Given Maya confirms the handoff on FEAT-20.SPEC-001, when the request is built, then it carries each unticked Grocery List Item's ingredient_name, quantity_and_unit, and aisle, plus an opaque attempt reference -- and no Household-entity field (unit_system, currency), household member identity, address, or payment data.

**FEAT-20.SPEC-002-AC-02:** Given Sam's handoff request is accepted in full, when the response arrives, then FEAT-20.SPEC-001 shows the Success state with no unavailable items called out.

**FEAT-20.SPEC-002-AC-03:** Given Maya's handoff request is accepted with some items unavailable, when the response arrives, then FEAT-20.SPEC-001 shows the Success state listing each unavailable item by name.

**FEAT-20.SPEC-002-AC-04:** Given Sam's handoff request is rejected by the capability, when the response arrives, then FEAT-20.SPEC-001 shows the Failure state with Retry and Return to In-App Shopping options.

**FEAT-20.SPEC-002-AC-05:** Given Maya's handoff request receives no response within the capability's normal response window, when the timeout occurs, then this integration treats it as a failed handoff and FEAT-20.SPEC-001 shows the Failure state.

**FEAT-20.SPEC-002-AC-06:** Given the capability is responding slowly, when Sam has been in the Confirming state past the normal wait, then the screen shows "Still working -- this can take a moment." while the request remains pending.

**FEAT-20.SPEC-002-AC-07:** Given the capability is entirely unreachable, when Maya taps Confirm Handoff, then the request is not sent and the screen shows "Online grocery ordering isn't available right now. Your grocery list is safe -- try again later."

**FEAT-20.SPEC-002-AC-08:** Given the capability rejects Sam's submitted list outright, when the rejection is received, then the screen shows "We couldn't hand off your list to online ordering. Nothing has changed -- your grocery list is exactly as it was." with Retry and Return to In-App Shopping.

**FEAT-20.SPEC-002-AC-09:** Given Maya has never handed off a list before, when she taps Confirm Handoff for the first time, then the first-handoff disclosure notice appears with the exact wording naming what is shared, and "Continue"/"Cancel" options, before any data leaves the product.

**FEAT-20.SPEC-002-AC-10:** Given Sam has already seen and accepted the first-handoff disclosure, when he initiates a later handoff, then the disclosure does not reappear automatically, and he can reopen the same wording through "How this data is shared" on FEAT-20.SPEC-001.

**FEAT-20.SPEC-002-AC-11:** Given a household navigates away from FEAT-20.SPEC-001 before a response arrives, when the response is later received, then it is discarded and no outcome is shown or queued for that attempt.

**FEAT-20.SPEC-002-AC-12:** Given the same acceptance event is delivered twice for one handoff attempt, when the second delivery arrives, then nothing further changes and no duplicate confirmation is shown.

**FEAT-20.SPEC-002-AC-13:** Given a rejection event and an earlier partial-acceptance event both concern the same attempt, when both are received, then only the first to reach the still-open FEAT-20.SPEC-001 is displayed, and the later one is discarded.

**FEAT-20.SPEC-002-AC-14:** Given the capability goes down after Maya's request has been sent but before any response is confirmed, when this occurs, then the screen shows the capability-down message and no local record reflects a half-completed handoff, since no handoff entity is ever persisted.

**FEAT-20.SPEC-002-AC-15:** Given some Grocery List Items are ticked at the moment Sam confirms the handoff, when the outbound request is built, then only the unticked items are included.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 2 | 2 |
| Inbound Events | 4 | 4 |
| Degradation Paths | 3 | 3 |
| Consent and Disclosure | 2 | 2 |
| Edge Cases | 5 | 5 |

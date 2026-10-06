---
document_type: feature-overview
feature_number: FEAT-20
feature_name: Online Grocery Ordering Handoff
feature_slug: online-grocery-ordering-handoff
priority_tier: Nice-to-Have
feature_type: Platform
produced_by: feature-analyst
status: final
created: 2026-09-27
spec_count: 3
screen_count: 1
automation_count: 0
logic_rule_count: 1
integration_count: 1
notification_count: 0
---

# Feature Breakdown Brief: Online Grocery Ordering Handoff

## Summary

**Feature:** Online Grocery Ordering Handoff
**ID:** FEAT-20
**Description:** The household's grocery list can be handed off to an online grocery ordering capability for delivery, instead of shopping in person.
**Priority:** Nice-to-Have
**Phase:** Later
**Type:** Platform
**Rationale:** Named directly in BRIEF.md's Ecosystem & Integrations section as "desirable later, not v1." Phased to Later per the brief's own explicit timing; documented here rather than only as a deferral note because it is a genuine feature the product will eventually need, not merely an idea. Competitor context (Samsung Food integrates with 23 grocery retailers across 4 regions, while AnyList and Cozi offer none) frames this as a differentiator rather than a common feature, supporting the Later timing.

**Key Capabilities:**
- Hand off the list — Household sends its current grocery list to an online-ordering capability for fulfillment
- See handoff status — Household sees whether the handoff succeeded and can return to in-person shopping if it did not

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-20.SPEC-001 | Grocery Handoff Screen | Screen | Maya, Sam | Household initiates a handoff and sees its progress, confirmation, partial-availability callouts, or failure with a fallback to in-app shopping |
| FEAT-20.SPEC-002 | Online Grocery Ordering Integration | Integration | Maya, Sam | Product sends the household's current grocery list to the online-ordering capability and receives back acceptance, rejection, or per-item availability |
| FEAT-20.SPEC-003 | Handoff Eligibility & Authorization Rules | Logic/Rule | All | Governs whether the handoff entry point and screen are available, combining regional online-ordering availability with the Access Matrix's per-role handoff permission |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Hand off the list | FEAT-20.SPEC-001, FEAT-20.SPEC-002 | The screen collects the household's confirmation to proceed and displays progress; the integration spec carries the list to the external ordering capability and returns its response | Phase 2 (Explicit) |
| See handoff status | FEAT-20.SPEC-001 | The screen displays the loading indicator, the success confirmation with any unavailable items called out, or the failure message with the fallback to in-app shopping | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-20.SPEC-002 | Online Grocery Ordering Integration | Phase 4 (External Dependencies lens) | assumptions-constraints.md's ASMP-37 names online grocery-ordering as a category-level external capability this feature depends on; sending the list and receiving acceptance/rejection/availability crosses the product boundary and needed its own Integration spec rather than living inline in the screen |
| FEAT-20.SPEC-003 | Handoff Eligibility & Authorization Rules | Phase 5 (Rule Discovery) | Two conditions gate the feature — the "Validation & Limits" regional-availability rule and the Access field's per-role handoff permission (Maya/Sam Full; older-kid login Full for add/tick only, not handoff; young-kid and Riley None) — and both conditions are evaluated at two points: the "hand off list" entry point on FEAT-06's Grocery List screen and this feature's own screen. A rule evaluated at two points across two features clears the "shared across multiple screens" threshold for a standalone Logic/Rule spec rather than staying inline in one screen |

## Entity-Lifecycle Coverage Matrix

This feature manages no entity of its own. Both Connected Entities named in product-features.md (Grocery List, Grocery List Item) are marked read-only for FEAT-20, and the Domain Entity Inventory defines no "handoff attempt" or "handoff history" entity — product-features.md's Data Notes field states "Derived: none beyond the handoff attempt itself," meaning the attempt's outcome is displayed transiently on FEAT-20.SPEC-001 and is not persisted as a queryable record. No CRUD Coverage Matrix therefore applies; see Referenced Entities below and Non-Goals for the explicit decision not to persist handoff history.

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Grocery List | FEAT-20.SPEC-002 | Read as the payload sent to the online-ordering capability at handoff time |
| Grocery List Item | FEAT-20.SPEC-001, FEAT-20.SPEC-002 | SPEC-002 reads each item as part of the handoff payload; SPEC-001 displays which items the ordering capability's response marks unavailable |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Household taps "hand off list" on the Grocery List screen (FEAT-06) | Navigate into this feature's Handoff screen and evaluate eligibility before allowing the handoff to proceed | Cross-feature -- entry point owned by FEAT-06, evaluated against SPEC-003 | FEAT-20.SPEC-001 / FEAT-20.SPEC-003 |
| Handoff screen loads | Check regional online-ordering availability and the initiating member's role permission | Standalone Logic/Rule | FEAT-20.SPEC-003 |
| Household confirms the handoff on the Handoff screen | Send the current Grocery List and its items to the online-ordering capability | Standalone Integration | FEAT-20.SPEC-002 |
| Online-ordering capability accepts the list, in full or in part | Show a success confirmation on the Handoff screen, calling out any items the capability marked unavailable so the household knows what still needs an in-person trip; fire `grocery_handoff_succeeded` | Inline in triggering screen -- same-screen result with no separate delivery channel named in product-features.md's Communications field | FEAT-20.SPEC-001 |
| Online-ordering capability rejects the list or the integration call fails | Show a clear failure message on the Handoff screen and fall back cleanly to the standard in-app grocery list, preserving all list data; fire `grocery_handoff_failed` | Inline in triggering screen -- same reasoning as above; the "confirmation or failure notice to whoever initiated it" is the initiator's own view of this same screen, not a separate email/push/SMS delivery | FEAT-20.SPEC-001 |
| Household initiates a handoff (any outcome) | Fire `grocery_handoff_initiated` for analytics | Inline in triggering screen | FEAT-20.SPEC-001 |
| No online-ordering capability is available for the household's region | Hide or disable the "hand off list" entry point on the Grocery List screen and, if reached directly, show the Handoff screen's ineligible state | Standalone Logic/Rule | FEAT-20.SPEC-003 |
| Member without handoff permission (older-kid Later-phase login, young-kid profile, Riley, or an unauthorized visitor) reaches the Handoff screen or entry point | Entry point is not shown; direct navigation is denied with an access message | Standalone Logic/Rule, rendered via Permission Denied state on the screen | FEAT-20.SPEC-003 / FEAT-20.SPEC-001 |
| Device goes offline while the Grocery List screen or Handoff screen is open | Handoff entry point and any in-progress handoff are unavailable (handoff requires connectivity, per product-features.md's States field); the standard grocery list itself remains fully usable offline, unaffected by this feature | Inline in triggering screen (Offline/Degraded state); the unaffected grocery list behavior is FEAT-06's own responsibility | FEAT-20.SPEC-001 |
| Grocery List is edited (tick, add, remove) by another member while a handoff is in progress | The handoff proceeds against the snapshot of the list taken at the moment the household confirmed the handoff; edits made afterward are not included and simply remain on the standard list for next time | Inline in triggering screen -- a single confirm-and-send interaction with no separate queuing behavior, consistent with the "brief" progress indicator in product-features.md's States field | FEAT-20.SPEC-001 / FEAT-20.SPEC-002 |

## Shared Context

**Shared Entities:**
- Grocery List -- read by FEAT-20.SPEC-002 as the handoff payload; not created, updated, or deleted by this feature (owned end-to-end by FEAT-06).
- Grocery List Item -- read by FEAT-20.SPEC-001 (to display which items the ordering capability marked unavailable) and FEAT-20.SPEC-002 (as line items in the handoff payload); not created, updated, or deleted by this feature.

**Shared UI Patterns:**
N/A -- this feature produces a single Screen spec (FEAT-20.SPEC-001), so there is no pattern shared across multiple screens within this feature. The screen's progress/confirmation/failure layout should stay visually consistent with the Grocery List screen it is reached from (FEAT-06), which is a cross-feature consistency note rather than an intra-feature shared pattern.

**Shared Validation:**
- FEAT-20.SPEC-003 (Handoff Eligibility & Authorization Rules) is the single source of truth for both the regional-availability gate and the per-role handoff permission. FEAT-20.SPEC-001 references it to decide whether to render the handoff action or a Permission Denied / ineligible state, and FEAT-20.SPEC-002 references it to refuse initiating an integration call for an ineligible household or member rather than re-deriving either check.

## Internal Dependency Map

```
SPEC-001 (Grocery Handoff Screen) -> [screen loads] -> SPEC-003 (Handoff Eligibility & Authorization Rules) -> [eligible] -> SPEC-001 (renders the confirm-handoff action)
SPEC-001 (Grocery Handoff Screen) -> [screen loads] -> SPEC-003 (Handoff Eligibility & Authorization Rules) -> [ineligible: region or role] -> SPEC-001 (renders ineligible / Permission Denied state)
SPEC-001 (Grocery Handoff Screen) -> [household confirms handoff] -> SPEC-002 (Online Grocery Ordering Integration) -> [acceptance, in full or in part] -> SPEC-001 (renders success confirmation with unavailable items called out)
SPEC-001 (Grocery Handoff Screen) -> [household confirms handoff] -> SPEC-002 (Online Grocery Ordering Integration) -> [rejection or failure] -> SPEC-001 (renders failure message and falls back to the in-app list)
```

**Default Entry:** SPEC-001 (Grocery Handoff Screen) -- this feature has no independent navigation entry point of its own; it is reached only by tapping "hand off list" on the Shared Grocery List screen (FEAT-06), gated by SPEC-003.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-20.SPEC-001 | Inbound | FEAT-06 (Shared Grocery List) | Household is taken from the Grocery List screen into this feature's Handoff screen, gated by FEAT-20.SPEC-003 | Tap "hand off list" |
| FEAT-20.SPEC-002 | Inbound | FEAT-06 (Shared Grocery List) | Reads the household's current Grocery List and Grocery List Items as the handoff payload | Household confirms the handoff on FEAT-20.SPEC-001 |
| FEAT-20.SPEC-001 | Outbound | FEAT-06 (Shared Grocery List) | A failed or declined handoff falls back cleanly to the standard in-app grocery list, with no data lost | Handoff fails, or the household chooses to return to in-person shopping |
| FEAT-20.SPEC-003 | Inbound | FEAT-06 (Shared Grocery List) | Determines whether the "hand off list" entry point is shown on the Grocery List screen at all | Grocery List screen renders for the current member |

## Non-Functional Notes

**Data volumes / growth:** N/A -- this feature persists no entity of its own; the only volume in play is the existing Grocery List's item count, which is already governed by FEAT-06's non-functional expectations.

**Responsiveness:** The handoff shows a brief, explained progress indicator while the online-ordering capability responds (product-features.md, States field); the household should never be left on an unexplained blank wait. `grocery_handoff_initiated`, `grocery_handoff_succeeded`, and `grocery_handoff_failed` (product-features.md, Signals field) give the product visibility into how often handoffs are attempted and how often they complete, so responsiveness regressions and failure-rate trends can both be tracked.

**Data sensitivity / privacy:** Low -- the Grocery List and Grocery List Item data this feature reads carries the dependency map's Low data-sensitivity classification, is household personal data private to the household, and is never sold (assumptions-constraints.md ASMP-14). Handing the list to an external online-ordering capability is the one point where this household data leaves the product boundary; FEAT-20.SPEC-002 (the Integration spec) is where that data-sharing contract is specified, and it inherits this same privacy floor.

**Compliance flags:** N/A -- no compliance regime is named for this feature beyond the general household-data privacy posture; the children's-data privacy rule (ASMP-26) does not apply here because no kid profile (young-kid or Later-phase older-kid login) has handoff access per the Access Matrix.

## Non-Goals

- **Selling or fulfilling grocery orders within the product** -- Excluded per scope-boundaries.md (SC-07): the product plans and lists groceries; it hands the list off to an external online-ordering capability rather than selling or shipping ingredients itself.
- **Naming or comparing specific online-ordering retailers** -- Excluded per pipeline-rules.md's functional-language-only constraint and assumptions-constraints.md's ASMP-37: Stage 2–3 documents name only the category-level "online grocery ordering" capability; selecting and comparing named providers is Stage 4's responsibility.
- **Delivery tracking or order-status updates after handoff** -- Not part of the Key Capabilities, which name only "hand off the list" and "see handoff status" (i.e., whether the handoff itself succeeded); once the online-ordering capability has accepted the list, further order lifecycle (packing, delivery, driver tracking) happens entirely within that external capability and is outside this feature's and this product's scope.
- **Older-kid, young-kid, or unauthenticated-visitor initiated handoff** -- Excluded per the Access Matrix in user-persona.md: the Later-phase older-kid login's Grocery List Full access covers adding and ticking only, not handoff; young-kid profiles and unauthorized visitors have no access at all to this or any household data.
- **Persisted handoff history** -- Intentional lifecycle decision surfaced by entity-lifecycle analysis: product-features.md's Data Notes field states "Derived: none beyond the handoff attempt itself," and no handoff entity appears in the Domain Entity Inventory. Each handoff is a transient, single-attempt interaction shown on FEAT-20.SPEC-001; no history log, audit trail, or repeat-order record is created. Weekly Plan History (FEAT-19) covers historical plan and list review, not handoff attempts.

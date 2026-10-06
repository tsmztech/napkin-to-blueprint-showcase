# FEAT-20 — Online Grocery Ordering Handoff

This chapter covers FEAT-20, Online Grocery Ordering Handoff, a Nice-to-Have-tier feature. It contains 3 specifications carrying 50 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-20.SPEC-001 | Grocery Handoff Screen | screen | 21 |
| FEAT-20.SPEC-002 | Online Grocery Ordering Integration | integration | 15 |
| FEAT-20.SPEC-003 | Handoff Eligibility & Authorization Rules | logic-rule | 14 |

The feature breakdown brief follows, then every specification in full.


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



# Screen Spec: Grocery Handoff Screen

## Overview

**Name:** Grocery Handoff Screen
**ID:** FEAT-20.SPEC-001
**Type:** Screen
**Purpose:** The household initiates a handoff of its current grocery list to an online-ordering capability and sees its progress, confirmation (with any partial-availability callouts), or failure with a fallback to in-app shopping.
**Parent Feature:** FEAT-20 -- Online Grocery Ordering Handoff

## Scope and Non-Goals

**In Scope:**
- Showing the eligible or ineligible entry state for the household reaching this screen
- Collecting the household's confirmation to hand off the current grocery list
- Showing progress while the handoff is in flight
- Showing the success confirmation, calling out any items the online-ordering capability marked unavailable
- Showing a failure message with a clean fallback back to the standard in-app grocery list

**Non-Goals:**
- The regional-availability and per-role eligibility determination itself -- governed by FEAT-20.SPEC-003 (Handoff Eligibility & Authorization Rules); this screen only renders the outcome of that determination
- The mechanics of sending the list to the online-ordering capability and interpreting its response -- governed by FEAT-20.SPEC-002 (Online Grocery Ordering Integration); this screen triggers it and displays its outcome
- Delivery tracking or order-status updates after the online-ordering capability accepts the list -- excluded per feature-overview.md's Non-Goals: once the capability has accepted the list, further order lifecycle happens entirely within that external capability, outside this feature and this product
- Persisting a handoff history or audit trail -- excluded per feature-overview.md's Non-Goals: product-features.md's Data Notes state "Derived: none beyond the handoff attempt itself," so each handoff is a transient, single-attempt interaction with no history log

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-06.SPEC-001 (Grocery List) | Maya or Sam taps the "Hand off list" header control (shown only while the household's region is handoff-eligible, per FEAT-20.SPEC-003) | The household's current Grocery List reference; eligibility is re-evaluated on load per FEAT-20.SPEC-003 rather than trusted from the entry point |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | Confirm the handoff, retry after failure, return to in-app shopping | -- |
| Sam (Other Adult Member) | Full screen | Confirm the handoff, retry after failure, return to in-app shopping | -- |
| Jordan (older kid, limited login -- Later) | Only the Permission Denied state | No handoff action | "This action needs an adult household member." -- shown regardless of the household's region eligibility, since this role's Grocery List access (FEAT-06.SPEC-009) covers adding and ticking only, not handoff (FEAT-20.SPEC-003) |
| Jordan (young kid profile, no login -- MVP) | No | No | Has no login at all and cannot reach this screen through any path |
| Riley (Operator, support -- from v1) | No | No | Entry point and direct navigation are both denied; Riley holds no handoff permission under any condition (product-features.md, Access field) |
| Unauthenticated | No | No | Redirected to the sign-in screen; no household grocery data is exposed |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- if the expiry is detected before Confirm Handoff was tapped, no handoff is initiated on the member's behalf |

## Layout and Content

**Header:** Screen title "Hand Off Grocery List" with a back arrow (returns to FEAT-06.SPEC-001, the Grocery List).

**Body:** Content depends on the current screen state (see States):
- **Loading state:** A brief, unlabeled progress indicator in place of the summary area, shown while the item-count read and the FEAT-20.SPEC-003 eligibility check are in flight.
- **Error state:** A load-time failure message ("We couldn't load your handoff details. Try again.") with a "Retry" button and a "Return to In-App Shopping" link.
- **Ineligible state:** A single message area explaining why the handoff is unavailable (regional unavailability or role restriction), with no further controls.
- **Ready-to-Confirm state:** A summary area listing the count of unticked items currently on the household's Grocery List, above a single "Confirm Handoff" button.
- **Confirming state:** The summary area from Ready-to-Confirm, with the Confirm Handoff button replaced by a brief, explained progress indicator ("Sending your list...").
- **Success state:** A confirmation message ("Your list is on its way"), followed -- only when the response marks some items unavailable -- by a list of those items by name under the heading "Still need these from the store:".
- **Failure state:** A failure message area, below which sit two buttons side by side: "Retry" and "Return to In-App Shopping".
- **Offline/Degraded state:** The Ready-to-Confirm summary area with the Confirm Handoff button disabled, and a banner above it explaining the connectivity requirement.

**Footer:** None -- all actions live in the body.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described above, full width; Retry and Return to In-App Shopping stack vertically in the Failure state.
- **Medium size class and above:** Content remains single-column, capped at a consistent platform-wide form width (exact value is the design layer's decision) and horizontally centered; Retry and Return to In-App Shopping sit side by side in the Failure state instead of stacking.
- **Unavailable-items list:** Uniform scaling, no structural change across breakpoints.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-06.SPEC-001 (Grocery List) | Screen closes; no handoff initiated if not already confirmed | Animated transition back to the Grocery List |
| Confirm Handoff button | Tap | Triggers FEAT-20.SPEC-002 with a snapshot of the current unticked Grocery List Items | Screen enters the Confirming state | Progress indicator with the message "Sending your list..." |
| Confirm Handoff button (while Confirming) | Tap | No action -- debounced | None | Button/indicator remains in the Confirming state |
| Retry button (Failure state) | Tap | Re-triggers FEAT-20.SPEC-002 with a fresh snapshot of the current list | Screen re-enters the Confirming state | Progress indicator with the message "Sending your list..." |
| Return to In-App Shopping button (Failure state) | Tap | Navigate to FEAT-06.SPEC-001 (Grocery List) | Screen closes; Grocery List is unchanged | Animated transition back to the Grocery List |
| Unavailable-items list (Success state) | None | Display-only -- not interactive | None | Read-only list of item names |
| Retry button (Error state) | Tap | Re-runs the item-count read and the FEAT-20.SPEC-003 eligibility check | Screen re-enters the Loading state | Progress indicator shown in place of the error message |
| Return to In-App Shopping link (Error state) | Tap | Navigate to FEAT-06.SPEC-001 (Grocery List) | Screen closes; no handoff initiated | Animated transition back to the Grocery List |

### Accessibility Notes

- **Focus order:** Back arrow -> (state-dependent body content) -> Confirm Handoff, or Retry -> Return to In-App Shopping, in that order.
- **State announcements:** Entering the Loading, Error, Confirming, Success, Failure, and Ineligible states each triggers an announcement to assistive technology of that state's message text.
- **Offline banner:** Announced immediately when connectivity is lost while this screen is open.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty | N/A -- handoff only applies once a grocery list exists (product-features.md, States field); a Grocery List always exists by the time this screen is reachable, since FEAT-06 creates one from any generated or manually picked week | -- | -- |
| Loading | A brief, unlabeled progress indicator in place of the summary area, while the screen reads the current Grocery List / Grocery List Item item-count summary and FEAT-20.SPEC-003 evaluates eligibility; no Confirm control or message is shown yet | Screen is navigated to from FEAT-06.SPEC-001's "hand off list" trigger, before the item-count read and the FEAT-20.SPEC-003 eligibility check both return | The item-count read and the eligibility check both complete, resolving to Ineligible or Ready to Confirm; or either fails, resolving to Error |
| Error | A load-time failure message ("We couldn't load your handoff details. Try again.") with a single "Retry" control that re-runs the item-count read and the FEAT-20.SPEC-003 eligibility check, and a "Return to In-App Shopping" link back to FEAT-06.SPEC-001 | The item-count read or the FEAT-20.SPEC-003 eligibility check fails while the device is online (a non-connectivity load failure; a connectivity loss instead enters Offline/Degraded) | Household taps Retry (re-enters Loading) or Return to In-App Shopping (leaves the screen) |
| Ineligible | Message explaining the handoff is unavailable, either "Online grocery ordering isn't available in your area yet." (region) or "This action needs an adult household member." (role); no Confirm control shown | Loading completes and FEAT-20.SPEC-003 evaluates the household or member as ineligible | Household leaves the screen |
| Ready to Confirm (default) | Item-count summary with an enabled Confirm Handoff button | Loading completes and FEAT-20.SPEC-003 evaluates the household and member as eligible | Household taps Confirm Handoff |
| Confirming | Item-count summary with the Confirm control replaced by a progress indicator | Confirm Handoff or Retry tapped | FEAT-20.SPEC-002 returns a result, or the request fails/times out |
| Success | Confirmation message, plus an unavailable-items list when the response is partial | FEAT-20.SPEC-002 reports full or partial acceptance | Household leaves the screen |
| Failure | Failure message with Retry and Return to In-App Shopping | FEAT-20.SPEC-002 reports rejection or a failed/timed-out call | Household taps Retry (re-enters Confirming) or Return to In-App Shopping (leaves the screen) |
| Offline/Degraded | Ready-to-Confirm summary with Confirm Handoff disabled and a banner reading "Hand-off needs a connection. Reconnect to send your list." | Connectivity is lost while this screen is open | Connectivity is restored -- Confirm Handoff re-enables; nothing was queued, since handoff requires an active connection to initiate |

## Validation Rules

Validation governed by FEAT-20.SPEC-003 (Handoff Eligibility & Authorization Rules). See that spec for the regional-availability and per-role eligibility conditions. This screen applies the eligibility check on screen load (shown as the Loading state), before rendering the Ineligible or Ready-to-Confirm state; if the check or the item-count read fails while online, the screen renders the Error state instead.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-06.SPEC-001 (Grocery List) | FEAT-06 (Shared Grocery List) |
| Return to In-App Shopping tap (Failure state) | FEAT-06.SPEC-001 (Grocery List) | FEAT-06 (Shared Grocery List) |

## Data Model

**Creates:** None -- this feature persists no entity of its own; the handoff outcome is displayed transiently and is not written to any record (feature-overview.md's Entity-Lifecycle Coverage Matrix).
**Reads:** Grocery List -- week, status (to confirm a current list exists and is Active); Grocery List Item -- ingredient_name, quantity_and_unit, aisle, ticked (to build the item-count summary and, at confirm time, the snapshot sent by FEAT-20.SPEC-002). Both entities are owned end-to-end by FEAT-06 and are never created, updated, or deleted by this screen.
**Updates:** None.
**Deletes:** None.

## Business Rules

- Eligibility (region and role) is governed by FEAT-20.SPEC-003 and evaluated fresh on this screen's load -- the screen never trusts an eligibility result carried from the FEAT-06.SPEC-001 entry point.
- Confirming the handoff triggers FEAT-20.SPEC-002 (Online Grocery Ordering Integration), which owns the request/response contract with the online-ordering capability.
- The handoff proceeds against the snapshot of the Grocery List Items taken at the moment the household taps Confirm Handoff (or Retry); edits made by another member after that moment are not included and simply remain on the standard list for next time.
- Double-submit is prevented: while a request is in flight (Confirming state), a repeated tap on Confirm Handoff has no effect.

## Edge Cases

- **Household taps Confirm Handoff twice in rapid succession** -- The second tap is ignored while the first request is in flight; only one call reaches FEAT-20.SPEC-002.
- **Grocery List is edited by another member while a handoff is in the Confirming state** -- No live conflict is shown on this screen. Resolution: the handoff proceeds against the snapshot taken at the moment Confirm was tapped (per the dependency map's "merge" Contention note for Grocery List and its High-contention Grocery List Item); the new edit is not included and remains on the standard list for next time.
- **Household navigates away from this screen while a handoff is still in the Confirming state** -- The in-flight request is not cancelled by leaving the screen, but no outcome is queued or displayed later, since no handoff history is persisted; returning to this screen later shows a fresh Ready-to-Confirm state.
- **Device goes offline after Confirm Handoff was tapped but before a response arrives** -- The request cannot complete; the screen shows the Failure state with the connectivity-specific message from FEAT-20.SPEC-002's Degradation Behavior, and the household's Grocery List remains unchanged and fully usable offline.
- **Household reopens the Handoff screen after a completed handoff (this session or a previous one)** -- The screen always shows a fresh Ready-to-Confirm (or Ineligible) state; no prior outcome is retained or shown, consistent with the feature's decision not to persist handoff history.
- **The item-count read or the FEAT-20.SPEC-003 eligibility check fails while the screen is loading, with connectivity still present** -- The screen shows the Error state rather than Offline/Degraded, since connectivity is not the cause; tapping Retry re-runs both checks from Loading, and Return to In-App Shopping returns to FEAT-06.SPEC-001 with the Grocery List unchanged.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-001 (Grocery List) | Navigation (inbound and outbound) | Household arrives here from "hand off list"; the back arrow and both post-outcome paths return there |
| FEAT-20.SPEC-002 (Online Grocery Ordering Integration) | Triggers (outbound) | Confirm Handoff and Retry both trigger this integration with a snapshot of the current list |
| FEAT-20.SPEC-003 (Handoff Eligibility & Authorization Rules) | References (inbound) | Screen-load eligibility check determines the Ineligible vs. Ready-to-Confirm rendering |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| grocery_handoff_initiated | initiating role, item count in the snapshot | Household taps Confirm Handoff or Retry | N/A -- no metric in success-metrics.md names Online Grocery Ordering Handoff as its Connected Feature; retained for operational visibility per product-features.md's Signals field, so handoff attempt volume is observable even without a Stage 2 target |
| grocery_handoff_succeeded | outcome (full / partial), count of unavailable items | FEAT-20.SPEC-002 reports full or partial acceptance | N/A -- same reason as above |
| grocery_handoff_failed | failure reason category (rejected / capability down / timeout) | FEAT-20.SPEC-002 reports rejection, or the call fails or times out | N/A -- same reason as above |

## Acceptance Criteria

**FEAT-20.SPEC-001-AC-01:** Given the household's region has an available online-ordering capability, when Maya (Organiser) taps "hand off list" on FEAT-06.SPEC-001 and this screen loads, then it shows the Ready-to-Confirm state with a count of the household's current unticked Grocery List Items and an enabled Confirm Handoff button.

**FEAT-20.SPEC-001-AC-02:** Given Sam (Other Adult Member) is on this screen in the Ready-to-Confirm state, when he taps Confirm Handoff, then the screen enters the Confirming state showing "Sending your list..." and this spec triggers FEAT-20.SPEC-002 with a snapshot of the current unticked items.

**FEAT-20.SPEC-001-AC-03:** Given Maya is on this screen in the Confirming state, when FEAT-20.SPEC-002 returns full acceptance, then the screen shows the Success state with the message "Your list is on its way" and no unavailable-items list.

**FEAT-20.SPEC-001-AC-04:** Given Sam is on this screen in the Confirming state, when FEAT-20.SPEC-002 returns partial acceptance, then the screen shows the Success state and lists each unavailable item by name under "Still need these from the store:".

**FEAT-20.SPEC-001-AC-05:** Given Maya is on this screen in the Confirming state, when FEAT-20.SPEC-002 reports rejection or the call fails, then the screen shows the Failure state with a Retry button and a Return to In-App Shopping button.

**FEAT-20.SPEC-001-AC-06:** Given Sam is on this screen in the Failure state, when he taps Return to In-App Shopping, then he is navigated to FEAT-06.SPEC-001 and the Grocery List is unchanged.

**FEAT-20.SPEC-001-AC-07:** Given Maya is on this screen in the Failure state, when she taps Retry, then the screen re-enters the Confirming state and FEAT-20.SPEC-002 is triggered again with a fresh snapshot of the current list.

**FEAT-20.SPEC-001-AC-08:** Given Maya is on this screen at any point before tapping Confirm Handoff, when she taps the back arrow, then she is navigated to FEAT-06.SPEC-001 and no handoff is initiated.

**FEAT-20.SPEC-001-AC-09:** Given the household's region has no available online-ordering capability, when any eligible-role member reaches this screen, then it shows the Ineligible state with "Online grocery ordering isn't available in your area yet." and no Confirm Handoff control.

**FEAT-20.SPEC-001-AC-10:** Given Jordan (older kid, limited login) reaches this screen directly, when the screen loads, then it shows "This action needs an adult household member." with no Confirm Handoff control, regardless of the household's region eligibility.

**FEAT-20.SPEC-001-AC-11:** Given Jordan (young kid profile, no login), when any attempt is made to reach this screen, then it never succeeds, since this profile has no login at all.

**FEAT-20.SPEC-001-AC-12:** Given Riley (Operator, support) attempts to open a household's Handoff screen, when the attempt is made, then access is denied entirely and no household grocery data is shown.

**FEAT-20.SPEC-001-AC-13:** Given an unauthenticated visitor requests this screen directly, when the request is made, then they are redirected to the sign-in screen and no household data is exposed.

**FEAT-20.SPEC-001-AC-14:** Given Maya's session expires while she is on this screen before she has tapped Confirm Handoff, when the expiry is detected, then the dialog "Your session has expired. Sign in to continue." appears and no handoff is initiated on her behalf.

**FEAT-20.SPEC-001-AC-15:** Given Sam loses connectivity while this screen is in the Ready-to-Confirm state, when connectivity drops, then Confirm Handoff disables and the banner "Hand-off needs a connection. Reconnect to send your list." appears, while the underlying Grocery List (FEAT-06) remains fully usable elsewhere.

**FEAT-20.SPEC-001-AC-16:** Given Maya taps Confirm Handoff twice in rapid succession, when the second tap occurs, then it is ignored while the first request is in flight, and only one call reaches FEAT-20.SPEC-002.

**FEAT-20.SPEC-001-AC-17:** Given another household member edits the Grocery List while Maya's handoff is in the Confirming state, when the edit is made, then it is not included in the in-flight handoff, which proceeds against the snapshot taken when Confirm was tapped, and the new edit remains on the standard list for next time.

**FEAT-20.SPEC-001-AC-18:** Given Sam navigates away from this screen while a handoff is still in the Confirming state, when he returns to the screen later, then no outcome is queued or shown for the earlier attempt, and the screen shows a fresh Ready-to-Confirm state.

**FEAT-20.SPEC-001-AC-19:** Given Maya taps "hand off list" on FEAT-06.SPEC-001, when this screen begins loading, then it shows the Loading state's progress indicator with no message, Confirm control, or state-specific content until the item-count read and the FEAT-20.SPEC-003 eligibility check both complete.

**FEAT-20.SPEC-001-AC-20:** Given Sam is online and this screen is in the Loading state, when the FEAT-20.SPEC-003 eligibility check fails to return a result, then the screen shows the Error state with "We couldn't load your handoff details. Try again." and Retry and Return to In-App Shopping controls, rather than the Offline/Degraded state.

**FEAT-20.SPEC-001-AC-21:** Given Maya is on this screen in the Error state, when she taps Retry, then the screen re-enters the Loading state and re-runs the item-count read and the FEAT-20.SPEC-003 eligibility check.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 8 (Loading, Error, Ineligible, Ready to Confirm, Confirming, Success, Failure, Offline/Degraded -- Empty is N/A) | 9 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



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



# Logic/Rule Spec: Handoff Eligibility & Authorization Rules

## Overview

**Name:** Handoff Eligibility & Authorization Rules
**ID:** FEAT-20.SPEC-003
**Type:** Logic/Rule
**Purpose:** Governs whether the handoff entry point and screen are available, combining regional online-ordering availability with the Access Matrix's per-role handoff permission.
**Parent Feature:** FEAT-20 -- Online Grocery Ordering Handoff
**Governed Entity:** Grocery List (read-only, for eligibility determination only)

## Scope and Non-Goals

**In Scope:**
- The single regional-availability gate: whether an online-ordering capability exists for the household's region
- The per-role handoff permission: which roles may see the entry point, reach the screen, and confirm a handoff
- What each role and eligibility outcome experiences at both evaluation points -- FEAT-06's Grocery List screen entry point and this feature's own Handoff screen
- Keeping the two evaluation points consistent with each other and with the Access Matrix in user-persona.md

**Non-Goals:**
- Field-level validation or derivation of Grocery List or Grocery List Item content -- owned by FEAT-06 (FEAT-06.SPEC-006, FEAT-06.SPEC-007, FEAT-06.SPEC-009); this spec governs handoff eligibility only, not list content
- The request/response contract with the online-ordering capability -- owned by FEAT-20.SPEC-002; this spec only decides whether that integration may be initiated
- The screen mechanics of the Ineligible, Confirming, Success, and Failure states -- owned by FEAT-20.SPEC-001; this spec defines only the eligibility outcome those states render
- Persisting eligibility history or an audit trail of eligibility checks -- excluded per feature-overview.md's Non-Goals: this feature persists no entity of its own, and eligibility is re-evaluated fresh at each check rather than logged

## Governed Entity

**Entity:** Grocery List
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| week | date | The plan week this list serves -- not used by this spec's eligibility check |
| aisle_grouping | text | The household's configured aisle names and order -- not used by this spec's eligibility check |
| status | enum | Generated, Active, or Archived -- not used by this spec's eligibility check |

This spec's actual governed object is the handoff action itself (an action evaluated against the Household's region and the initiating member's role), rather than any field of the Grocery List or Grocery List Item entities. The Grocery List entity is named as the nominal governed entity because the handoff action is reached from and evaluated against it; its fields carry no validation rules from this spec, which are owned entirely by FEAT-06.

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-06.SPEC-001 | Grocery List | On screen render, to decide whether the "hand off list" entry point is shown at all |
| FEAT-20.SPEC-001 | Grocery Handoff Screen | On screen load, to decide the Ineligible vs. Ready-to-Confirm rendering |
| FEAT-20.SPEC-002 | Online Grocery Ordering Integration | Refuses to initiate a request for an ineligible household or member rather than re-deriving either check itself, per the Brief's Shared Validation section |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| week | No validation beyond data type -- this spec governs handoff eligibility and authorization, not list field content; owned by FEAT-06 | Always | -- | -- | -- |
| aisle_grouping | No validation beyond data type -- owned by FEAT-16 and FEAT-06; read-only from this spec | Always | -- | -- | -- |
| status | No validation beyond data type -- transitions owned by FEAT-06.SPEC-002 and FEAT-06.SPEC-004 | Always | -- | -- | -- |

## Cross-Field Rules

N/A -- this spec governs a single regional-availability gate combined with per-role authorization, not multi-field validation of Grocery List or Grocery List Item content. Field- and cross-field validation for those entities is governed entirely by FEAT-06.

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View the "hand off list" entry point on FEAT-06.SPEC-001 | Maya (Organiser) | Only while the household's region has an available online-ordering capability | Entry point is not shown on FEAT-06.SPEC-001 at all |
| View the "hand off list" entry point on FEAT-06.SPEC-001 | Sam (Other Adult Member) | Only while the household's region has an available online-ordering capability | Entry point is not shown on FEAT-06.SPEC-001 at all |
| View the "hand off list" entry point on FEAT-06.SPEC-001 | Jordan (older kid, limited login -- Later) | Never | Entry point is never shown to this role regardless of the household's region, since this role's Grocery List Full access (FEAT-06.SPEC-009) covers adding and ticking only, not handoff (user-persona.md, Access Matrix notes) |
| View the "hand off list" entry point on FEAT-06.SPEC-001 | Jordan (young kid profile, no login -- MVP) | Never | Has no login and cannot reach FEAT-06.SPEC-001 or any household screen at all |
| View the "hand off list" entry point on FEAT-06.SPEC-001 | Riley (Operator, support -- from v1) | Never | Entry point is never shown; Riley's Grocery List View access does not extend to the handoff action, which carries no access for this role (product-features.md, Access field) |
| View the "hand off list" entry point on FEAT-06.SPEC-001 | Unauthenticated visitor | Never | Cannot reach FEAT-06.SPEC-001 at all; redirected to the sign-in screen |
| Reach the Handoff screen (FEAT-20.SPEC-001), including direct navigation | Maya (Organiser) | Only while the household's region has an available online-ordering capability | Screen loads in the Ineligible state: "Online grocery ordering isn't available in your area yet." |
| Reach the Handoff screen (FEAT-20.SPEC-001), including direct navigation | Sam (Other Adult Member) | Only while the household's region has an available online-ordering capability | Screen loads in the Ineligible state: "Online grocery ordering isn't available in your area yet." |
| Reach the Handoff screen (FEAT-20.SPEC-001), including direct navigation | Jordan (older kid, limited login -- Later) | Never | Screen loads in the Permission Denied state: "This action needs an adult household member." -- shown regardless of region eligibility |
| Reach the Handoff screen (FEAT-20.SPEC-001), including direct navigation | Jordan (young kid profile, no login -- MVP) | Never | Has no login and cannot reach this screen through any path |
| Reach the Handoff screen (FEAT-20.SPEC-001), including direct navigation | Riley (Operator, support -- from v1) | Never | Access is denied entirely; no household grocery data is shown |
| Reach the Handoff screen (FEAT-20.SPEC-001), including direct navigation | Unauthenticated visitor | Never | Redirected to the sign-in screen; no household data is exposed |
| Confirm the handoff (send the list) | Maya (Organiser) | Only while the household's region has an available online-ordering capability and the screen has loaded in the Ready-to-Confirm state | Confirm Handoff control is not rendered; the screen shows the Ineligible state instead |
| Confirm the handoff (send the list) | Sam (Other Adult Member) | Only while the household's region has an available online-ordering capability and the screen has loaded in the Ready-to-Confirm state | Confirm Handoff control is not rendered; the screen shows the Ineligible state instead |
| Confirm the handoff (send the list) | Jordan (older kid, limited login -- Later) | Never | Confirm Handoff control is never rendered for this role |
| Confirm the handoff (send the list) | Jordan (young kid profile, no login -- MVP) | Never | Has no login and cannot reach the control at all |
| Confirm the handoff (send the list) | Riley (Operator, support -- from v1) | Never | Confirm Handoff control is never rendered; Riley's access to this feature is denied entirely |
| Confirm the handoff (send the list) | Unauthenticated visitor | Never | Redirected to the sign-in screen before any control is reachable |

## Defaults and Derivations

N/A -- this spec governs eligibility gating and authorization only; it defines no default or derived field values. Grocery List and Grocery List Item field derivations are governed entirely by FEAT-06 (FEAT-06.SPEC-006, FEAT-06.SPEC-007).

## Business Rules

- Regional online-ordering availability is checked identically at both evaluation points -- the "hand off list" entry point on FEAT-06.SPEC-001 and the Handoff screen's own load on FEAT-20.SPEC-001 -- per feature-overview.md's Side-Effect Inventory, so the household never sees an entry point that leads to an inconsistent eligibility outcome on the destination screen.
- The older-kid role's Grocery List Full access (FEAT-06.SPEC-009) never extends to this feature's handoff action, consistent with the Access Matrix notes in user-persona.md: "The older-kid row's Grocery List Full covers adding and ticking items, not household list settings ... or grocery-ordering handoff."
- This spec's Authorization Rules table is the single source of truth for both FEAT-06.SPEC-001's entry-point visibility and FEAT-20.SPEC-001's Access and Visibility table -- the three must never diverge.
- FEAT-20.SPEC-002 refuses to initiate a request for an ineligible household or member rather than re-deriving either the regional or role check itself, per the Feature Breakdown Brief's Shared Validation section.

## Edge Cases

- **The household's region availability changes from eligible to ineligible while the Handoff screen is already open, before Confirm Handoff is tapped** -- The next evaluation (screen reload or re-entry) shows the Ineligible state; a confirm already sent before the change completes as initiated, since eligibility is not re-checked mid-flight for an already-triggered request.
- **The household's region becomes eligible after previously being ineligible, while a member is viewing the Ineligible state** -- The screen does not update automatically; the member must return to FEAT-06.SPEC-001 and re-enter the Handoff screen to see the updated eligibility, since no live-polling behavior is defined for this rarely-changing condition.
- **Jordan (older kid, limited login) attempts direct URL navigation to the Handoff screen, bypassing the FEAT-06.SPEC-001 entry point** -- Denied identically to reaching it through the entry point: eligibility is enforced on the Handoff screen's own load, not only through entry-point visibility.
- **The organiser role is handed over from Maya to Sam (FEAT-09) while a handoff confirm is mid-flight** -- Unaffected: both Maya and Sam hold identical Full handoff eligibility under this spec, so the hand-over changes nothing about the in-flight request or either member's standing eligibility.

## Acceptance Criteria

**FEAT-20.SPEC-003-AC-01:** Given the household's region has an available online-ordering capability, when Maya (Organiser) opens FEAT-06.SPEC-001, then the "hand off list" entry point is shown to her.

**FEAT-20.SPEC-003-AC-02:** Given the household's region has no available online-ordering capability, when Maya opens FEAT-06.SPEC-001, then the "hand off list" entry point is not shown at all.

**FEAT-20.SPEC-003-AC-03:** Given the household's region is eligible, when Sam taps "hand off list" and reaches FEAT-20.SPEC-001, then it loads in the Ready-to-Confirm state.

**FEAT-20.SPEC-003-AC-04:** Given the household's region is ineligible, when Sam reaches FEAT-20.SPEC-001 by any means, then it shows the Ineligible state: "Online grocery ordering isn't available in your area yet."

**FEAT-20.SPEC-003-AC-05:** Given Maya is on FEAT-20.SPEC-001 with an eligible region, when she taps Confirm Handoff, then the handoff proceeds and FEAT-20.SPEC-002 is triggered.

**FEAT-20.SPEC-003-AC-06:** Given Jordan (older kid, limited login) attempts to reach FEAT-20.SPEC-001 directly, when the attempt is made, then it is denied with "This action needs an adult household member." regardless of the household's region eligibility.

**FEAT-20.SPEC-003-AC-07:** Given Jordan (older kid, limited login) is viewing FEAT-06.SPEC-001, when he looks for the "hand off list" entry point, then it is never shown to him, even when the household's region is eligible.

**FEAT-20.SPEC-003-AC-08:** Given Jordan (young kid profile, no login), when any attempt is made to reach either FEAT-06.SPEC-001's entry point or FEAT-20.SPEC-001, then it never succeeds, since this profile has no login at all.

**FEAT-20.SPEC-003-AC-09:** Given Riley (Operator, support) attempts to open a household's Handoff screen, when the attempt is made, then it is denied entirely under every condition, since Riley holds no handoff permission at all.

**FEAT-20.SPEC-003-AC-10:** Given an unauthenticated visitor requests FEAT-20.SPEC-001 directly, when the request is made, then they are redirected to the sign-in screen and no household data is exposed.

**FEAT-20.SPEC-003-AC-11:** Given the household's region availability changes from eligible to ineligible while Maya has FEAT-20.SPEC-001 open but has not yet tapped Confirm Handoff, when she reloads or re-enters the screen, then it shows the Ineligible state.

**FEAT-20.SPEC-003-AC-12:** Given a household's region becomes eligible after previously being ineligible, when Sam is viewing the Ineligible state, then the screen does not update automatically; he must return to FEAT-06.SPEC-001 and re-enter to see the current eligibility.

**FEAT-20.SPEC-003-AC-13:** Given the organiser role is handed over from Maya to Sam mid-session (FEAT-09), when the hand-over completes, then both continue to have identical Full handoff eligibility under this spec, unaffected by the hand-over.

**FEAT-20.SPEC-003-AC-14:** Given Jordan (older kid) attempts direct URL navigation to FEAT-20.SPEC-001 bypassing the FEAT-06.SPEC-001 entry point, when the attempt is made, then it is denied identically to reaching it through the entry point, since eligibility is enforced on the screen's own load.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 3 | 3 |
| Cross-Field Rules | 0 (N/A -- see Cross-Field Rules section) | 0 |
| Authorization Rules | 18 | 18 |
| Defaults/Derivations | 0 (N/A -- see Defaults and Derivations section) | 0 |
| Business Rules | 4 | 4 |
| Edge Cases | 4 | 4 |

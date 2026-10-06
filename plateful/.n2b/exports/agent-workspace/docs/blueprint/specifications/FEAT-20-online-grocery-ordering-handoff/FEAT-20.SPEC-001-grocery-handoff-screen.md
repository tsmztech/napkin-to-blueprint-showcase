---
document_type: spec
spec_type: screen
spec_id: FEAT-20.SPEC-001
spec_name: Grocery Handoff Screen
spec_slug: grocery-handoff-screen
parent_feature: FEAT-20
parent_feature_name: Online Grocery Ordering Handoff
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 21
---

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

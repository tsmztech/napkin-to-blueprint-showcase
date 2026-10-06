---
document_type: spec
spec_type: screen
spec_id: FEAT-14.SPEC-004
spec_name: Downgrade / Cancel
spec_slug: downgrade-cancel
parent_feature: FEAT-14
parent_feature_name: Subscription & Billing Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Screen Spec: Downgrade / Cancel

## Overview

**Name:** Downgrade / Cancel
**ID:** FEAT-14.SPEC-004
**Type:** Screen
**Purpose:** Maya reviews exactly what is kept and what is lost, then confirms a downgrade to the free tier or a cancellation of the paid subscription.
**Parent Feature:** FEAT-14 -- Subscription & Billing Management

## Scope and Non-Goals

**In Scope:**
- Presenting the downgrade path and the cancellation path, each with an explanation of what is kept and what is lost
- Confirming a downgrade or cancellation request

**Non-Goals:**
- Executing the reversion to free at period end -- owned by FEAT-14.SPEC-008 (Apply Subscription Change), triggered by this screen's confirmation
- Determining the exact timing (end of current paid period, never mid-period) and the no-partial-refund rule -- governed by FEAT-14.SPEC-005 (Billing State & Refund Rules)
- Re-upgrading after a downgrade or cancellation -- owned by FEAT-14.SPEC-002 (Upgrade to Paid); this screen only ever moves the household toward free

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-14.SPEC-003 (Billing & Payment Management) | Maya taps "Downgrade to Free" | Path context: downgrade |
| FEAT-14.SPEC-003 (Billing & Payment Management) | Maya taps "Cancel Subscription" | Path context: cancellation |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | All actions: review and confirm downgrade or cancellation | -- |
| Sam (Other Adult Member) | No | No | No entry point to this screen exists; a direct link resolves to "You don't have access to billing for this household." |
| Jordan (young kid profile, no login -- MVP) | No | No | Screen is unreachable -- a no-login profile has no sign-in path to any screen |
| Jordan (older kid, limited login -- Later) | No | No | A direct link resolves to "You don't have access to billing for this household." |
| Riley (Operator, support -- from v1) | No | No | Riley's Billing access never extends to this screen; unreachable under any support scenario |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in, the user lands on FEAT-14.SPEC-001, not this screen |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue."; the chosen path (downgrade or cancellation) is preserved and this screen is restored after re-authentication, since no destructive action has occurred yet |

## Layout and Content

**Header:** Screen title "Downgrade to Free" or "Cancel Subscription" (matching the entered path) with a back arrow (returns to FEAT-14.SPEC-003).

**Body:**
- **What you keep / what you lose block**: identical content structure to the tier-inclusion summary on FEAT-14.SPEC-001 and FEAT-14.SPEC-002 per the Brief's Shared UI Patterns, framed for this screen as two labeled lists -- "You keep" (every past plan, rating, recipe, pantry item, and the shared list; manual weekly planning) and "You lose" (new AI-generated plans, pantry-aware plan weighting, and rating-based learning).
- **Timing explanation**: a plain-language statement that the change takes effect at the end of the current paid period (the exact date), and paid features remain active until then -- sourced from FEAT-14.SPEC-005.
- **No-refund note**: a plain-language line stating no partial refund is made for the unused remainder of the period, per FEAT-14.SPEC-005.
- **Confirmation control**: a single "Confirm downgrade" or "Confirm cancellation" button (matching the entered path).

**Footer:** None -- confirmation is inline in the body.

### Responsive Behavior

- **Compact breakpoint:** All blocks stack vertically, full width, one-thumb reachable per ASMP-29.
- **Medium size class and above:** "You keep" and "You lose" lists render side by side; the timing explanation, no-refund note, and confirmation control remain full width below.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-14.SPEC-003 (Billing & Payment Management) without confirming | Screen closes | Standard transition; no change is made |
| "What you keep / what you lose" block | -- | Display-only, non-interactive | None | -- |
| "Confirm downgrade" (downgrade path) | Tap | 1. Validate the household is currently paid with no change already pending, per FEAT-14.SPEC-005. 2. Trigger FEAT-14.SPEC-008 (Apply Subscription Change) to record the downgrade as pending against current_period_end_date; billing_state stays Active throughout. | Button shows loading state | Success: toast "Your household will move to the free tier on {current_period_end_date}. Paid features stay active until then." and navigation to FEAT-14.SPEC-001. Failure: inline error with retry |
| "Confirm cancellation" (cancellation path) | Tap | 1. Validate the household is currently paid with billing_state not already Cancelled, per FEAT-14.SPEC-005. 2. Trigger FEAT-14.SPEC-008 to set billing_state to Cancelled immediately and record the cancellation as pending against current_period_end_date. | Button shows loading state | Success: toast "Your subscription is cancelled. Paid features stay active until {current_period_end_date}, with no partial refund." and navigation to FEAT-14.SPEC-001. Failure: inline error with retry |

### Accessibility Notes

- **Focus order:** Back arrow -> "You keep" list -> "You lose" list -> timing explanation -> no-refund note -> confirmation button.
- **Dynamic announcements:** The success toast and any submission-failure banner are announced as live-region changes.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Reviewing (default) | Full explanation shown, confirmation button enabled | Screen opens from FEAT-14.SPEC-003 | Maya taps back or confirms |
| Confirming | Confirmation button shows a loading state | Maya taps "Confirm downgrade" or "Confirm cancellation" | Confirmation completes (success or failure) |
| Error | Banner: "Couldn't process your request. Check your connection and try again." with a Retry button; the review content remains unchanged | The confirmation request fails | Maya taps Retry and it succeeds |
| Offline/Degraded | Banner: "Changing billing requires a connection. Nothing has changed yet." Confirmation button is disabled | Connectivity is lost while this screen is open, or it is opened with no connectivity | Connectivity restored -- confirmation button re-enables |

## Validation Rules

Validation governed by FEAT-14.SPEC-005 (Billing State & Refund Rules). See that spec for the downgrade/cancellation timing rule and the condition that the household must currently be paid to reach a valid confirmation.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-14.SPEC-003 (Billing & Payment Management) | -- |
| Successful confirmation | FEAT-14.SPEC-001 (Plan Tier Overview) | -- |

## Data Model

**Creates:** None.
**Reads:** Subscription -- tier, billing_period, billing_state, pending_change, current_period_end_date (to confirm the household is currently paid, has no change already pending, and to compute the period-end date shown).
**Updates:** None directly -- the downgrade or cancellation is written by FEAT-14.SPEC-008, triggered from this screen.
**Deletes:** None.

## Business Rules

- A downgrade or cancellation always takes effect at the end of the current paid period, never mid-period, per FEAT-14.SPEC-005.
- A cancellation confirmation immediately sets billing_state to Cancelled (paid features remain active) and schedules the reversion to Reverted to free for current_period_end_date; a downgrade confirmation never changes billing_state during its pending window (it stays Active) and resolves to Active once tier reverts to free, per FEAT-14.SPEC-005 and FEAT-14.SPEC-008.
- No partial refund is made for the unused remainder of a period, per FEAT-14.SPEC-005 and product-features.md's Validation & Limits.
- XBR-05: every past plan, rating, recipe, pantry item, and the shared list stay fully available after the reversion completes -- nothing a free household already had is taken away. This is stated explicitly on this screen so the choice is fully informed, per the Non-Goals field's grounding in documented trust backlash from retroactive paywalling (Cozi, Trustpilot average 2.1/5, HIGH confidence).
- Only Maya reaches this screen, per FEAT-14.SPEC-006 (Tier & Billing Access Authorization).

## Edge Cases

- **Household's billing_state changes between load and confirmation (e.g., a grace-period reversion to free completes elsewhere while this screen is open)** -- Confirmation is rejected with refresh: "Your household is already on the free tier." and the screen redirects to FEAT-14.SPEC-001, consistent with the dependency map's Contention note for Subscription (reject-with-refresh against a stale billing state).
- **Maya taps Confirm twice in quick succession** -- The second tap is ignored while the first request is in progress; at most one downgrade or cancellation request is recorded per confirmation.
- **Maya backs out after reading the "what you lose" list** -- No change is made; tapping back returns her to FEAT-14.SPEC-003 with the subscription unchanged.
- **Maya cancels, then reconsiders before the period ends** -- Re-upgrading (FEAT-14.SPEC-002) before period end reverses the pending cancellation, since the household is still paid until then; this screen does not itself offer an "undo," but FEAT-14.SPEC-002 remains reachable throughout the remaining paid period.
- **Household has a pending period-switch request (from FEAT-14.SPEC-003) when Maya confirms a downgrade or cancellation here** -- The downgrade or cancellation supersedes the pending period switch; the switch never takes effect, since the household reverts to free before the next renewal it was scheduled for.
- **Maya reaches this screen via a stale link while a change is already pending for the household (e.g., billing_state is already Cancelled, or pending_change is already downgrade)** -- Confirmation is rejected with refresh: "Your household already has a change scheduled for {current_period_end_date}." and she is redirected to FEAT-14.SPEC-003, consistent with the dependency map's Contention note for Subscription.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-14.SPEC-003 (Billing & Payment Management) | Navigation (inbound) | "Downgrade to Free" and "Cancel Subscription" hand off to this screen |
| FEAT-14.SPEC-005 (Billing State & Refund Rules) | References (inbound) | Timing and no-refund rules enforced here |
| FEAT-14.SPEC-008 (Apply Subscription Change) | Triggers (outbound) | Confirmation triggers the recorded downgrade or cancellation |
| FEAT-14.SPEC-001 (Plan Tier Overview) | Navigation (outbound) | Successful confirmation returns here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| downgrade_or_cancel_reviewed | path (downgrade / cancel) | Screen finishes loading | supports success-metrics.md: "Paying Household Retention" |
| downgrade_confirmed | -- | Maya confirms a downgrade | supports success-metrics.md: "Paying Household Retention" |
| cancellation_confirmed | -- | Maya confirms a cancellation | supports success-metrics.md: "Paying Household Retention" |
| downgrade_or_cancel_abandoned | path | Maya taps back without confirming | N/A -- no Stage 2 metric measures abandonment of this flow directly; retained as a diagnostic signal for retention efforts |

## Acceptance Criteria

**FEAT-14.SPEC-004-AC-01:** Given Maya arrives on the downgrade path, when the screen loads, then she sees "You keep" listing past plans, ratings, recipes, pantry items, and the shared list, and "You lose" listing AI-generated plans, pantry-aware weighting, and learning.

**FEAT-14.SPEC-004-AC-02:** Given Maya arrives on the cancellation path, when the screen loads, then the title reads "Cancel Subscription" and the same what-you-keep/what-you-lose content is shown with cancellation-specific confirmation wording.

**FEAT-14.SPEC-004-AC-03:** Given Maya is on the downgrade path, when she taps "Confirm downgrade", then a toast reads "Your household will move to the free tier on {period end date}. Paid features stay active until then." and she is returned to FEAT-14.SPEC-001.

**FEAT-14.SPEC-004-AC-04:** Given Maya is on the cancellation path, when she taps "Confirm cancellation", then a toast reads "Your subscription is cancelled. Paid features stay active until {period end date}, with no partial refund." and she is returned to FEAT-14.SPEC-001.

**FEAT-14.SPEC-004-AC-05:** Given Maya taps back on either path, when she has not confirmed, then no change is made to the Subscription and she returns to FEAT-14.SPEC-003.

**FEAT-14.SPEC-004-AC-06:** Given Maya's household reverts to free from a grace-period expiry while this screen is open, when she taps Confirm, then the request is rejected with "Your household is already on the free tier." and she is redirected to FEAT-14.SPEC-001.

**FEAT-14.SPEC-004-AC-07:** Given Maya taps Confirm twice in quick succession, when the first request is still in progress, then the second tap has no effect and only one request is recorded.

**FEAT-14.SPEC-004-AC-08:** Given Maya loses connectivity on this screen, when she taps Confirm, then the banner "Changing billing requires a connection. Nothing has changed yet." appears and the button is disabled.

**FEAT-14.SPEC-004-AC-09:** Given the confirmation request fails for a reason other than connectivity, when the failure occurs, then the banner "Couldn't process your request. Check your connection and try again." appears with a Retry button.

**FEAT-14.SPEC-004-AC-10:** Given Sam attempts to reach this screen directly, when the attempt is made, then he sees "You don't have access to billing for this household." and no content is shown.

**FEAT-14.SPEC-004-AC-11:** Given Maya's session expires while she is reviewing this screen, when she re-authenticates, then she is restored to this screen on the same path (downgrade or cancellation) she was reviewing.

**FEAT-14.SPEC-004-AC-12:** Given Maya's household already has a change pending (billing_state Cancelled, or pending_change downgrade) when she reaches this screen via a stale link, when she attempts to confirm, then the request is rejected with "Your household already has a change scheduled for {current_period_end_date}." and she is redirected to FEAT-14.SPEC-003.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 3 | 3 |
| States | 4 (reviewing, confirming, error, offline) | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

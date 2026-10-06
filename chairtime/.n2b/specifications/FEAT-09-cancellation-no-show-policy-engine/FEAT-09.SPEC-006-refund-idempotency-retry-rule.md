---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-09.SPEC-006
spec_name: Refund Idempotency & Retry Rule
spec_slug: refund-idempotency-retry-rule
parent_feature: FEAT-09
parent_feature_name: Cancellation & No-Show Policy Engine
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 12
acceptance_criteria_count: 13
---

# Logic/Rule Spec: Refund Idempotency & Retry Rule

## Overview

**Name:** Refund Idempotency & Retry Rule
**ID:** FEAT-09.SPEC-006
**Type:** Logic/Rule
**Purpose:** Guarantees a refund completes exactly once per Deposit Transaction, retries automatically and indefinitely when it cannot complete immediately, and keeps the Pro's dashboard flag and the client's "in progress" status consistent with the true state until the refund resolves.
**Parent Feature:** FEAT-09 -- Cancellation & No-Show Policy Engine
**Governed Entity:** Deposit Transaction (refund-completion invariant: status Refunded / Refund in Progress and the retry state that governs the transition between them)

## Scope and Non-Goals

**In Scope:**
- The guarantee that at most one successful refund is ever recorded against a given Deposit Transaction, regardless of retries or duplicate delivery of the underlying capability's events
- The automatic, indefinite retry cadence applied when a refund request cannot complete immediately
- Keeping Talia's dashboard flag and Riley's "in progress" status consistent with the Deposit Transaction's true state throughout the retry period
- Resolution behavior when a retry succeeds, and when the underlying blocking condition (e.g., the Pro's payout account) is resolved by a Pro action mid-retry

**Non-Goals:**
- Determining that a refund is due, or deciding its amount -- owned by FEAT-09.SPEC-004 (Cancellation & No-Show Outcome Evaluation) and FEAT-09.SPEC-003 (Deposit Outcome Rules); this spec governs correctness *after* a refund has already been determined and requested
- Composing or sending the refund request itself, or defining the capability's inbound events -- owned by FEAT-09.SPEC-005 (Automatic Deposit Refund); this spec defines the guarantee that spec's request/response contract must uphold when a single attempt does not resolve
- Forfeiture correctness or the no-show/cancellation-inside-window path -- excluded per the Entity-Lifecycle Coverage Matrix: this spec governs only the refund path, mirroring the idempotency pattern FEAT-07.SPEC-004 applies to deposit capture, not the forfeiture path FEAT-11 owns
- A Pro-facing manual "retry now" control -- excluded per product-features.md's Error state, which states the refund "is retried automatically" with no manual action described; the Pro's only visible control is the attention flag itself, not a retry trigger

## Governed Entity

**Entity:** Deposit Transaction (refund-completion slice)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|--------------|
| Deposit Transaction (refund-attempt existence) | derived | Whether a refund attempt for a given Deposit Transaction has already succeeded -- this spec guarantees this fact is single-valued (succeeded at most once) |
| Deposit Transaction.status | enum | At this spec's stage: Refund in Progress -> Refunded. This spec guarantees the status set here is the true, final outcome of the one attempt that ultimately succeeds |
| Refund retry state (derived, not a dependency-map field) | derived | Whether a retry is currently scheduled, and when the next attempt will occur; exists only for the duration a Deposit Transaction sits at Refund in Progress |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-09.SPEC-005 | Automatic Deposit Refund | Applies this spec's idempotency key discipline when submitting a refund request, and calls this spec's retry loop whenever the capability reports it cannot complete a refund immediately |
| FEAT-12 (Pro Daily Schedule Dashboard) | Cross-feature | Displays the attention flag this spec keeps consistent with the Deposit Transaction's true state throughout the retry period |

## Field Validation Rules

No input fields exist on this spec's governed slice -- the refund-completion invariant is a system guarantee, never a value any role enters directly.

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| Deposit Transaction.status (Refund in Progress / Refunded) | Must never show as Refunded unless exactly one successful refund has been confirmed by the payment-processing capability for it, and must never remain Refund in Progress once that confirmation exists | Always | Continuously -- this is an invariant this spec's retry loop maintains, not a point-in-time check | N/A -- this is a system invariant, not a validated input | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| At-most-once refund | Deposit Transaction (refund-attempt existence), status | Once a Deposit Transaction reaches Refunded, no further refund request is ever submitted for it, by this spec's retry loop or by any resubmission arriving through FEAT-09.SPEC-005 | N/A -- structural guarantee, no client-facing error |
| Idempotency key discipline | Deposit Transaction (refund-attempt existence), the refund request itself | Every refund request FEAT-09.SPEC-005 submits carries an identifier tied to this specific refund attempt on this specific Deposit Transaction, so a resubmitted request (from a retry or a duplicate delivery) is recognized by the capability as the same attempt rather than a second refund | N/A -- structural guarantee at the integration boundary, not a client-facing error |
| Attention-flag consistency | Deposit Transaction.status, Talia's dashboard flag, Riley's "in progress" status | Both surfaces reflect the Deposit Transaction's true current status at all times during the retry period; neither is ever cleared before the true state changes, and neither ever shows "failed" while a retry is still pending | "A refund for {client name}'s cancelled booking is in progress and will complete automatically." (Talia); "Your deposit refund is in progress and will complete automatically." (Riley) |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View the refund's current status (in progress / refunded) | The Pro (Talia), The Client (Riley) | Talia: any of her own bookings; Riley: her own booking only | -- |
| View refund retry status for support purposes | Platform Operator (Support) | View-only, for the Pro account under an active help request | -- |
| Manually trigger a retry attempt | Any role | Never -- retries are always automatic, per product-features.md's Error state | No manual "retry now" control exists anywhere in the product for this action |
| Manually mark a stuck refund as resolved without the capability's confirmation | Any role | Never -- the Deposit Transaction only moves to Refunded on the capability's own confirmation | No control anywhere lets any role force the status to Refunded; if Talia believes a refund is taking too long, her only path is contacting support, which surfaces the true status, never a manual override |
| Cancel or abandon a pending automatic refund | Any role | Never -- once a refund is due (FEAT-09.SPEC-004), it is retried until it completes; XBR-10 states refunds are "never dropped" | No control exists to cancel or abandon an in-progress automatic refund |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Retry schedule | Retried automatically on a fixed cadence (platform parameter: `refund-retry-interval-hours`) after a not-yet-completable report, continuing indefinitely until the capability confirms success | Whenever a Deposit Transaction sits at Refund in Progress | No |
| Talia's dashboard flag, on screen load | Derived from the Deposit Transaction's true current status at the moment the dashboard loads: shown if status is Refund in Progress, cleared if status is Refunded | Whenever FEAT-12 loads or re-loads | No -- always read fresh from the true state, never cached |
| Riley's "in progress" status, on screen load | Derived identically from the Deposit Transaction's true current status | Whenever her booking view loads or re-loads | No |

## Business Rules

- **Never refund twice:** at most one successful refund is ever recorded against a Deposit Transaction, however many retries occur; a retry that resubmits a request already fulfilled is recognized as the same attempt by the capability rather than processed as a second refund (idempotency key discipline).
- **Never leave an ambiguous outcome:** every refund attempt resolves, from the product's point of view, to exactly one of Refunded or still-in-progress. There is no third "failed and abandoned" state -- XBR-10 requires refunds to be "never dropped," so a not-yet-completable report always leads to another scheduled retry, never a terminal failure state.
- **Automatic, indefinite retry until resolution, never after:** once Refund in Progress, the retry loop continues on the fixed cadence (platform parameter: `refund-retry-interval-hours`) until the capability confirms success; once Refunded, no further retry of that Deposit Transaction ever occurs.
- **Surfaces are always the true state, never assumed:** Talia's dashboard flag and Riley's "in progress" status are both derived fresh from the Deposit Transaction's current status on every load, never from a cached or previously-shown value, so neither party is ever shown a stale "in progress" after the refund has in fact completed, or a stale "refunded" before it has.
- **Consistent with FEAT-07.SPEC-004's idempotency pattern:** this spec mirrors, for the refund path, the same never-double-process and never-ambiguous-outcome guarantees FEAT-07.SPEC-004 applies to deposit capture, using the same idempotency-key discipline at the capability boundary.

## Edge Cases

- **The Pro's payout account is reconnected mid-retry (the blocking condition resolves before the next scheduled attempt)** -- The next scheduled retry (per the fixed cadence) picks up the now-resolved payout account automatically; no separate trigger is needed, and the refund completes on that attempt without any manual action from Talia.
- **The capability reports success for a retry attempt the product had scheduled, but a different retry attempt for the same Deposit Transaction is also in flight (overlapping retries)** -- Only one can result in a Refunded status; the other, whichever resolves second, finds the Deposit Transaction already Refunded (via the idempotency key at the capability boundary) and is treated as a no-op with no duplicate refund and no duplicate confirmation content.
- **Talia views her dashboard while a refund is mid-retry, then reloads moments after it succeeds** -- The reload shows the true current state (Refunded, flag cleared); she is never shown a stale "in progress" flag after the underlying refund has actually completed.
- **Riley closes her booking view while a refund is in progress and reopens it days later** -- Her view re-derives the current status fresh on reopen; if it has since completed, she sees it reflected as refunded, never a stale "in progress."
- **A refund stays at Refund in Progress for an extended period because the underlying blocking condition never resolves** -- The retry loop continues indefinitely on its fixed cadence; the Deposit Transaction is never silently abandoned or moved to a terminal failure state, and Talia's attention flag persists for as long as the condition remains unresolved, consistent with XBR-10's "never dropped" guarantee.
- **The payment-processing capability reports a very late refund success for a retry attempt the product's own retry loop had already superseded with a newer attempt** -- The late success is still applied if no successful refund has yet been recorded for the Deposit Transaction; if a different attempt already succeeded in the meantime, the late report is treated as the duplicate-success case and produces no double refund or double confirmation.

## Acceptance Criteria

**FEAT-09.SPEC-006-AC-01:** Given a refund cannot complete on its first attempt, when the retry loop's next scheduled attempt runs (platform parameter: `refund-retry-interval-hours` after the previous attempt), then a new refund request is submitted carrying the same attempt's idempotency key.

**FEAT-09.SPEC-006-AC-02:** Given a Deposit Transaction is already Refunded, when any further retry logic runs for that same attempt, then no further refund request is submitted -- only the already-resolved state is reflected on any surface that reads it.

**FEAT-09.SPEC-006-AC-03:** Given two overlapping retry attempts for the same Deposit Transaction are both processed, when both resolve, then only one results in a Refunded status, and the other is treated as a no-op with no duplicate refund.

**FEAT-09.SPEC-006-AC-04:** Given Talia reconnects her payout account while a refund sits at Refund in Progress, when the next scheduled retry runs, then the refund completes automatically with no separate action required from Talia beyond having reconnected the account.

**FEAT-09.SPEC-006-AC-05:** Given Talia views her dashboard while a refund is Refund in Progress, then she sees "A refund for {client name}'s cancelled booking is in progress and will complete automatically."

**FEAT-09.SPEC-006-AC-06:** Given Talia reloads her dashboard immediately after a refund has completed, then the attention flag is cleared and no stale "in progress" state is shown.

**FEAT-09.SPEC-006-AC-07:** Given Riley views her booking while her refund is Refund in Progress, then she sees "Your deposit refund is in progress and will complete automatically." -- never a failure message.

**FEAT-09.SPEC-006-AC-08:** Given Riley reopens her booking view days after her refund actually completed, then she sees it reflected as refunded, never a stale "in progress" state.

**FEAT-09.SPEC-006-AC-09:** Given any role looks for a manual "retry now" control for a refund in progress, then none exists anywhere in the product.

**FEAT-09.SPEC-006-AC-10:** Given any role looks for a way to manually mark a stuck refund as resolved, then no such control exists -- the status changes only on the capability's own confirmation.

**FEAT-09.SPEC-006-AC-11:** Given any role looks for a way to cancel or abandon an in-progress automatic refund, then no such control exists anywhere in the product.

**FEAT-09.SPEC-006-AC-12:** Given a refund's blocking condition never resolves, when successive scheduled retries continue to fail, then the Deposit Transaction remains Refund in Progress indefinitely rather than moving to any terminal failure state, and Talia's attention flag persists throughout.

**FEAT-09.SPEC-006-AC-13:** Given a very late refund-success report arrives for an attempt the retry loop had already superseded, when a different attempt already succeeded in the meantime, then the late report produces no second refund and no duplicate confirmation.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 1 | 1 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

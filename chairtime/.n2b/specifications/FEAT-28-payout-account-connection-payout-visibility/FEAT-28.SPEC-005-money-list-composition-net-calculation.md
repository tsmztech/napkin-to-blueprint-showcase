---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-28.SPEC-005
spec_name: Money List Composition & Net Calculation
spec_slug: money-list-composition-net-calculation
parent_feature: FEAT-28
parent_feature_name: Payout Account Connection & Payout Visibility
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 24
acceptance_criteria_count: 20
---

# Logic/Rule Spec: Money List Composition & Net Calculation

## Overview

**Name:** Money List Composition & Net Calculation
**ID:** FEAT-28.SPEC-005
**Type:** Logic/Rule
**Purpose:** Defines how deposits, refunds, processor fees, and payouts are assembled into the money list and how net amount received per period is derived.
**Parent Feature:** FEAT-28 -- Payout Account Connection & Payout Visibility
**Governed Entity:** Money List row (a derived, read-only composition over Deposit Transaction, Balance Payment, and the Payout Account's reported payouts -- not a stored entity of its own)

## Scope and Non-Goals

**In Scope:**
- Row-composition rules for each money-list row type (deposit, refund, processor fee display, payout)
- Ordering, grouping, and the "in progress" display rule for a refund that has not yet cleared
- The net-amount-received-per-period derivation
- Authorization rules for who may view the composed list

**Non-Goals:**
- Creating or updating the underlying Deposit Transaction, Balance Payment, or Payout Account records -- owned by FEAT-07, FEAT-09, FEAT-30, FEAT-22, and FEAT-28.SPEC-003 respectively; this spec only reads and composes them for display
- The screen that renders the composed list -- owned by FEAT-28.SPEC-002 (Payout Status & Money Dashboard); this spec supplies the assembled data, not the layout
- Determining Payout Account eligibility or the zero-Chairtime-fee standing rule itself -- owned by FEAT-28.SPEC-004; this spec applies that rule when composing a deposit row's deduction, it does not define the rule
- Partial or tiered refund display -- excluded per SC-18: the underlying cancellation rule is binary, so no partial-percentage refund row is ever composed

## Governed Entity

**Entity:** Money List row -- a derived, read-only view composed at display time from three source entities. No independent field list of its own; instead, each row type's source fields are enumerated below.

**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| Deposit Transaction.amount / currency | number / enum | The captured deposit amount and currency |
| Deposit Transaction.status | enum | Authorized \| Captured \| Applied \| Refunded \| Refund in Progress \| Forfeited \| Disputed |
| Deposit Transaction.processor_fee | number | The processor's own card fee on this transaction |
| Deposit Transaction.outcome_reason / timestamps | text / date | Which rule or action produced the current status, and when |
| Balance Payment.amount | number | Price minus deposit, from v1 (FEAT-22) |
| Balance Payment.tip | number | Optional, non-negative, Later (FEAT-23) |
| Balance Payment.state | enum | Attempted \| Succeeded \| Failed \| Refunded |
| Payout Account.payout_schedule | derived | The processor's reported payout cadence |
| Payout Account.recent payouts | derived | The processor's reported list of recent bank transfers (amount, date, status) |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-28.SPEC-002 | Payout Status & Money Dashboard | On every money list load and refresh; authorization on screen entry |
| FEAT-25 | Booking & Revenue Insights | Reads this spec's composed data and net derivation once that feature exists |

## Field Validation Rules

Every source field is written by its owning feature (FEAT-07, FEAT-09, FEAT-30, FEAT-22, FEAT-28.SPEC-003) under that feature's own validation rules; this spec only composes and derives from already-valid data, so no field below carries composition-time validation of its own beyond confirming it is present. The Cross-Field Rules section governs how these fields combine into displayed rows, which is where this spec's own logic lives.

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| Deposit Transaction.amount / currency | No validation beyond data type -- validity is owned by FEAT-07.SPEC-003 at capture time | Always | -- | -- | -- |
| Deposit Transaction.status | No validation beyond data type -- validity is owned by FEAT-07, FEAT-09, FEAT-11, FEAT-30 as they write it | Always | -- | -- | -- |
| Deposit Transaction.processor_fee | Treated as zero for display if missing or null (see Edge Cases), rather than failing composition | When the field is absent | On composition | N/A -- not a user-facing validation error; a display fallback, not a blocking failure | No |
| Deposit Transaction.outcome_reason / timestamps | No validation beyond data type | Always | -- | -- | -- |
| Balance Payment.amount / tip / state | No validation beyond data type -- validity is owned by FEAT-22 (and FEAT-23 for tip) | Always | -- | -- | -- |
| Payout Account.payout_schedule | No validation beyond data type -- shown exactly as the processor reports it | Always | -- | -- | -- |
| Payout Account.recent payouts | No validation beyond data type -- shown exactly as the processor reports it | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Deposit row composition | Deposit Transaction.amount, processor_fee, status | Every Deposit Transaction in status Captured, Applied, Refunded, Refund in Progress, Forfeited, or Disputed produces exactly one deposit row, showing the gross amount, the processor_fee as the only deduction, and the net (amount minus processor_fee); a Deposit Transaction still Authorized (not yet captured) produces no row | N/A -- display composition, not a user-facing error |
| Refund row composition | Deposit Transaction.status, outcome_reason, timestamps | A Deposit Transaction whose status is Refunded or Refund in Progress additionally produces a linked refund row beneath its deposit row, showing the refunded amount, the refund date (if Refunded) or "in progress" (if Refund in Progress), and referencing the original deposit row | N/A |
| Forfeited deposit display | Deposit Transaction.status = Forfeited | A forfeited deposit shows no refund row; the deposit row itself shows "Kept" as its running status, per the binary outcome rule (SC-18) | N/A |
| Disputed deposit display | Deposit Transaction.status = Disputed | The deposit row (and its refund row, if any) carries a "Disputed" overlay alongside its existing status, per XBR-22; the underlying outcome (kept, refunded, or refund in progress) remains visible beneath the overlay | N/A |
| Balance payment row composition (from v1) | Balance Payment.amount, tip, state | Once FEAT-22 exists, a Balance Payment in state Succeeded produces a balance-payment row; a Refunded state additionally produces a linked refund row, following the same pattern as deposits | N/A |
| Payout row composition | Payout Account.recent payouts | Each entry in the processor's reported recent payouts produces one payout row (amount, date, status: upcoming or completed); no Chairtime-side computation of payout amounts occurs -- the processor's own reported figures are shown as-is | N/A |
| Net-per-period derivation | Deposit Transaction.amount, processor_fee, status; Balance Payment.amount, tip, state (from v1) | Net amount received for a selected period = sum of (amount minus processor_fee) for every deposit row with status Captured, Applied, or Refund in Progress dated within the period, minus the amount of every completed refund dated within the period, plus (from v1) the net of any Balance Payment rows in the same period; a Forfeited deposit's full amount minus its processor_fee counts toward net, since it is kept | N/A |
| Zero-Chairtime-fee enforcement at composition | Deposit Transaction.processor_fee | The composition logic has no field or code path that renders a Chairtime-owned fee on any row, per XBR-07 and FEAT-28.SPEC-004's standing rule | N/A |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|----------------------------------------------|
| View composed money list | The Pro (Talia) | Own Payout Account's data only, always | -- |
| View composed money list | Platform Operator (Support) | Same composed data Talia sees (Access Matrix, Payouts = View), never bank or identity details (none of which are among this spec's source fields in any case) | -- |
| View composed money list | The Client (Riley) | Never -- Clients have no access to Payouts at all | The money list is not reachable by any Client navigation path |
| Trigger recomposition (open the money list / change the period selector) | The Pro, Platform Operator (Support) | Always, read-only action | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Deposit row net amount | amount minus processor_fee | Computed at every composition | No |
| Net amount received per period | See the net-per-period cross-field rule above | Recomputed whenever the money list loads or the period selector changes | Yes -- the Pro (and Support, view-only) may change the selected period, which recomputes the figure; the underlying rows never change |
| Default period | This month, from the first day of the current calendar month in the Pro's timezone (XBR-25) to the current date | On money list load, before any period change | Yes -- changeable via the period selector on FEAT-28.SPEC-002 |
| Refund row's "in progress" label | Derived directly from Deposit Transaction.status = Refund in Progress | Whenever a linked refund row is composed | No -- the label always mirrors the source status exactly |

## Business Rules

- XBR-07: Chairtime's own fee is always zero; the only deduction this composition ever shows on any row is the payment processor's own card fee, per FEAT-28.SPEC-004's standing rule.
- FEAT-23.SPEC-003 (Tip Payout & Refund Rule) is enforced here: a succeeded Balance Payment's `tip` contributes in full to its row and to the net-per-period figure, with no fee line deducted.
- XBR-10: a refund that cannot complete immediately is shown as "in progress," never as failed or silently dropped, and the underlying retry (owned by FEAT-09/FEAT-30) is what eventually moves the row to its completed refund state.
- SC-18: the cancellation rule producing these outcomes is binary (refunded or kept); the composition never renders a partial-percentage refund row.
- XBR-22: a Disputed overlay never erases the underlying outcome; the composition shows both together.
- ASMP-22 / ASMP-27: the composition remains equally responsive as history accumulates over multiple years, and every load shows an in-place loading indicator with nothing tappable until real data has loaded, matching FEAT-28.SPEC-002's States section.

## Edge Cases

- **A Deposit Transaction transitions from Refund in Progress to Refunded while the money list is open** -- The linked refund row updates from "in progress" to its completed status and date on the list's next refresh (FEAT-28.SPEC-002), and the net-per-period figure recalculates to include it if the refund date falls in the selected period.
- **A deposit and its refund fall in different periods (deposit last month, refund this month)** -- Each event counts toward the net figure of the period its own date falls in; the deposit's full net (minus fee) counted toward last month's total is not retroactively reversed -- this month's total instead reflects the refund as a negative entry.
- **A Deposit Transaction is Disputed while also Refund in Progress** -- Both overlays compose onto the same row: the "in progress" refund status and the "Disputed" overlay are shown together, since a dispute never erases the underlying refund-in-progress outcome (XBR-22).
- **Balance Payment rows before FEAT-22 ships** -- No Balance Payment rows or net contribution are composed at all; the money list and net figure are computed entirely from Deposit Transaction data until FEAT-22 exists, consistent with the Brief's Non-Goals.
- **The processor reports a recent payout for an amount that does not match the sum of the Deposit Transaction rows composed for that period** -- The payout row is shown exactly as the processor reports it (amount, date, status), since the processor's own payout timing may span multiple periods or batching windows; no reconciliation or discrepancy flag is computed by this spec, and the payout row and the deposit/refund rows are never forced to reconcile to the same total.
- **The money list is opened for a Pro with multiple years of history (per ASMP-22)** -- Composition and the net-per-period derivation operate only over the selected period's rows plus the always-shown full history list; performance does not degrade as total history grows, since the period derivation never requires scanning the Pro's entire multi-year history to compute a shorter period's figure.

## Acceptance Criteria

**FEAT-28.SPEC-005-AC-01:** Given a Deposit Transaction is Captured, when the money list composes, then a deposit row shows the gross amount, the processor's own fee, and the net (amount minus fee).

**FEAT-28.SPEC-005-AC-02:** Given a Deposit Transaction is Authorized but not yet Captured, when the money list composes, then no row is shown for it.

**FEAT-28.SPEC-005-AC-03:** Given a Deposit Transaction is Refunded, when the money list composes, then a linked refund row appears beneath the deposit row showing the refunded amount and refund date.

**FEAT-28.SPEC-005-AC-04:** Given a Deposit Transaction is Refund in Progress, when the money list composes, then the linked refund row shows "in progress," never a failure state.

**FEAT-28.SPEC-005-AC-05:** Given a Deposit Transaction is Forfeited, when the money list composes, then the deposit row shows "Kept" and no refund row is composed.

**FEAT-28.SPEC-005-AC-06:** Given a Deposit Transaction is Disputed and also Refund in Progress, when the money list composes, then the row shows both the "in progress" refund status and the "Disputed" overlay together.

**FEAT-28.SPEC-005-AC-07:** Given the processor reports a recent payout, when the money list composes, then a payout row shows the amount, date, and status (upcoming or completed) exactly as reported.

**FEAT-28.SPEC-005-AC-08:** Given Talia opens the money list with the default period, when the net figure is computed, then it covers from the first day of the current calendar month (her timezone) to today.

**FEAT-28.SPEC-005-AC-09:** Given Talia changes the period selector, when the new period is applied, then the net figure recomputes for that period and the full money list itself is unaffected.

**FEAT-28.SPEC-005-AC-10:** Given a deposit is captured in one period and refunded in a later period, when both periods' net figures are computed, then the deposit's net counts toward the period it was captured in and the refund counts as a negative entry in the period it completed in.

**FEAT-28.SPEC-005-AC-11:** Given a deposit row is composed, when its deduction is displayed, then only the processor's own card fee ever appears -- never a Chairtime fee line.

**FEAT-28.SPEC-005-AC-12:** Given FEAT-22 does not yet exist for this Pro, when the money list composes, then no Balance Payment rows or net contribution appear.

**FEAT-28.SPEC-005-AC-13:** Given the Client (Riley) attempts to view any Pro's money list, when the attempt is made, then no such access path exists for her role.

**FEAT-28.SPEC-005-AC-14:** Given Support views a Pro's money list, when composed, then Support sees exactly the same rows Talia sees, since none of this spec's source fields are bank or identity data.

**FEAT-28.SPEC-005-AC-15:** Given the processor's reported recent payout amount does not match the sum of a period's deposit rows, when the money list composes, then the payout row is shown exactly as reported with no reconciliation discrepancy flag.

**FEAT-28.SPEC-005-AC-16:** Given Talia's history spans multiple years, when she selects a one-month period, then the net figure computes only over that period's rows without scanning her full history.

**FEAT-28.SPEC-005-AC-17:** Given a refund transitions from in progress to completed while the list is open, when the list next refreshes, then the row updates and the net-per-period figure recalculates to reflect it if the refund date falls in the selected period.

**FEAT-28.SPEC-005-AC-18:** Given a cancellation rule produces a refund outcome, when the money list composes it, then only a full refund or a kept deposit is ever shown -- never a partial-percentage row, per SC-18.

**FEAT-28.SPEC-005-AC-19:** Given a Deposit Transaction's processor_fee field somehow carries a null or missing value, when the money list composes its row, then the row still renders with the amount and net shown as the amount itself (fee treated as zero for display), rather than the row failing to compose.

**FEAT-28.SPEC-005-AC-20:** Given a Deposit Transaction that is Disputed but not refunded or forfeited, when the money list composes, then the deposit row shows its existing status (e.g., Captured) with the Disputed overlay, and the underlying outcome remains visible beneath it.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 7 | 7 |
| Cross-Field Rules | 8 | 8 |
| Authorization Rules | 4 | 4 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

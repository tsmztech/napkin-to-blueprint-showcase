---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-23.SPEC-002
spec_name: Tip Amount Validation
spec_slug: tip-amount-validation
parent_feature: FEAT-23
parent_feature_name: Tipping at Checkout
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 4
acceptance_criteria_count: 9
---

# Logic/Rule Spec: Tip Amount Validation

## Overview

**Name:** Tip Amount Validation
**ID:** FEAT-23.SPEC-002
**Type:** Logic/Rule
**Purpose:** Governs the tip amount's own constraints on the Balance Payment record's `tip` field -- non-negative when given, valid as a monetary amount, and never pre-selected to a default -- independent of the balance amount's own rules, which FEAT-22 owns.
**Parent Feature:** FEAT-23 -- Tipping at Checkout
**Governed Entity:** Balance Payment (the `tip` field only)

## Scope and Non-Goals

**In Scope:**
- Field validation for the `tip` field: format, sign, and when it is checked
- The rule that `tip` is never pre-selected to a default value
- Authorization for who may set, skip, or view a tip amount

**Non-Goals:**
- Validation of the Balance Payment's `amount` field or eligibility preconditions -- owned by FEAT-22.SPEC-003 (Balance Amount & Eligibility Rules); this spec addresses only `tip`.
- Balance Payment state-transition consistency (Attempted / Succeeded / Failed) -- owned by FEAT-22.SPEC-004 (Balance Payment Outcome Consistency & Cancellation Contention).
- Where a validated tip's money goes and its refund behavior on cancellation -- owned by FEAT-23.SPEC-003 (Tip Payout & Refund Rule), which assumes this spec's validation has already passed, per feature-overview.md's Shared Validation section.
- Imposing an upper bound on the tip amount -- excluded because product-features.md's Validation & Limits field for this feature defines only a non-negative floor and the no-default rule; no maximum is named in Stage 2, so none is invented here.

## Governed Entity

**Entity:** Balance Payment
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| amount | number | Price minus deposit, not client-alterable -- **out of scope for this spec**; validated by FEAT-22.SPEC-003 |
| tip | number | Optional, non-negative, never pre-selected -- **governed by this spec** |
| state | enum | Attempted \| Succeeded \| Failed \| Refunded -- **out of scope for this spec**; owned by FEAT-22.SPEC-002 and FEAT-22.SPEC-004 (transitions), and FEAT-23.SPEC-003 (refund behavior tied to the Refunded transition) |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-23.SPEC-001 | Tip Selection | On the tip amount input's blur, and again when FEAT-22.SPEC-001's Pay action is tapped (the tip value is validated before being handed to that submission) |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| tip | Must be a valid monetary amount in the Pro's account currency (digits and at most one decimal separator, up to 2 decimal places; no letters, currency symbols, or other characters) | When the field is non-empty | On blur, and on the host screen's Pay action | "Enter a valid amount." | Yes |
| tip | Must be non-negative (zero or greater) | When the field is non-empty | On blur, and on the host screen's Pay action | "Tip amount can't be negative." | Yes |
| tip | May be left entirely empty (null) -- absence of a tip is always valid and never itself an error | Always | On blur, and on the host screen's Pay action | -- (no error; empty is a valid, final state, not a pending one) | No |
| amount | No validation beyond data type in this spec | Always | -- | -- (owned by FEAT-22.SPEC-003) | -- |
| state | No validation beyond data type in this spec | Always | -- | -- (owned by FEAT-22.SPEC-002 / FEAT-22.SPEC-004) | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| No cross-field dependency | tip, amount | The `tip` field's validity and value are entirely independent of `amount`: a tip is checked against its own rules regardless of the balance amount, and the balance amount's computation (FEAT-22.SPEC-003) never reads or is altered by `tip` | N/A -- this is a structural independence guarantee, not a rejectable condition |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Set or edit the tip amount on an in-progress balance payment | The Client (Riley) | Only on their own booking's in-progress balance payment (Own-only, per the Access Matrix's Booking & Payment row) | -- |
| Set or edit the tip amount on an in-progress balance payment | The Pro (Talia) | Never | No control exists in any Pro-facing screen to set or edit a tip on a Client's balance payment; the tip is client-chosen only, per product-features.md's Access field for this feature |
| Set or edit the tip amount on an in-progress balance payment | Platform Operator (Support) | Never | No edit control exists in the Support view; Support's access to Booking & Payment is View-only, per the Access Matrix |
| Skip tipping entirely | The Client (Riley) | Always, on their own balance payment | -- |
| View a tip amount already recorded on a completed Balance Payment | The Pro (Talia) | Always, on their own bookings (Full, Booking & Payment) | -- |
| View a tip amount already recorded on a completed Balance Payment | The Client (Riley) | Only their own booking's Balance Payment (Own-only) | -- |
| View a tip amount already recorded on a completed Balance Payment | Platform Operator (Support) | Always, status/amount only, never card data (View, per the Access Matrix) | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| tip | None -- the field starts unset with no default value or pre-selected amount of any kind, per product-features.md's Validation & Limits: "never pre-selected to a default that could feel presumptive" | Every time the tip step (FEAT-23.SPEC-001) is shown, on each fresh balance payment attempt | N/A -- no default exists to override; the Client enters the value directly, or leaves it unset |

## Business Rules

- Field validation on `tip` runs before the value is handed to FEAT-22.SPEC-001's own payment submission -- an invalid tip blocks that submission, per FEAT-23.SPEC-001's Interactions.
- Validation on `tip` applies identically whether the client is entering a first attempt or retrying after a decline -- the product definition establishes no retry-only or first-attempt-only rule for the tip field.
- An empty `tip` field is a fully valid, final choice (equivalent to skipping), never a pending or incomplete state that blocks submission.
- This spec's rules govern the field's own constraints only; XBR-07 (zero platform cut) and XBR-23 (full refund with the balance) are enforced by FEAT-23.SPEC-003, not restated here.

## Edge Cases

- **Tip entered as exactly "0"** -- Passes validation (zero is non-negative). Functionally equivalent to leaving the field empty: no tip is recorded on the Balance Payment, so a client who deliberately types "0" and one who skips entirely produce the same outcome.
- **Tip entered with a negative sign ("-5")** -- Fails validation with "Tip amount can't be negative." before the value ever reaches the host screen's payment submission.
- **Tip entered with more than 2 decimal places ("5.999")** -- Fails validation with "Enter a valid amount."
- **Tip entered as a very large number (e.g., far exceeding the balance amount)** -- Passes validation; no upper bound is defined in Stage 2 for this feature, so none is imposed here (see Non-Goals).
- **Tip field left as whitespace only** -- Treated the same as empty: valid, no tip recorded.
- **Client clears a previously valid tip amount back to empty before submitting** -- Field re-validates as empty (valid, no error); the "no tip" outcome applies exactly as if the client had never typed anything.
- **A Pro attempts, through any means outside the product's own screens, to influence a Client's in-progress tip amount** -- Out of scope for this spec: no Pro-facing control exists to do so (see Authorization Rules); this scenario has no in-product surface to specify further.

## Acceptance Criteria

**FEAT-23.SPEC-002-AC-01:** Given Riley enters "10" in the tip amount input, when she moves focus away, then the value passes validation with no error shown.

**FEAT-23.SPEC-002-AC-02:** Given Riley enters "-3" in the tip amount input, when she moves focus away, then the field shows the error "Tip amount can't be negative." and the balance payment is not submitted while the field remains invalid.

**FEAT-23.SPEC-002-AC-03:** Given Riley enters "12.999" in the tip amount input, when she moves focus away, then the field shows the error "Enter a valid amount."

**FEAT-23.SPEC-002-AC-04:** Given Riley leaves the tip amount input completely empty, when she taps the host screen's Pay action, then no validation error is shown, and the balance payment proceeds with no tip.

**FEAT-23.SPEC-002-AC-05:** Given Riley enters exactly "0" as her tip amount, when validation runs, then it passes as non-negative, and the resulting Balance Payment records no tip, identically to leaving the field empty.

**FEAT-23.SPEC-002-AC-06:** Given the tip amount step first renders for Riley, when she looks at the input, then it shows no pre-filled or highlighted amount of any kind.

**FEAT-23.SPEC-002-AC-07:** Given Talia (the Pro) is signed in to her own account, when she looks for any control to set or edit a tip on a Client's balance payment, then no such control exists anywhere in her account.

**FEAT-23.SPEC-002-AC-08:** Given Platform Operator (Support) is viewing a Pro's account, when they look for a way to edit a tip amount, then no edit control exists -- their access to Booking & Payment is View-only.

**FEAT-23.SPEC-002-AC-09:** Given Riley's balance payment already includes a valid tip of "15" and she changes it to "8" before submitting, then the amount field re-validates against the new value, and "8" is the amount handed to the payment submission if it passes.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 3 | 3 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 7 | 7 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |

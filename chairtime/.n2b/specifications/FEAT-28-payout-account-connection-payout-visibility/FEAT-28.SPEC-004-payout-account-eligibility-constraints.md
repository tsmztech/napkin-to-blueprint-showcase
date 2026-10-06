---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-28.SPEC-004
spec_name: Payout Account Eligibility & Constraints
spec_slug: payout-account-eligibility-constraints
parent_feature: FEAT-28
parent_feature_name: Payout Account Connection & Payout Visibility
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 22
acceptance_criteria_count: 19
---

# Logic/Rule Spec: Payout Account Eligibility & Constraints

## Overview

**Name:** Payout Account Eligibility & Constraints
**ID:** FEAT-28.SPEC-004
**Type:** Logic/Rule
**Purpose:** Governs one-payout-account-per-Pro, country/currency matching to the Pro Account, and the standing zero-Chairtime-fee rule that the money list and go-live gate both depend on.
**Parent Feature:** FEAT-28 -- Payout Account Connection & Payout Visibility
**Governed Entity:** Payout Account

## Scope and Non-Goals

**In Scope:**
- Field-level rules for every Payout Account field
- The one-account-per-Pro-Account constraint
- The country/currency match rule against the Pro Account (XBR-25)
- The standing zero-Chairtime-fee rule (XBR-07) that governs every deduction ever shown against this entity
- Authorization rules for every action on the Payout Account, per role

**Non-Goals:**
- Writing the Payout Account's status field from processor reports -- owned by FEAT-28.SPEC-003 (Payout Account Status Processing); this spec defines the eligibility gate that automation applies at creation, not the write itself
- Composing the money list or deriving net-per-period -- owned by FEAT-28.SPEC-005 (Money List Composition & Net Calculation); this spec only fixes the zero-fee rule that spec's derivation must respect
- The visual treatment of the eligibility rejection message -- owned by FEAT-28.SPEC-001 and FEAT-28.SPEC-006, which display the exact denied text this spec defines
- Timezone rules -- owned by FEAT-27 (Pro Profile & Booking Page Settings) under XBR-25; this spec only checks that the Payout Account's currency matches the Pro Account's, not timezone

## Governed Entity

**Entity:** Payout Account
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| processor_account_reference | text | Reference to the Pro's account with the payment-processing capability; bank and identity details stay with the capability |
| status | enum | Not Connected \| Verification Pending \| Active \| Action Required \| Disconnected |
| country | enum | Must match the Pro Account's country |
| currency | enum | Must match the Pro Account's currency |
| payout_schedule | derived | The processor's own reported payout cadence -- no validation beyond data type |
| recent payouts | derived | The processor's own reported list of recent bank transfers -- no validation beyond data type |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-28.SPEC-001 | Payout Account Connection | On the first connection handoff; authorization on screen entry |
| FEAT-28.SPEC-002 | Payout Status & Money Dashboard | On every display of status and money-list deductions; authorization on screen entry |
| FEAT-28.SPEC-003 | Payout Account Status Processing | On every candidate Payout Account creation, before writing the record |
| FEAT-28.SPEC-006 | Payout Account Connection & Verification | On the outbound handoff and on the inbound eligibility rejection from the capability |
| FEAT-07 | Deposit Payment at Booking | Reads the Active-status precondition before any deposit charge (XBR-06) |
| FEAT-15 | Pro Onboarding & Setup Wizard | Reads the Active-status precondition for the go-live check (XBR-06, XBR-26) |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| processor_account_reference | Required, non-empty; set only from the capability's own report, never entered by the Pro | Always, once a Payout Account exists | On Payout Account creation | N/A -- not a user-facing field; no direct entry point exists for it | Yes |
| status | Must be one of the five defined enum values | Always | On every write (FEAT-28.SPEC-003) | N/A -- not user-entered; an unrecognized value from the capability is treated as a processing error, not a validation failure shown to the Pro | Yes |
| country | Must exactly match the Pro Account's country at the time of connection | Always, checked at Payout Account creation | On Payout Account creation (via the handoff outcome) | "Your payout account's country doesn't match your Chairtime account's country. Payout accounts must be in the same country you signed up with." | Yes |
| currency | Must exactly match the Pro Account's currency at the time of connection | Always, checked at Payout Account creation | On Payout Account creation (via the handoff outcome) | "Your payout account's currency doesn't match your Chairtime account's currency. Payout accounts must use the same currency as your Chairtime account." | Yes |
| payout_schedule | No validation beyond data type | Always | -- | -- | -- |
| recent payouts | No validation beyond data type | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| One Payout Account per Pro Account | Pro Account reference (implicit), Payout Account existence | A Payout Account can be created for a given Pro Account only when no Payout Account already exists for it | "You already have a connected payout account. Contact support if you need to change it." (shown only in the unreachable case this check is ever triggered outside the normal single-connection flow, since FEAT-28.SPEC-001 is never re-shown once a Payout Account exists) |
| Country/currency match to Pro Account | country, currency, Pro Account.country, Pro Account.currency | Both country and currency on the Payout Account must equal the Pro Account's values at the moment of connection | See Field Validation Rules above (separate messages per field, shown together if both mismatch) |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|----------------------------------------------|
| Connect a Payout Account | The Pro (Talia) | Only when no Payout Account yet exists for this Pro Account | FEAT-28.SPEC-001 is not shown a second time once a Payout Account exists; a direct attempt shows "You already have a connected payout account." |
| View Payout Account status | The Pro (Talia) | Always, own account only | -- |
| View Payout Account status | Platform Operator (Support) | Always, status and money list only (Access Matrix, Payouts = View) | -- |
| View bank or identity details | Platform Operator (Support) | Never -- the product never holds these details at all (ASMP-31) | No control to view them exists anywhere in the product; the interface has no such field to show |
| View bank or identity details | The Client | Never -- Clients have no access to Payouts at all (Access Matrix, Payouts = None) | The Payouts area is not reachable by a Client under any navigation path |
| Resolve an Action Required flag | The Pro (Talia) | Only through the processor's own flow (FEAT-28.SPEC-006), never by editing a field inside the product | "Resolve now" hands Talia directly to the processor; the product has no in-app form to clear this flag itself |
| Resolve an Action Required flag | Platform Operator (Support) | Never | No "Resolve now" or equivalent control is shown to Support, per SC-05 |
| Disconnect a Payout Account | The Pro (Talia) | Only through the processor's own flow, never through a Chairtime control (the dependency map's Payout Account Contention: the Pro acts through the processor's own flow) | No standalone "disconnect" control exists inside the product; disconnection is a status the processor reports, not an action taken here |
| Delete a Payout Account record | Nobody, directly | Never inside this feature -- removed only as part of Pro Account closure (FEAT-29's 30-day cooling-off), per the Brief's CRUD matrix | No delete control exists anywhere for this entity |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| country | Defaulted from the Pro Account's country at the moment of first connection | On Payout Account creation | No -- the Pro changes their Chairtime account's country/currency through FEAT-27, not through this entity |
| currency | Defaulted from the Pro Account's currency at the moment of first connection | On Payout Account creation | No |
| Chairtime's own fee on any deposit, balance, or tip | Always exactly zero (XBR-07); never a field with a stored value, since it never varies | Always | No -- this is a standing product rule, not a per-account setting |

## Business Rules

- XBR-06: no deposit can be taken, and the booking link cannot go live, unless the Payout Account's status is Active; every reader of this entity (FEAT-07, FEAT-15) treats Active as the sole qualifying state.
- XBR-07: money never rests with the platform -- Chairtime's fee on deposits, balances, and tips is always zero; the only deduction ever shown against a Payout Account's transactions is the payment processor's own card fee (FEAT-28.SPEC-005 enforces this at display time; this spec is the standing rule's single source of truth).
- XBR-25: the Payout Account's country and currency must match the Pro Account's; once the Pro Account's currency locks at its first deposit (XBR-25), the Payout Account's currency is likewise fixed for the life of the account.
- The Validation & Limits field (product-features.md, FEAT-28) and SC-20 fix exactly one Payout Account per Pro Account, spanning exactly one country and currency; multi-country payout support is out of scope for this feature (deferred to the geography phase-in).
- A Pro never edits bank details inside Chairtime (dependency map, Payout Account Contention); every write to processor_account_reference and status originates from the capability's own report, applied by FEAT-28.SPEC-003.

## Edge Cases

- **A Pro's Chairtime account country/currency changes after the Payout Account was already connected** -- Cannot occur under normal operation: FEAT-27 locks currency at the first deposit (XBR-25) and country is not user-editable after account creation, so no post-connection mismatch scenario exists; if the capability itself ever reports a changed country/currency for an existing account, that report is treated as an Action Required condition requiring the Pro to reconnect, rather than silently accepted.
- **A candidate connection reports a country/currency combination Chairtime does not yet support (outside the US at MVP)** -- Rejected with the same country mismatch message; SC-20's geography phase-in note applies -- this rule does not change until that phase-in ships.
- **Two connection attempts race for the same Pro Account (e.g., two browser tabs)** -- The first committed report wins and creates the Payout Account; the second is evaluated against the one-account-per-Pro rule and rejected, consistent with reject-with-refresh handling elsewhere in the product.
- **The zero-Chairtime-fee rule is checked against a transaction that somehow carries a non-zero platform fee value (a data anomaly)** -- Treated as a processing error, never displayed to the Pro as a fee; FEAT-28.SPEC-005's composition never renders a Chairtime fee line under any circumstance, per XBR-07.
- **Support attempts to view or infer bank/identity details indirectly (e.g., by cross-referencing the processor_account_reference)** -- The reference itself carries no bank or identity information; it is an opaque pointer to the capability's own record, so no inference is possible from data the product holds.
- **A Pro Account closes while its Payout Account is Action Required** -- The Payout Account is not independently deleted; it is soft-removed only as part of FEAT-29's 30-day cooling-off closure, with no cascade to already-created Deposit Transactions, per the explicit non-goal in the Brief.

## Acceptance Criteria

**FEAT-28.SPEC-004-AC-01:** Given Talia has no Payout Account, when a connection is reported with country and currency matching her Pro Account, then the Payout Account is created successfully.

**FEAT-28.SPEC-004-AC-02:** Given a candidate connection reports a country different from Talia's Pro Account, when the eligibility check runs, then no Payout Account is created and "Your payout account's country doesn't match your Chairtime account's country. Payout accounts must be in the same country you signed up with." is shown.

**FEAT-28.SPEC-004-AC-03:** Given a candidate connection reports a currency different from Talia's Pro Account, when the eligibility check runs, then no Payout Account is created and the currency mismatch message is shown.

**FEAT-28.SPEC-004-AC-04:** Given Talia already has a Payout Account, when a second connection attempt is made, then it is rejected with "You already have a connected payout account. Contact support if you need to change it."

**FEAT-28.SPEC-004-AC-05:** Given a captured deposit is composed for the money list, when the deduction is shown, then only the processor's own card fee appears -- never a Chairtime fee line.

**FEAT-28.SPEC-004-AC-06:** Given Talia's Payout Account status is anything other than Active, when FEAT-07 checks eligibility for a deposit charge, then the charge is blocked per XBR-06.

**FEAT-28.SPEC-004-AC-07:** Given Talia's Payout Account status is Active, when FEAT-15 evaluates go-live readiness, then this precondition is satisfied.

**FEAT-28.SPEC-004-AC-08:** Given the Client (Riley) attempts to reach any Payouts-related screen, when the attempt is made, then no such navigation path exists for her role.

**FEAT-28.SPEC-004-AC-09:** Given Platform Operator (Support) views a Pro's Payout Account, when Support looks for bank or identity details, then none are shown, since the product never holds them.

**FEAT-28.SPEC-004-AC-10:** Given Talia's Payout Account is Action Required, when she looks for an in-app control to clear the flag directly, then none exists -- only "Resolve now," which hands her to the processor's own flow.

**FEAT-28.SPEC-004-AC-11:** Given Support views a Pro's Action Required banner, when Support looks for a resolution control, then none is shown, per SC-05.

**FEAT-28.SPEC-004-AC-12:** Given Talia wants to disconnect her Payout Account, when she looks for a Chairtime control to do so, then none exists -- disconnection is reflected only from the processor's own report.

**FEAT-28.SPEC-004-AC-13:** Given Talia's Pro Account currency locks at her first deposit (XBR-25), when the Payout Account's currency is evaluated afterward, then it remains fixed and is never independently editable.

**FEAT-28.SPEC-004-AC-14:** Given a candidate connection reports a country/currency combination outside the US at MVP, when the eligibility check runs, then it is rejected with the same country mismatch message, per SC-20's phase-in note.

**FEAT-28.SPEC-004-AC-15:** Given two connection attempts race for the same Pro Account, when both are evaluated, then only the first-committed report creates the Payout Account and the second is rejected under the one-account-per-Pro rule.

**FEAT-28.SPEC-004-AC-16:** Given a Pro Account closes while its Payout Account is Action Required, when FEAT-29's closure process runs, then the Payout Account is soft-removed only as part of that closure, with no independent deletion beforehand.

**FEAT-28.SPEC-004-AC-17:** Given Talia's Payout Account has never been connected (Not Connected), when FEAT-07 checks eligibility, then the deposit charge is blocked exactly as it would be for any other non-Active status.

**FEAT-28.SPEC-004-AC-18:** Given the capability reports processor_account_reference and status values for a new account, when FEAT-28.SPEC-003 processes them, then no user-facing validation error can occur on these fields, since they are never directly entered by the Pro.

**FEAT-28.SPEC-004-AC-19:** Given a data anomaly reports a non-zero platform fee on a transaction, when the money list composes that row, then no Chairtime fee line is ever rendered, per XBR-07.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 6 | 6 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 9 | 9 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

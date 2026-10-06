---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-27.SPEC-008
spec_name: Currency Lock Rule
spec_slug: currency-lock-rule
parent_feature: FEAT-27
parent_feature_name: Pro Profile & Booking Page Settings
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 3
acceptance_criteria_count: 9
---

# Logic/Rule Spec: Currency Lock Rule

## Overview

**Name:** Currency Lock Rule
**ID:** FEAT-27.SPEC-008
**Type:** Logic/Rule
**Purpose:** Determines whether the Pro Account's currency is still editable, locking it permanently the moment the account's first deposit is taken.
**Parent Feature:** FEAT-27 -- Pro Profile & Booking Page Settings
**Governed Entity:** Pro Account -- the currency field, evaluated against Deposit Transaction (read-only)

## Scope and Non-Goals

**In Scope:**
- The lock condition itself: whether currency is still editable
- The exact message shown when a currency change is blocked
- Authorization for who may attempt a currency change

**Non-Goals:**
- The timezone/currency screen's layout and save flow -- owned by FEAT-27.SPEC-003 (Timezone & Currency Settings); this spec only supplies the lock determination it defers to
- Computing deposit amounts or determining what counts as a "deposit taken" for pricing purposes -- owned by FEAT-07 (Deposit Payment at Booking); this spec only reads whether any Deposit Transaction exists for the account
- Unlocking currency after account reopening or any other lifecycle event -- product-features.md and BRIEF.md describe this as a permanent, one-way lock with no reset path; excluded as a deliberate design choice protecting every prior booking's price integrity

## Governed Entity

**Entity:** Pro Account (currency field)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| currency | text (enum of supported currencies) | The currency all of the Pro's prices, deposits, and payouts are expressed in; required per account |

**Referenced Entity (read-only):** Deposit Transaction -- checked for existence to determine whether the account's first deposit has occurred (FEAT-07's entity; fields not modified by this spec).

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-27.SPEC-003 | Timezone & Currency Settings | On screen load (to show the selector as editable or locked) and on submit (final rejection if locked) |
| FEAT-07.SPEC-003 | Deposit Amount & Eligibility Rules | As a charge precondition -- currency must already equal the Pro Account's locked value before any deposit charge is authorized |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| currency | Editable only while no Deposit Transaction exists for the account | Always (this is the lock condition itself) | On screen load and on submit | "Your currency is locked because you've already taken a deposit. To keep every past booking's price accurate, currency can't change once you've started collecting payments." | Yes |
| currency | Must be one of the product's supported currency values | Always, while still unlocked | On selection and on submit | "Choose a supported currency" | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Lock derives from Deposit Transaction existence, not from currency's own state | currency, Deposit Transaction (existence) | The lock is a derived boolean: locked = (at least one Deposit Transaction exists for this Pro Account). currency's own value never determines its own lock. | N/A -- this is a derivation rule, not a user-facing error |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Change currency | The Pro (Talia) | Only while unlocked (no Deposit Transaction exists yet for the account) | Once locked: the currency selector on FEAT-27.SPEC-003 is disabled and shows the exact locked explanation above; a direct submit attempt (for example, from a stale screen state) is rejected with the same message |
| Change currency | The Client (Riley) | Never | No control of any kind is exposed to clients; currency is an account-wide Pro setting |
| Change currency | Platform Operator (Support) | Never | The currency selector is shown disabled and labeled "View-only in support mode" on FEAT-27.SPEC-003, regardless of lock state |
| View currency lock status | The Pro (Talia) | Always | -- |
| View currency lock status | Platform Operator (Support) | Always, view-only (ASMP-20) | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| currency (initial value) | Set during onboarding (FEAT-15) based on the Pro's stated country/market | On Pro Account creation | Yes -- freely, until the lock condition below becomes true |
| is_locked (derived, not a stored field) | True if and only if at least one Deposit Transaction exists for this Pro Account; false otherwise | Evaluated live on every screen load and every submit attempt | No -- this is a system-derived state, never directly set by any role |

## Business Rules

- XBR-25: currency is fixed once the first deposit is taken; this is the authoritative business rule this spec implements for the Pro Account's currency field.
- FEAT-07.SPEC-003 separately checks the resulting lock as a charge precondition (currency must match the locked value before any deposit charge is authorized) -- this spec owns the lock's determination; FEAT-07.SPEC-003 consumes it, never redefines it.
- The lock is permanent and one-directional: once locked, no lifecycle event (account pause, reopening, or any other state change) unlocks it again.
- FEAT-28.SPEC-004 checks that the payout account's country and currency match the Pro Account's currency (SC-20); this spec's lock does not itself validate that match -- FEAT-28 surfaces any resulting mismatch independently.

## Edge Cases

- **Talia's very first deposit transaction is created for her account while she has the Timezone & Currency Settings screen open** -- The screen is not live-updating; her lock state is re-evaluated fresh on her next submit attempt, and if a Deposit Transaction now exists, the change is rejected with the exact locked message even though the selector appeared editable when the screen loaded.
- **Talia's account has a Deposit Transaction that was later fully refunded** -- The lock remains permanent: the rule is existence of a Deposit Transaction record, not its current status, so a refunded deposit still counts as "already taken" for locking purposes.
- **A brand-new Pro Account with zero bookings and zero deposits** -- currency is fully editable; the lock condition (at least one Deposit Transaction exists) is false.
- **Support views a locked account's currency setting** -- Shown disabled per the Authorization Rules, identically to how it would appear if Support could otherwise act (which it never can, locked or not).
- **Talia attempts to change currency at the exact same moment her first-ever booking's deposit charge succeeds** -- Whichever completes first is authoritative: if the deposit charge commits first, her currency-change submission is rejected with the locked message; if her currency-change submission somehow committed first (not possible under normal use since the lock check is evaluated at submit time against the current Deposit Transaction existence), no ambiguity arises because the check is a live existence read, not a cached flag.

## Acceptance Criteria

**FEAT-27.SPEC-008-AC-01:** Given Talia's Pro Account has never had a Deposit Transaction, when she views the Timezone & Currency Settings screen, then the currency selector is editable.

**FEAT-27.SPEC-008-AC-02:** Given Talia's Pro Account has at least one Deposit Transaction, when she views the currency selector, then it is disabled and shows "Your currency is locked because you've already taken a deposit. To keep every past booking's price accurate, currency can't change once you've started collecting payments."

**FEAT-27.SPEC-008-AC-03:** Given Talia's account locks between her screen load and her submit attempt, when she submits a currency change, then it is rejected with the exact locked message even though the selector appeared editable on load.

**FEAT-27.SPEC-008-AC-04:** Given Talia's account has a Deposit Transaction that was later fully refunded, when she views the currency selector, then it remains locked -- the refund does not unlock it.

**FEAT-27.SPEC-008-AC-05:** Given FEAT-07.SPEC-003 evaluates a deposit charge's eligibility, when it checks currency, then it relies on this spec's lock determination rather than re-deriving its own.

**FEAT-27.SPEC-008-AC-06:** Given Talia (the Pro) attempts to change currency while unlocked, when she selects a supported currency and saves, then the change is allowed.

**FEAT-27.SPEC-008-AC-07:** Given Talia attempts to select an unsupported currency value, when she submits, then she sees "Choose a supported currency."

**FEAT-27.SPEC-008-AC-08:** Given a support operator views a locked account's currency setting, when they look for an edit control, then none is offered -- the selector is disabled and labeled "View-only in support mode," regardless of the lock state.

**FEAT-27.SPEC-008-AC-09:** Given Talia's account has zero bookings and zero deposits, when she views the currency selector, then it is fully editable and shows no lock message.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 2 | 2 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |

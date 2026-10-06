---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-15.SPEC-004
spec_name: Currency & Tax Lock After First Invoice
spec_slug: currency-tax-lock-after-first-invoice
parent_feature: FEAT-15
parent_feature_name: Currency & Tax Handling
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 9
acceptance_criteria_count: 9
---

# Logic/Rule Spec: Currency & Tax Lock After First Invoice

## Overview

**Name:** Currency & Tax Lock After First Invoice
**ID:** FEAT-15.SPEC-004
**Type:** Logic/Rule
**Purpose:** Locks a project's currency and its paired tax line the moment its first invoice is sent, and blocks any later change attempt with an explanation.
**Parent Feature:** FEAT-15 -- Currency & Tax Handling
**Governed Entity:** Project (the `currency`, `tax_label`, and `tax_rate` fields; the lock state itself is derived from Invoice)

## Scope and Non-Goals

**In Scope:**
- The state transition from editable to locked, fired by a project's first invoice being sent
- The permanent, no-unlock nature of the lock for that project
- Blocking any change attempt on a locked project's currency/tax fields, with an explanation
- Reading Invoice to determine whether a project already has a sent invoice

**Non-Goals:**
- Deciding what counts as a valid currency or tax shape -- owned entirely by FEAT-15.SPEC-003 (Currency & Tax Validation Rules); this spec governs *whether a change may be attempted at all*, not whether an attempted value would be valid
- Who may attempt a change while a project is unlocked -- owned entirely by FEAT-15.SPEC-005 (Currency & Tax Configuration Access Rules); this spec governs the lock state, not role entitlement
- Sending the first invoice itself, or any other invoice lifecycle behavior -- owned entirely by Invoice Generation & Sending (FEAT-09); this spec only reacts to the "first invoice sent" event as a trigger
- Any versioned history of pre-lock configuration changes -- excluded per this feature's Non-Goals: because the lock is permanent and irreversible, no scenario produces multiple historical configurations to version

## Governed Entity

**Entity:** Project
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|--------------|
| currency | enum (recognized world currency code) | The project's billing currency; the field this rule locks |
| tax_label | text | Freelancer-entered tax line label, or absent; the field this rule locks |
| tax_rate | number (percentage) | Freelancer-entered tax rate, or absent; the field this rule locks |
| (derived) lock_state | derived boolean | Whether the project's currency/tax fields are currently editable or locked; derived from whether any Invoice belonging to this project has ever reached status Sent or later |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|--------------------|
| FEAT-15.SPEC-001 | Project Currency & Tax Configuration | On screen load (to decide editable vs. locked rendering) and on any save attempt |
| FEAT-15.SPEC-008 | Invoice Currency & Tax Line Application | Immediately after a project's first invoice is sent, to trigger the state transition |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| currency | No new value may be written once the project's lock_state is Locked | Project has a sent invoice | On any change attempt | "Currency is fixed once billing has begun for this project." | Yes |
| tax_label | No new value may be written once the project's lock_state is Locked | Project has a sent invoice | On any change attempt | "Tax label is fixed once billing has begun for this project." | Yes |
| tax_rate | No new value may be written once the project's lock_state is Locked | Project has a sent invoice | On any change attempt | "Tax rate is fixed once billing has begun for this project." | Yes |
| lock_state | No validation beyond data type -- this is a system-derived field, never directly entered by any user | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|-----------------|
| Locked triad | currency, tax_label, tax_rate | The lock applies to all three fields together -- there is no partial lock where one field remains editable while the others are fixed | "Currency and tax are fixed once billing has begun for this project." |

## Authorization Rules

The role-action matrix for who may attempt a currency/tax change is owned by FEAT-15.SPEC-005. This spec's own authorization concern is narrower: whether the lock state itself blocks even an otherwise-permitted actor.

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Change currency/tax on a locked project | Nadia (Freelancer) | Never, once lock_state is Locked -- even though FEAT-15.SPEC-005 grants Nadia Full access to this configuration while unlocked | No editable field is ever rendered for a locked project (FEAT-15.SPEC-001); a direct, out-of-band attempt is refused with "Currency and tax are fixed once billing has begun for this project." |
| Change currency/tax on an unlocked project | Nadia (Freelancer) | Always, while lock_state is Editable (subject to FEAT-15.SPEC-003's validation and FEAT-15.SPEC-005's access grant) | -- |
| View a locked project's currency/tax | Nadia (Freelancer), Dana (Support Operator) | Always -- the lock never restricts viewing, only writing | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-----------------------|
| lock_state | Derived: Editable by default; transitions to Locked the moment any Invoice belonging to this project first reaches status Sent | Recomputed at the moment the project's first invoice is sent (FEAT-15.SPEC-008); read on every access to FEAT-15.SPEC-001 | No -- lock_state is never directly set by any user, and once Locked it is never overridable by anyone, including Nadia |

## Business Rules

- XBR-17: the currency cannot change once the first invoice is sent -- this spec is the rule's sole enforcement point.
- The lock is permanent for a project: there is no unlock path, no administrative override, and no scenario (including account support access) that reopens a locked project's currency/tax fields, per this feature's Entity-Lifecycle Coverage Matrix.
- The lock fires from a cross-feature trigger: "a project's first invoice is sent (event from Invoice Generation & Sending, FEAT-09)," per this feature's Side-Effect Inventory and Internal Dependency Map.
- The lock check (this spec) always runs before the validation check (FEAT-15.SPEC-003): a locked project blocks any change attempt outright, regardless of whether the attempted new value would itself be valid.
- A credit note or a corrected/re-sent invoice on the same project does not create a second "first invoice sent" event and does not re-lock or re-evaluate an already-locked project -- the lock is keyed to the project's *first* invoice reaching Sent, a one-time, non-repeating transition.

## Edge Cases

- **Two of Nadia's sessions both have the currency/tax screen open, unlocked, when the project's first invoice is sent by FEAT-09** -- Both sessions are rejected-with-refresh: neither in-flight edit is saved, and both re-render to the Locked state on their next interaction (consistent with FEAT-15.SPEC-001's Edge Cases).
- **Nadia attempts to save a currency/tax change at the exact moment the first invoice's send is being processed (race between save and lock)** -- The lock check re-evaluates at the moment of the save attempt, not at the moment the screen was loaded: if the invoice's Sent status has landed by the time the save is processed, the save is refused even though the screen appeared editable when it was opened.
- **A project has an unsent (Draft or Generated, not yet Sent) first invoice** -- lock_state remains Editable; only a status of Sent or later on any invoice belonging to the project triggers the lock. A draft invoice never locks the project.
- **The project's would-be first invoice is voided or fails to send before reaching Sent** -- No lock fires; the project remains Editable until an invoice genuinely reaches Sent.
- **Dana views a locked project's currency/tax inside a support session** -- She sees it exactly as Nadia would (read-only display with the lock explanation); the lock applies identically regardless of viewer, and Dana's session grants no override of any kind.

## Acceptance Criteria

**FEAT-15.SPEC-004-AC-01:** Given a project has no invoice that has ever reached Sent, when Nadia opens its currency/tax configuration, then the fields render editable and lock_state is Editable.

**FEAT-15.SPEC-004-AC-02:** Given a project's first invoice is sent by FEAT-09, when that event lands, then the project's lock_state transitions to Locked immediately.

**FEAT-15.SPEC-004-AC-03:** Given a project's lock_state is Locked, when Nadia opens its currency/tax configuration, then no editable field is rendered and the explanation "currency and tax are fixed after your first invoice" is shown.

**FEAT-15.SPEC-004-AC-04:** Given a project's lock_state is Locked, when any out-of-band attempt tries to change its currency, then it is refused with "Currency is fixed once billing has begun for this project."

**FEAT-15.SPEC-004-AC-05:** Given a project has a Draft invoice that has never reached Sent, when Nadia opens its currency/tax configuration, then the fields remain editable -- the draft alone does not trigger the lock.

**FEAT-15.SPEC-004-AC-06:** Given a project's would-be first invoice is voided before reaching Sent, when Nadia later opens its currency/tax configuration, then the fields remain editable and no lock has fired.

**FEAT-15.SPEC-004-AC-07:** Given Nadia's save attempt reaches the server after the project's first invoice has already been sent in the interim, when the lock check runs, then the save is refused even though the screen appeared editable when it was opened.

**FEAT-15.SPEC-004-AC-08:** Given a project's currency/tax is locked, when Dana views it inside a support session, then she sees the identical locked, read-only display Nadia would see -- no override is available to her.

**FEAT-15.SPEC-004-AC-09:** Given a project's currency/tax is already locked, when a credit note or a corrected invoice is later issued for that project, then no re-evaluation of the lock occurs and the project remains Locked exactly as before.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 3 | 3 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |

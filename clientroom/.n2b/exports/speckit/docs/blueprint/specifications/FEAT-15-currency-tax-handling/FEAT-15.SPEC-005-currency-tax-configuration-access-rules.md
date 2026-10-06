---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-15.SPEC-005
spec_name: Currency & Tax Configuration Access Rules
spec_slug: currency-tax-configuration-access-rules
parent_feature: FEAT-15
parent_feature_name: Currency & Tax Handling
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 8
acceptance_criteria_count: 10
---

# Logic/Rule Spec: Currency & Tax Configuration Access Rules

## Overview

**Name:** Currency & Tax Configuration Access Rules
**ID:** FEAT-15.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs who may view or change a project's currency/tax configuration -- Nadia Full, Dana read-only in a logged session, Owen and Priya no direct access to the configuration surface at all.
**Parent Feature:** FEAT-15 -- Currency & Tax Handling
**Governed Entity:** Project (the `currency`, `tax_label`, and `tax_rate` fields, and the configuration screen that surfaces them)

## Scope and Non-Goals

**In Scope:**
- The complete role-action matrix for viewing and changing a project's currency/tax configuration
- The exact experience for each role that is denied access, including the two roles with no access at all (Owen, Priya)
- This spec is the single source of truth FEAT-15.SPEC-001 (screen) and FEAT-15.SPEC-003 (validation) reference rather than restating the gate, per this feature's Shared UI Patterns note

**Non-Goals:**
- What shape a valid currency or tax line takes -- owned entirely by FEAT-15.SPEC-003 (Currency & Tax Validation Rules); this spec governs who may attempt an action, not whether the attempted value is valid
- Whether a project's configuration can still be changed at all, independent of role -- owned entirely by FEAT-15.SPEC-004 (Currency & Tax Lock After First Invoice); this spec governs role entitlement, not lock state
- Dana's support session lifecycle (how it opens, how long it lasts, how it is announced) -- owned entirely by Operator Support Access (FEAT-31); this spec only states what Dana may do with currency/tax data once inside a session
- Access to invoice content that happens to display a project's currency and tax line after generation -- owned by Invoice Generation & Sending (FEAT-09, e.g. its own Access Rules); this spec governs only the configuration surface, not the resulting invoice

## Governed Entity

**Entity:** Project
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|--------------|
| currency | enum (recognized world currency code) | The project's billing currency |
| tax_label | text | Freelancer-entered tax line label, or absent |
| tax_rate | number (percentage) | Freelancer-entered tax rate, or absent |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|--------------------|
| FEAT-15.SPEC-001 | Project Currency & Tax Configuration | On screen entry (deciding whether the screen renders at all, and in which mode) and on every save attempt |

## Field Validation Rules

Not applicable -- this spec governs role access to the configuration screen and its fields, not the shape of the values themselves. Field-level validation is defined entirely by FEAT-15.SPEC-003.

## Cross-Field Rules

Not applicable -- no cross-field rule in this spec; cross-field validation is defined entirely by FEAT-15.SPEC-003.

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View project currency/tax configuration | Nadia (Freelancer) | Always, for her own projects | -- |
| View project currency/tax configuration | Dana (Support Operator) | Only inside a logged, time-limited support session (FEAT-31) | Outside an open support session, this screen is not reachable at all -- no route exists for Dana without an active session |
| View project currency/tax configuration | Owen (Client Primary Contact) | Never | The configuration screen is not shown at all -- absent from his navigation entirely; a direct link redirects to his portal home with no partial content ever rendered |
| View project currency/tax configuration | Priya (Client Reviewer Contact) | Never | The configuration screen is not shown at all -- absent from her navigation entirely; a direct link redirects to her portal home with no partial content ever rendered |
| Set or change project currency/tax | Nadia (Freelancer) | For her own projects, while lock_state is Editable (FEAT-15.SPEC-004) and the attempted value passes FEAT-15.SPEC-003 | Once locked, no editable control is shown; a direct out-of-band attempt is refused per FEAT-15.SPEC-004's exact message |
| Set or change project currency/tax | Dana (Support Operator) | Never | No Save control or any editable input is ever rendered in a support session, regardless of the project's lock state; a direct, out-of-band change attempt is refused and the screen re-renders in its read-only form |
| Set or change project currency/tax | Owen (Client Primary Contact) | Never | The configuration screen is not shown at all, so no change control is ever reachable |
| Set or change project currency/tax | Priya (Client Reviewer Contact) | Never | The configuration screen is not shown at all, so no change control is ever reachable |

## Defaults and Derivations

Not applicable -- this spec governs access, not data defaults. Field defaults and derivations are defined entirely by FEAT-15.SPEC-003.

## Business Rules

- This spec is the single authoritative home for the role-action matrix on the Project currency/tax configuration; FEAT-15.SPEC-001 and FEAT-15.SPEC-003 reference it rather than restating the gate, per this feature's Shared UI Patterns note.
- Owen and Priya's exclusion is total, not merely read-restricted: the configuration screen is not shown at all, distinct from a "view but not edit" pattern -- consistent with the Brief's Side-Effect Inventory entry "Owen or Priya attempts to reach the currency/tax configuration screen -> Show nothing."
- Dana's access is always read-only and always session-scoped: she never reaches this screen outside an active support session, and even inside one she holds no change capability of any kind, consistent with XBR-29 (operator sessions are read-only in every feature).
- This matrix governs the configuration surface only; it does not govern what Owen sees on an invoice once one has been generated (FEAT-09 owns that surface and its own access rules) -- Owen's Access field in product-features.md states he sees the resulting amounts and tax line on his invoices, which is a distinct, downstream surface from this one.

## Edge Cases

- **Owen's support-request context somehow includes a link to this configuration screen (e.g., forwarded from an internal tool)** -- The redirect-to-portal-home behavior applies regardless of how the link was obtained; no context bypasses the role check.
- **Dana's support session expires while she is viewing this screen** -- The screen becomes unreachable the moment the session closes (governed by FEAT-31); any further attempt to view or interact resolves as if she had never had a session.
- **Nadia is viewing this screen when a support session for her account happens to open (Dana starts viewing concurrently)** -- No conflict: Dana's session is read-only and makes no writes, so Nadia's own editing (subject to FEAT-15.SPEC-003 and FEAT-15.SPEC-004) proceeds unaffected by Dana's concurrent, separate view.
- **A future role is added to the product that the Access Matrix does not yet name** -- Out of scope for this spec: per pipeline-rules.md's grounded-roles constraint, this spec's matrix covers exactly the four roles the Access Matrix in user-persona.md establishes today (Nadia, Owen, Priya, Dana), and no invented role is added here.

## Acceptance Criteria

**FEAT-15.SPEC-005-AC-01:** Given Nadia opens the currency/tax configuration for one of her own projects, when the screen loads, then she has full view and change access, subject to lock state.

**FEAT-15.SPEC-005-AC-02:** Given Dana has no active support session on a freelancer's account, when she attempts to reach that account's currency/tax configuration screen, then no route to it exists.

**FEAT-15.SPEC-005-AC-03:** Given Dana has an active, logged support session open, when she views a project's currency/tax configuration, then she sees it read-only with no Save control or editable input rendered.

**FEAT-15.SPEC-005-AC-04:** Given Owen attempts to reach the currency/tax configuration screen by any direct link, when the link resolves, then he is redirected to his portal home with no configuration content ever rendered.

**FEAT-15.SPEC-005-AC-05:** Given Priya attempts to reach the currency/tax configuration screen by any direct link, when the link resolves, then she is redirected to her portal home with no configuration content ever rendered.

**FEAT-15.SPEC-005-AC-06:** Given Dana is inside a support session, when she attempts an out-of-band change to a project's currency, then it is refused and the screen re-renders in its read-only form.

**FEAT-15.SPEC-005-AC-07:** Given Nadia's project is unlocked, when she attempts to change its currency/tax, then the action is allowed, subject to passing FEAT-15.SPEC-003's validation.

**FEAT-15.SPEC-005-AC-08:** Given Nadia's project is locked (FEAT-15.SPEC-004), when she attempts to change its currency/tax, then no editable control is shown and any out-of-band attempt is refused per FEAT-15.SPEC-004's exact message.

**FEAT-15.SPEC-005-AC-09:** Given Dana's support session expires while she is viewing this screen, when the session closes, then the screen becomes unreachable to her as if no session had ever existed.

**FEAT-15.SPEC-005-AC-10:** Given Owen later opens an invoice generated for one of his company's projects, when he views its content, then he sees the resulting currency and tax line on the invoice itself -- a distinct surface governed by FEAT-09, not by this spec's configuration-screen access rule.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 0 (N/A -- owned by FEAT-15.SPEC-003) | 0 |
| Cross-Field Rules | 0 (N/A -- owned by FEAT-15.SPEC-003) | 0 |
| Authorization Rules | 8 | 8 |
| Defaults/Derivations | 0 (N/A -- owned by FEAT-15.SPEC-003) | 0 |
| Business Rules | 4 | 4 |
| Edge Cases | 4 | 4 |

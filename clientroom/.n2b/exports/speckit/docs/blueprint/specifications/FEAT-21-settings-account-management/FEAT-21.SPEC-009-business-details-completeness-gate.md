---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-21.SPEC-009
spec_name: Business Details Completeness Gate
spec_slug: business-details-completeness-gate
parent_feature: FEAT-21
parent_feature_name: Settings & Account Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 5
acceptance_criteria_count: 10
---

# Logic/Rule Spec: Business Details Completeness Gate

## Overview

**Name:** Business Details Completeness Gate
**ID:** FEAT-21.SPEC-009
**Type:** Logic/Rule
**Purpose:** Tracks whether business details and payment terms are complete and blocks the first invoice send until they are (XBR-16).
**Parent Feature:** FEAT-21 -- Settings & Account Management
**Governed Entity:** Freelancer Account (business_name, business_address, default_payment_terms fields)

## Scope and Non-Goals

**In Scope:**
- The completeness determination: which fields must be non-empty for business details to count as complete
- Re-evaluating completeness on every business-details save
- Exposing the completeness state to FEAT-21.SPEC-004 (for the indicator line) and to FEAT-09 (for the send-time gate)

**Non-Goals:**
- Performing the actual block on invoice sending -- that check and its user-facing blocked message are executed by FEAT-09 (Invoice Generation & Sending) at send time; this spec only computes and exposes the true/false completeness state XBR-16 requires
- Field-level validation (required/format/length) for the individual business-detail fields -- owned by FEAT-21.SPEC-007; this spec consumes those fields' current values, it does not validate their format
- Client billing details completeness -- a separate half of XBR-16 owned by FEAT-01 (Client & Project Management); this spec covers only the freelancer's own business details

## Governed Entity

**Entity:** Freelancer Account (business_name, business_address, default_payment_terms fields)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| business_name | text | Required before the first invoice is sent (dependency map, Freelancer Account fields) |
| business_address | text | Required before the first invoice is sent |
| default_payment_terms | enum | Required before the first invoice is sent |
| tax_id | text | Optional -- not part of the completeness determination (dependency map lists tax_id without a required-before-invoicing note) |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-21.SPEC-004 | Business Details & Payment Terms | Re-evaluated on every successful save; result drives the screen's completeness indicator line |
| FEAT-09 (Invoice Generation & Sending) | -- | Checked at the moment a first-invoice send is attempted (XBR-16); the send is blocked while this spec's completeness result is false |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| business_name | No validation beyond data type in this spec -- format/required rules owned by FEAT-21.SPEC-007; this spec only reads whether it is non-empty | Always | -- | -- | -- |
| business_address | No validation beyond data type in this spec -- see above | Always | -- | -- | -- |
| default_payment_terms | No validation beyond data type in this spec -- see above | Always | -- | -- | -- |
| tax_id | No validation beyond data type -- not part of the completeness determination | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Business details completeness | business_name, business_address, default_payment_terms | Complete (true) only when all three are non-empty/selected; incomplete (false) if any one is empty | On FEAT-21.SPEC-004: "Business details are incomplete -- required before your first invoice can be sent." On FEAT-09 at send time: the invoice send is blocked with a message directing Nadia to complete business details in Settings (FEAT-09 owns the exact blocked-send message text) |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View completeness state | Nadia (Freelancer) | Always, her own account only | -- |
| Change the fields that determine completeness | Nadia (Freelancer) | Always, her own account only, via FEAT-21.SPEC-004 | -- |
| View completeness state | Dana (Support Operator) | Read-only, inside a logged FEAT-31 support session (FEAT-21.SPEC-010) | -- |
| Change the fields that determine completeness | Dana (Support Operator) | Never | No save controls are rendered on FEAT-21.SPEC-004 for Dana; a direct attempt is refused with "Support sessions are read-only." |
| View or change completeness-related fields | Owen (Client Primary Contact) | Never | Settings is not shown in navigation at all |
| View or change completeness-related fields | Priya (Client Reviewer Contact) | Never | Settings is not shown in navigation at all |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| completeness (derived) | Computed as business_name non-empty AND business_address non-empty AND default_payment_terms selected | Recomputed on every business-details save and on every FEAT-09 send-time check | No -- it is fully derived, never directly set |

## Business Rules

- XBR-16 is the authority for this spec's central rule: "Every invoice carries a due date from the freelancer's default payment terms ... a unique sequential number per freelancer, her business details, and the client's billing details; sending is blocked until both sets of details exist." This spec owns the freelancer's-own-details half of that check.
- Completeness applies only to the *first* invoice send in the strict sense that once business details are complete, they can later be cleared and re-completed -- the gate re-evaluates the current state at every send attempt (not just the first), so a freelancer who later clears a required field would find sending blocked again, consistent with the field values genuinely being empty at that moment.
- Saving is never blocked by incompleteness: FEAT-21.SPEC-007's required-before-invoicing rules are advisory, so Nadia can save any partial or cleared state on FEAT-21.SPEC-004; the only enforcement of completeness is FEAT-09's send-time block.
- tax_id is deliberately excluded from the completeness determination -- the dependency map records it as optional, with no required-before-invoicing note, unlike business_name, business_address, and default_payment_terms.
- Authorization here is consistent with, and never overrides, FEAT-21.SPEC-010's account-wide read-only scope rules.

## Edge Cases

- **Nadia completes the last missing field and saves while a send attempt from FEAT-09 is already mid-flight against the previously incomplete state** -- FEAT-09's own send-time check is authoritative for that specific attempt; if FEAT-09's check already read "incomplete" before this save committed, that attempt is blocked, and Nadia must retry the send after the save (which will then pass).
- **Nadia clears business_address after it was previously complete, with no first invoice yet sent** -- Completeness reverts to false immediately on that save; any subsequent send attempt is blocked until it is filled again.
- **Nadia clears business_address after the first invoice has already been sent** -- Completeness still reverts to false for the purpose of any *future* first-time gate check (XBR-16 gates the first invoice; a project's first invoice already sent is unaffected retroactively, but any other project of Nadia's whose first invoice has not yet been sent would now be blocked, since completeness is evaluated on the account's current field state, not per project).
- **Nadia saves a partial form (e.g., only tax ID filled, or one required field cleared)** -- The save succeeds (it is never blocked by this rule); completeness is recomputed on that save and is false until all three required fields are filled.
- **All three required fields are filled with only whitespace** -- Treated as empty by FEAT-21.SPEC-007's trim behavior before this spec ever evaluates them, so completeness remains false.
- **Business details were captured during FEAT-20's guided onboarding setup** -- Completeness is evaluated the same way regardless of when the fields were set; onboarding-captured values that satisfy all three conditions yield a complete result with no distinct code path.

## Acceptance Criteria

**FEAT-21.SPEC-009-AC-01:** Given Nadia has business_name, business_address, and default_payment_terms all filled, when this rule evaluates completeness, then the result is complete and FEAT-21.SPEC-004 shows "Business details are complete."

**FEAT-21.SPEC-009-AC-02:** Given Nadia has business_address empty, when this rule evaluates completeness, then the result is incomplete and FEAT-21.SPEC-004 shows "Business details are incomplete -- required before your first invoice can be sent."

**FEAT-21.SPEC-009-AC-03:** Given Nadia's business details are incomplete, when a first-invoice send is attempted through FEAT-09, then FEAT-09 blocks the send and directs her to complete business details in Settings.

**FEAT-21.SPEC-009-AC-04:** Given Nadia's business details are complete, when a first-invoice send is attempted through FEAT-09, then the send proceeds and is not blocked by this rule.

**FEAT-21.SPEC-009-AC-05:** Given Nadia leaves tax_id empty while the three required fields are filled, when this rule evaluates completeness, then the result is complete (tax_id is not part of the determination).

**FEAT-21.SPEC-009-AC-06:** Given Nadia clears default_payment_terms after previously completing business details, when she saves, then the save succeeds (it is not blocked) and completeness reverts to incomplete.

**FEAT-21.SPEC-009-AC-10:** Given Nadia fills only tax_id and leaves business_name, business_address, and default_payment_terms empty, when she saves on FEAT-21.SPEC-004, then the save succeeds and this rule evaluates completeness as incomplete.

**FEAT-21.SPEC-009-AC-07:** Given Dana (Support Operator) is inside a logged support session, when she views the completeness indicator, then she sees its current state read-only with no ability to change the underlying fields.

**FEAT-21.SPEC-009-AC-08:** Given Owen (Client Primary Contact) attempts to reach any Settings screen showing completeness, then he finds none in navigation and no direct access exists.

**FEAT-21.SPEC-009-AC-09:** Given all three required fields contain only whitespace, when this rule evaluates completeness, then the result is incomplete, since whitespace-only values are treated as empty.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 (all N/A -- owned by FEAT-21.SPEC-007) | 4 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 6 | 6 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

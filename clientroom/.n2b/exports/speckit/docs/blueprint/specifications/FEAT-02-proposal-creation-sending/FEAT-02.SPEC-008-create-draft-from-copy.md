---
document_type: spec
spec_type: automation
spec_id: FEAT-02.SPEC-008
spec_name: Create Draft From Copy
spec_slug: create-draft-from-copy
parent_feature: FEAT-02
parent_feature_name: Proposal Creation & Sending
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 7
---

# Automation Spec: Create Draft From Copy

## Overview

**Name:** Create Draft From Copy
**ID:** FEAT-02.SPEC-008
**Type:** Automation
**Purpose:** Creates a new Draft proposal for the target project, pre-filled from a selected earlier proposal's scope, price, and currency.
**Parent Feature:** FEAT-02 -- Proposal Creation & Sending

## Scope and Non-Goals

**In Scope:**
- Copying scope_description and price from a selected source proposal into a new Draft
- Recording the copied_from reference for traceability
- Handling a source proposal in a different currency than the target project

**Non-Goals:**
- Selecting the source proposal -- owned by FEAT-02.SPEC-004 (Reuse Proposal Picker); this automation only acts once a source is chosen.
- Editing the copied content -- the freelancer edits the resulting Draft in FEAT-02.SPEC-001 like any other Draft; this automation only performs the initial copy.
- Copying the payment_schedule_reference -- the new Draft references the target project's own Payment Schedule (FEAT-04), never the source proposal's, since payment schedules are project-specific and the target project may have a different schedule or none yet.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nadia selects a source proposal row | FEAT-02.SPEC-004 (Reuse Proposal Picker) | The target project has no other active (non-voided) proposal | Source proposal id (scope_description, price, currency), target project reference |

## Processing Logic

1. Read the target project's current proposal state; if an active (Draft, Sent, or Accepted) proposal already exists for it, stop and report the one-active-proposal-cap failure.
2. Read the source proposal's scope_description and price.
3. Read the target project's own currency (FEAT-15).
4. Create a new Proposal record for the target project with status Draft: scope_description copied verbatim from the source; price copied as a numeric amount, expressed in the target project's currency without any conversion (see Business Rules); payment_schedule_reference set to the target project's own Payment Schedule (or left unset if the target project has none yet, matching a blank-start Draft).
5. Set copied_from to the source proposal's id.
6. Open the new Draft in FEAT-02.SPEC-001 (Proposal Draft Editor) for the freelancer to review and adjust before saving or sending.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Copy succeeds | Target project has no active proposal | New Proposal created (Draft), copied_from set | Editor opens pre-filled, with the copied-from banner shown | FEAT-02.SPEC-001 |
| Blocked -- active proposal exists | Target project already has a Draft, Sent, or Accepted proposal | None | Reuse Picker shows an inline error on the row: "This project already has a proposal. Open it from the project instead." | FEAT-02.SPEC-004 |
| Failure | Processing error (e.g., connectivity lost mid-copy) | None | Reuse Picker shows an inline error on the row: "Could not start from this proposal. Try again." and Nadia remains on the picker | FEAT-02.SPEC-004 |

## Data Model

**Reads:** Proposal (source) -- scope_description, price, currency. Project (target) -- currency, existing proposal state, Payment Schedule reference.
**Creates:** Proposal (new Draft) -- scope_description, price, currency (target project's own), payment_schedule_reference (target project's own), status Draft, copied_from (source proposal id).
**Updates:** None.
**Deletes:** None.

## Business Rules

- The one-active-proposal-per-project cap (dependency map: Project relationships) is checked before the copy is created -- a project already carrying a Draft, Sent, or Accepted proposal cannot receive a second one from this automation.
- Price is copied as a numeric amount only; if the source proposal's currency differs from the target project's currency, the number is carried over unconverted (no automatic currency conversion exists in the product, per XBR-18) and the freelancer is expected to review and adjust it in the editor before sending -- the copied-from banner and the visible currency field make the source's original currency and the target's current currency both apparent for that review.
- copied_from is set once at creation and is never altered afterward -- it is a permanent provenance reference to the source proposal, not a live link.

## Edge Cases

- **The source proposal's currency differs from the target project's currency** -- The price value is copied as-is; the editor displays the target project's currency next to the copied number so Nadia can adjust it before sending (see Business Rules). No automatic conversion is performed or implied.
- **The target project already has a Draft, Sent, or Accepted proposal by the time this automation runs (created concurrently in another session)** -- Reported as the blocked outcome; no duplicate active proposal is created for the project.
- **Concurrent trigger firing (two copy operations targeting the same project fire at effectively the same time)** -- Only the first to pass the one-active-proposal check creates the Draft; the second finds an active proposal already present and receives the blocked outcome.
- **A trigger fires while a previous copy for the same target project is still in flight** -- The Reuse Picker's row shows a loading state during the operation (FEAT-02.SPEC-004), preventing a second selection from the same session; a second session racing in is covered by the concurrent-trigger-firing case above.
- **The source proposal is later voided or discarded after this copy completes** -- Has no effect on the new Draft; copied_from is a point-in-time provenance reference, not a live dependency, and the new Draft is fully independent once created.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-004 (Reuse Proposal Picker) | Triggered by (inbound) | Row selection triggers this automation with the chosen source |
| FEAT-02.SPEC-001 (Proposal Draft Editor) | Affects (outbound) | The resulting Draft opens here for review before save or send |
| FEAT-15 (Currency & Tax Handling) | References (inbound) | Target project's currency governs the new Draft's currency |
| FEAT-04 (Milestone & Payment Schedule Setup) | References (inbound) | Target project's own Payment Schedule is referenced, never the source's |

## Analytics and Success Signals

- **proposal_created_from_copy** (source proposal age in days, currency match: same / different) -- supports success-metrics.md: "Proposal Send Speed"
- **proposal_copy_blocked_active_proposal_exists** (-- ) -- N/A -- no Stage 2 metric measures this specific block; retained for operational visibility into how often the one-active-proposal cap is hit via the reuse path

## Acceptance Criteria

**FEAT-02.SPEC-008-AC-01:** Given Nadia selects a source proposal for a target project with no existing proposal, when the copy completes, then a new Draft is created with the source's scope description and price, the target project's own currency and payment schedule reference, and copied_from set to the source.

**FEAT-02.SPEC-008-AC-02:** Given the target project already has a Draft proposal, when Nadia selects a source to copy from, then the copy is blocked with "This project already has a proposal. Open it from the project instead." and no new Draft is created.

**FEAT-02.SPEC-008-AC-03:** Given the source proposal's currency differs from the target project's currency, when the copy completes, then the price number is carried over unconverted and the editor displays the target project's own currency alongside it.

**FEAT-02.SPEC-008-AC-04:** Given a processing failure occurs during the copy, then the Reuse Picker shows "Could not start from this proposal. Try again." and Nadia remains on the picker.

**FEAT-02.SPEC-008-AC-05:** Given two copy operations for the same target project fire at effectively the same time, then only the first creates a Draft and the second receives the active-proposal-exists blocked outcome.

**FEAT-02.SPEC-008-AC-06:** Given the copy succeeds, when the editor opens, then it shows the copied-from banner naming the source project.

**FEAT-02.SPEC-008-AC-07:** Given the source proposal is voided in another session after this copy has already completed, then the resulting Draft is unaffected and remains fully editable.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 (success, blocked, failure) | 3 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |

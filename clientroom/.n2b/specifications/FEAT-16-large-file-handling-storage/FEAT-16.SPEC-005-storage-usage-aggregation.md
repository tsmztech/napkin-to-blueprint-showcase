---
document_type: spec
spec_type: automation
spec_id: FEAT-16.SPEC-005
spec_name: Storage Usage Aggregation
spec_slug: storage-usage-aggregation
parent_feature: FEAT-16
parent_feature_name: Large File Handling & Storage
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Automation Spec: Storage Usage Aggregation

## Overview

**Name:** Storage Usage Aggregation
**ID:** FEAT-16.SPEC-005
**Type:** Automation
**Purpose:** Recalculates the freelancer's total stored bytes whenever a file finishes uploading, a deliverable is removed, or stored bytes are purged, feeding the usage summary screen and the limit rule.
**Parent Feature:** FEAT-16 -- Large File Handling & Storage

## Scope and Non-Goals

**In Scope:**
- Recalculating the freelancer's aggregated total stored bytes on every event that changes it
- Making the recalculated total available to FEAT-16.SPEC-001 (display) and FEAT-16.SPEC-004 (the limit and warning checks)

**Non-Goals:**
- Defining the storage allowance or the warning threshold the recalculated total is checked against -- owned by FEAT-16.SPEC-004 (Storage Limit & Size Ceiling Rules); this automation only produces the total, it does not decide what the total means
- Performing the upload transfer that produces a newly stored file's size -- owned by FEAT-16.SPEC-002 (Resumable Upload Transfer), which signals this automation on completion
- Performing the actual byte-level purge on account deletion -- owned by FEAT-16.SPEC-006 (Stored File Purge on Account Deletion), which signals this automation once the purge completes
- Displaying the total -- owned by FEAT-16.SPEC-001 (Storage Usage Summary), which reads this automation's most recent result

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| An upload transfer completes fully | FEAT-16.SPEC-002 (Resumable Upload Transfer) | Fires the instant a transfer's "Transfer completes fully" outcome is reached | The newly stored file's final size, the freelancer account it belongs to |
| A deliverable is removed | FEAT-06.SPEC-005 (Deliverable Validation & Removal Eligibility Rules) | Fires when Nadia removes a deliverable whose bytes are released (not retained) -- see Business Rules; a removal that retains bytes triggers no recalculation | Deliverable reference, freelancer account |
| Stored bytes are purged | FEAT-16.SPEC-006 (Stored File Purge on Account Deletion) | Fires once a purge completes | The freelancer account whose bytes were purged, total bytes purged |

## Processing Logic

1. Receive the triggering event (upload completion, deliverable removal, or purge completion) and the affected freelancer account.
2. Read the current set of the freelancer's non-purged Deliverable Versions and their stored sizes.
3. Sum the size across every one of those versions to produce the freelancer's current total stored bytes.
4. Persist the recalculated total as the freelancer's current aggregated figure, replacing the previous value.
5. Compare the new total against the freelancer's storage allowance and warning threshold (FEAT-16.SPEC-004) to determine whether the Near Limit or At Capacity condition now applies.
6. Make the recalculated total and the resulting Near Limit/At Capacity condition available for the next time FEAT-16.SPEC-001 loads or FEAT-16.SPEC-004 is checked.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Recalculation completes, total unchanged in warning status | New total is still below the warning threshold (or still above it, if it already was) | Aggregated total updated to the new value | No immediate feedback -- the updated figure is picked up the next time FEAT-16.SPEC-001 loads | FEAT-16.SPEC-001, FEAT-16.SPEC-004 |
| Recalculation completes, Near Limit newly reached | New total crosses into the warning threshold for the first time | Aggregated total updated; Near Limit condition now true | No push notification (this feature's Communications field is N/A) -- the warning appears the next time FEAT-16.SPEC-001 loads, or the next time an upload screen in FEAT-06 checks the condition | FEAT-16.SPEC-001, FEAT-06 (upload screen's inline warning) |
| Recalculation completes, total drops back below the threshold | A removal or purge brings the total back under the warning threshold | Aggregated total updated; Near Limit/At Capacity condition now false | No feedback beyond the figure updating on next view | FEAT-16.SPEC-001, FEAT-16.SPEC-004 |
| Recalculation fails | Processing error while reading versions or summing sizes | The previous total remains in place (no partial or zeroed total is ever persisted) | No user-visible failure -- the last known-good total continues to display until the next successful recalculation | FEAT-16.SPEC-001 |

## Data Model

**Reads:** Deliverable Version -- the current size of every non-purged version belonging to the freelancer's account.
**Creates:** None.
**Updates:** The freelancer's aggregated stored-bytes total (the derived field named in this feature's Non-Functional Notes: "storage usage totals per freelancer, used for plan-limit warnings").
**Deletes:** None.

## Business Rules

- This automation is the sole owner of the freelancer's aggregated stored-bytes total; no other spec computes or caches this figure independently -- FEAT-16.SPEC-001 and FEAT-16.SPEC-004 both read this automation's result rather than recalculating it themselves.
- Removing a deliverable in FEAT-06 does not, by itself, purge its stored bytes -- per this feature's Non-Goals, storage is retained for the life of the account with no automatic purge while active; this automation only recalculates the total when FEAT-06.SPEC-005 explicitly signals a removal that does affect the count (i.e., only when bytes are actually released, not on every removal). Where a removed deliverable's version bytes are retained rather than released, this automation makes no change and the total is unaffected.
- A failed recalculation never zeroes or corrupts the previously known total -- the last successful value remains authoritative until a subsequent recalculation succeeds.
- Recalculation runs synchronously with its trigger and completes before the total is considered current for the next display or limit check -- there is no separate scheduled recalculation pass.

## Edge Cases

- **Two triggers arrive for the same freelancer at effectively the same time (an upload completes while a purge is also finishing)** -- Each recalculation reads the version set as it stands at that moment; the recalculation that reads last reflects both changes, and the total converges to the correct value once both triggers have been processed, even if an intermediate read briefly reflects only one of them.
- **Trigger fires while a previous recalculation for the same freelancer is still in flight** -- The second trigger's recalculation waits for the first to finish reading and summing before it begins its own pass, so recalculations for one freelancer never interleave and never persist an inconsistent total.
- **The freelancer has zero stored bytes (new account, or every version purged)** -- The recalculated total is zero; FEAT-16.SPEC-001 displays "0 of {allowance} used" with no warning.
- **A recalculation is triggered for a freelancer account that no longer exists (a race with the final stage of account deletion)** -- The recalculation is a no-op; there is no remaining account to update, and no error is surfaced, since the account and its figures are being removed regardless.
- **Recalculation runs while FEAT-16.SPEC-001 is open and being viewed** -- The screen does not live-update mid-view; the newly recalculated total is picked up the next time the screen loads or is refreshed, consistent with FEAT-16.SPEC-001's own Edge Cases.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-16.SPEC-002 (Resumable Upload Transfer) | Triggered by (inbound) | Transfer completion signals recalculation |
| FEAT-06.SPEC-005 (Deliverable Validation & Removal Eligibility Rules) | Triggered by (inbound) | A removal that releases stored bytes signals recalculation |
| FEAT-16.SPEC-006 (Stored File Purge on Account Deletion) | Triggered by (inbound) | Purge completion signals recalculation |
| FEAT-16.SPEC-001 (Storage Usage Summary) | Affects (outbound) | Reads this automation's most recent aggregated total for display |
| FEAT-16.SPEC-004 (Storage Limit & Size Ceiling Rules) | Affects (outbound) | The recalculated total is what this spec's allowance and warning-threshold checks are evaluated against |

## Analytics and Success Signals

- **storage_usage_recalculated** (trigger: upload_completed / deliverable_removed / storage_purged, new_total_bytes, warning_state: normal / near_limit / at_capacity) -- supports success-metrics.md: "Large File Upload Success at Scale"
- **storage_usage_recalculation_failed** (trigger) -- N/A -- no Stage 2 metric measures recalculation failure directly; retained so a stale-total condition is observable and diagnosable

## Acceptance Criteria

**FEAT-16.SPEC-005-AC-01:** Given Nadia's upload transfer completes fully, when FEAT-16.SPEC-002 signals completion, then this automation recalculates her total stored bytes to include the newly stored file's size.

**FEAT-16.SPEC-005-AC-02:** Given a removal in FEAT-06.SPEC-005 releases stored bytes, when the removal completes, then this automation recalculates her total to exclude the released bytes.

**FEAT-16.SPEC-005-AC-03:** Given FEAT-16.SPEC-006 completes a storage purge on account deletion, when the purge finishes, then this automation recalculates the total to reflect the purge (down to zero, if all bytes were purged).

**FEAT-16.SPEC-005-AC-04:** Given a recalculation brings Nadia's total to exactly platform parameter: `storage-usage-warning-threshold-percent` of her allowance, when the recalculation completes, then the Near Limit condition becomes true for the next FEAT-16.SPEC-001 load.

**FEAT-16.SPEC-005-AC-05:** Given a recalculation brings Nadia's total back below the warning threshold after a removal, when the recalculation completes, then the Near Limit condition becomes false.

**FEAT-16.SPEC-005-AC-06:** Given a recalculation fails due to a processing error, when the failure occurs, then the previously known total remains in place and is not zeroed or corrupted.

**FEAT-16.SPEC-005-AC-07:** Given a removal in FEAT-06 does not release any stored bytes (the version is retained per this feature's Non-Goals), when the removal completes, then no recalculation is triggered and the total is unaffected.

**FEAT-16.SPEC-005-AC-08:** Given an upload completes and a purge finishes for the same freelancer at effectively the same time, when both recalculations process, then the total converges to correctly reflect both changes.

**FEAT-16.SPEC-005-AC-09:** Given a second recalculation trigger arrives for a freelancer while a prior recalculation for that same freelancer is still in flight, when the second trigger fires, then it waits for the first to finish before it begins, so no inconsistent total is ever persisted.

**FEAT-16.SPEC-005-AC-10:** Given a freelancer has zero stored bytes, when a recalculation runs, then the total resolves to zero and FEAT-16.SPEC-001 shows "0 of {allowance} used" with no warning.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 (upload completed, deliverable removed, storage purged) | 3 |
| Outcome Paths | 4 | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |

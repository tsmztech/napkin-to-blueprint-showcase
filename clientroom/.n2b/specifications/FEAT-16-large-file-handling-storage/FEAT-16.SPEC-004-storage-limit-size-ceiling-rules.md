---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-16.SPEC-004
spec_name: Storage Limit & Size Ceiling Rules
spec_slug: storage-limit-size-ceiling-rules
parent_feature: FEAT-16
parent_feature_name: Large File Handling & Storage
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 9
acceptance_criteria_count: 14
---

# Logic/Rule Spec: Storage Limit & Size Ceiling Rules

## Overview

**Name:** Storage Limit & Size Ceiling Rules
**ID:** FEAT-16.SPEC-004
**Type:** Logic/Rule
**Purpose:** Defines the per-file size ceiling and the per-freelancer storage allowance -- the only two limits this feature enforces -- and the warning threshold that triggers a pre-limit warning, along with who may trigger these checks.
**Parent Feature:** FEAT-16 -- Large File Handling & Storage
**Governed Entity:** Deliverable (storage-capacity fields only)

## Scope and Non-Goals

**In Scope:**
- The per-file size ceiling for an uploaded deliverable file (accommodating video files somewhat over 1 GB)
- The per-freelancer total storage allowance, resolved from the freelancer's Subscription Plan tier
- The warning threshold that triggers a pre-limit, in-product warning
- Authorization for who may trigger a storage-affecting upload and who may view storage figures

**Non-Goals:**
- Validating every other Deliverable field (kind, link validity, milestone requirement) -- owned by FEAT-06.SPEC-005 (Deliverable Validation & Removal Eligibility Rules); this spec governs storage-capacity fields only
- Performing the byte-level transfer or the actual size-ceiling and allowance checks at transfer time -- owned by FEAT-16.SPEC-002 (Resumable Upload Transfer), which enforces this spec's rules during the transfer
- Recalculating the freelancer's current aggregated total -- owned by FEAT-16.SPEC-005 (Storage Usage Aggregation); this spec defines the allowance and threshold the aggregation is checked against, not the aggregation itself
- Displaying the usage figures -- owned by FEAT-16.SPEC-001 (Storage Usage Summary), which reads this spec's allowance and threshold values

## Governed Entity

**Entity:** Deliverable
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| kind | enum | Uploaded file or linked external asset -- out of scope for this spec |
| file or link | text/binary | The uploaded file or reachable link -- this spec governs the file's size only, for uploaded files |
| milestone | reference | Owning Milestone -- out of scope for this spec |
| uploaded_at | date | Upload timestamp -- out of scope for this spec |
| size | number (bytes) | For uploaded files, the file's committed size -- governed by this spec's per-file ceiling rule |
| status | enum | Uploading, Active, Superseded, Removed -- out of scope for this spec |
| link_status | enum | Reachable or flagged -- out of scope for this spec |
| first_client_view_at | date | Timestamp of first client view -- out of scope for this spec |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-16.SPEC-002 | Resumable Upload Transfer | Storage-allowance check, before a transfer begins and again at the moment each transfer's bytes are about to commit |
| FEAT-06.SPEC-005 | Deliverable Validation & Removal Eligibility Rules | Per-file size ceiling check, on file selection, before the transfer begins (restates this spec's ceiling value via the shared platform parameter: `deliverable-file-size-ceiling`) |
| FEAT-16.SPEC-001 | Storage Usage Summary | Displays the allowance, the warning threshold, and the At Capacity condition this spec defines; triggers no check itself |
| FEAT-16.SPEC-005 | Storage Usage Aggregation | Reads the allowance and threshold this spec defines to determine whether the recalculated total should raise a warning |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| kind | No validation beyond data type -- governed by FEAT-06.SPEC-005 | Always | -- | -- | No |
| file or link | For uploaded files, the file's size must not exceed platform parameter: `deliverable-file-size-ceiling` | kind = uploaded file | Before the transfer begins (checked authoritatively by FEAT-16.SPEC-002; the message is shown by the calling screen via FEAT-06.SPEC-005) | "This file is larger than the size limit for deliverables. Compress it or share it by link instead." | Yes |
| milestone | No validation beyond data type -- governed by FEAT-06.SPEC-005 | Always | -- | -- | No |
| uploaded_at | No validation beyond data type | Always | -- | -- | No |
| size | Once committed, the persisted size can never exceed platform parameter: `deliverable-file-size-ceiling` (restates the file/link rule on the finalized value) | For uploaded files, once the transfer commits | On transfer completion | Same message as the file/link rule above -- a committed size over the ceiling cannot occur, because the transfer itself is stopped before completion (FEAT-16.SPEC-002) | Yes |
| status | No validation beyond data type -- Deliverable's status vocabulary is owned by FEAT-06/FEAT-17 | Always | -- | -- | No |
| link_status | No validation beyond data type -- governed by FEAT-06.SPEC-004 | Always | -- | -- | No |
| first_client_view_at | No validation beyond data type -- owned by FEAT-13 | Always | -- | -- | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Storage allowance check | Deliverable.size (the file being transferred), the freelancer's aggregated stored-bytes total (external, maintained by FEAT-16.SPEC-005), Subscription Plan.tier | The freelancer's current aggregated total plus this file's size must not exceed the allowance resolved from her plan tier: platform parameter: `free-tier-storage-allowance` for Free, platform parameter: `paid-tier-storage-allowance` for Paid | "You've reached your storage allowance." |
| Warning threshold | The freelancer's aggregated stored-bytes total, her resolved storage allowance | When the total reaches platform parameter: `storage-usage-warning-threshold-percent` of the allowance, the Near Limit warning condition is true for display on FEAT-16.SPEC-001 | N/A -- non-blocking advisory, not an error; displayed inline per FEAT-16.SPEC-001, never as a delivery-channel notification (this feature's Communications field is N/A) |
| At capacity condition | The freelancer's aggregated stored-bytes total, her resolved storage allowance | When the total reaches (or would be pushed to) 100% of the allowance, the At Capacity condition is true, and a further upload is blocked by the storage allowance check above | "You've reached your storage allowance. New uploads won't complete until you free up space or upgrade your plan." |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Upload a file (subject to the size ceiling and storage allowance) | Nadia (Freelancer) | Always, subject to the Field Validation and Cross-Field rules above | -- |
| Upload a file (subject to the size ceiling and storage allowance) | Owen (Client Primary Contact), Priya (Client Reviewer Contact) | Never | No upload control exists on any client-facing screen; the Access Matrix's Milestones & Deliverables entitlement for both roles is Own-only (view, comment, approve for Owen; view, comment for Priya), with no upload capability |
| Upload a file (subject to the size ceiling and storage allowance) | Dana (Support Operator) | Never | No upload control exists in a support session; Dana's Milestones & Deliverables entitlement is View, with no file downloads or uploads (ASMP-18, XBR-29) |
| View storage usage figures (total, allowance, warning state) | Nadia (Freelancer) | Always | -- |
| View storage usage figures (total, allowance, warning state) | Dana (Support Operator) | Always, read-only, only during an active support session on that freelancer's account (FEAT-31.SPEC-003) | Outside an active support session, the figures are unreachable; no separate denied experience, since no navigation path exists |
| View storage usage figures (total, allowance, warning state) | Owen (Client Primary Contact), Priya (Client Reviewer Contact) | Never | No storage-usage surface exists in the client portal; the Access Matrix's Subscription & Account Data entitlement for both roles is None |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Deliverable.size | Derived from the committed transfer's final byte count (FEAT-16.SPEC-002) | On transfer completion | No |
| Freelancer's aggregated stored-bytes total (external) | Sum of the current size across every non-purged Deliverable Version the freelancer owns, recalculated by FEAT-16.SPEC-005 | On upload completion, deliverable removal, or storage purge | No |
| Resolved storage allowance | Looked up from the freelancer's current Subscription Plan tier: platform parameter: `free-tier-storage-allowance` for Free, platform parameter: `paid-tier-storage-allowance` for Paid | Always -- re-evaluated at each check against the live plan, never cached across a plan change | No |
| Near Limit / At Capacity condition | Derived by comparing the aggregated total to the resolved allowance and the warning threshold | Always -- re-evaluated on every display or enforcement check | No |

## Business Rules

- XBR-14: the per-file size ceiling and the per-freelancer storage allowance are the only two limits this feature enforces, and a pre-limit warning is raised before the allowance is reached; the ceiling is set to accommodate video files somewhat over 1 GB.
- The per-file ceiling and the per-freelancer allowance are independent checks: a file can pass the size ceiling and still be rejected by the storage allowance if the freelancer's account is already near full, and vice versa a file within the allowance's remaining headroom is still rejected if it individually exceeds the per-file ceiling.
- A linked external asset (Figma, Drive, Dropbox) never counts against the per-freelancer storage allowance, since this feature's storage capability only ever stores files the freelancer actually uploads (Non-Goals: linked assets are referenced by URL, never copied in).
- The storage allowance is read from the freelancer's live Subscription Plan tier at the moment of every check -- a plan upgrade takes effect for the very next check, with no separate re-provisioning step.

## Edge Cases

- **File size exactly at the per-file ceiling** -- Passes validation; one byte over fails.
- **Freelancer's projected total exactly at her allowance** -- Passes (the allowance is inclusive of its own boundary); one byte over fails.
- **Freelancer's total exactly at the warning threshold** -- The Near Limit condition is true; the boundary is inclusive of the threshold value.
- **Freelancer downgrades her plan while already over the new, lower allowance's threshold** -- No existing stored bytes are purged or blocked from delivery; the At Capacity condition becomes true immediately for any further upload, and the warning shows on FEAT-16.SPEC-001, but nothing already stored is affected (consistent with this feature's Non-Goal: no automatic purge of active deliverables' storage while the account remains active).
- **Freelancer's plan is upgraded mid-transfer (the allowance increases while a transfer that would have exceeded the old allowance is in flight)** -- The transfer's commit-time allowance re-check (FEAT-16.SPEC-002) uses the plan as it stands at that moment, so an upgrade that lands before the commit point lets the transfer complete against the new, higher allowance.
- **Dana's support session is opened for a freelancer already At Capacity** -- Dana sees the same At Capacity figures read-only; no upload control is ever shown to her regardless of the freelancer's capacity state.

## Acceptance Criteria

**FEAT-16.SPEC-004-AC-01:** Given Nadia selects a file exactly at platform parameter: `deliverable-file-size-ceiling`, when the size check runs, then the file passes.

**FEAT-16.SPEC-004-AC-02:** Given Nadia selects a file one byte over platform parameter: `deliverable-file-size-ceiling`, when the size check runs, then she sees "This file is larger than the size limit for deliverables. Compress it or share it by link instead." and the transfer does not begin.

**FEAT-16.SPEC-004-AC-03:** Given Nadia's projected total (current total plus the new file's size) is exactly at her resolved storage allowance, when the allowance check runs, then the transfer proceeds.

**FEAT-16.SPEC-004-AC-04:** Given Nadia's projected total would be one byte over her resolved storage allowance, when the allowance check runs, then she sees "You've reached your storage allowance." and the transfer does not begin.

**FEAT-16.SPEC-004-AC-05:** Given Nadia's aggregated total reaches exactly platform parameter: `storage-usage-warning-threshold-percent` of her allowance, when FEAT-16.SPEC-001 next loads, then it shows the Near Limit warning.

**FEAT-16.SPEC-004-AC-06:** Given Nadia's aggregated total is below platform parameter: `storage-usage-warning-threshold-percent` of her allowance, when FEAT-16.SPEC-001 next loads, then no warning is shown.

**FEAT-16.SPEC-004-AC-07:** Given Nadia is on the free tier, when her resolved allowance is looked up, then it equals platform parameter: `free-tier-storage-allowance`; given she is on a paid tier, then it equals platform parameter: `paid-tier-storage-allowance`.

**FEAT-16.SPEC-004-AC-08:** Given Nadia attempts to upload a file, when the action is evaluated, then it is allowed (subject to the size and allowance checks); given Owen, Priya, or Dana attempts the same action, then no upload control is available to them at all.

**FEAT-16.SPEC-004-AC-09:** Given Dana opens a support session on a freelancer's account, when she views the storage figures, then she sees them read-only; given Owen or Priya looks for any storage-usage surface, then none exists in the client portal.

**FEAT-16.SPEC-004-AC-10:** Given Nadia references a Figma or Drive link as a deliverable rather than uploading a file, when her aggregated total is recalculated, then the linked asset contributes nothing to it.

**FEAT-16.SPEC-004-AC-11:** Given Nadia downgrades her plan and her existing stored bytes now exceed the new, lower allowance, when the downgrade takes effect, then no existing stored file is purged or blocked from delivery, and any further upload is blocked by the At Capacity condition.

**FEAT-16.SPEC-004-AC-12:** Given a transfer that would have exceeded Nadia's old allowance is in flight when she completes a plan upgrade, when the transfer reaches its commit-time allowance re-check, then it is evaluated against the new, higher allowance.

**FEAT-16.SPEC-004-AC-13:** Given Nadia's committed Deliverable.size is checked after a transfer completes, when the check runs, then it can never exceed platform parameter: `deliverable-file-size-ceiling`, since FEAT-16.SPEC-002 stops any transfer that would breach it before completion.

**FEAT-16.SPEC-004-AC-14:** Given the freelancer's storage allowance is read for any check, when it is looked up, then it always reflects her current Subscription Plan tier at that exact moment, never a cached or stale value.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 8 | 8 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 6 | 6 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |

---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-17.SPEC-004
spec_name: Version Numbering, Immutability & Retention Rules
spec_slug: version-numbering-immutability-retention-rules
parent_feature: FEAT-17
parent_feature_name: Deliverable Version History
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 13
acceptance_criteria_count: 14
---

# Logic/Rule Spec: Version Numbering, Immutability & Retention Rules

## Overview

**Name:** Version Numbering, Immutability & Retention Rules
**ID:** FEAT-17.SPEC-004
**Type:** Logic/Rule
**Purpose:** Governs sequential round numbering, immutability once uploaded, the uncapped-but-storage-bounded version count, and the "latest version" derivation for every Deliverable Version.
**Parent Feature:** FEAT-17 -- Deliverable Version History
**Governed Entity:** Deliverable Version

## Scope and Non-Goals

**In Scope:**
- Round_number sequencing: assignment, ordering, and gaplessness
- Immutability: what can never be changed on an existing version, and by whom
- Version-count and retention rules: the uncapped-but-storage-bounded ceiling, and how long versions are kept
- The `is_latest` derivation formula (highest round_number for the deliverable; derived, never stored)
- Authorization outcomes for actions that would violate immutability (edit, delete, manual "latest" override), for every role

**Non-Goals:**
- Who may view or create a version in the first place -- governed by FEAT-17.SPEC-005 (Version Access & Comment-Anchoring Rules); this spec addresses only the numbering, immutability, and retention constraints on the entity once a create is already authorized
- The comment-anchoring rule (a comment stays attached to the version it was posted on) -- governed by FEAT-17.SPEC-005, since it concerns the Comment entity's relationship to a version, not the version's own numbering or immutability
- The mechanics of the re-upload transfer itself (progress, pause/resume, failure handling) -- owned by FEAT-17.SPEC-003 (Version Creation & Preservation), which applies the rules defined here

## Governed Entity

**Entity:** Deliverable Version
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| round_number | number | Sequential position of this version among the deliverable's rounds, starting at 1 |
| file | file reference | The uploaded file for this specific round |
| uploaded_at | date/time | Timestamp this round's transfer completed |
| is_latest | derived (boolean) | True only for the version with the highest round_number for its deliverable |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-06.SPEC-003 | Resumable Upload Handling | Creates round_number = 1 on the deliverable's first upload completion; establishes the sequence this spec continues |
| FEAT-17.SPEC-003 | Version Creation & Preservation | Assigns round_number = previous highest + 1 on every subsequent completion (create-only); never writes to any field of any prior version, and stores no is_latest value -- the new round becomes latest purely by holding the highest round_number |
| FEAT-17.SPEC-001 | New Version Upload | Displays the round_number just added ("Round {N} added") after the transfer completes, reflecting this spec's sequencing formula |
| FEAT-17.SPEC-002 | Version Browser & Comparison | Displays every round ordered by round_number and labels the is_latest round, per this spec's derivation; renders no edit or delete control for any version |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| round_number | Must equal the deliverable's current highest round_number plus one; never user-entered | Always | On version creation (FEAT-17.SPEC-003) | N/A -- system-assigned, never presented to any user as an editable field | Yes |
| file | Required; must be a successfully and fully transferred file (no partial-transfer commit); subject to the reused per-file size ceiling (platform parameter: `deliverable-file-size-ceiling`, per FEAT-06.SPEC-005) | Always | On version creation (FEAT-17.SPEC-003, before commit) | "This file is larger than the size limit for deliverables. Compress it or share it by link instead." (size ceiling); "This file appears to be empty." (zero bytes) | Yes |
| uploaded_at | Required; system-assigned to the transfer's completion time; never user-entered | Always | On version creation | N/A -- system-assigned | Yes |
| is_latest | No validation beyond data type -- purely derived (evaluated from round_number on read, never stored, never user-entered or user-editable) | Always | Evaluated whenever a version list is read | N/A -- not a user-facing input | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Round sequence continuity | round_number (across all versions of one deliverable) | For a given deliverable, the set of round_numbers is exactly {1, 2, ..., N} with no gaps and no duplicates, where N is the deliverable's current version count | N/A -- enforced structurally by FEAT-17.SPEC-003's assignment logic; no user-facing error exists because no user ever supplies this value |
| Latest-uniqueness | is_latest (across all versions of one deliverable) | Exactly one version per deliverable has is_latest = true at any moment: the one with the highest round_number | N/A -- follows structurally from derivation: the version with the highest round_number is the latest, so committing a new highest round_number changes which version qualifies without any write to another version; never presented as an editable state |

## Authorization Rules

Who may create, edit, delete, or otherwise change a Deliverable Version's numbering, content, or latest status. Who may view or create a version at all is governed by FEAT-17.SPEC-005; this spec's authorization table addresses only the actions that immutability and retention constrain.

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create the next Deliverable Version | Nadia | Only via a fully completed re-upload transfer (FEAT-17.SPEC-003); who else may trigger this at all is governed by FEAT-17.SPEC-005 | No create control is rendered for Owen, Priya, or Dana anywhere in the product (FEAT-17.SPEC-005) |
| Edit any field of an existing Deliverable Version (round_number, file, uploaded_at) | No one | Never -- immutable once uploaded, regardless of role | No edit control exists anywhere in the product for any role; a version's file and metadata are permanently fixed the instant it is created |
| Delete or purge an individual Deliverable Version while the account is active | No one | Never -- "never deleted in-product; removed only by FEAT-24" (dependency map) | No delete or purge control exists anywhere in the product; the only lifecycle paths are superseding with a new round (FEAT-17.SPEC-001/003) or full account deletion (FEAT-24), executed at the storage layer by FEAT-16.SPEC-006 |
| Manually mark an older round as "latest" / current | No one | Never -- is_latest is purely derived from round_number with no override mechanism described anywhere in Stage 2 | No override control exists; the only way to make an older round's content current again is to upload it again as a brand-new round through FEAT-17.SPEC-001/003 |
| View a version's round_number, uploaded_at, and file | Governed by FEAT-17.SPEC-005 | See FEAT-17.SPEC-005 (Version Access & Comment-Anchoring Rules) | See FEAT-17.SPEC-005 |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| round_number | 1 for a deliverable's first version (FEAT-06.SPEC-003); for every subsequent version, the deliverable's current highest round_number + 1 | On create only, computed at the moment a transfer commits | No |
| uploaded_at | Current date/time at the moment the transfer completes fully | On create only | No |
| is_latest | True for the version whose round_number is the highest among all versions of its deliverable; false for every other version of that deliverable | Evaluated on every read; it changes for a deliverable the instant a new version with a higher round_number commits, with no write to any version | No (no override mechanism exists anywhere in Stage 2) |

## Business Rules

- XBR-13: each re-upload creates a new immutable version; comments stay attached to the version they were made on (FEAT-17.SPEC-005); all versions count against the freelancer's storage allowance.
- XBR-14: no fixed cap exists on version count per deliverable; the only limit is the freelancer's aggregate storage allowance (platform parameter: `free-tier-storage-allowance` for Free, platform parameter: `paid-tier-storage-allowance` for Paid), enforced by FEAT-16.SPEC-004 at the moment a new transfer would begin -- not by this spec, which defines only that no version-count ceiling of its own exists.
- Retention: every Deliverable Version is retained for the life of the account with no automatic purge while the account is active (ASMP-22, dependency map); it is removed only as part of account deletion, executed at the storage layer by FEAT-16.SPEC-006 (FEAT-24).
- A version's `is_latest` state is never itself a stored field a person sets -- it is evaluated as a pure function of round_number (highest round_number for the deliverable) each time versions are read, and no write to any existing version is ever involved when a new version is created (dependency map: "Derived: 'latest version' is the most recent upload").
- Immutability holds by omission: the product defines no edit path for any field of an existing Deliverable Version anywhere in Stage 2, so this spec's Authorization Rules table records "never" rather than describing a blocked-but-existing control.

## Edge Cases

- **Two near-simultaneous re-upload transfers for the same deliverable both complete** -- No round_number collision occurs: FEAT-17.SPEC-003 re-reads the current highest round_number at the moment each transfer commits, so whichever commits first is assigned the next number and the second, committing moments later, is assigned the number after that. Both are preserved; the sequence stays gapless.
- **A re-upload transfer fails partway through** -- No Deliverable Version row is ever created for that attempt; round_number sequencing is untouched -- no round is reserved, skipped, or left as a placeholder before a transfer fully completes.
- **A deliverable has exactly one version** -- Its round_number is 1 and its is_latest is trivially true; the "Latest" label carries no special meaning until a second round exists (surfaced by FEAT-17.SPEC-002 hiding the selector entirely in this case).
- **A deliverable accumulates a very large number of versions (hundreds of rounds over a long-running project)** -- No rule in this spec blocks further creation; the only possible blocker is the freelancer's storage allowance being exhausted (FEAT-16.SPEC-004), which is a separate feature's rule, not a version-count ceiling of this spec's own.
- **An attempt is made to reorder or renumber existing rounds (e.g., after discovering an upload was made in error)** -- Not applicable: the product defines no reordering or renumbering capability anywhere in Stage 2; the only corrective path is uploading a further new round, which always takes the next sequential number regardless of the reason for the upload.
- **Account deletion is requested while multiple versions exist** -- Every Deliverable Version and its stored bytes is removed as part of FEAT-24's account deletion, executed at the storage layer by FEAT-16.SPEC-006; this spec's retention rule ends exactly there, with no partial or selective retention of some rounds over others.

## Acceptance Criteria

**FEAT-17.SPEC-004-AC-01:** Given a deliverable's current highest round is 2, when FEAT-17.SPEC-003 commits a new version, then it is assigned round_number 3.

**FEAT-17.SPEC-004-AC-02:** Given a new version commits at round_number 3, when is_latest is evaluated, then Round 3 qualifies as latest and Round 2 does not, with no field of Round 1 or Round 2 written or changed.

**FEAT-17.SPEC-004-AC-03:** Given Nadia (or anyone) looks for an edit control on any existing Deliverable Version anywhere in the product, when the search is made, then none exists -- every field is permanently fixed once the version is created.

**FEAT-17.SPEC-004-AC-04:** Given Nadia looks for a delete or purge control on an individual Deliverable Version while her account is active, when the search is made, then none exists; the only path to a changed current state is uploading a new round.

**FEAT-17.SPEC-004-AC-05:** Given Nadia has just uploaded a new, later round, when she looks for a way to mark an older round "current" again, then no such control exists anywhere in the product.

**FEAT-17.SPEC-004-AC-06:** Given a deliverable has only one version, when its is_latest is evaluated, then it is true, and FEAT-17.SPEC-002 shows no version selector as a result.

**FEAT-17.SPEC-004-AC-07:** Given two of Nadia's sessions each complete a re-upload transfer for the same deliverable within moments of each other, when both commit, then they are assigned sequential, non-colliding round_numbers (e.g., 3 and 4) with no gap and no duplicate.

**FEAT-17.SPEC-004-AC-08:** Given a re-upload transfer fails partway through, when the failure is recorded, then no Deliverable Version row exists for that attempt and the round_number sequence is unaffected.

**FEAT-17.SPEC-004-AC-09:** Given a deliverable has accumulated 40 versions over the life of a long-running project, when a 41st re-upload is attempted, then no version-count rule in this spec blocks it -- only the freelancer's storage allowance (FEAT-16.SPEC-004) can prevent the transfer from completing.

**FEAT-17.SPEC-004-AC-10:** Given a file selected for a new version is zero bytes, when FEAT-17.SPEC-003 evaluates it, then no Deliverable Version is created and the file field validation rejects it before any round_number is assigned.

**FEAT-17.SPEC-004-AC-11:** Given a file selected for a new version exceeds platform parameter: `deliverable-file-size-ceiling`, when FEAT-17.SPEC-003 evaluates it, then the file field validation rejects it and no Deliverable Version is created.

**FEAT-17.SPEC-004-AC-12:** Given Nadia's account is deleted (FEAT-24), when the deletion completes, then every Deliverable Version and its stored bytes belonging to her account has been removed, with no rounds retained selectively.

**FEAT-17.SPEC-004-AC-13:** Given Owen, Priya, or Dana looks for a create control for a new Deliverable Version, when the search is made, then none is rendered for any of them -- only Nadia can trigger a re-upload (per FEAT-17.SPEC-005's authority).

**FEAT-17.SPEC-004-AC-14:** Given a deliverable's version history is inspected at any point in its life, when the round_number sequence is examined, then it forms an unbroken run from 1 to the current version count with no gaps and no duplicate numbers.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

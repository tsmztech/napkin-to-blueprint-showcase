---
document_type: feature-overview
feature_number: FEAT-17
feature_name: Deliverable Version History
feature_slug: deliverable-version-history
priority_tier: Important
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 5
screen_count: 2
automation_count: 1
logic_rule_count: 2
integration_count: 0
notification_count: 0
---

# Feature Breakdown Brief: Deliverable Version History

## Summary

**Feature:** Deliverable Version History
**ID:** FEAT-17
**Description:** Every re-upload to a deliverable keeps the prior version rather than overwriting it, so both freelancer and client can see and open any earlier round rather than losing track in a folder of "final_v3_REAL" files.
**Priority:** Important
**Phase:** MVP
**Type:** User-Facing
**Rationale:** BRIEF.md's Problem Statement names this exact failure mode by name ("final_v3_REAL" versions), and its Scale section lists "version history per deliverable" as a day-one norm. Ranked Important rather than Core because a single-round deliverable still works without it. [MODIFIED: phase moved from v1 to MVP based on BRIEF.md's Scale section naming version history per deliverable as the norm, the draft scope boundaries (Scale Expectations) already committing to it from MVP, and research finding versioned deliverable handling absent from all 5 profiled competitors (Feature Landscape, Absent Features) — the product's most direct fix for the brief's named problem should not wait for a later release] [INFERRED: carried from Visionary draft]

**Key Capabilities:**
- Preserve every version — a re-upload never overwrites a prior round
- Browse by version — open and compare any round by number
- Version-anchored comments — feedback stays attached to the version it was made on

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-17.SPEC-001 | New Version Upload | Screen | Nadia | Nadia re-uploads a revised file to an existing deliverable, adding a new preserved round rather than replacing the current one |
| FEAT-17.SPEC-002 | Version Browser & Comparison | Screen | Nadia, Owen, Priya, Dana | Any viewer opens the version selector on a deliverable, browses every round by number, and opens any earlier round alongside the latest |
| FEAT-17.SPEC-003 | Version Creation & Preservation | Automation | Nadia | On a fully completed re-upload, creates the next immutable Deliverable Version, leaves every prior version untouched, and updates which round is "latest" |
| FEAT-17.SPEC-004 | Version Numbering, Immutability & Retention Rules | Logic/Rule | Nadia | Governs sequential round numbering, immutability once uploaded, the uncapped-but-storage-bounded version count, and the "latest version" derivation |
| FEAT-17.SPEC-005 | Version Access & Comment-Anchoring Rules | Logic/Rule | Nadia, Owen, Priya, Dana | Governs who may view or add versions per the Access Matrix, and the rule that a comment stays attached to the version it was posted on even after newer rounds exist |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Preserve every version | FEAT-17.SPEC-001, FEAT-17.SPEC-003, FEAT-17.SPEC-004 | The upload screen collects the re-upload; the creation automation commits it as a new immutable round without touching prior rounds; the numbering/immutability rule spec is the standing contract that makes "never overwrites" true on every future round | Phase 2 (Explicit) |
| Browse by version | FEAT-17.SPEC-002 | The version browser lists every round by number and opens any one, hidden entirely when only one round exists | Phase 2 (Explicit) |
| Version-anchored comments | FEAT-17.SPEC-005 | The access/anchoring rule spec is the authority FEAT-07's comment screens reference so a comment stays tied to the round it was posted on regardless of later uploads | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-17.SPEC-003 | Version Creation & Preservation | Phase 4 (Trigger-Response Analysis) | The feature's States field ("a failed version upload does not overwrite the existing version") and Primary Flows & Alternates describe processing logic with a real failure mode -- version creation is gated on full upload completion, not a direct field write -- crossing the standalone-Automation threshold |
| FEAT-17.SPEC-004 | Version Numbering, Immutability & Retention Rules | Phase 5 (Rule-Constraint Discovery) | The Validation & Limits field ("no fixed cap on version count... within the plan's storage allowance," "each version is immutable once uploaded") and the Data Notes field's derivation rule ("latest version" is the most recent upload) are conditional, cross-screen rules that both SPEC-001 and SPEC-003 must obey consistently, crossing the shared-rule threshold |
| FEAT-17.SPEC-005 | Version Access & Comment-Anchoring Rules | Phase 5 (Rule-Constraint Discovery) | The Access field carries four distinct role permissions (Nadia Full, Owen/Priya Own-only view, Dana View-only inside a logged support session) and the Key Capabilities' comment-anchoring rule is authorization- and state-dependent logic shared across this feature's own screens and FEAT-07's comment screens -- not a simple inline validation |

## Entity-Lifecycle Coverage Matrix

**Entity: Deliverable Version**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-17.SPEC-003 | On a fully completed re-upload, creates a new Deliverable Version at the next sequential round_number, carrying the uploaded file | The first version (round_number 1) is created by FEAT-06.SPEC-003 on the deliverable's initial upload -- out of this feature's scope; this feature owns round 2 onward |
| Read (single) | FEAT-17.SPEC-002 | Version browser opens one round's file and metadata (round_number, uploaded_at) | Also read by FEAT-07 (version-anchored comment screens) and FEAT-31 (Dana's support session), outside this feature's own screens |
| Read (list) | FEAT-17.SPEC-002 | Version browser lists every round for a deliverable, ordered by round_number, hidden when only one round exists (Empty state) | -- |
| Update | N/A | The dependency map states Deliverable Version is "never updated (immutable once uploaded)" -- an explicit product decision (Validation & Limits: "each version is immutable once uploaded"), not a gap | Recorded in Non-Goals |
| Delete/Archive | N/A | "Never deleted in-product; removed only by FEAT-24" (dependency map). No soft-delete state exists for a version: hard deletion happens only at account deletion, executed at the storage layer by FEAT-16.SPEC-006; no restore path is needed because no in-product deletion path exists; no cascade beyond the deleted account's own records; retained for the life of the account with no automatic purge while the account is active (ASMP-22) | An explicit non-goal, not an omission -- recorded in Non-Goals |
| State Transition | N/A | `is_latest` is a derived field (Data Notes: "Derived: 'latest version' is the most recent upload"), not a stored state; the entity carries no status field | Derivation logic is owned by FEAT-17.SPEC-004 |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Deliverable | FEAT-17.SPEC-001, FEAT-17.SPEC-002 | The re-upload screen is scoped to one existing deliverable; the version browser is reached from that deliverable's view and shows the deliverable's own identity (name, milestone) alongside its version list |
| Comment | FEAT-17.SPEC-005 | The anchoring rule reasons about which version a comment is pinned to, but comment creation, editing, and retraction remain owned by FEAT-07 -- this feature never writes a Comment record |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Nadia uploads a revised file to an existing deliverable | Begin the re-upload transfer; the deliverable's current (prior) version stays Active and visible throughout | Standalone Automation | FEAT-17.SPEC-003 |
| The re-upload transfer completes fully | Create the next Deliverable Version at the next round_number, mark it `is_latest`, leave every prior version's own record untouched; emit `version_uploaded` | Standalone Automation | FEAT-17.SPEC-003 |
| The re-upload transfer fails or is interrupted | Do not create a new version and do not touch the existing one -- the prior version remains intact and current; no premature client notification fires (XBR-12) | Standalone Automation | FEAT-17.SPEC-003 |
| A new version becomes fully ready | Reuse FEAT-06's "new deliverable ready" notification for the relevant client contacts, for this version | Cross-feature | FEAT-06.SPEC-006 (reused as-is; no new Notification spec required -- see Shared Context) |
| A viewer opens the version browser on a deliverable | Determine round numbering and which round is "latest" | Standalone Logic/Rule | FEAT-17.SPEC-004 |
| A viewer opens the version browser or a re-upload screen | Check role-based access (Nadia Full; Owen/Priya Own-only view; Dana View-only, no download, inside a logged support session) before showing the screen | Standalone Logic/Rule | FEAT-17.SPEC-005 |
| A viewer opens the version browser | Show every round by number; hide the selector entirely when the deliverable has only one version | Inline in triggering screen | FEAT-17.SPEC-002 |
| A viewer switches between versions of a large file | Show a brief loading indicator while the selected round's file loads | Inline in triggering screen | FEAT-17.SPEC-002 |
| Nadia opens the re-upload screen and submits | Confirm the round was added and return to the deliverable/version view | Inline in triggering screen | FEAT-17.SPEC-001 |
| A client contact posts a comment while viewing an older round | The comment stays anchored to that round even after a newer round is uploaded | Standalone Logic/Rule | FEAT-17.SPEC-005 (write itself stays inline in FEAT-07's comment screens) |
| Any version upload | Transfers and stores bytes through the shared large-file storage capability and counts against the freelancer's storage allowance (XBR-13, XBR-14) | Cross-feature | FEAT-16.SPEC-002 / FEAT-16.SPEC-007 responsibility |
| A new version is created, or a removal is blocked on an approved milestone (XBR-11) | An append-only activity-trail entry is written with actor and timestamp | Cross-feature | FEAT-13 responsibility |
| Dana opens a read-only support session | Sees the version browser's list and metadata but never downloads a version's file | Cross-feature | FEAT-31 responsibility (governs the session; FEAT-17.SPEC-005 enforces the no-download constraint within it) |

## Shared Context

**Shared Entities:**
- Deliverable Version -- created (round 2 onward) by SPEC-003; listed and opened by SPEC-002; governed by SPEC-004 (numbering, immutability, retention) and SPEC-005 (access, anchoring). Fields: round_number, file, uploaded_at, is_latest (derived).
- Deliverable (read-only) -- read by SPEC-001 to scope the re-upload and by SPEC-002 to anchor the version browser to its parent deliverable; created and owned entirely by FEAT-06.

**Shared UI Patterns:**
- Version selector -- SPEC-002 is the single implementation FEAT-07's deliverable view navigates into (dependency map navigation row: "deliverable view → version selector / earlier round"); it is hidden when a deliverable has exactly one version (Empty state) and shows a brief loading indicator when switching rounds on a large file (Loading state). Spec Writers should describe this component once and have FEAT-07 reference it rather than re-specifying it.
- Round-number and "latest" labeling -- the same round_number and is_latest vocabulary appears in SPEC-001's post-upload confirmation and throughout SPEC-002's browser; Spec Writers should keep the labeling identical across both.

**Shared Validation:**
- SPEC-004 defines every numbering, immutability, and retention rule (sequential round_number with no gaps, immutability once uploaded, no fixed version-count cap bounded by the storage allowance, is_latest derivation). SPEC-001 and SPEC-003 both reference SPEC-004 rather than duplicating these rules.
- SPEC-005 defines the only access and anchoring rules this feature has. SPEC-001, SPEC-002, and FEAT-07's comment screens all reference SPEC-005 rather than re-deriving who may view/upload a version or how a comment binds to its round.

**Flagged discrepancy (not resolved by this Brief):** This feature carries `integration_count: 0` and `notification_count: 0` by deliberate resolution, not omission. The dependency map's External Touchpoints row lists this feature's large-file storage dependency as "pending, assigned when its batch is validated" -- resolved here the same way FEAT-06 (already validated) resolved its identical dependency: this feature consumes the shared storage capability entirely through FEAT-16.SPEC-007 and owns no Integration spec of its own (mirrors FEAT-06.SPEC-003 → FEAT-16, XBR-14). Separately, the Communications field names only a reuse of FEAT-06's existing "new deliverable ready" notification for each new version, with no new channel, audience, or content of its own -- so no new Notification spec is warranted here either. Both zero counts are surfaced for the Requirements Architect, not silently left as gaps.

## Internal Dependency Map

```
SPEC-001 (New Version Upload) -> [Nadia re-uploads a file to an existing deliverable] -> SPEC-003 (Version Creation & Preservation)
SPEC-003 (Version Creation & Preservation) -> [applies numbering/immutability using] -> SPEC-004 (Version Numbering, Immutability & Retention Rules)
SPEC-003 (Version Creation & Preservation) -> [checks role access using] -> SPEC-005 (Version Access & Comment-Anchoring Rules)
SPEC-003 (Version Creation & Preservation) -> [new round created, is_latest updated] -> SPEC-002 (Version Browser & Comparison)
SPEC-003 (Version Creation & Preservation) -> [version fully ready] -> FEAT-06.SPEC-006 (Deliverable Ready Notification, reused per version)
SPEC-002 (Version Browser & Comparison) -> [checks role access using] -> SPEC-005 (Version Access & Comment-Anchoring Rules)
SPEC-002 (Version Browser & Comparison) -> [viewer opens an older round and comments] -> FEAT-07 (Deliverable Review & Feedback comment thread) -> [comment bound to that round using] -> SPEC-005
```

**Default Entry:** SPEC-002 (Version Browser & Comparison) -- reached from FEAT-07's deliverable view via the version selector whenever more than one round exists; SPEC-001 is reached separately when Nadia chooses to add a new round to an existing deliverable.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-17.SPEC-001 | Inbound | FEAT-06 (Deliverable Upload & Sharing) | Nadia reaches "Upload New Version" from an existing deliverable's management screen | Nadia chooses "replace" on FEAT-06.SPEC-002 |
| FEAT-17.SPEC-003 | Outbound | FEAT-16 (Large File Handling & Storage) | The re-upload's bytes are transferred and stored through FEAT-16's resumable transfer, and count against the storage allowance | Nadia uploads a new version (XBR-13, XBR-14) |
| FEAT-17.SPEC-003 | Outbound | FEAT-06 (Deliverable Upload & Sharing) | On full completion, reuses FEAT-06.SPEC-006's deliverable-ready notification for the new version, never for a partial or failed upload | A new version finishes uploading (XBR-12) |
| FEAT-17.SPEC-003 | Inbound | FEAT-08 (Milestone Approval) | An approved milestone blocks outright deliverable removal but still allows superseding through a new version, which this feature provides | Owen approves the milestone; Nadia later uploads a new round (XBR-11) |
| FEAT-17.SPEC-002 | Inbound | FEAT-07 (Deliverable Review & Feedback) | The deliverable view's version selector opens directly into this feature's version browser | A viewer opens an earlier version of the deliverable |
| FEAT-17.SPEC-005 | Outbound | FEAT-07 (Deliverable Review & Feedback) | Governs that a comment posted on an older version stays anchored to that version regardless of later uploads | Nadia, Owen, or Priya comments on any version |
| FEAT-17.SPEC-002 / FEAT-17.SPEC-005 | Inbound | FEAT-31 (Operator Support Access) | Dana's read-only, logged support session can browse versions but never download a version's file | Dana opens a support session on the freelancer's account |
| FEAT-17.SPEC-003 | Outbound | FEAT-13 (Immutable Activity & Audit Trail) | Every new version creation, and every removal blocked in favor of supersession, writes an append-only trail entry with actor and timestamp | A new version is created, or removal is blocked on an approved milestone |
| FEAT-17.SPEC-003 / FEAT-17.SPEC-004 | Outbound | FEAT-24 (Account Deletion) | Deliverable Version records and their stored bytes are removed only as part of account deletion, executed at the storage layer by FEAT-16.SPEC-006 | Nadia's account is deleted |

## Non-Functional Notes

**Data volumes / growth:** Every preserved version counts against the freelancer's storage allowance (XBR-13), and version accumulation is named directly as the main driver of year-one storage volume (success-metrics.md, Large File Upload Success at Scale); versions are retained with no fixed cap and no automatic purge for the life of the account (ASMP-22), so this feature's own data volume grows without bound across an account's lifetime by design.

**Responsiveness:** Switching between versions of a large file shows a brief, visible loading indicator rather than an indefinite spinner (feature's States field, ASMP-27); the client-facing version browser and comment-anchoring flow fall under the client-facing 2-second interactivity target (ASMP-21), directly measured by the Client Portal Mobile Responsiveness success metric since clients switch between and open versions almost entirely on mobile browsers.

**Data sensitivity / privacy:** Each Deliverable Version carries the same client-confidential work product as its parent Deliverable (design files, videos, documents that may themselves contain personal data), strictly isolated per client (ASMP-23); Dana's read-only support session can list and browse versions but never download the underlying file (ASMP-23, mirroring FEAT-06/FEAT-16's Support Access boundary).

**Compliance flags:** GDPR-class handling applies only to the extent a version's file content or its anchored comments carry personal data (ASMP-24) -- the version metadata this feature itself manages (round_number, uploaded_at) carries no personal data of its own; no card or payment data is ever involved in this feature.

## Non-Goals

- **Editing or replacing the file of an already-uploaded version** -- Excluded by the feature's own Validation & Limits field ("each version is immutable once uploaded"); the only path to a changed file is a new round through FEAT-17.SPEC-001/SPEC-003, never an in-place edit of an existing round.
- **Manually deleting or purging an individual version while the account is active** -- Intentional lifecycle decision surfaced by the CRUD matrix and carried from the dependency map ("never deleted in-product; removed only by FEAT-24") and ASMP-22 (retained for the life of the account); a version's evidentiary and version-history role means no in-product delete or purge action exists.
- **Manually marking an older round as "current" or "latest"** -- Intentional design decision: Data Notes defines `is_latest` purely as a derived field ("the most recent upload"), with no override mechanism described anywhere in Stage 2; the only way to make an older round's content current again is to upload it again as a brand-new round.
- **Automated cross-version content diffing or side-by-side visual comparison** -- The Key Capabilities line ("open and compare any round by number") and the Data Notes field describe comparison as opening any round through the version selector, with no diff mechanics, supported file types, or comparison UI described anywhere in Stage 2; a computed content-diff tool is out of scope for this Brief absent that elaboration.
- **Scoped upload or view permissions for a freelancer-side team** -- Excluded per scope-boundaries.md (SC-01): the product is solo-freelancer only for v1, so this feature has exactly one writer role (Nadia); no bookkeeper- or contractor-style scoped version permission exists.
- **Operator downloading a version's file, or editing/removing versions in a support session** -- Excluded per scope-boundaries.md (SC-04): Dana's support session is read-only and never downloads deliverable files or their versions, and never edits or removes anything.
- **Native mobile version-browsing app** -- Excluded per scope-boundaries.md (SC-06): the product is a web app with no native apps; version browsing and comparison happen entirely through the mobile-and-desktop-browser client experience.

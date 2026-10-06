---
document_type: feature-overview
feature_number: FEAT-24
feature_name: Data Export & Account Deletion
feature_slug: data-export-account-deletion
priority_tier: Important
feature_type: Lifecycle
produced_by: feature-analyst
status: final
created: 2026-09-27
spec_count: 9
screen_count: 2
automation_count: 3
logic_rule_count: 2
integration_count: 0
notification_count: 2
---

# Feature Breakdown Brief: Data Export & Account Deletion

## Summary

**Feature:** Data Export & Account Deletion
**ID:** FEAT-24
**Description:** The freelancer can export all of her own data and permanently delete her account and its data.
**Priority:** Important
**Phase:** MVP
**Type:** Lifecycle
**Rationale:** BRIEF.md, Constraints: "freelancers must be able to export and delete their data. Personal data of worldwide clients falls under GDPR." Ranked Important because it is a compliance and trust requirement rather than part of the daily value loop; phased MVP since this is a hard regulatory requirement that must exist from launch, not added later.

**Key Capabilities:**
- Full data export — a complete archive of clients, projects, proposals, invoices, and activity trail
- Account deletion — permanently remove the account and its data, with explicit confirmation

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-24.SPEC-001 | Data Export Screen | Screen | Nadia (Freelancer) | Nadia requests a full data export, tracks its progress, and downloads the completed archive |
| FEAT-24.SPEC-002 | Account Deletion Screen | Screen | Nadia (Freelancer) | Nadia reviews any open-item warnings and gives the explicit confirmation required to permanently delete her account |
| FEAT-24.SPEC-003 | Data Export Archive Generation | Automation | Nadia (Freelancer) | Aggregates every client, project, proposal, invoice, and activity record she owns into a downloadable archive, retries cleanly on failure, and expires the archive after its download window |
| FEAT-24.SPEC-004 | Account Deletion Processing | Automation | Nadia (Freelancer) | Cascades the permanent removal of the freelancer's account and all owned data across every data-holding feature once she confirms, disconnecting her payment account, and leaving the account fully intact if processing fails |
| FEAT-24.SPEC-005 | Legal Retention Purge | Automation | Nadia (Freelancer) | Purges the financial records held back from an otherwise-completed account deletion once their legal retention period lapses |
| FEAT-24.SPEC-006 | Pre-Deletion Warning & Retention Determination Rules | Logic/Rule | Nadia (Freelancer) | Determines which open items (unpaid invoices, pending approvals) trigger a specific warning without blocking deletion, and which records are legally retained versus immediately deleted |
| FEAT-24.SPEC-007 | Export & Deletion Access Rules | Logic/Rule | All | Restricts export and deletion to Nadia alone, with no access of any kind for client contacts or the Support Operator |
| FEAT-24.SPEC-008 | Export Ready Notification | Notification | Nadia (Freelancer) | Emails Nadia when her requested data export archive is ready to download |
| FEAT-24.SPEC-009 | Account Deletion Final Warning Notification | Notification | Nadia (Freelancer) | Emails Nadia a final confirmation notice before her account deletion becomes irreversibly complete |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Full data export — a complete archive of clients, projects, proposals, invoices, and activity trail | FEAT-24.SPEC-001, FEAT-24.SPEC-003, FEAT-24.SPEC-007, FEAT-24.SPEC-008 | The screen offers the request/status/download surface; the automation aggregates every owned record into the archive; the access rule confines the request to Nadia; the notification tells her when it is ready | Phase 2 (Explicit) |
| Account deletion — permanently remove the account and its data, with explicit confirmation | FEAT-24.SPEC-002, FEAT-24.SPEC-004, FEAT-24.SPEC-005, FEAT-24.SPEC-006, FEAT-24.SPEC-007, FEAT-24.SPEC-009 | The screen surfaces open-item warnings and captures explicit confirmation; the processing automation cascades the permanent removal; the retention automation purges what was held back once legal retention lapses; the rules spec decides what is warned versus retained; the access rule confines the request to Nadia; the notification delivers the final warning before the point of no return | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-24.SPEC-003 | Data Export Archive Generation | Phase 4 (Trigger-Response Analysis) | Aggregating every owned entity into a single archive file, retrying cleanly on failure, and expiring the archive after a limited download window is processing logic with real failure modes, not a direct data write -- past the standalone-Automation threshold. The archive's storage and delivery run through the large-file storage & delivery capability already owned by FEAT-16.SPEC-007 (External Dependencies lens), so no new Integration spec is created here |
| FEAT-24.SPEC-004 | Account Deletion Processing | Phase 4 (Trigger-Response Analysis) | Cascading removal reaches across every data-holding feature (FEAT-01 through FEAT-33) with defined cross-feature effects (disconnecting Payment Account Connection, purging stored files, purging the activity trail) -- the archetypal cross-feature-effect case for a standalone Automation |
| FEAT-24.SPEC-005 | Legal Retention Purge | Phase 4 (Time-based triggers lens) | The Validation & Limits field states deletion "honors any legal retention requirement for financial records before final purge" -- a distinct, later-firing trigger (retention-window elapse) independent of the deletion-confirmation trigger that drives SPEC-004, warranting its own Automation rather than folding a second trigger into SPEC-004 |
| FEAT-24.SPEC-006 | Pre-Deletion Warning & Retention Determination Rules | Phase 5 (Rule-Constraint Discovery) | Conditional logic with multiple interacting combinations (unpaid invoice present or not, pending approval present or not, record class subject to legal retention or not) that must produce defined behavior for every combination -- past the inline-validation threshold and shared between SPEC-002 and SPEC-004 |
| FEAT-24.SPEC-007 | Export & Deletion Access Rules | Phase 5 (Rule-Constraint Discovery) / grounded-roles | The Access field is atypical for this product: Dana (Support Operator), who has View access almost everywhere else, has no access at all here, and no client contact has any account-level surface to reach this feature from. This total exclusion, applying to every role and every spec in the feature, needed an explicit standalone rule rather than being implied by omission |
| FEAT-24.SPEC-008 | Export Ready Notification | Phase 4 (Notification surfacing lens) | The Communications field names "confirmation email when an export is ready to download" -- a message with a defined channel, audience, and trigger, which requires a standalone Notification spec rather than an inline toast. Delivery runs through the transactional email capability already owned by FEAT-14.SPEC-001, so no new Integration spec is created here |
| FEAT-24.SPEC-009 | Account Deletion Final Warning Notification | Phase 4 (Notification surfacing lens) | The Communications field names "a final confirmation email before deletion is irreversibly completed" -- a distinct message from the export-ready email, with its own trigger (explicit confirmation given, before the point of no return) and audience, requiring its own standalone Notification spec |

## Entity-Lifecycle Coverage Matrix

**Entity: Data Export Archive**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-24.SPEC-003 | Generated on request from every entity Nadia owns (clients, projects, proposals, invoices, activity trail, and the rest of the domain per the feature's Connected Entities field) | -- |
| Read (single) | FEAT-24.SPEC-001 | The Data Export Screen shows the current archive's status and, once ready, serves the Download action | -- |
| Read (list) | N/A | product-features.md's Domain Entity Inventory marks this entity "Managed by: N/A -- a point-in-time generated file"; there is at most one active archive at a time, so no list/history view applies | -- |
| Update | FEAT-24.SPEC-003 | Advances the archive through its own lifecycle states as generation and delivery progress | Same automation as Create; no separate edit path exists for a generated archive |
| Delete/Archive | FEAT-24.SPEC-003 | Hard delete: the archive and its underlying file are automatically purged once the limited download window elapses (retention/purge: expires and is purged, not retained further); no restore path -- a fresh request always regenerates a new archive from current data rather than recovering an expired one; no cascade, since nothing else in the product references a generated archive | -- |
| State Transition | FEAT-24.SPEC-003 | Requested -> Ready -> Downloaded -> Expired (product-features.md, Domain Entity Inventory, Lifecycle field) | A failed generation does not advance the state and is retried without leaving partial, corrupted output (States field) |

**Entity: Freelancer Account (this feature's Delete/Archive responsibility only)**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A | Owned by Onboarding & First-Run Setup (FEAT-20) at sign-up; not this feature's responsibility | -- |
| Read (single) | N/A | Owned by other features (FEAT-09, FEAT-14, FEAT-21, FEAT-23, FEAT-31, FEAT-33 per the dependency map); this feature reads the account and its owned records only transiently, to build the export archive and to execute deletion | See Referenced Entities below |
| Read (list) | N/A | A single freelancer's own account; no list applies | -- |
| Update | N/A | Owned by Settings & Account Management (FEAT-21); this feature never edits account fields | -- |
| Delete/Archive | FEAT-24.SPEC-004 | Hard delete of the Freelancer Account and, per XBR-33, a cascading hard delete across every entity it owns (see Referenced Entities below), plus disconnecting the Payment Account Connection; no restore path anywhere -- irreversible once the explicit confirmation is given (Validation & Limits field). The sole exception is financial records (Invoice, Payment) held under legal retention (scope-boundaries.md SC-24), which FEAT-24.SPEC-005 purges once that retention period lapses | Settings & Account Management (FEAT-21) explicitly routes "close account" here rather than deleting the account itself |
| State Transition | FEAT-24.SPEC-004 | Active -> Deletion Requested -> Deletion Confirmed -> Deleted, with a reversion to Active (no partial change) if processing fails partway (States field: "a failed deletion leaves the account fully intact rather than half-deleted") | -- |

**Referenced Entities (read for export content; hard-deleted or disconnected by the FEAT-24.SPEC-004 cascade, except where noted):**

| Entity | Read By | Context |
|--------|---------|---------|
| Client | FEAT-24.SPEC-003 (export) | Deleted by FEAT-24.SPEC-004's cascade (dependency map: "Deleted by ... FEAT-24 (account deletion)") |
| Client Contact | FEAT-24.SPEC-003 (export) | Deleted by FEAT-24.SPEC-004's cascade, including the contact's personal data (name, email), per GDPR-class handling (dependency map: "Deleted by FEAT-24") |
| Project | FEAT-24.SPEC-003 (export) | Deleted by FEAT-24.SPEC-004's cascade (dependency map: "Deleted by FEAT-24 only -- no in-product project delete") |
| Proposal | FEAT-24.SPEC-003 (export) | Deleted by FEAT-24.SPEC-004's cascade (dependency map: "Deleted by FEAT-02 (unsent drafts only) and FEAT-24") |
| Milestone | FEAT-24.SPEC-003 (export) | Deleted by FEAT-24.SPEC-004's cascade (dependency map: "Deleted by FEAT-04 ... and FEAT-24") |
| Payment Schedule | FEAT-24.SPEC-003 (export) | Deleted by FEAT-24.SPEC-004's cascade (dependency map: "Deleted by FEAT-24") |
| Deliverable | FEAT-24.SPEC-003 (export) | Deleted by FEAT-24.SPEC-004's cascade (dependency map: "Deleted by FEAT-24") |
| Deliverable Version | FEAT-24.SPEC-003 (export, as version history) | Deleted by FEAT-24.SPEC-004's cascade, which also purges the underlying stored bytes through FEAT-16.SPEC-006 (dependency map: "removed only by FEAT-24") |
| Comment | FEAT-24.SPEC-003 (export, as part of the activity trail context) | Deleted by FEAT-24.SPEC-004's cascade (dependency map: "Deleted by FEAT-24 only") |
| Invoice | FEAT-24.SPEC-003 (export) | Deleted by FEAT-24.SPEC-004's cascade **subject to legal retention** -- held back and purged later by FEAT-24.SPEC-005 (dependency map: "Deleted by FEAT-24 (subject to legal retention)") |
| Payment | FEAT-24.SPEC-003 (export) | Deleted by FEAT-24.SPEC-004's cascade **subject to legal retention** -- held back and purged later by FEAT-24.SPEC-005 (dependency map: "removed by FEAT-24 subject to legal retention") |
| Reminder Log | FEAT-24.SPEC-003 (export, as part of invoice history) | Deleted by FEAT-24.SPEC-004's cascade (dependency map: "Deleted by FEAT-24") |
| Activity Log Entry | FEAT-24.SPEC-003 (the activity trail export) | Deleted by FEAT-24.SPEC-004's cascade, executed through FEAT-13's own retention rule (FEAT-13.SPEC-006), subject to the same financial-record retention exception (dependency map: "removed only by FEAT-24 account deletion") |
| Branding Profile | Not exported (not client/project/proposal/invoice/activity data) | Deleted by FEAT-24.SPEC-004's cascade (dependency map: "Deleted by FEAT-24") |
| Subscription Plan | Not exported (billing relationship, not client data) | Deleted by FEAT-24.SPEC-004's cascade (dependency map: "Deleted by FEAT-24") |
| Custom Domain Record | Not exported (configuration, not client data) | Deleted by FEAT-24.SPEC-004's cascade (dependency map: "Deleted by FEAT-27 (removing the domain) and FEAT-24") |
| Notification | Not exported (delivery metadata, not client data) | Deleted by FEAT-24.SPEC-004's cascade (dependency map: "Deleted by FEAT-24") |
| Payment Account Connection | Not exported (financial-account linkage, not client data) | Disconnected by FEAT-24.SPEC-004's cascade via FEAT-32.SPEC-004 (dependency map: "Deleted by FEAT-32 (disconnect) and FEAT-24 (disconnect on account deletion)"; XBR-33) |
| Support Access Session | Not exported (operator record, not client data) | Deleted by FEAT-24.SPEC-004's cascade (dependency map: "Deleted by FEAT-24") |
| Referral Attribution | Not exported (growth-measurement record, not client data) | Deleted by FEAT-24.SPEC-004's cascade (dependency map: "Deleted by FEAT-24 with the account") |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Nadia requests a full data export | Confirm she is the requester and begin aggregating every owned record into an archive | Standalone Automation | SPEC-003 |
| Archive generation completes | Mark the archive Ready and offer the Download action | Inline in triggering screen (state display) | SPEC-001 |
| Archive becomes Ready | Send the export-ready confirmation email | Standalone Notification | SPEC-008 |
| Archive generation fails partway | Retry generation cleanly, producing no partial or corrupted output (States field) | Standalone Automation (failure handling) | SPEC-003 |
| The generated archive sits unfetched past its download window | Expire the archive and purge its underlying file | Standalone Automation (time-based) | SPEC-003 |
| Nadia opens the Account Deletion Screen | Evaluate open items (unpaid invoices, pending approvals) and which records would be legally retained | Standalone Logic/Rule | SPEC-006 |
| Open items are found | Warn specifically about the consequence, without blocking; Nadia decides how to proceed (product-features.md, Primary Flows & Alternates) | Inline in triggering screen (warning display), governed by SPEC-006 | SPEC-002 |
| Nadia gives explicit confirmation to delete | Send the final confirmation email before the deletion becomes irreversible | Standalone Notification | SPEC-009 |
| Confirmation is given | Begin cascading, permanent removal of the account and every owned entity | Standalone Automation | SPEC-004 |
| Deletion processing reaches the Payment Account Connection | Disconnect it | Cross-feature -- owned by Payment Account Connection (FEAT-32.SPEC-004) | FEAT-32 responsibility |
| Deletion processing reaches stored deliverable files and versions | Purge every stored byte from the storage capability | Cross-feature -- owned by Large File Handling & Storage (FEAT-16.SPEC-006) | FEAT-16 responsibility |
| Deletion processing reaches the activity trail | Remove entries under FEAT-13's own retention rule, subject to the financial-record retention exception | Cross-feature -- owned by Immutable Activity & Audit Trail (FEAT-13.SPEC-006) | FEAT-13 responsibility |
| Deletion processing fails partway | Leave the account fully intact rather than half-deleted (States field) and revert to Active | Standalone Automation (failure handling) | SPEC-004 |
| Deletion reaches an Invoice or Payment record | Hold it back from immediate deletion under legal financial-record retention (SC-24) rather than purging it | Standalone Logic/Rule | SPEC-006 |
| A retained financial record's legal retention period elapses | Purge the record | Standalone Automation (time-based) | SPEC-005 |
| Dana (Support Operator) attempts to reach export or deletion | Deny access entirely -- no view, unlike this product's other features | Standalone Logic/Rule | SPEC-007 |
| A client contact (Owen or Priya) attempts to reach export or deletion | Deny access entirely -- no account-level surface exists for a client contact | Standalone Logic/Rule | SPEC-007 |

## Shared Context

**Shared Entities:**
- Data Export Archive -- created, advanced through its lifecycle states, and expired by SPEC-003; its current status and download link are shown by SPEC-001. Fields (functional): requested-at, status (Requested, Ready, Downloaded, Expired), download link/window.
- Freelancer Account and everything it owns (Client, Client Contact, Project, Proposal, Milestone, Payment Schedule, Deliverable, Deliverable Version, Comment, Invoice, Payment, Reminder Log, Activity Log Entry, Branding Profile, Subscription Plan, Custom Domain Record, Notification, Payment Account Connection, Support Access Session, Referral Attribution) -- read in full by SPEC-003 to build the export archive; hard-deleted (or, for Payment Account Connection, disconnected) in full by SPEC-004's cascade, except the financial records SPEC-006 flags for legal retention and SPEC-005 later purges.

**Shared UI Patterns:**
- Two-step, linear settings flow -- SPEC-001 (export) and SPEC-002 (deletion) are the feature's only screens, matching the journey's own sequencing ("requests a full data export -> receives a complete archive -> separately requests account deletion"); Spec Writers should keep the same settings-surface visual language across both rather than treating them as unrelated screens.
- Explicit-confirmation pattern -- SPEC-002's confirmation step is deliberately heavier than a normal destructive-action confirm, per the Validation & Limits field's "requires explicit confirmation given its irreversibility"; it is paired with SPEC-009's final warning email rather than resolved by an in-screen dialog alone.

**Shared Validation:**
- SPEC-006 defines the open-item warning conditions and the retention-vs-immediate-delete determination; SPEC-002 and SPEC-004 both reference it rather than duplicating the logic.
- SPEC-007 defines the Nadia-only access gate; SPEC-001, SPEC-002, SPEC-003, and SPEC-004 all reference it rather than restating who may reach this feature.

## Internal Dependency Map

```
SPEC-001 (Data Export Screen) -> [Nadia requests a full export] -> SPEC-003 (Data Export Archive Generation)
SPEC-003 (Data Export Archive Generation) -> [archive ready] -> SPEC-001 (Data Export Screen shows the Download action)
SPEC-003 (Data Export Archive Generation) -> [archive ready] -> SPEC-008 (Export Ready Notification)
SPEC-003 (Data Export Archive Generation) -> [download window elapses unfetched] -> SPEC-003 (marks the archive Expired)
SPEC-001 (Data Export Screen) -> [Nadia separately chooses to close her account] -> SPEC-002 (Account Deletion Screen)
SPEC-002 (Account Deletion Screen) -> [screen loads] -> SPEC-006 (Pre-Deletion Warning & Retention Determination Rules) -> [open items found] -> SPEC-002 (shows the specific warning)
SPEC-002 (Account Deletion Screen) -> [Nadia gives explicit confirmation] -> SPEC-009 (Account Deletion Final Warning Notification)
SPEC-002 (Account Deletion Screen) -> [confirmation given] -> SPEC-004 (Account Deletion Processing)
SPEC-004 (Account Deletion Processing) -> [reaches a financial record] -> SPEC-006 (Pre-Deletion Warning & Retention Determination Rules decides retain-vs-delete)
SPEC-004 (Account Deletion Processing) -> [a retained record's legal retention period elapses] -> SPEC-005 (Legal Retention Purge)
SPEC-001 (Data Export Screen), SPEC-002 (Account Deletion Screen), SPEC-003 (Data Export Archive Generation), SPEC-004 (Account Deletion Processing) -> [every request] -> SPEC-007 (Export & Deletion Access Rules)
```

**Default Entry:** SPEC-001 (Data Export Screen) -- reached from the navigation connection out of Settings & Account Management (FEAT-21, "settings: close account"); the journey's export step precedes the deletion step, so the export screen is the feature's landing point and offers the path onward to SPEC-002.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-24.SPEC-001, FEAT-24.SPEC-002 | Inbound | FEAT-21 (Settings & Account Management) | Nadia navigates here from "settings: close account"; FEAT-21 itself owns no deletion flow of its own | Nadia chooses to close her account |
| FEAT-24.SPEC-003 | Outbound | FEAT-16 (Large File Handling & Storage) | The generated archive file is stored and delivered for download through the large-file storage & delivery capability (FEAT-16.SPEC-007) | Export requested |
| FEAT-24.SPEC-004 | Outbound | FEAT-16 (Large File Handling & Storage) | Account deletion purges every stored file and version's bytes (FEAT-16.SPEC-006) | Deletion confirmed and finalized |
| FEAT-24.SPEC-004 | Outbound | FEAT-32 (Payment Account Connection) | Account deletion disconnects the payment account as part of its warn-then-remove sequence (FEAT-32.SPEC-004; XBR-33) | Deletion confirmed and finalized |
| FEAT-24.SPEC-004 | Outbound | FEAT-13 (Immutable Activity & Audit Trail) | Account deletion removes Activity Log Entries under FEAT-13's own retention & purge rule (FEAT-13.SPEC-006), subject to the financial-record retention exception | Deletion confirmed and finalized |
| FEAT-24.SPEC-004 | Outbound | FEAT-01 through FEAT-33 (every data-holding feature) | Cascading hard-delete removes every record these features created, per XBR-33 and the dependency map's per-entity "Deleted by FEAT-24" notes | Deletion confirmed and finalized |
| FEAT-24.SPEC-008, FEAT-24.SPEC-009 | Outbound | FEAT-14 (Notifications (Email)) | Export-ready and deletion final-warning emails are dispatched and tracked through the shared notification/delivery engine (FEAT-14.SPEC-001, FEAT-14.SPEC-002) | Archive ready / explicit deletion confirmation given |
| FEAT-24.SPEC-007 | Inbound | FEAT-31 (Operator Support Access) | XBR-29 excludes data/accounting exports and account-lifecycle actions from every support session; this feature's access rule enforces that exclusion on its own side by denying Dana entirely | Dana attempts any access during a support session |

## Non-Functional Notes

**Data volumes / growth:** A freelancer account holds 3-15 active clients' worth of records plus deliverables that are typically tens of MB and sometimes over 1 GB, retained for the life of the account (assumptions-constraints.md, ASMP-22); the export archive must aggregate this full history in one pass, and the deletion cascade must remove it in full, so both operations are sized to whatever a substantial, multi-year account has accumulated. This feature emits `data_export_requested`, `data_export_ready`, `account_deletion_requested`, and `account_deletion_completed` signals (product-features.md, Signals field), fired respectively by SPEC-003 (requested, ready) and SPEC-004 (requested, completed).

**Responsiveness:** Both export generation and deletion processing show real progress for accounts with substantial history rather than a silent wait (product-features.md, States field; assumptions-constraints.md, ASMP-27); a failed export is retried without partial, corrupted output, and a failed deletion leaves the account fully intact rather than half-deleted (product-features.md, States field).

**Data sensitivity / privacy:** This feature reads and then removes the full breadth of the freelancer's data, including a client contact's own personal data (email, comments), which is included in the export and removed on deletion per GDPR-class handling (product-features.md, Access field; assumptions-constraints.md, ASMP-24). It is the single feature in the product with the broadest data-sensitivity surface, since it touches every entity in the domain rather than one feature's slice of it.

**Compliance flags:** GDPR-class handling governs both the export and the deletion (assumptions-constraints.md, ASMP-24; ASMP-23: "freelancers can export and delete their own data on request"). Deletion honors any legal retention requirement for financial records before final purge (product-features.md, Validation & Limits field; scope-boundaries.md, SC-24). A removed client contact's evidentiary records (acceptances, approvals) stay on the record under their name even after this feature erases the account holding them, per ASMP-20's stated balance between record immutability and the right to erasure -- ASMP-20 also notes this balance's legal basis is to be confirmed with qualified privacy advice before launch, which this feature inherits rather than resolves.

## Non-Goals

- **Partial or selective export (exporting only some data types)** -- Excluded per product-features.md's Key Capabilities, which name "Full data export -- a complete archive of clients, projects, proposals, invoices, and activity trail"; the product commits to whole-account export only, with no per-category export controls.
- **Account deactivation or temporary suspension as an alternative to deletion** -- Excluded per product-features.md's Key Capabilities, which name exactly two lifecycle actions (export, permanent deletion); no paused or suspended account state exists anywhere in the Domain Entity Inventory.
- **Export or deletion initiated by any client contact or by the Support Operator** -- Excluded per product-features.md's Access field ("Nadia only ... no client contact can export or delete the freelancer's account ... Dana ... has no access") and scope-boundaries.md SC-04 (the operator's access is read-only and "never edits, sends, approves, pays, downloads deliverable files, or acts as a client contact"); the Access Matrix gives Owen and Priya "None" on Subscription & Account Data.
- **Retention of data beyond the legal financial-record requirement** -- Excluded per BRIEF.md's Constraints and scope-boundaries.md SC-24: once the legal retention period for financial records lapses, SPEC-005 purges them; nothing in this feature is kept "just in case" beyond what SC-24 requires.
- **Restoring a deleted account, or any undo after explicit confirmation** -- Excluded per product-features.md's Validation & Limits field ("Account deletion requires explicit confirmation given its irreversibility"); unlike this product's soft-delete/archive/restore pattern used elsewhere (e.g., Client, Contact), account deletion is designed to be terminal with no restore path once confirmed.

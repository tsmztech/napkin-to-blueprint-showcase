# FEAT-24 — Data Export & Account Deletion

This chapter covers Data Export & Account Deletion, a Important-tier feature. It contains the feature breakdown brief followed by every specification in full: 9 specifications carrying 126 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-24.SPEC-001 | Data Export Screen | screen | 15 |
| FEAT-24.SPEC-002 | Account Deletion Screen | screen | 19 |
| FEAT-24.SPEC-003 | Data Export Archive Generation | automation | 13 |
| FEAT-24.SPEC-004 | Account Deletion Processing | automation | 18 |
| FEAT-24.SPEC-005 | Legal Retention Purge | automation | 10 |
| FEAT-24.SPEC-006 | Pre-Deletion Warning & Retention Determination Rules | logic-rule | 18 |
| FEAT-24.SPEC-007 | Export & Deletion Access Rules | logic-rule | 12 |
| FEAT-24.SPEC-008 | Export Ready Notification | notification | 11 |
| FEAT-24.SPEC-009 | Account Deletion Final Warning Notification | notification | 10 |

The feature breakdown brief follows, then every specification in full.


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



# Screen Spec: Data Export Screen

## Overview

**Name:** Data Export Screen
**ID:** FEAT-24.SPEC-001
**Type:** Screen
**Purpose:** Nadia requests a full export of all her own data, tracks the archive's progress, and downloads it once ready.
**Parent Feature:** FEAT-24 -- Data Export & Account Deletion

## Scope and Non-Goals

**In Scope:**
- Requesting a full data export and showing its current status (Requested, Ready, Downloaded, Expired)
- Downloading the completed archive
- The path onward to closing the account (navigation to the Account Deletion Screen)

**Non-Goals:**
- Choosing which data types to include -- excluded per product-features.md's Key Capabilities, which name only a whole-account export; no per-category selection control exists on this screen.
- Aggregating the archive's contents or handling generation failure/retry -- owned by FEAT-24.SPEC-003 (Data Export Archive Generation); this screen only reflects that automation's state.
- Deciding who may reach this screen -- owned by FEAT-24.SPEC-007 (Export & Deletion Access Rules); this screen's Access and Visibility table below is consistent with, but does not redefine, that spec.
- Account deletion itself -- owned entirely by FEAT-24.SPEC-002 (Account Deletion Screen); this screen only offers the path there.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-21.SPEC-001 (Account Profile) | Nadia taps "Close account" in the Settings navigation shell | None -- this is the feature's default entry point; the screen loads the current (or absent) Data Export Archive state for her account |
| FEAT-24.SPEC-008 (Export Ready Notification) | Nadia taps the email's "Download your export" CTA | None -- the screen loads the current Data Export Archive state for her account, showing the ready archive |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Request export, download archive, navigate to Account Deletion Screen | -- |
| Owen (Client Primary Contact) | No | No | No navigation path in the client portal reaches this screen; a direct link shows the same out-of-scope explanation used elsewhere in the portal (XBR-09), never this screen's content |
| Priya (Client Reviewer Contact) | No | No | Same as Owen -- no client-portal surface exists for this feature (feature-overview.md, Non-Goals) |
| Dana (Support Operator) | No | No | This screen is excluded entirely from every read-only support session (XBR-29; FEAT-24.SPEC-007); a direct link during a session shows "This isn't available during a support session," not a read-only view |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in, the user lands on FEAT-12 (Freelancer Financial Dashboard), not this screen |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." No in-progress request is lost, since this screen holds no unsaved input -- after re-authentication the current archive status is reloaded exactly as it stood |

## Layout and Content

**Header:** Screen title "Export Your Data" with a back arrow (returns to FEAT-21.SPEC-001, Account Profile). To the left of the title, the shared Settings navigation shell (also used by FEAT-21.SPEC-001 through SPEC-004 and FEAT-19.SPEC-001) lists "Profile," "Notification Preferences," "Login & Security," "Business Details & Payment Terms," "Branding," and "Close account" at the bottom, visually separated; "Close account" is shown selected/active since this screen is what it routes to.

**Body:** A single-column content area below the header:
- Explanatory text: "Download a complete copy of your clients, projects, proposals, invoices, and activity trail."
- A status card showing the current archive's state (see States below): its requested date, current status, and, once Ready or Downloaded, a download action and the date the archive becomes unavailable.
- A "Request Export" action (button), whose label and enabled state depend on the current archive state.
- Below the status card, a visually separated section: "Want to close your account instead?" with a "Continue to close your account" link.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described above, full width; the Settings navigation shell collapses to a top row of tabs, consistent with FEAT-21.SPEC-001's own responsive behavior.
- **Medium size class and above:** The Settings navigation shell sits to the left of the main content, consistent with FEAT-21.SPEC-001; the status card and actions remain single-column, capped at a consistent platform-wide content width and horizontally centered within the content area.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-21.SPEC-001 (Account Profile) | Screen closes | Standard navigation transition |
| "Request Export" button | Tap (Empty, Expired, Ready, or Downloaded state) | Triggers FEAT-24.SPEC-003 (Data Export Archive Generation) | Status card enters Requested/Generating | Button shows loading state; status card text updates to "Preparing your export -- this can take a few minutes for accounts with a lot of history." |
| "Request Export" button (while Requested/Generating) | Tap | No action -- debounced | None | Button remains disabled/loading |
| "Download" action (Ready or Downloaded state only) | Tap | Initiates delivery of the archive file via FEAT-16.SPEC-007 (Large-File Storage & Delivery Capability) | Archive transitions from Ready to Downloaded on first successful download (stays downloadable until it expires) | Standard file-download behavior begins; status card shows "Downloaded on {date}" once the transfer completes |
| "Continue to close your account" link | Tap | Navigate to FEAT-24.SPEC-002 (Account Deletion Screen) | Screen leaves Data Export Screen | Standard navigation transition |

### Accessibility Notes

- **Focus order:** Back arrow -> Settings navigation shell items -> explanatory text -> status card -> Request Export button -> Download action (when present) -> "Continue to close your account" link.
- **Status announcements:** When the status card's state changes (e.g., Requested/Generating -> Ready, or -> Error), the new status text is announced to assistive technology.
- **Download feedback:** The "Downloaded on {date}" update is announced once a download completes.
- **Keyboard alternatives:** Every action on this screen (Request Export, Download, Continue to close account, back arrow) is a standard activatable control reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (no export requested) | Status card reads "You haven't requested an export yet." Request Export button enabled | Screen first opens with no prior or active archive for this account | Nadia taps Request Export |
| Requested/Generating | Status card shows a progress indicator and "Preparing your export -- this can take a few minutes for accounts with a lot of history." Request Export button disabled | FEAT-24.SPEC-003 begins processing the request | FEAT-24.SPEC-003 reports Ready or, after all retries, Error |
| Ready | Status card shows "Your export is ready." with a Download action and "Available until {expiry_date}." Request Export button re-enabled, labeled "Request New Export" | FEAT-24.SPEC-003 reports the archive Ready | Nadia downloads (-> Downloaded), the window elapses unfetched (-> Expired), or she requests a new export (replaces this archive) |
| Downloaded | Status card shows "Downloaded on {date}." with the Download action still available and "Available until {expiry_date}." Request Export button re-enabled, labeled "Request New Export" | Nadia completes a download while the archive is Ready | The window elapses (-> Expired) or she requests a new export |
| Expired | Status card shows "Your last export has expired. Request a new one to download your data again." Request Export button enabled, labeled "Request Export" | The download window elapses without a fresh request superseding it (FEAT-24.SPEC-003) | Nadia requests a new export |
| Error | Error banner: "We couldn't generate your export. Try again." Request Export button re-enabled | FEAT-24.SPEC-003 reports generation failure after exhausting its retries | Nadia requests a new export |
| Offline/Degraded | Banner: "You're offline. Reconnect to request or download your export." Request Export and Download actions disabled; the last-known status card content remains visible | Connectivity lost while this screen is open | Connectivity restored -- the current archive status is re-fetched and the screen returns to the state matching it |

## Validation Rules

N/A -- this screen has no user-entered fields. Its only actions (Request Export, Download, navigate onward) require no input validation; authorization for reaching the screen at all is governed by FEAT-24.SPEC-007 (Export & Deletion Access Rules).

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-21.SPEC-001 (Account Profile) | FEAT-21 (Settings & Account Management) |
| "Continue to close your account" link | FEAT-24.SPEC-002 (Account Deletion Screen) | -- |

## Data Model

**Creates:** None directly -- Request Export triggers FEAT-24.SPEC-003, which creates the Data Export Archive record.
**Reads:** Data Export Archive -- status, requested-at, download link/window, displayed in the status card.
**Updates:** None directly -- FEAT-24.SPEC-003 advances the archive's lifecycle states; this screen only reflects them.
**Deletes:** None.

## Business Rules

- At most one active archive exists at a time (feature-overview.md, Entity-Lifecycle Coverage Matrix); requesting a new export while one is Ready, Downloaded, or Expired supersedes it, per FEAT-24.SPEC-003.
- Full data export only -- no per-category selection exists on this screen (product-features.md, Key Capabilities).
- FEAT-24.SPEC-007 confines this entire screen, and every action on it, to Nadia and her own account.

## Edge Cases

- **Nadia taps Request Export twice in quick succession** -- The second tap is ignored while the button is disabled during Requested/Generating (debounced).
- **Network failure while initiating a request** -- Error banner: "Could not start your export. Check your connection and try again." No archive record is created; the screen returns to its prior state (Empty, Expired, Ready, or Downloaded).
- **Nadia has this screen open in two browser tabs and requests an export in one** -- This screen is a snapshot, not live-updating; the second tab continues to show its last-loaded state until Nadia reloads or re-navigates to it, at which point it reflects the newly requested archive. No conflict dialog is shown, since the underlying Data Export Archive is a single-writer, non-shared entity (feature-dependency-map.md notes it as a single-feature entity with no Contention entry) and FEAT-24.SPEC-003 resolves the two requests deterministically (the later request supersedes).
- **The archive expires while this screen is open and idle** -- The status card does not update in real time; it shows Expired on the next load or manual refresh.
- **Nadia navigates away mid-generation and returns later** -- The screen re-fetches and displays whatever state FEAT-24.SPEC-003 has reached (Generating, Ready, or Error); nothing is lost, since generation continues independently of whether this screen is open.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-24.SPEC-003 (Data Export Archive Generation) | Triggers (outbound) | Request Export starts this automation; its outcomes drive every state on this screen |
| FEAT-24.SPEC-007 (Export & Deletion Access Rules) | References (inbound) | Governs who can reach and act on this screen |
| FEAT-24.SPEC-008 (Export Ready Notification) | References (inbound) | This notification's CTA deep-links back to this screen |
| FEAT-21.SPEC-001 (Account Profile) | Navigation (inbound) | "Close account" navigation item routes here as the feature's default entry |
| FEAT-24.SPEC-002 (Account Deletion Screen) | Navigation (outbound) | "Continue to close your account" link |
| FEAT-16.SPEC-007 (Large-File Storage & Delivery Capability) | References (outbound) | Underlying delivery capability the Download action uses |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| data_export_requested | entry state (empty / expired / ready / downloaded) | Nadia taps Request Export | N/A -- no success-metrics.md metric is connected to Data Export & Account Deletion; this event is retained because product-features.md's Signals field names it explicitly as this feature's defined signal, kept observable even without a Stage 2 metric consuming it |
| data_export_download_initiated | archive age (days since requested) | Nadia taps Download | N/A -- same reason as above; retained for observability of whether Nadia actually retrieves what she requested |

## Acceptance Criteria

**FEAT-24.SPEC-001-AC-01:** Given Nadia is on FEAT-21.SPEC-001 (Account Profile), when she taps "Close account," then she lands on this screen showing her account's current export status.

**FEAT-24.SPEC-001-AC-02:** Given Nadia is on the Data Export Screen with no prior export, when she taps "Request Export," then FEAT-24.SPEC-003 begins and the status card shows "Preparing your export -- this can take a few minutes for accounts with a lot of history."

**FEAT-24.SPEC-001-AC-03:** Given Nadia's archive is in the Requested/Generating state, when she looks at the Request Export button, then it is disabled and a second tap has no effect.

**FEAT-24.SPEC-001-AC-04:** Given FEAT-24.SPEC-003 reports the archive Ready, when Nadia views this screen, then the status card shows "Your export is ready." with a Download action and "Available until {expiry_date}."

**FEAT-24.SPEC-001-AC-05:** Given Nadia's archive is Ready, when she taps Download, then the file transfer begins through FEAT-16.SPEC-007 and, once it completes, the status card shows "Downloaded on {date}."

**FEAT-24.SPEC-001-AC-06:** Given Nadia's archive has passed its download window unfetched, when she views this screen, then the status card shows "Your last export has expired. Request a new one to download your data again." and the Download action is no longer shown.

**FEAT-24.SPEC-001-AC-07:** Given FEAT-24.SPEC-003 reports generation failure after exhausting its retries, when Nadia views this screen, then she sees the error banner "We couldn't generate your export. Try again." and the Request Export button is enabled.

**FEAT-24.SPEC-001-AC-08:** Given Nadia is on the Data Export Screen, when she taps "Continue to close your account," then she is navigated to FEAT-24.SPEC-002 (Account Deletion Screen).

**FEAT-24.SPEC-001-AC-09:** Given Nadia loses connectivity while this screen is open, when the connection drops, then the banner "You're offline. Reconnect to request or download your export." appears and both Request Export and Download are disabled.

**FEAT-24.SPEC-001-AC-10:** Given Owen (Client Primary Contact) has no navigation path to this screen, when he follows any link in the client portal, then he never reaches it.

**FEAT-24.SPEC-001-AC-11:** Given Dana (Support Operator) is in an active read-only support session, when she attempts to open this screen directly, then she sees "This isn't available during a support session," not a read-only view of Nadia's export.

**FEAT-24.SPEC-001-AC-12:** Given an unauthenticated visitor opens this screen's link, then they are redirected to sign-in and, after signing in, land on FEAT-12 (Freelancer Financial Dashboard), not this screen.

**FEAT-24.SPEC-001-AC-13:** Given Nadia's session expires while this screen is open, when she is shown the expired-session dialog and signs back in, then the screen reloads showing the current archive status exactly as it stood before expiry.

**FEAT-24.SPEC-001-AC-14:** Given Nadia's archive is already Ready, when she taps "Request New Export," then a fresh request supersedes the current archive per FEAT-24.SPEC-003, and the status card returns to Requested/Generating.

**FEAT-24.SPEC-001-AC-15:** Given Nadia has this screen open in two tabs and requests an export in one, when she switches to the other tab without reloading it, then that tab still shows its prior state until she reloads or re-navigates to it.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 7 (empty, requested/generating, ready, downloaded, expired, error, offline) | 7 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



# Screen Spec: Account Deletion Screen

## Overview

**Name:** Account Deletion Screen
**ID:** FEAT-24.SPEC-002
**Type:** Screen
**Purpose:** Nadia reviews any open-item warnings and gives the explicit confirmation required to permanently delete her account and all its data.
**Parent Feature:** FEAT-24 -- Data Export & Account Deletion

## Scope and Non-Goals

**In Scope:**
- Displaying open-item warnings (unpaid invoices, pending approvals) evaluated by FEAT-24.SPEC-006
- Capturing Nadia's explicit, heavier-than-normal confirmation of permanent deletion
- Showing deletion progress and the outcome (success or a reverted, intact account on failure)

**Non-Goals:**
- Determining which open items warrant a warning or which records are legally retained -- owned entirely by FEAT-24.SPEC-006 (Pre-Deletion Warning & Retention Determination Rules); this screen only displays that determination's output.
- Executing the cascading deletion itself -- owned by FEAT-24.SPEC-004 (Account Deletion Processing); this screen only captures confirmation and shows that automation's progress and outcome.
- Exporting data before deletion -- handled separately by FEAT-24.SPEC-001 (Data Export Screen); this screen does not repeat or require that step, consistent with the Brief's Primary Flows describing export and deletion as separate, independently initiated actions.
- Offering any undo or restore path after confirmation -- excluded per product-features.md's Validation & Limits field ("Account deletion requires explicit confirmation given its irreversibility"); no cancel action exists once the cascade begins.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-24.SPEC-001 (Data Export Screen) | Nadia taps "Continue to close your account" | None -- this screen loads and evaluates open items and retention determinations fresh via FEAT-24.SPEC-006 |
| FEAT-21.SPEC-001 (Account Profile) | Nadia taps "Close account" in the Settings navigation shell (alternate direct path, since the shell's link routes to FEAT-24 generally and the feature's default entry is FEAT-24.SPEC-001; this row documents that this screen is never itself the shell's direct destination) | N/A -- see Non-Goals: the shell always lands on FEAT-24.SPEC-001 first per the Brief's Default Entry |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Review warnings, confirm, and initiate permanent deletion | -- |
| Owen (Client Primary Contact) | No | No | No navigation path in the client portal reaches this screen; a direct link shows the same out-of-scope explanation used elsewhere (XBR-09) |
| Priya (Client Reviewer Contact) | No | No | Same as Owen -- no client-portal surface exists for this feature |
| Dana (Support Operator) | No | No | Excluded entirely from every read-only support session (XBR-29; FEAT-24.SPEC-007); a direct link during a session shows "This isn't available during a support session" |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in, the user lands on FEAT-12 (Freelancer Financial Dashboard), not this screen |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." Any in-progress confirmation state (the acknowledgment checkbox and typed confirmation text) is discarded rather than preserved -- resuming a partially confirmed deletion after re-authentication is treated as starting over, since this is a security-sensitive, irreversible action |

## Layout and Content

**Header:** Screen title "Delete Your Account" with a back arrow (returns to FEAT-24.SPEC-001, Data Export Screen). The shared Settings navigation shell is not shown on this screen -- unlike the other Settings-area screens, this one is presented as a focused, full-attention flow rather than one item among a navigable list, consistent with the Shared UI Pattern's "deliberately heavier" confirmation step.

**Body:** A single-column content area:
- Warning region (only shown when FEAT-24.SPEC-006 finds open items): up to two warning banners, the unpaid-invoice banner first and the pending-approval banner second, each rendered with the exact title and body text defined in FEAT-24.SPEC-006 (Business Rules, "Exact warning and notice text") with its count and amount placeholders filled, and neither blocking progress.
- Changed-warnings notice (only shown in the Loaded -- warnings changed state): a line directly above the warning region reading "Your open items changed. Review the updated warnings, then tap Delete My Account again."
- Retention notice: always shown, rendered with the exact text defined in FEAT-24.SPEC-006 (Business Rules, "Exact warning and notice text"), with the retention period filled from platform parameter: `financial-record-legal-retention-period`.
- Confirmation region, positioned below the warnings and retention notice:
  - An acknowledgment checkbox: "I understand this permanently deletes my account and all its data, and this cannot be undone."
  - A confirmation text input, labeled "Type DELETE to confirm."
  - A "Delete My Account" button, disabled until both the checkbox is checked and the typed text exactly matches "DELETE."
  - A "Cancel" button/link, always enabled, returning to FEAT-24.SPEC-001.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described above, full width; warning banners stack vertically; the Delete My Account and Cancel buttons stack full-width, Delete My Account above Cancel.
- **Medium size class and above:** Content is capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping. Delete My Account and Cancel sit side by side, Cancel to the left.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-24.SPEC-001 (Data Export Screen) | Screen closes | Standard navigation transition |
| Acknowledgment checkbox | Tap | Toggles acknowledgment | Delete My Account re-evaluates its enabled condition | Checkbox shows checked/unchecked state |
| Confirmation text input | Type | Captures typed text | Delete My Account re-evaluates its enabled condition on each keystroke | Standard input focus and text-entry state |
| "Delete My Account" button | Tap (enabled only when checkbox is checked AND input exactly equals "DELETE") | 1. Re-runs FEAT-24.SPEC-006 and compares the result with the warnings currently displayed (open_unpaid_invoice_present, open_invoice_count, open_invoice_totals, pending_approval_present, pending_approval_count). 2a. If identical: triggers FEAT-24.SPEC-009 (Account Deletion Final Warning Notification), then FEAT-24.SPEC-004 (Account Deletion Processing). 2b. If any value differs: the tap is rejected -- neither FEAT-24.SPEC-009 nor FEAT-24.SPEC-004 is triggered | 2a: Screen enters Processing state. 2b: Screen enters Loaded -- warnings changed state | 2a: Button shows loading state; screen shows "Deleting your account -- this can take a few minutes for accounts with a lot of history." 2b: Warning region shows the current banners (or none), the changed-warnings notice appears, and focus moves to the notice; the checkbox and typed text are kept |
| "Cancel" button/link | Tap | Navigate to FEAT-24.SPEC-001 (Data Export Screen); no deletion action taken | Screen closes | Standard navigation transition |

### Accessibility Notes

- **Focus order:** Back arrow -> warning banners (if present) -> retention notice -> acknowledgment checkbox -> confirmation text input -> Delete My Account -> Cancel.
- **Changed-warnings announcement:** The changed-warnings notice and the refreshed banner text are announced when the tap is rejected, and focus moves to the notice.
- **Warning announcements:** Each warning banner's text is announced to assistive technology when the screen loads with open items present.
- **Confirmation-state announcements:** The Delete My Account button's enabled/disabled transition is announced when the checkbox and typed text together satisfy or stop satisfying the confirmation condition.
- **Processing announcements:** The "Deleting your account..." progress message is announced when processing begins.
- **Keyboard alternatives:** Every action (checkbox, text input, Delete My Account, Cancel, back arrow) is a standard keyboard-operable control; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | A loading indicator is shown in place of the warning region and confirmation controls while FEAT-24.SPEC-006 evaluates open items and retention determinations | Screen first opens | Evaluation completes |
| Loaded -- no warnings | No warning banners; retention notice and confirmation region shown directly | FEAT-24.SPEC-006 finds no open items | Nadia confirms or cancels |
| Loaded -- with warnings | One or more warning banners shown above the retention notice and confirmation region | FEAT-24.SPEC-006 finds one or more open items | Nadia confirms or cancels |
| Confirming | Delete My Account button enabled | Checkbox checked AND typed text exactly "DELETE" | Nadia taps Delete My Account (re-check identical: Processing; re-check differs: Loaded -- warnings changed), or un-checks/edits the text (returns to Loaded) |
| Loaded -- warnings changed | The current warning banners (or none, if every open item has since resolved) are shown, with the changed-warnings notice above them. Checkbox stays checked, typed text is kept, and Delete My Account stays enabled; nothing has been sent to FEAT-24.SPEC-009 or FEAT-24.SPEC-004 | Delete My Account tapped and the FEAT-24.SPEC-006 re-check differs from the warnings displayed | Nadia taps Delete My Account again (re-check against the now-displayed warnings: Processing if identical, this state again if they changed once more), un-checks/edits the text (returns to Loaded), or cancels |
| Processing | Progress indicator and "Deleting your account -- this can take a few minutes for accounts with a lot of history." All controls disabled | Delete My Account tapped and the re-check is identical to the displayed warnings | FEAT-24.SPEC-004 reports completion (its commit report, after which Nadia is signed out) or a pre-commit failure |
| Error (deletion failed) | Error banner: "We couldn't complete account deletion. Your account has not been changed. Try again." Confirmation region re-enabled, checkbox and text cleared | FEAT-24.SPEC-004 reports failure and reverts the account to Active | Nadia re-confirms |
| Offline/Degraded | Banner: "You're offline. Reconnect to continue." Confirmation region disabled; any evaluation or processing already in flight is unaffected server-side | Connectivity lost while this screen is open | Connectivity restored -- the screen re-fetches current state |

## Validation Rules

**Option B -- Inline (simple validation not warranting a standalone spec):**

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| Confirmation text input | Must exactly match "DELETE" (case-sensitive) | On change (continuously re-evaluated) | No error message shown -- the Delete My Account button simply stays disabled until the text matches exactly |

Open-item warning content (exact banner text and placeholders), the retention-notice text, and retention classification are governed by FEAT-24.SPEC-006 (Pre-Deletion Warning & Retention Determination Rules); this screen only displays that spec's output verbatim.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-24.SPEC-001 (Data Export Screen) | -- |
| Cancel button/link | FEAT-24.SPEC-001 (Data Export Screen) | -- |
| Deletion completes | None -- Nadia is signed out; there is no screen to return to since her account no longer exists | -- |

## Data Model

**Creates:** None directly -- Delete My Account triggers FEAT-24.SPEC-009 and FEAT-24.SPEC-004, which perform the actual notification and deletion.
**Reads:** Freelancer Account and its open-item indicators (unpaid Invoice status, pending-approval Milestone status) and retention classification, all via FEAT-24.SPEC-006.
**Updates:** None directly.
**Deletes:** None directly -- the cascade is entirely FEAT-24.SPEC-004's responsibility.

## Business Rules

- Deletion requires explicit confirmation -- the acknowledgment checkbox AND an exact-match typed "DELETE" -- before the Delete My Account button is enabled (product-features.md, Validation & Limits field).
- Open items warn without blocking: FEAT-24.SPEC-006's warnings are informational only and never disable the confirmation controls (product-features.md, Primary Flows & Alternates).
- The re-check at tap time gates everything: FEAT-24.SPEC-009 and FEAT-24.SPEC-004 fire only after the re-check confirms the warnings Nadia is looking at are current. A changed picture rejects the tap and requires a fresh tap.
- FEAT-24.SPEC-009 (final warning email) fires at the moment of confirmation (an accepted tap), not after FEAT-24.SPEC-004 completes -- it is Nadia's own record of exactly when she confirmed, independent of how long processing takes.
- A failed deletion (FEAT-24.SPEC-004 outcome) always leaves the account fully intact and re-enables the confirmation controls -- there is no partially deleted state ever shown (product-features.md, States field).
- FEAT-24.SPEC-007 confines this entire screen, and every action on it, to Nadia and her own account.

## Edge Cases

- **Open items change between screen load and confirmation (e.g., Nadia sends a new invoice in another tab, then returns here and confirms)** -- The confirmation re-checks open items and retention determinations at the moment Delete My Account is tapped, per FEAT-24.SPEC-006; if any compared value differs from what is displayed (including a warning that has disappeared), the tap is rejected: the screen moves to Loaded -- warnings changed, shows the refreshed banners and the changed-warnings notice, and fires neither FEAT-24.SPEC-009 nor FEAT-24.SPEC-004. Nadia must tap Delete My Account again to proceed, and that second tap is compared against the now-displayed warnings. Resolution: reject-with-refresh -- the checkbox and typed text are kept, so only the fresh tap is needed. A re-check that cannot complete (offline or evaluation error) also rejects the tap and shows the Offline/Degraded banner or "We couldn't check your open items. Try again."; nothing is triggered.
- **Nadia navigates away while Processing is underway** -- The deletion cascade (FEAT-24.SPEC-004) continues server-side regardless of whether this screen remains open; if she returns to any product screen before it completes, she is signed out once it finishes.
- **Nadia double-taps Delete My Account** -- The second tap is ignored while the button is in its loading/disabled state during Processing.
- **The typed confirmation text is pasted rather than typed** -- Treated identically to typed input; the exact-match check applies the same way.
- **Nadia edits the confirmation text after Delete My Account is already Processing** -- Not possible; all controls are disabled during Processing.
- **A retained financial record's classification changes between the warning display and the actual cascade (e.g., an invoice is paid in the moments after the warning was shown)** -- FEAT-24.SPEC-004 re-evaluates retention classification at cascade execution time via FEAT-24.SPEC-006, independent of what this screen displayed at load; the displayed warning reflects the state at evaluation time and is not guaranteed to be identical to the state at the exact moment the cascade runs, since a warning is advisory, not a lock.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-24.SPEC-006 (Pre-Deletion Warning & Retention Determination Rules) | References (inbound) | Supplies the open-item warning content and retention classification this screen displays |
| FEAT-24.SPEC-009 (Account Deletion Final Warning Notification) | Triggers (outbound) | Confirmation fires this notification |
| FEAT-24.SPEC-004 (Account Deletion Processing) | Triggers (outbound) | Confirmation starts the cascading deletion; this screen shows its progress and outcome |
| FEAT-24.SPEC-007 (Export & Deletion Access Rules) | References (inbound) | Governs who can reach and act on this screen |
| FEAT-24.SPEC-001 (Data Export Screen) | Navigation (inbound / outbound) | Entry point via "Continue to close your account"; Cancel and the back arrow return here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| account_deletion_requested | open_items_present (yes/no) | Nadia opens this screen and FEAT-24.SPEC-006's evaluation completes | N/A -- no success-metrics.md metric is connected to Data Export & Account Deletion; retained because product-features.md's Signals field names this event explicitly |
| account_deletion_confirmed | open_items_present (yes/no) | Nadia taps Delete My Account with a valid confirmation | N/A -- same reason as above; retained for observability of the confirmation moment distinct from the eventual completion signal emitted by FEAT-24.SPEC-004 |

## Acceptance Criteria

**FEAT-24.SPEC-002-AC-01:** Given Nadia is on FEAT-24.SPEC-001 and taps "Continue to close your account," when this screen loads, then it evaluates open items and retention determinations via FEAT-24.SPEC-006 before showing any warning or confirmation content.

**FEAT-24.SPEC-002-AC-02:** Given Nadia has an unpaid invoice, when this screen finishes loading, then a warning banner describes that specific consequence without disabling the confirmation controls.

**FEAT-24.SPEC-002-AC-03:** Given Nadia has no open items, when this screen finishes loading, then no warning banners appear and only the retention notice and confirmation region are shown.

**FEAT-24.SPEC-002-AC-04:** Given Nadia checks the acknowledgment checkbox but leaves the confirmation text empty, when she looks at "Delete My Account," then it remains disabled.

**FEAT-24.SPEC-002-AC-05:** Given Nadia checks the acknowledgment checkbox and types "delete" (lowercase), when she looks at "Delete My Account," then it remains disabled, since the match is case-sensitive.

**FEAT-24.SPEC-002-AC-06:** Given Nadia checks the acknowledgment checkbox and types "DELETE" exactly, when she looks at "Delete My Account," then it becomes enabled.

**FEAT-24.SPEC-002-AC-07:** Given Nadia has satisfied both confirmation conditions, when she taps "Delete My Account," then FEAT-24.SPEC-009 fires immediately and FEAT-24.SPEC-004 begins, and the screen shows "Deleting your account -- this can take a few minutes for accounts with a lot of history."

**FEAT-24.SPEC-002-AC-08:** Given deletion processing is underway, when Nadia looks at the screen, then every control is disabled and a second tap on Delete My Account has no effect.

**FEAT-24.SPEC-002-AC-09:** Given FEAT-24.SPEC-004 reports successful completion, when processing finishes, then Nadia is signed out and no screen in the product remains for her account.

**FEAT-24.SPEC-002-AC-10:** Given FEAT-24.SPEC-004 reports failure, when processing ends, then the screen shows "We couldn't complete account deletion. Your account has not been changed. Try again." with the checkbox and confirmation text cleared.

**FEAT-24.SPEC-002-AC-11:** Given Nadia taps Cancel at any point before confirming, when she does so, then no deletion action is taken and she returns to FEAT-24.SPEC-001.

**FEAT-24.SPEC-002-AC-12:** Given Owen (Client Primary Contact) has no navigation path to this screen, when he follows any link in the client portal, then he never reaches it.

**FEAT-24.SPEC-002-AC-13:** Given Dana (Support Operator) is in an active read-only support session, when she attempts to open this screen directly, then she sees "This isn't available during a support session."

**FEAT-24.SPEC-002-AC-14:** Given Nadia's session expires while she has partially filled the confirmation region, when she is shown the expired-session dialog and signs back in, then the confirmation region is reset (checkbox unchecked, text cleared) rather than restored.

**FEAT-24.SPEC-002-AC-15:** Given Nadia sends a new unpaid invoice in another tab after this screen has already loaded, when she taps "Delete My Account" here, then the open-item evaluation re-runs at that moment, the tap is rejected, the screen shows the new unpaid-invoice banner and the changed-warnings notice, and neither FEAT-24.SPEC-009 nor FEAT-24.SPEC-004 has fired.

**FEAT-24.SPEC-002-AC-16:** Given Nadia loses connectivity while reviewing this screen, when the connection drops, then the banner "You're offline. Reconnect to continue." appears and the confirmation region is disabled.

**FEAT-24.SPEC-002-AC-17:** Given the screen is in Loaded -- warnings changed after a rejected tap and the checkbox and text are still valid, when Nadia taps "Delete My Account" again and the re-check matches the warnings now displayed, then FEAT-24.SPEC-009 fires, FEAT-24.SPEC-004 begins, and the screen enters Processing.

**FEAT-24.SPEC-002-AC-18:** Given Nadia has 1 unpaid invoice of $1,200.00 displayed and pays it in another tab, when she taps "Delete My Account," then the tap is rejected, the unpaid-invoice banner is removed, the notice "Your open items changed. Review the updated warnings, then tap Delete My Account again." appears, and no notification or deletion is triggered.

**FEAT-24.SPEC-002-AC-19:** Given Nadia has an open invoice and a pending approval, when this screen loads, then the unpaid-invoice banner appears above the pending-approval banner and above the retention notice, each with the exact text defined in FEAT-24.SPEC-006.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 8 (loading, loaded-no-warnings, loaded-with-warnings, loaded-warnings-changed, confirming, processing, error, offline) | 8 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |



# Automation Spec: Data Export Archive Generation

## Overview

**Name:** Data Export Archive Generation
**ID:** FEAT-24.SPEC-003
**Type:** Automation
**Purpose:** Aggregates every client, project, proposal, invoice, and activity record Nadia owns into a single downloadable archive, retries cleanly on failure, and expires the archive after its limited download window.
**Parent Feature:** FEAT-24 -- Data Export & Account Deletion

## Scope and Non-Goals

**In Scope:**
- Aggregating every entity Nadia owns into one archive file on request
- Storing and delivering the file through the large-file storage & delivery capability
- Retrying generation failures without leaving partial or corrupted output
- Advancing the archive through its lifecycle states and expiring it after its download window

**Non-Goals:**
- Requesting the export and displaying its status -- owned by FEAT-24.SPEC-001 (Data Export Screen); this automation only performs and reports on the generation itself.
- Partial or selective export by data type -- excluded per product-features.md's Key Capabilities, which name only a whole-account export; this automation always aggregates the full set of owned records.
- Deciding who may request an export -- owned by FEAT-24.SPEC-007 (Export & Deletion Access Rules); this automation trusts that authorization has already passed by the time its trigger fires.
- Permanently removing the freelancer's underlying data -- owned by FEAT-24.SPEC-004 (Account Deletion Processing); this automation only reads existing data to build a copy, never deletes the source records.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nadia requests a full export | FEAT-24.SPEC-001 (Data Export Screen) | Fires whenever Request Export (or Request New Export) is tapped, always on successful authorization via FEAT-24.SPEC-007 | Freelancer account reference |
| Archive successfully delivered | FEAT-16.SPEC-007 (Large-File Storage & Delivery Capability) | Fires when the storage & delivery capability reports the download transfer for this archive completed | Archive reference, transfer confirmation |
| Download window elapses unfetched | System (scheduled sweep) | Fires when an archive's download window (platform parameter: `data-export-download-window`) has elapsed since it reached Ready, regardless of whether it was ever downloaded | Archive reference, elapsed time since Ready |

## Processing Logic

1. Receive the export request from FEAT-24.SPEC-001, already authorized via FEAT-24.SPEC-007 (Nadia, her own account only).
2. If an existing Data Export Archive record for this account is in Ready, Downloaded, or Expired state, discard it and any underlying stored file via FEAT-16.SPEC-007 -- only one active archive exists per account at a time.
3. Create a new Data Export Archive record with status Requested and requested-at set to the current time.
4. Aggregate every record Nadia owns into one archive file: Client, Client Contact, Project, Proposal, Milestone, Payment Schedule, Deliverable, Deliverable Version (as version history), Comment, Invoice, Payment, Reminder Log, and Activity Log Entry (feature-overview.md, Referenced Entities table), scoped strictly to this one freelancer account (ASMP-23 isolation).
5. Submit the assembled file to the large-file storage & delivery capability (FEAT-16.SPEC-007) for storage and delivery.
6. On confirmed storage, set the archive's status to Ready and its download link/window to expire at platform parameter: `data-export-download-window` from the moment it reaches Ready.
7. Signal FEAT-24.SPEC-001 to reflect the Ready state.
8. Trigger FEAT-24.SPEC-008 (Export Ready Notification).
9. When FEAT-16.SPEC-007 reports a completed download transfer for this archive, set its status to Downloaded (the download action itself remains available until the archive expires).
10. When the download window elapses (whether the archive is Ready or Downloaded), set its status to Expired and purge its underlying stored file via FEAT-16.SPEC-007; there is no restore path for an expired archive.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Archive Ready | Aggregation and storage succeed | Data Export Archive: Requested -> Ready; download link/window set | FEAT-24.SPEC-001 shows the Ready status card with a Download action; FEAT-24.SPEC-008 fires | FEAT-24.SPEC-001, FEAT-24.SPEC-008 |
| Archive Downloaded | FEAT-16.SPEC-007 reports a completed transfer | Data Export Archive: Ready -> Downloaded | FEAT-24.SPEC-001 shows "Downloaded on {date}," Download action remains available | FEAT-24.SPEC-001 |
| Generation retried after transient failure | Aggregation or storage submission fails mid-process | No partial file persisted; the same Requested archive record retries automatically | FEAT-24.SPEC-001 continues showing "Preparing your export..." | FEAT-24.SPEC-001 |
| Generation failed (retries exhausted) | Every retry up to platform parameter: `data-export-generation-retry-count` fails | Archive record carries no download link; status remains a failed Requested state | FEAT-24.SPEC-001 shows "We couldn't generate your export. Try again." | FEAT-24.SPEC-001 |
| Archive Expired | Download window elapses since Ready, regardless of Downloaded state | Data Export Archive: -> Expired; underlying stored file purged via FEAT-16.SPEC-007 | FEAT-24.SPEC-001 shows "Your last export has expired." | FEAT-24.SPEC-001 |
| No-action (sweep finds nothing due) | Scheduled sweep runs and no archive has reached its window | None | None | -- |

## Data Model

**Reads:** Client, Client Contact, Project, Proposal, Milestone, Payment Schedule, Deliverable, Deliverable Version, Comment, Invoice, Payment, Reminder Log, Activity Log Entry -- every field of each, scoped to the requesting freelancer's own account.
**Creates:** Data Export Archive -- requested-at, status, download link/window.
**Updates:** Data Export Archive -- status transitions Requested -> Ready -> Downloaded -> Expired.
**Deletes:** Data Export Archive's underlying stored file (via FEAT-16.SPEC-007) once superseded by a new request or once Expired; the record itself is not deleted, it simply carries no further download link.

## Business Rules

- Only one active archive exists per account at a time; a new request always supersedes any prior archive rather than queuing behind it (feature-overview.md, Entity-Lifecycle Coverage Matrix).
- Whole-account export only -- no partial or selective aggregation exists (product-features.md, Key Capabilities).
- The export reflects current data at request time, including Invoice and Payment records that would later be subject to legal retention on account deletion (SC-24) -- retention rules apply only to deletion, never to what an export may include.
- XBR-29: an operator support session never triggers, views, or generates a data export; this automation's trigger is exclusively FEAT-24.SPEC-001, which FEAT-24.SPEC-007 confines to Nadia.
- A failed generation never leaves a partial or corrupted archive visible to Nadia (product-features.md, States field) -- retries occur before any failure is surfaced.

## Edge Cases

- **Nadia's account has almost no data (a brand-new account)** -- The archive still generates normally, containing whatever minimal set of records exists; States field: "this always operates on whatever data exists, even a near-empty account."
- **A deliverable file referenced by a Deliverable Version cannot be included at generation time (e.g., storage capability degradation)** -- Generation retries per the failure path; if retries exhaust with this file still unavailable, the whole generation is reported failed rather than producing an incomplete archive, since a partial archive contradicts the "no partial, corrupted output" guarantee.
- **The archive expires at the same moment Nadia taps Download** -- Whichever completes first wins: if the expiry sweep processes first, the Download action fails with "This export has expired. Request a new one."; if the download transfer is already in flight, it is allowed to complete and the archive is marked Downloaded rather than Expired.
- **Concurrent trigger firing (Nadia requests a new export from two open tabs at effectively the same time)** -- Both requests reach step 2 independently; whichever's discard-and-create sequence commits first becomes the active archive, and the second request's discard step then finds and supersedes that first archive in turn, leaving exactly one active archive reflecting the later request. Neither request errors.
- **Trigger fires while a previous generation run is still in flight** -- A second request for the same account interrupts the in-flight generation by proceeding through the same discard step (step 2); the in-flight run's eventual completion is discarded since its archive record no longer exists, avoiding two archives being finalized for one account.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-24.SPEC-001 (Data Export Screen) | Triggered by (inbound) | Request Export fires this automation |
| FEAT-24.SPEC-001 (Data Export Screen) | Affects (outbound) | Every outcome updates the status card shown there |
| FEAT-24.SPEC-007 (Export & Deletion Access Rules) | References (inbound) | Authorization gate this automation trusts has already passed |
| FEAT-24.SPEC-008 (Export Ready Notification) | Triggers (outbound) | Fires when the archive reaches Ready |
| FEAT-16.SPEC-007 (Large-File Storage & Delivery Capability) | Triggers (outbound) / Triggered by (inbound) | Performs the actual storage, delivery, and purge of the archive file; its "Archive download delivered" inbound event triggers the Ready -> Downloaded transition |

## Analytics and Success Signals

- **data_export_archive_ready** (generation_duration_bucket) -- N/A -- no success-metrics.md metric is connected to Data Export & Account Deletion; this event is retained because product-features.md's Signals field names "data_export_ready" explicitly as this feature's defined signal.
- **data_export_generation_failed** (failure_point: aggregation / storage_submission; retries_exhausted: yes/no) -- N/A -- same reason; retained so a generation path that never resolves stays observable rather than silent.
- **data_export_archive_expired** (was_downloaded: yes/no) -- N/A -- same reason; retained to distinguish an archive Nadia never came back for from one she already retrieved.

## Acceptance Criteria

**FEAT-24.SPEC-003-AC-01:** Given Nadia taps Request Export on FEAT-24.SPEC-001 with no prior archive, when this automation runs, then it aggregates every owned record into a new archive and, on success, sets the archive to Ready with a download link/window of platform parameter: `data-export-download-window`.

**FEAT-24.SPEC-003-AC-02:** Given the archive reaches Ready, when this outcome applies, then FEAT-24.SPEC-008 fires and FEAT-24.SPEC-001 shows the Download action.

**FEAT-24.SPEC-003-AC-03:** Given the storage & delivery capability reports a completed download transfer for the archive, when this event arrives, then the archive's status advances from Ready to Downloaded and the Download action remains available.

**FEAT-24.SPEC-003-AC-04:** Given aggregation fails transiently on the first attempt, when the failure occurs, then generation retries automatically without exposing a partial archive to Nadia.

**FEAT-24.SPEC-003-AC-05:** Given every retry up to platform parameter: `data-export-generation-retry-count` fails, when the final retry fails, then the archive carries no download link and FEAT-24.SPEC-001 shows "We couldn't generate your export. Try again."

**FEAT-24.SPEC-003-AC-06:** Given an archive is Ready or Downloaded, when its download window (platform parameter: `data-export-download-window`) elapses, then its status becomes Expired and its underlying stored file is purged via FEAT-16.SPEC-007.

**FEAT-24.SPEC-003-AC-07:** Given Nadia requests a new export while a prior archive is Ready, Downloaded, or Expired, when the new request is received, then the prior archive and its file are discarded and a fresh archive begins generating.

**FEAT-24.SPEC-003-AC-08:** Given a freelancer account with almost no data, when an export is requested, then the archive still generates successfully, containing whatever records exist.

**FEAT-24.SPEC-003-AC-09:** Given the archive's download window elapses at the same moment Nadia's download transfer is already in flight, when both occur, then the in-flight transfer is allowed to complete and the archive is marked Downloaded rather than Expired.

**FEAT-24.SPEC-003-AC-10:** Given the archive's download window elapses before Nadia starts a download, when she then attempts to download, then the attempt fails with "This export has expired. Request a new one." and no file transfer occurs.

**FEAT-24.SPEC-003-AC-11:** Given Nadia requests a new export from two open tabs at effectively the same time, when both requests are processed, then exactly one active archive results, reflecting the later request.

**FEAT-24.SPEC-003-AC-12:** Given a generation run is already in flight when a new request for the same account arrives, when the new request is processed, then it supersedes the in-flight run so that only the new request's resulting archive is ever finalized.

**FEAT-24.SPEC-003-AC-13:** Given this automation aggregates data for one freelancer's export, when it runs, then it never includes another freelancer's records.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 | 3 |
| Outcome Paths | 6 | 6 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |



# Automation Spec: Account Deletion Processing

## Overview

**Name:** Account Deletion Processing
**ID:** FEAT-24.SPEC-004
**Type:** Automation
**Purpose:** Cascades the permanent removal of the freelancer's account and all owned data across every data-holding feature once she confirms, disconnecting her payment account, and leaving the account fully intact if processing fails.
**Parent Feature:** FEAT-24 -- Data Export & Account Deletion

## Scope and Non-Goals

**In Scope:**
- Executing the cascading hard delete of the Freelancer Account and every entity it owns, once Nadia's explicit confirmation is received
- Disconnecting the Payment Account Connection as part of the cascade
- Holding back financial records (Invoice, Payment) subject to legal retention rather than deleting them immediately
- Ordering the cascade into a reversible hold phase, one explicit commit point (the payment-account disconnect), and an irreversible finalization phase, so a failure before the commit point reverts the account fully to Active and nothing is ever left half-deleted
- Completing the irreversible finalization phase by automatic retry once the commit point has passed (it is never reverted)

**Non-Goals:**
- Capturing Nadia's confirmation or displaying open-item warnings -- owned by FEAT-24.SPEC-002 (Account Deletion Screen); this automation begins only once that confirmation is already given.
- Determining which open items warrant a warning or which records are legally retained versus immediately deleted -- owned by FEAT-24.SPEC-006 (Pre-Deletion Warning & Retention Determination Rules); this automation executes that determination's output.
- Purging financial records once their legal retention period elapses -- owned by FEAT-24.SPEC-005 (Legal Retention Purge); this automation only holds them back and hands them off, it never itself purges a retained record.
- Restoring a deleted account -- excluded per product-features.md's Validation & Limits field ("irreversibility"); no restore path exists once this automation reports completion.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nadia gives explicit confirmation | FEAT-24.SPEC-002 (Account Deletion Screen) | Fires only after the acknowledgment checkbox is checked and the confirmation text exactly matches "DELETE," and after FEAT-24.SPEC-007's authorization passes | Freelancer account reference, confirmation timestamp |

## Processing Logic

1. Receive the confirmed deletion instruction from FEAT-24.SPEC-002, already authorized via FEAT-24.SPEC-007 (Nadia, her own account only).
2. Set the Freelancer Account's state to Deletion Requested, then immediately to Deletion Confirmed (Active -> Deletion Requested -> Deletion Confirmed -> Deleted, per feature-overview.md's Entity-Lifecycle Coverage Matrix). In parallel, FEAT-24.SPEC-009 (Account Deletion Final Warning Notification) fires from the triggering confirmation itself; this automation does not wait for that email to send before proceeding, since the confirmation is the point of no return, not the email's delivery.
3. **Classify (reversible; read-only).** For every Invoice and Payment record the account holds, determine retain-versus-delete via FEAT-24.SPEC-006 (Pre-Deletion Warning & Retention Determination Rules). Nothing is changed in this step.
4. **Hold phase (reversible).** Mark every entity the account owns as pending-delete: Client, Client Contact (including the contact's personal data), Project, Proposal, Milestone, Payment Schedule, Deliverable, Deliverable Version, Comment, Reminder Log, Branding Profile, Subscription Plan, Custom Domain Record, Notification, Support Access Session, and Referral Attribution. Mark every Invoice and Payment flagged Retained in step 3 as retained-hold. A pending-delete or retained-hold entity is hidden from every role and every screen but no row, file, or Activity Log Entry is removed or altered beyond that marker, so the marker can be cleared to restore the entity exactly as it was. Confirm every marker is written; if any marker cannot be written after its retries, go to step 10 (pre-commit failure).
5. **Commit point -- disconnect the Payment Account Connection** via FEAT-32.SPEC-004 (Disconnect Payment Account). This is the first irreversible action, so it is deliberately the only irreversible action that can still fail the deletion: no earlier step is irreversible and no later step reverts. If the disconnect fails after FEAT-32.SPEC-004's own retries, go to step 10 (pre-commit failure). If it succeeds, the commit point has passed: from here the account is never reverted to Active.
6. **Report the commit to FEAT-24.SPEC-002** and sign Nadia out. Every entity is already hidden by step 4, so Nadia is signed out here rather than waiting for the finalization phase. FEAT-24.SPEC-002 treats this report as completion.
7. **Finalize -- Activity Log removal (irreversible).** Remove Activity Log Entries under FEAT-13's own retention and purge rule (FEAT-13.SPEC-006), which applies the same financial-record retention exception as step 3.
8. **Finalize -- stored-byte purge (irreversible).** Purge every stored deliverable file and version's bytes via FEAT-16.SPEC-006 (Stored File Purge on Account Deletion).
9. **Finalize -- hard delete and hand-off (irreversible).** Hard-delete every pending-delete entity from step 4. Hand every retained-hold Invoice and Payment to FEAT-24.SPEC-005 (Legal Retention Purge) for later removal once its legal retention period lapses; they stay in a retained, no-longer-accessible state until then. Then set the Freelancer Account's state to Deleted -- the account record itself is hard-deleted, with no restore path.
10. **Failure handling, by phase.**
    - **Pre-commit failure (any failure in steps 4 or 5, unrecoverable after retries):** halt, clear every pending-delete and retained-hold marker written in step 4, and revert the Freelancer Account's state to Active. Because steps 3 to 5 changed nothing irreversibly, every entity is exactly as it was before step 2 began. FEAT-24.SPEC-002 shows its deletion-failed error.
    - **Post-commit failure (any failure in steps 7, 8 or 9):** never revert. Every step in the finalization phase is idempotent, so the failed step is retried automatically at each platform parameter: `deletion-finalization-retry-interval` until it succeeds, then the remaining steps continue in order. Nadia has already been signed out and every entity stays hidden throughout, so no half-deleted data is ever visible or usable. The failure is recorded through account_deletion_failed with reverted: no.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Deletion completes | The commit point (step 5) succeeds and every finalization step (7-9) then succeeds, on first attempt or by retry | Freelancer Account and every owned entity are hard-deleted, except Invoice and Payment records flagged for retention, which are held in a retained, inaccessible state | Nadia is signed out at the commit report (step 6); no screen remains for her account | FEAT-32.SPEC-004, FEAT-13.SPEC-006, FEAT-16.SPEC-006, FEAT-24.SPEC-005 (receives the retained records) |
| Deletion fails before the commit point | Step 4 (hold) or step 5 (payment-account disconnect) fails and cannot be retried to success | Nothing irreversible has occurred; every pending-delete and retained-hold marker is cleared and the Freelancer Account reverts fully to Active | FEAT-24.SPEC-002 shows "We couldn't complete account deletion. Your account has not been changed. Try again." | FEAT-24.SPEC-002 |
| Finalization step fails after the commit point | Step 7, 8 or 9 fails after the commit report | No revert; entities stay hidden and the failed idempotent step is retried automatically until it succeeds | None to Nadia (already signed out); account_deletion_failed recorded with reverted: no | FEAT-13.SPEC-006, FEAT-16.SPEC-006, FEAT-24.SPEC-005 |
| No-action (already deleted) | The trigger fires for an account already in the Deleted state (e.g., a stale retry) | None | N/A -- no account remains to notify | -- |

## Data Model

**Reads:** Freelancer Account and every entity it owns (Client, Client Contact, Project, Proposal, Milestone, Payment Schedule, Deliverable, Deliverable Version, Comment, Invoice, Payment, Reminder Log, Activity Log Entry, Branding Profile, Subscription Plan, Custom Domain Record, Notification, Payment Account Connection, Support Access Session, Referral Attribution).
**Creates:** None.
**Updates:** Every owned entity -- pending-delete marker set in step 4 (cleared again on a pre-commit failure); Invoice and Payment -- retained-hold marker set in step 4, then transitioned to a retained, inaccessible state pending later purge by FEAT-24.SPEC-005; Freelancer Account -- state transitions Active -> Deletion Requested -> Deletion Confirmed -> Deleted (or reverted to Active on failure).
**Deletes:** Freelancer Account, Client, Client Contact, Project, Proposal, Milestone, Payment Schedule, Deliverable, Deliverable Version, Comment, Reminder Log, Activity Log Entry (non-retained), Branding Profile, Subscription Plan, Custom Domain Record, Notification, Payment Account Connection (disconnected), Support Access Session, Referral Attribution -- all hard-deleted in step 9 (Activity Log Entries in step 7, Payment Account Connection disconnected in step 5), no restore path once the commit point passes.

## Business Rules

- XBR-33: account deletion warns about open items (owned by FEAT-24.SPEC-002/SPEC-006), disconnects the payment account, removes all the freelancer's data including client contacts' personal data, and keeps only financial records under legal retention.
- The legal financial-record retention period is a platform-set policy value, referenced only as platform parameter: `financial-record-legal-retention-period` (the same marker FEAT-13.SPEC-006 uses for its own Activity Log Entry retention exception). The retry interval for post-commit finalization is likewise a platform-set value, referenced only as platform parameter: `deletion-finalization-retry-interval`.
- A deletion that fails before the commit point always reverts the account fully to Active -- no partial removal is ever left standing (product-features.md, States field). That guarantee holds because nothing irreversible runs before the commit point (step 5) and everything irreversible after it (steps 7-9) is retried to success rather than reverted, with all data hidden throughout.
- Reversible steps: 3 (classify) and 4 (hold). Commit point: 5 (payment-account disconnect). Irreversible steps: 5, 7, 8 and 9. The FEAT-13.SPEC-006 and FEAT-16.SPEC-006 steps are invoked only after the commit point and are never run while a revert is still possible.
- Deletion is irreversible once completed; no restore path exists anywhere in the product (feature-overview.md, Non-Goals).
- FEAT-24.SPEC-006 is the sole authority on which records are retained versus immediately deleted; this automation never makes that determination itself.

## Edge Cases

- **Concurrent trigger firing (two confirmed-deletion instructions for the same account arrive at effectively the same time, e.g., from two sessions of Nadia's own)** -- Whichever instruction reaches step 2 first proceeds through the full cascade; the second is treated as the trigger-fires-while-a-previous-run-is-in-flight case below rather than as an independent second cascade.
- **Trigger fires while a previous run is still in flight** -- A second confirmed-deletion instruction for the same account while step 2 has already moved it out of Active does not start a second concurrent cascade; it is discarded as redundant, since the account is already mid-transition per the first instruction.
- **Failure specifically at the Payment Account Connection disconnect step (step 5)** -- Retried per FEAT-32.SPEC-004's own failure handling; if the final retry still fails, the cascade halts and reverts per step 10 (pre-commit failure), clearing every hold marker. Nothing else has been changed irreversibly at that point.
- **Failure in the stored-byte purge (step 8) or Activity Log removal (step 7) after the commit point** -- Not reverted; the account is already signed out and hidden, and the step is retried until it succeeds (step 10, post-commit failure). Nadia sees no error, since the outcome she was shown (commit report) is already true.
- **Process interruption between the hold phase and the commit point (e.g., the worker restarts mid-run)** -- The account is still in Deletion Confirmed with only reversible markers written; the run is resumed from step 4, and if it cannot resume, step 10 (pre-commit failure) reverts it to Active. Process interruption after the commit point resumes at the first unfinished finalization step.
- **Nadia has open unpaid invoices or pending approvals at the moment of confirmation** -- Deletion still proceeds; the warning shown by FEAT-24.SPEC-002 was informational only, and any invoice flagged for retention by FEAT-24.SPEC-006 is held back rather than blocking the rest of the cascade.
- **A client contact's personal data (name, email) exists at deletion time** -- Removed in full as part of the Client Contact cascade, per GDPR-class handling (product-features.md, Access field), even though the contact's own acceptances and approvals elsewhere in the product retain the contact's name as evidence under ASMP-20 (a separate feature's own retention behavior, not reversed by this automation).
- **The confirmed-deletion instruction targets an account already in the Deleted state (a stale retry)** -- No-action outcome; nothing is re-processed and no error is raised.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-24.SPEC-002 (Account Deletion Screen) | Triggered by (inbound) | Confirmed deletion fires this automation |
| FEAT-24.SPEC-002 (Account Deletion Screen) | Affects (outbound) | Failure outcome is shown there with the account reverted |
| FEAT-24.SPEC-006 (Pre-Deletion Warning & Retention Determination Rules) | References (inbound) | Supplies the retain-versus-delete determination this automation executes |
| FEAT-24.SPEC-009 (Account Deletion Final Warning Notification) | Triggers (outbound) | Fires in parallel with the start of this automation, from the same confirmation |
| FEAT-24.SPEC-005 (Legal Retention Purge) | Affects (outbound) | Receives every retained Invoice and Payment record for later purge once retention lapses |
| FEAT-24.SPEC-007 (Export & Deletion Access Rules) | References (inbound) | Authorization gate this automation trusts has already passed |
| FEAT-32.SPEC-004 (Disconnect Payment Account) | Triggers (outbound) | Executes the payment-account disconnect step of the cascade (XBR-33) |
| FEAT-13.SPEC-006 (Retention & Account-Deletion Purge Rule) | Triggers (outbound) | Executes the Activity Log Entry removal step, subject to the same retention exception |
| FEAT-16.SPEC-006 (Stored File Purge on Account Deletion) | Triggers (outbound) | Executes the stored-file purge step of the cascade |
| FEAT-01 through FEAT-33 (every data-holding feature) | Affects (outbound) | Cascading hard-delete removes every record these features created, per the dependency map's per-entity "Deleted by FEAT-24" notes and XBR-33 |

## Analytics and Success Signals

- **account_deletion_requested** (open_items_present: yes/no) -- N/A -- no success-metrics.md metric is connected to Data Export & Account Deletion; retained because product-features.md's Signals field names this event explicitly (this automation's own emission point for the signal, distinct from FEAT-24.SPEC-002's screen-level emission of the same event name at confirmation).
- **account_deletion_completed** (retained_financial_records_count) -- N/A -- same reason; retained as this feature's defined completion signal.
- **account_deletion_failed** (failure_step: hold / payment_disconnect / activity_purge / storage_purge / hard_delete; reverted: yes for hold and payment_disconnect, no for activity_purge, storage_purge and hard_delete) -- N/A -- same reason; retained so a reverted attempt and a retried post-commit finalization both stay observable.

## Acceptance Criteria

**FEAT-24.SPEC-004-AC-01:** Given Nadia has confirmed deletion on FEAT-24.SPEC-002, when this automation runs, then the Freelancer Account transitions through Deletion Requested and Deletion Confirmed before the hold phase (step 4) begins.

**FEAT-24.SPEC-004-AC-02:** Given the confirmation is received, when this automation starts, then FEAT-24.SPEC-009 fires in parallel without this automation waiting for that email to send.

**FEAT-24.SPEC-004-AC-03:** Given the cascade reaches an Invoice subject to legal retention, when FEAT-24.SPEC-006 flags it for retention, then that Invoice is held in a retained, inaccessible state rather than hard-deleted, and handed to FEAT-24.SPEC-005.

**FEAT-24.SPEC-004-AC-04:** Given the cascade proceeds, when it reaches the Payment Account Connection, then FEAT-32.SPEC-004 disconnects it in step 5, after every entity has been marked pending-delete, as the commit point of this cascade.

**FEAT-24.SPEC-004-AC-05:** Given the cascade proceeds, when it reaches the Activity Log Entries, then they are removed in step 7, after the commit point, under FEAT-13.SPEC-006's own retention rule, subject to the same financial-record retention exception.

**FEAT-24.SPEC-004-AC-06:** Given the cascade proceeds, when it reaches stored deliverable files and versions, then FEAT-16.SPEC-006 purges every stored byte in step 8, only after the commit point has passed.

**FEAT-24.SPEC-004-AC-07:** Given the commit point succeeds, when this automation reports the commit, then Nadia is signed out; and when every finalization step (7-9) has then succeeded, the Freelancer Account is set to Deleted and hard-deleted.

**FEAT-24.SPEC-004-AC-08:** Given the Payment Account Connection disconnect step fails after exhausting its own retries, when the failure is reported, then the cascade halts, every pending-delete and retained-hold marker is cleared, and the Freelancer Account reverts fully to Active with every entity as before.

**FEAT-24.SPEC-004-AC-09:** Given Nadia has open unpaid invoices or pending approvals at confirmation, when the cascade runs, then deletion still proceeds, with only the retention-flagged records held back.

**FEAT-24.SPEC-004-AC-10:** Given a Client Contact exists with personal data at deletion time, when the cascade reaches Client Contact, then the contact's name and email are removed in full.

**FEAT-24.SPEC-004-AC-11:** Given a confirmed-deletion instruction arrives for an account already in the Deleted state, when this automation processes it, then nothing changes and no error is raised.

**FEAT-24.SPEC-004-AC-12:** Given a second confirmed-deletion instruction for the same account arrives while a first cascade is already in flight, when it arrives, then it is discarded as redundant rather than starting a second concurrent cascade.

**FEAT-24.SPEC-004-AC-13:** Given two confirmed-deletion instructions for the same account arrive at effectively the same time, when both are processed, then only one cascade actually runs to completion.

**FEAT-24.SPEC-004-AC-14:** Given this automation processes a deletion for one freelancer's account, when the cascade runs, then it never touches another freelancer's data.

**FEAT-24.SPEC-004-AC-15:** Given the hold phase (step 4) is running, when Nadia's Client, Project, and Deliverable records are marked pending-delete, then no row or stored byte has been removed and clearing the markers restores each exactly as it was.

**FEAT-24.SPEC-004-AC-16:** Given the payment-account disconnect (step 5) has not yet succeeded, when any earlier step fails, then no Activity Log Entry has been removed and no stored byte has been purged, and the account reverts to Active.

**FEAT-24.SPEC-004-AC-17:** Given the commit point has passed and the stored-byte purge (step 8) fails, when the failure is reported, then the account is not reverted to Active, Nadia's data stays hidden, the step is retried at platform parameter: `deletion-finalization-retry-interval` until it succeeds, and account_deletion_failed is recorded with reverted: no.

**FEAT-24.SPEC-004-AC-18:** Given the hold phase (step 4) cannot write a pending-delete marker for one entity after its retries, when the failure is reported, then no payment-account disconnect is attempted, all markers written so far are cleared, and the account reverts to Active.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 4 | 4 |
| Business Rules | 6 | 6 |
| Edge Cases | 8 | 8 |



# Automation Spec: Legal Retention Purge

## Overview

**Name:** Legal Retention Purge
**ID:** FEAT-24.SPEC-005
**Type:** Automation
**Purpose:** Purges the Invoice and Payment records held back from an otherwise-completed account deletion once their legal retention period lapses.
**Parent Feature:** FEAT-24 -- Data Export & Account Deletion

## Scope and Non-Goals

**In Scope:**
- Periodically checking every Invoice and Payment record held back under legal retention by a completed FEAT-24.SPEC-004 deletion
- Permanently purging each such record once its legal retention period elapses
- Leaving no restore path once a record is purged

**Non-Goals:**
- Determining that an Invoice or Payment must be retained in the first place -- owned by FEAT-24.SPEC-006 (Pre-Deletion Warning & Retention Determination Rules); this automation only acts on records FEAT-24.SPEC-004 has already flagged and held back.
- Purging Activity Log Entries subject to the same retention exception -- owned entirely by FEAT-13.SPEC-006 (Retention & Account-Deletion Purge Rule), which purges its own retained entries on the same elapsed-period condition independently of this automation, to avoid duplicating that ownership.
- Any purge not tied to a completed account deletion -- excluded per feature-overview.md's Non-Goals ("Retention of data beyond the legal financial-record requirement"); this automation never runs against an active (non-deleted) account.
- Notifying anyone when a purge completes -- product-features.md's Communications field for this feature names only the export-ready and deletion-final-warning emails; no account or person remains to notify once a deletion has already completed and this later purge runs.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A retained financial record's legal retention period elapses | System (scheduled sweep, interval: platform parameter: `legal-retention-purge-sweep-interval`) | Fires when the sweep finds an Invoice or Payment held back by a completed FEAT-24.SPEC-004 deletion whose retention period (platform parameter: `financial-record-legal-retention-period`), counted from that deletion's completion, has elapsed | The retained record's reference and the deletion-completion date it is measured from |

## Processing Logic

1. On each scheduled sweep, identify every Invoice and Payment record currently held in the retained, inaccessible state established by FEAT-24.SPEC-004.
2. For each such record, calculate the elapsed time since the account's deletion completion date (the date FEAT-24.SPEC-004 set the Freelancer Account to Deleted).
3. If the elapsed time is at or beyond platform parameter: `financial-record-legal-retention-period`, permanently purge that record. If it is not yet at the threshold, leave it untouched until a later sweep.
4. Confirm the purge before considering the record removed; a failed purge attempt is retried on the next sweep rather than reported as complete.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Record purged | Elapsed time meets or exceeds platform parameter: `financial-record-legal-retention-period` | The Invoice or Payment record is permanently removed, with no restore path | None -- the freelancer account no longer exists by this point | -- |
| Not yet due | Elapsed time is below the threshold | None | None | -- |
| Purge retried | The purge attempt itself fails (a processing error) | Record remains retained, untouched | None | -- |
| No-action (sweep finds nothing due) | No retained record has reached its threshold at this sweep | None | None | -- |

## Data Model

**Reads:** Invoice and Payment -- every record currently held in the retained, inaccessible state, plus the deletion-completion date each is measured against.
**Creates:** None.
**Updates:** None.
**Deletes:** Invoice and Payment -- each record once its retention period elapses, permanently and without a restore path.

## Business Rules

- The legal retention period is a platform-set policy value, referenced only as platform parameter: `financial-record-legal-retention-period` -- the same marker FEAT-24.SPEC-004 and FEAT-13.SPEC-006 use for the identical retention exception.
- This automation never runs against an active account -- it acts exclusively on records a completed FEAT-24.SPEC-004 deletion has already flagged and held back (SC-24).
- Once a record's retention period elapses, its purge is unconditional -- there is no further extension, hold, or manual review step (feature-overview.md's Non-Goals: "no retention of data beyond the legal financial-record requirement").
- Activity Log Entries subject to the same retention window are purged independently by FEAT-13.SPEC-006, not by this automation, avoiding duplicate ownership of one retention exception.

## Edge Cases

- **Concurrent sweep runs overlap (two scheduled sweeps fire close together)** -- A record already purged by one sweep is simply absent from the next sweep's candidate set; no error occurs from encountering an already-removed record.
- **Trigger fires while a previous sweep is still in flight** -- A new sweep for the same records waits for the first to finish rather than processing the same candidate set concurrently, avoiding two processes racing to purge the same record.
- **A record's retention period elapses between two sweep cycles rather than exactly at a sweep's run time** -- It is purged at the first sweep that runs at or after the threshold, per platform parameter: `legal-retention-purge-sweep-interval`, not at the exact expiry instant.
- **A purge attempt fails on its first try** -- The record remains retained and is reprocessed at the next sweep rather than being reported purged.
- **An account's deletion is later re-attempted after already completing (a stale retry reaching FEAT-24.SPEC-004)** -- No effect on this automation: FEAT-24.SPEC-004's own no-action outcome for an already-deleted account means the deletion-completion date this automation measures against never changes.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-24.SPEC-004 (Account Deletion Processing) | Triggered by (inbound) | Supplies the retained Invoice and Payment records and the deletion-completion date this automation measures against |
| FEAT-24.SPEC-006 (Pre-Deletion Warning & Retention Determination Rules) | References (inbound) | Defines which records qualify as retained in the first place |
| FEAT-13.SPEC-006 (Retention & Account-Deletion Purge Rule) | References (outbound) | Purges the equivalent, financial-character Activity Log Entries independently, under the same retention window |

## Analytics and Success Signals

- **financial_record_retention_purge_completed** (record_type: invoice / payment) -- N/A -- no success-metrics.md metric is connected to Data Export & Account Deletion; retained as the audit-relevant record that a legally required purge actually completed.
- **financial_record_retention_purge_retried** (record_type: invoice / payment) -- N/A -- same reason; retained so a purge that keeps failing across sweeps stays observable.

## Acceptance Criteria

**FEAT-24.SPEC-005-AC-01:** Given an Invoice was held back by FEAT-24.SPEC-004 and its retention period (platform parameter: `financial-record-legal-retention-period`) has elapsed since deletion completion, when a scheduled sweep runs, then that Invoice is permanently purged.

**FEAT-24.SPEC-005-AC-02:** Given a Payment held back by FEAT-24.SPEC-004 and its retention period has elapsed, when a scheduled sweep runs, then that Payment is permanently purged.

**FEAT-24.SPEC-005-AC-03:** Given a retained record's elapsed time is below the retention threshold, when a scheduled sweep runs, then the record is left untouched.

**FEAT-24.SPEC-005-AC-04:** Given a purge attempt fails on its first try, when the failure occurs, then the record remains retained and is reprocessed at the next sweep.

**FEAT-24.SPEC-005-AC-05:** Given a record's retention period elapses between two scheduled sweeps, when the next sweep runs at or after the threshold, then the record is purged at that sweep, not exactly at the expiry instant.

**FEAT-24.SPEC-005-AC-06:** Given two scheduled sweeps fire close together, when the second sweep encounters a record the first sweep already purged, then no error occurs and the record is simply absent from the candidate set.

**FEAT-24.SPEC-005-AC-07:** Given a sweep is already in flight, when a new sweep is triggered before the first finishes, then the new sweep waits rather than processing the same candidates concurrently.

**FEAT-24.SPEC-005-AC-08:** Given no retained record has reached its threshold at a given sweep, when the sweep runs, then nothing changes.

**FEAT-24.SPEC-005-AC-09:** Given a retained record is purged, when the purge completes, then no restore path exists for it.

**FEAT-24.SPEC-005-AC-10:** Given Activity Log Entries under the same retention exception exist, when this automation runs, then it purges only Invoice and Payment records, leaving those entries to FEAT-13.SPEC-006's own purge.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 4 | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Logic/Rule Spec: Pre-Deletion Warning & Retention Determination Rules

## Overview

**Name:** Pre-Deletion Warning & Retention Determination Rules
**ID:** FEAT-24.SPEC-006
**Type:** Logic/Rule
**Purpose:** Determines which open items (unpaid invoices, pending approvals) trigger a specific warning without blocking deletion, and which records are legally retained versus immediately deleted.
**Parent Feature:** FEAT-24 -- Data Export & Account Deletion
**Governed Entity:** Deletion Readiness Determination (a derived evaluation, not a stored entity in its own right, computed from Invoice, Milestone, and the entity-class inventory in feature-overview.md's Entity-Lifecycle Coverage Matrix)

## Scope and Non-Goals

**In Scope:**
- Determining whether an open unpaid invoice exists for the account, and defining the exact banner text (with placeholders) for that warning
- Determining whether a pending-approval milestone exists for the account, and defining the exact banner text (with placeholders) for that warning
- Defining the exact retention-notice text shown alongside the warnings
- Determining, per entity class, whether a record is legally retained or immediately deleted at cascade time

**Non-Goals:**
- Displaying the warnings or capturing confirmation -- owned by FEAT-24.SPEC-002 (Account Deletion Screen); this spec only defines what is determined, not how it is shown.
- Executing the actual retention hold-back or deletion -- owned by FEAT-24.SPEC-004 (Account Deletion Processing), which executes this spec's determination.
- Purging a retained record once its retention period lapses -- owned by FEAT-24.SPEC-005 (Legal Retention Purge); this spec only classifies a record as retained, it does not schedule or perform the later purge.
- Who may reach the screens or automations that invoke this determination -- owned entirely by FEAT-24.SPEC-007 (Export & Deletion Access Rules); this spec's own Authorization Rules section defers to it rather than duplicating that gate.

## Governed Entity

**Entity:** Deletion Readiness Determination (derived; composite of Invoice, Milestone, and entity-class retention classification)
**Source:** Feature Dependency Map (Invoice, Milestone entities); feature-overview.md, Entity-Lifecycle Coverage Matrix (entity-class inventory)

| Field | Data Type | Description |
|-------|-----------|-------------|
| open_unpaid_invoice_present | derived (boolean) | Whether the account has at least one Invoice in an open/unresolved status at evaluation time |
| pending_approval_present | derived (boolean) | Whether the account has at least one Milestone awaiting approval at evaluation time |
| open_invoice_count | derived (integer >= 0) | Number of Invoices counted as open under the open_unpaid_invoice_present derivation |
| open_invoice_totals | derived (list of currency + amount) | Sum of each open Invoice's outstanding amount, one entry per invoice currency; empty when open_invoice_count is 0 |
| pending_approval_count | derived (integer >= 0) | Number of Milestones counted as awaiting approval under the pending_approval_present derivation |
| record_retention_classification | derived (enum: Retained \| Immediately Deleted) | Per entity class, whether records of that class are held back under legal retention or hard-deleted immediately during the FEAT-24.SPEC-004 cascade |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-24.SPEC-002 | Account Deletion Screen | On screen load (open-item warning display) and again at the moment Delete My Account is tapped (re-check before proceeding) |
| FEAT-24.SPEC-004 | Account Deletion Processing | During cascade execution, when the process reaches each Invoice and Payment record, to decide retain versus delete |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| open_unpaid_invoice_present | No user input -- system-derived from Invoice.status | Always | On evaluation (screen load and at confirmation) | -- | No |
| pending_approval_present | No user input -- system-derived from Milestone.status | Always | On evaluation (screen load and at confirmation) | -- | No |
| open_invoice_count | No user input -- system-derived count of open Invoices | Always | On evaluation (screen load and at confirmation) | -- | No |
| open_invoice_totals | No user input -- system-derived sum of outstanding amounts per currency | Always | On evaluation (screen load and at confirmation) | -- | No |
| pending_approval_count | No user input -- system-derived count of Milestones in Deliverable Uploaded status | Always | On evaluation (screen load and at confirmation) | -- | No |
| record_retention_classification | No user input -- system-derived, fixed per entity class | Always | On evaluation (at cascade execution) | -- | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Combined warning display | open_unpaid_invoice_present, pending_approval_present | When both are true, FEAT-24.SPEC-002 shows both warning banners together rather than merging them into one combined message, since each names a distinct, independent consequence | N/A -- not an error, an informational display rule |
| Exact warning and notice text | open_invoice_count, open_invoice_totals, pending_approval_count | The banner and notice text is exactly the literal text in Business Rules ("Exact warning and notice text"); FEAT-24.SPEC-002 renders it verbatim with placeholders filled from these fields and never paraphrases, merges, or abbreviates it | N/A -- not an error, a text-fidelity rule |
| Retention independent of open-item state | record_retention_classification, open_unpaid_invoice_present | An Invoice's retention classification (Retained if it is an Invoice or Payment record) applies regardless of whether that same invoice is also counted as "open" for warning purposes -- the two determinations are computed independently and never override each other | N/A -- not an error, a clarifying independence rule |

## Authorization Rules

This spec's determinations run only as part of FEAT-24.SPEC-002 and FEAT-24.SPEC-004, both of which FEAT-24.SPEC-007 (Export & Deletion Access Rules) already confines to Nadia and her own account. This section states that dependency rather than duplicating FEAT-24.SPEC-007's own role-action matrix.

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Trigger this determination (open-item evaluation, retention classification) | Nadia (Freelancer) | Only via FEAT-24.SPEC-002 (screen load / confirmation) or FEAT-24.SPEC-004 (cascade execution), both already gated to her own account by FEAT-24.SPEC-007 | -- |
| Trigger this determination | Owen (Client Primary Contact), Priya (Client Reviewer Contact), Dana (Support Operator) | Never | This determination never runs on behalf of these roles, since neither FEAT-24.SPEC-002 nor FEAT-24.SPEC-004 is ever reachable by them (FEAT-24.SPEC-007) |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| open_unpaid_invoice_present | True if any Invoice for the account has status Generated, Sent, Payment pending, Overdue, Partially refunded, or Disputed (i.e., not Paid, Paid (recorded by freelancer), Refunded, or Corrected) | On evaluation (screen load and at confirmation) | No -- always derived from current Invoice status |
| pending_approval_present | True if any Milestone for the account has status Deliverable Uploaded (awaiting Owen's approval) | On evaluation (screen load and at confirmation) | No -- always derived from current Milestone status |
| open_invoice_count | Count of Invoices meeting the open_unpaid_invoice_present status set | On evaluation (screen load and at confirmation) | No -- always derived |
| open_invoice_totals | For each currency, sum of the outstanding (unpaid) amount of the counted Invoices; Partially refunded and Disputed Invoices contribute their full invoiced amount less any amount already paid | On evaluation (screen load and at confirmation) | No -- always derived |
| pending_approval_count | Count of Milestones with status Deliverable Uploaded | On evaluation (screen load and at confirmation) | No -- always derived |
| record_retention_classification | Invoice and Payment classify as Retained, regardless of their own status field; every other entity class in feature-overview.md's Entity-Lifecycle Coverage Matrix classifies as Immediately Deleted | On evaluation (at cascade execution) | No -- fixed per entity class, never per individual record |

## Business Rules

- Open items warn without blocking deletion (product-features.md, Primary Flows & Alternates: "a freelancer with active unpaid invoices or pending approvals is warned about the consequences before deletion is finalized, not silently blocked").
- SC-24: financial records (Invoice, Payment) are retained under a legal requirement rather than deleted immediately; every other entity class has no such requirement and is deleted in full at cascade time.
- XBR-33: retention applies specifically to financial records, never to client contacts' personal data or any other entity class -- a Client Contact's name and email are always immediately deleted, with no retention exception, even though the account may separately have retained financial records.
- **Exact warning and notice text (FEAT-24.SPEC-002 renders these verbatim).** Placeholders in braces are filled from this spec's derived fields. Text never names individual invoices, clients, or milestones -- counts and amounts only.
  - **Unpaid-invoice banner** (shown only when open_unpaid_invoice_present is true). Title: "Unpaid invoices". Body when open_invoice_count is 1: "You have 1 unpaid invoice ({open_invoice_totals}). Deleting your account will not collect it, and your client will no longer be able to pay it." Body when open_invoice_count is 2 or more: "You have {open_invoice_count} unpaid invoices ({open_invoice_totals}). Deleting your account will not collect them, and your clients will no longer be able to pay them." When invoices span more than one currency, {open_invoice_totals} lists each currency total separated by " + " (e.g., "$1,200.00 + EUR 300.00"); with one currency it is that single total.
  - **Pending-approval banner** (shown only when pending_approval_present is true). Title: "Deliverables awaiting approval". Body when pending_approval_count is 1: "1 deliverable is waiting for your client's approval. Deleting your account will discard it without a decision, and your client will lose access to it." Body when pending_approval_count is 2 or more: "{pending_approval_count} deliverables are waiting for your clients' approval. Deleting your account will discard them without a decision, and your clients will lose access to them."
  - **Retention notice** (always shown, independent of open items): "Invoices and payment records are kept for {retention_period} after deletion, as legally required, and can no longer be accessed by you or your clients. Everything else, including your clients' names and email addresses, is deleted in full." {retention_period} is the value of platform parameter: `financial-record-legal-retention-period`, rendered in words (e.g., "seven years").
  - Both banners carry informational styling only: neither contains a button, a checkbox, or any text that says deletion is blocked.
- FEAT-13.SPEC-006 applies the same financial-record retention exception to Activity Log Entries that are themselves, or directly support, a financial record (invoice sent, credit note issued, manual payment recorded, refund/reversal/cancellation recorded) -- this spec's retention classification is the shared basis both FEAT-24.SPEC-004 and FEAT-13.SPEC-006 apply to their respective entities.

## Edge Cases

- **An invoice's status changes between screen load and confirmation (e.g., becomes Paid in another tab)** -- FEAT-24.SPEC-002 re-runs this determination at the moment of confirmation (Enforced By table), so the warning and retention picture reflect the state at that moment, not at screen load.
- **The account has zero invoices and zero milestones ever created** -- Both open_unpaid_invoice_present and pending_approval_present evaluate false; no warning is shown, and since no Invoice or Payment records exist, the cascade proceeds with no retained records at all.
- **An Invoice is in a Partially refunded or Disputed status at deletion time** -- Counted as open for warning purposes (money movement is unresolved), and separately classified Retained for the cascade, since both determinations key off it being an Invoice record, independent of its specific status.
- **A Payment record that failed or was never confirmed exists for the account** -- Still classified Retained, since retention is determined by entity class (Payment), not by the record's own status.
- **A Milestone is Reopened rather than Deliverable Uploaded at evaluation time** -- pending_approval_present evaluates false for that milestone, since Reopened is not the awaiting-approval status; if a Deliverable is later re-uploaded on it, the next evaluation reflects the new Deliverable Uploaded status.
- **Both an open invoice and a pending approval exist simultaneously** -- Both warning banners are shown together on FEAT-24.SPEC-002 (Cross-Field Rules), neither suppressing the other; the unpaid-invoice banner is listed first.
- **Invoices in different currencies are open at once** -- {open_invoice_totals} lists one total per currency, joined with " + ", and the banner uses the plural body whenever open_invoice_count is 2 or more, regardless of currency.

## Acceptance Criteria

**FEAT-24.SPEC-006-AC-01:** Given Nadia has an Invoice in Sent status, when this determination runs, then open_unpaid_invoice_present evaluates true and FEAT-24.SPEC-002 shows the unpaid-invoice warning.

**FEAT-24.SPEC-006-AC-02:** Given Nadia has every Invoice in Paid status, when this determination runs, then open_unpaid_invoice_present evaluates false and no unpaid-invoice warning is shown.

**FEAT-24.SPEC-006-AC-03:** Given Nadia has a Milestone in Deliverable Uploaded status, when this determination runs, then pending_approval_present evaluates true and FEAT-24.SPEC-002 shows the pending-approval warning.

**FEAT-24.SPEC-006-AC-04:** Given Nadia has no Milestone in Deliverable Uploaded status, when this determination runs, then pending_approval_present evaluates false and no pending-approval warning is shown.

**FEAT-24.SPEC-006-AC-05:** Given Nadia has both an open invoice and a pending approval, when this determination runs, then both warnings are shown together on FEAT-24.SPEC-002.

**FEAT-24.SPEC-006-AC-06:** Given the cascade in FEAT-24.SPEC-004 reaches an Invoice record, when this determination runs, then it is classified Retained regardless of the invoice's own status field.

**FEAT-24.SPEC-006-AC-07:** Given the cascade reaches a Payment record, when this determination runs, then it is classified Retained regardless of the payment's own status field.

**FEAT-24.SPEC-006-AC-08:** Given the cascade reaches a Client Contact record, when this determination runs, then it is classified Immediately Deleted, with no retention exception for the contact's personal data.

**FEAT-24.SPEC-006-AC-09:** Given an invoice's status changes from Sent to Paid in another tab after Nadia loaded FEAT-24.SPEC-002, when she then confirms deletion, then this determination re-runs and the warning reflects the current Paid status.

**FEAT-24.SPEC-006-AC-10:** Given Nadia's account has zero invoices and zero milestones, when this determination runs, then no warnings are shown and no records are classified Retained.

**FEAT-24.SPEC-006-AC-11:** Given an Invoice is in Partially refunded status, when this determination runs, then it counts as open for warning purposes and is separately classified Retained for the cascade.

**FEAT-24.SPEC-006-AC-12:** Given a Milestone is in Reopened status, when this determination runs, then pending_approval_present does not count it as awaiting approval.

**FEAT-24.SPEC-006-AC-13:** Given Nadia (Freelancer) triggers this determination through FEAT-24.SPEC-002 or FEAT-24.SPEC-004, when it runs, then it evaluates only her own account's records.

**FEAT-24.SPEC-006-AC-14:** Given Owen, Priya, or Dana attempts to reach any spec that would trigger this determination, when they try, then it never runs on their behalf, since FEAT-24.SPEC-007 excludes them from every screen and automation that invokes it.

**FEAT-24.SPEC-006-AC-15:** Given Nadia has exactly 1 open Invoice with an outstanding amount of $1,200.00, when this determination runs, then the unpaid-invoice banner reads title "Unpaid invoices" and body "You have 1 unpaid invoice ($1,200.00). Deleting your account will not collect it, and your client will no longer be able to pay it."

**FEAT-24.SPEC-006-AC-16:** Given Nadia has 3 open Invoices totaling $1,000.00 and EUR 300.00, when this determination runs, then the banner body reads "You have 3 unpaid invoices ($1,000.00 + EUR 300.00). Deleting your account will not collect them, and your clients will no longer be able to pay them."

**FEAT-24.SPEC-006-AC-17:** Given Nadia has 2 Milestones in Deliverable Uploaded status, when this determination runs, then the pending-approval banner reads title "Deliverables awaiting approval" and body "2 deliverables are waiting for your clients' approval. Deleting your account will discard them without a decision, and your clients will lose access to them."

**FEAT-24.SPEC-006-AC-18:** Given any open-item state (including none), when FEAT-24.SPEC-002 loads, then the retention notice reads "Invoices and payment records are kept for {retention_period} after deletion, as legally required, and can no longer be accessed by you or your clients. Everything else, including your clients' names and email addresses, is deleted in full." with {retention_period} filled from platform parameter: `financial-record-legal-retention-period`.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 6 | 6 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 2 | 2 |
| Defaults/Derivations | 6 | 6 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |



# Logic/Rule Spec: Export & Deletion Access Rules

## Overview

**Name:** Export & Deletion Access Rules
**ID:** FEAT-24.SPEC-007
**Type:** Logic/Rule
**Purpose:** Restricts every export and deletion action in this feature to Nadia alone, with no access of any kind for client contacts or the Support Operator.
**Parent Feature:** FEAT-24 -- Data Export & Account Deletion
**Governed Entity:** Data Export Archive and Freelancer Account (export & deletion authority) -- the access boundary applied to every screen and automation in this feature

## Scope and Non-Goals

**In Scope:**
- The single, feature-wide authorization gate applied to every screen and automation in FEAT-24
- The exact denied experience for each role that cannot reach this feature
- The total exclusion of the Support Operator, unlike her View access to almost every other feature

**Non-Goals:**
- Determining what content is shown once access is granted (open-item warnings, archive status, retention classification) -- owned by FEAT-24.SPEC-006 and the individual screen and automation specs; this spec governs only whether the action is reachable at all.
- Any ownership-based restriction within the feature -- excluded because there is exactly one governed account per Nadia's own session; the Access Matrix establishes no finer-grained ownership split within this feature than "her own account."
- Client-side account or portal-level permissions in general -- owned by FEAT-18 (Client Contact Management & Roles); this spec addresses only this one feature's total exclusion of client contacts.
- Operator support-session mechanics generally -- owned by FEAT-31 (Operator Support Access); this spec states only the FEAT-24-specific exclusion that XBR-29 requires of every support session.

## Governed Entity

**Entity:** Data Export Archive and Freelancer Account (export & deletion authority)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| Data Export Archive: requested-at | date | When the current export was requested |
| Data Export Archive: status | enum | Requested, Ready, Downloaded, Expired |
| Data Export Archive: download link/window | derived | Scoped, time-limited download link |
| Freelancer Account: deletion state | enum | Active, Deletion Requested, Deletion Confirmed, Deleted |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-24.SPEC-001 | Data Export Screen | On screen entry (route guard) and on every action (Request Export, Download) |
| FEAT-24.SPEC-002 | Account Deletion Screen | On screen entry (route guard) and on the confirmation action |
| FEAT-24.SPEC-003 | Data Export Archive Generation | On trigger receipt, before any aggregation begins |
| FEAT-24.SPEC-004 | Account Deletion Processing | On trigger receipt, before any cascade step begins |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| Data Export Archive: requested-at | No validation beyond data type -- system-set, not directly editable by any role | Always | -- | -- | -- |
| Data Export Archive: status | No validation beyond data type -- system-managed enum, transitions driven entirely by FEAT-24.SPEC-003 | Always | -- | -- | -- |
| Data Export Archive: download link/window | No validation beyond data type -- system-generated | Always | -- | -- | -- |
| Freelancer Account: deletion state | No validation beyond data type -- system-managed enum, transitions driven entirely by FEAT-24.SPEC-004 | Always | -- | -- | -- |

## Cross-Field Rules

No cross-field rules apply. Both governed fields (Data Export Archive's lifecycle fields and the Freelancer Account's deletion state) are each managed independently by their own automation (FEAT-24.SPEC-003 and FEAT-24.SPEC-004 respectively); this spec governs the authorization boundary around them, not any interaction between them.

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Request a data export | Nadia (Freelancer) | Always, her own account only | -- |
| View export status / download the completed archive | Nadia (Freelancer) | Always, her own account only | -- |
| View the Account Deletion Screen | Nadia (Freelancer) | Always, her own account only | -- |
| Give explicit deletion confirmation | Nadia (Freelancer) | Always, her own account only | -- |
| Request a data export, view export status, download an archive, view the Account Deletion Screen, or give deletion confirmation | Owen (Client Primary Contact) | Never | No navigation path in the client portal reaches any FEAT-24 screen; a direct link shows the client portal's standard out-of-scope explanation (XBR-09), never this feature's content |
| Request a data export, view export status, download an archive, view the Account Deletion Screen, or give deletion confirmation | Priya (Client Reviewer Contact) | Never | Same as Owen -- no client-portal surface exists for this feature |
| Request a data export, view export status, download an archive, view the Account Deletion Screen, give deletion confirmation, or view any output of this feature during a support session | Dana (Support Operator) | Never | Every FEAT-24 screen and automation output is excluded entirely from every read-only support session (XBR-29); a direct link during a session shows "This isn't available during a support session," not a read-only view -- unlike Dana's View access to nearly every other feature |

## Defaults and Derivations

N/A -- this spec governs access authorization only. Default values and derivations for the Data Export Archive and the Freelancer Account's deletion state are owned by FEAT-24.SPEC-003 and FEAT-24.SPEC-004 respectively, which create and transition those fields as part of their own processing logic.

## Business Rules

- XBR-29: the operator's support sessions exclude file downloads and data/accounting exports, and exclude account-lifecycle actions entirely -- this spec is FEAT-24's own enforcement of that exclusion, applied to every screen and automation in the feature rather than left to be inferred from omission.
- product-features.md's Access field: "Nadia only (Full, her own account); no client contact can export or delete the freelancer's account... Dana (Support Operator) has no access: she cannot request an export or delete an account on a freelancer's behalf."
- There is no administrative override anywhere in the product for any party other than Nadia to trigger an export or a deletion on her account.
- This exclusion is total and feature-wide: unlike other features where Dana holds View access and client contacts hold Own-only access to some capability groups, this feature grants neither role any access of any kind (Access Matrix, Subscription & Account Data column, as narrowed by feature-overview.md's Access field).

## Edge Cases

- **Dana attempts to reach a FEAT-24 screen via a direct link while an active support session is open on Nadia's account** -- Denied with "This isn't available during a support session," regardless of how the link was reached (bookmark, typed URL, or a link surfaced elsewhere in the product).
- **Owen or Priya attempts to reach a FEAT-24 screen via a direct link, without any support session involved** -- Denied with the client portal's standard out-of-scope explanation (XBR-09); there is no state in which a client contact reaches this feature's content.
- **Nadia's own session is authenticated but her account happens to be mid-deletion (Deletion Confirmed) when she (implausibly) attempts a fresh export request** -- Not reachable in practice, since FEAT-24.SPEC-002 disables all controls during Processing and Nadia is signed out once deletion completes; this spec's authorization gate is not the mechanism that prevents this scenario, FEAT-24.SPEC-002's own state handling is.
- **A support session is opened on Nadia's account by Dana after Nadia has already started (but not confirmed) an account deletion** -- Dana still has no access to FEAT-24.SPEC-002 or any indication of Nadia's in-progress deletion attempt; the total exclusion applies regardless of what state Nadia's own deletion flow is in.

## Acceptance Criteria

**FEAT-24.SPEC-007-AC-01:** Given Nadia (Freelancer) opens FEAT-24.SPEC-001, when the screen loads, then she can view and act on it in full.

**FEAT-24.SPEC-007-AC-02:** Given Nadia (Freelancer) opens FEAT-24.SPEC-002, when the screen loads, then she can view and act on it in full.

**FEAT-24.SPEC-007-AC-03:** Given Nadia triggers FEAT-24.SPEC-003 by requesting an export, when the trigger is received, then authorization passes since it is her own account.

**FEAT-24.SPEC-007-AC-04:** Given Nadia triggers FEAT-24.SPEC-004 by confirming deletion, when the trigger is received, then authorization passes since it is her own account.

**FEAT-24.SPEC-007-AC-05:** Given Owen (Client Primary Contact) attempts to reach FEAT-24.SPEC-001 via a direct link, when he does, then he sees the client portal's standard out-of-scope explanation, never this screen's content.

**FEAT-24.SPEC-007-AC-06:** Given Priya (Client Reviewer Contact) attempts to reach FEAT-24.SPEC-002 via a direct link, when she does, then she sees the same out-of-scope explanation as Owen.

**FEAT-24.SPEC-007-AC-07:** Given Dana (Support Operator) is in an active read-only support session on Nadia's account, when she attempts to open FEAT-24.SPEC-001 or FEAT-24.SPEC-002 directly, then she sees "This isn't available during a support session."

**FEAT-24.SPEC-007-AC-08:** Given Dana is in an active support session, when Nadia's export or deletion actions occur during that session, then Dana receives no indication of them through her support-session view, since this feature is excluded from it entirely.

**FEAT-24.SPEC-007-AC-09:** Given no client contact has any account-level surface to reach this feature from, when Owen or Priya browse their portal, then no navigation element anywhere leads to FEAT-24.

**FEAT-24.SPEC-007-AC-10:** Given Dana's support session grants View access to nearly every other feature, when she looks for an equivalent read-only view of this feature, then none exists -- her access here is None, not View.

**FEAT-24.SPEC-007-AC-11:** Given Nadia's own account is the only account she can act on, when she requests an export or confirms deletion, then the action is always scoped to her own account and never any other freelancer's.

**FEAT-24.SPEC-007-AC-12:** Given a support session is opened on Nadia's account while she has an in-progress (unconfirmed) deletion attempt, when Dana views her session, then Dana still has no access to any FEAT-24 screen or indication of that attempt.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 1 (N/A, stated) | 1 |
| Authorization Rules | 7 | 7 |
| Defaults/Derivations | 1 (N/A, stated) | 1 |
| Business Rules | 4 | 4 |
| Edge Cases | 4 | 4 |



# Notification Spec: Export Ready Notification

## Overview

**Name:** Export Ready Notification
**ID:** FEAT-24.SPEC-008
**Type:** Notification
**Purpose:** Emails Nadia when her requested data export archive is ready to download, so she learns about it even if she is away from the product while it generates.
**Parent Feature:** FEAT-24 -- Data Export & Account Deletion

## Scope and Non-Goals

**In Scope:**
- The confirmation email sent when the archive reaches Ready
- Delivery, retry, and expiry behavior for this email

**Non-Goals:**
- Deciding when the archive reaches Ready -- owned entirely by FEAT-24.SPEC-003 (Data Export Archive Generation); this spec begins where that automation's trigger fires.
- Notifying anyone about a generation failure -- product-features.md's Communications field for this feature names only the ready-to-download confirmation as triggering email; a failed generation is surfaced as an in-screen error on FEAT-24.SPEC-001, not by a separate email.
- Notifying Owen, Priya, or Dana -- none of these roles has any access to this entity (FEAT-24.SPEC-007: None for all three); this email carries content about Nadia's own data export that no one else is entitled to know exists.
- In-app notification center delivery -- product-features.md's Communications field names only the email channel for this feature; an in-app surface is owned separately by In-App Notification Center (FEAT-29, Later phase).

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, to Nadia, when the archive reaches Ready | Nadia works from a laptop or desktop throughout her day and is not necessarily inside the product at the moment a large archive finishes generating (user-persona.md, Behavioral Context); since the archive expires after a limited download window, email is the channel that ensures she learns it exists in time to retrieve it |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Archive becomes Ready | FEAT-24.SPEC-003 (Data Export Archive Generation) | Fires when FEAT-24.SPEC-003 applies the Ready outcome to Nadia's Data Export Archive | Freelancer Account name and sign-in email; the archive's download link/window expiry |

## Audience and Preferences

**Recipients:** Nadia (the Freelancer) only. FEAT-24.SPEC-007 confines every action and every piece of content in this feature to her; Owen, Priya, and Dana have no access to this entity at all.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| N/A -- always sent | -- | Always on | -- (this is a transactional email tied to a data-readiness event Nadia herself requested, not an optional notification) |

Notification preferences (FEAT-21.SPEC-002) can switch off only optional notifications; this email confirms the outcome of an action Nadia deliberately took (requesting her own export) and has no opt-out, consistent with XBR-30's treatment of record-core transactional email.

**Quiet Hours:** N/A -- this email is transactional and exempt from quiet hours (XBR-30 defines quiet hours only for optional, non-transactional notifications); it sends as soon as the archive reaches Ready regardless of the hour, since the download window begins counting from that moment.

## Content Definition

**Email:**
- **Subject:** Your data export is ready to download
- **Body:**
  Hi {nadia_first_name},

  Your requested data export is ready. Download it before {expiry_date} -- after that, you'll need to request a new export.
- **CTA (button):** Download your export -- deep-links to FEAT-24.SPEC-001 (Data Export Screen)

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {nadia_first_name} | Freelancer Account -- name (first name portion) | Nadia | Greeting renders as "Hi," |
| {expiry_date} | Data Export Archive -- download link/window (the expiry timestamp set when FEAT-24.SPEC-003 applies the Ready outcome) | October 4, 2026 | Never empty -- FEAT-24.SPEC-003 always sets the download window at the same moment it applies the Ready outcome that triggers this email |

## Delivery Rules

**Batching:** None -- at most one active archive exists per account (FEAT-24.SPEC-003), so there is never more than one pending instance of this email to batch.
**Deduplication:** At most one email per FEAT-24.SPEC-003 Ready application. A new request that supersedes the current archive (FEAT-24.SPEC-003's own discard-and-recreate step) cancels any pending email for the superseded archive; only the newest archive's Ready event produces a delivered email.
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001).
**Expiry:** If every retry fails and the archive has since reached its own Expired state (per FEAT-24.SPEC-003) before the email would be attempted again, the email is not sent at all -- an email urging Nadia to download an archive that no longer exists would be actively misleading. If the archive is still Ready or Downloaded when a delayed retry succeeds, the email still sends with the archive's actual current expiry date.

## Edge Cases

- **The archive expires before this email is delivered (all retries exhausted)** -- Per the Expiry rule above, the email is not sent; Nadia simply sees the Expired state directly on FEAT-24.SPEC-001 the next time she visits.
- **Nadia requests a new export before the email for the prior archive has been delivered** -- The pending email for the superseded archive is cancelled, since it would reference an archive that FEAT-24.SPEC-003 has already discarded; only the new request's eventual Ready event produces an email.
- **Nadia's account is deleted (FEAT-24.SPEC-004) while this email is still queued for retry** -- The pending delivery is cancelled once account deletion finalizes and removes her Freelancer Account and its Notifications, per the dependency map's Notification lifecycle (Deleted by FEAT-24).
- **Quiet hours colliding with expiry** -- N/A, since this email is transactional and exempt from quiet hours, so there is no quiet-hours hold to collide with the expiry cutoff.
- **Nadia downloads the archive before this email is even delivered (a fast retry succeeds after she has already returned to the product on her own)** -- The email still sends; it accurately confirms the export became ready and remains useful as a record of the event, even though she no longer needs its CTA to find it.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-24.SPEC-003 (Data Export Archive Generation) | Triggered by (inbound) | The Ready outcome fires this notification |
| FEAT-24.SPEC-001 (Data Export Screen) | Navigation (outbound) | The CTA deep-links here |
| FEAT-24.SPEC-007 (Export & Deletion Access Rules) | References (inbound) | Confines the recipient to Nadia alone |
| FEAT-14.SPEC-001 (Transactional Email Delivery) | References (outbound) | Underlying delivery, retry, and bounce/failure reporting capability this notification is sent through |

## Analytics and Success Signals

- **data_export_ready_email_delivered** () -- N/A -- no success-metrics.md metric is connected to Data Export & Account Deletion; retained because product-features.md's Signals field names "data_export_ready" explicitly as this feature's defined signal.
- **data_export_ready_email_delivery_failed** (retry_count) -- N/A -- same reason; retained as a standard delivery-quality signal so a silently undelivered export-ready email stays observable.

## Acceptance Criteria

**FEAT-24.SPEC-008-AC-01:** Given FEAT-24.SPEC-003 applies a Ready outcome to Nadia's archive, when this notification fires, then she receives an email with subject "Your data export is ready to download."

**FEAT-24.SPEC-008-AC-02:** Given Nadia opens the export-ready email, when she taps "Download your export," then she lands on FEAT-24.SPEC-001 (Data Export Screen).

**FEAT-24.SPEC-008-AC-03:** Given Nadia has no way to opt out of this email, when her notification preferences are checked, then no preference control exists for it and it always sends.

**FEAT-24.SPEC-008-AC-04:** Given the archive reaches Ready at any hour, when this notification fires, then it sends immediately regardless of Nadia's configured quiet hours.

**FEAT-24.SPEC-008-AC-05:** Given delivery fails on the first attempt, when the retry logic runs, then it retries up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`.

**FEAT-24.SPEC-008-AC-06:** Given every retry fails and the archive has since expired, when the final retry would otherwise be attempted, then the email is not sent.

**FEAT-24.SPEC-008-AC-07:** Given Nadia requests a new export before the prior archive's email is delivered, when the new request supersedes the prior archive, then the pending email for the superseded archive is cancelled.

**FEAT-24.SPEC-008-AC-08:** Given Nadia's account is deleted while this email is still queued for retry, when the deletion finalizes, then the pending delivery is cancelled.

**FEAT-24.SPEC-008-AC-09:** Given Owen, Priya, or Dana has no access to this entity, when the archive reaches Ready, then none of them receives any copy of this email.

**FEAT-24.SPEC-008-AC-10:** Given a delayed retry succeeds while the archive is still Ready, when the email is finally delivered, then it shows the archive's actual current expiry date.

**FEAT-24.SPEC-008-AC-11:** Given Nadia has already downloaded the archive by the time a delayed retry succeeds, when the email is delivered, then it still sends as an accurate record of the export becoming ready.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on -- no preference exists) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |



# Notification Spec: Account Deletion Final Warning Notification

## Overview

**Name:** Account Deletion Final Warning Notification
**ID:** FEAT-24.SPEC-009
**Type:** Notification
**Purpose:** Emails Nadia a final confirmation notice at the moment she confirms permanent account deletion, giving her a durable record of exactly when she gave that irreversible confirmation.
**Parent Feature:** FEAT-24 -- Data Export & Account Deletion

## Scope and Non-Goals

**In Scope:**
- The final-warning email sent at the moment Nadia's explicit deletion confirmation is given
- Delivery and retry behavior for this email

**Non-Goals:**
- Capturing the confirmation itself -- owned by FEAT-24.SPEC-002 (Account Deletion Screen); this spec begins where that screen's confirmation action fires.
- Executing the deletion -- owned by FEAT-24.SPEC-004 (Account Deletion Processing); this email fires in parallel with that automation and does not gate or wait for it.
- Offering any way to cancel or undo the deletion from this email -- excluded per product-features.md's Validation & Limits field ("irreversibility") and feature-overview.md's Non-Goals ("no undo after explicit confirmation"); this email carries no CTA, since the product deliberately offers no path to reverse a confirmation Nadia has already deliberately given.
- Notifying Owen, Priya, or Dana -- none of these roles has any access to this entity (FEAT-24.SPEC-007: None for all three); this email is Nadia's own record of her own action.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, to Nadia, the moment her explicit deletion confirmation is given | This is the single most consequential, irreversible action available in the product; email gives Nadia a durable, external record of exactly when she confirmed, independent of the product itself, which will shortly remove her access entirely |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nadia gives explicit confirmation | FEAT-24.SPEC-002 (Account Deletion Screen) | Fires the moment the acknowledgment checkbox is checked and the typed confirmation exactly matches "DELETE" and Delete My Account is tapped, in parallel with FEAT-24.SPEC-004 beginning | Freelancer Account name and sign-in email; the confirmation timestamp |

## Audience and Preferences

**Recipients:** Nadia (the Freelancer) only. FEAT-24.SPEC-007 confines every action and every piece of content in this feature to her; Owen, Priya, and Dana have no access to this entity at all.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| N/A -- always sent | -- | Always on | -- (this is a transactional, record-of-the-fact email about the single most consequential action in the product; it has no opt-out) |

This email is never optional, consistent with XBR-30's treatment of transactional record email -- a confirmation of permanent, irreversible account deletion is not a notification Nadia can choose not to receive a record of.

**Quiet Hours:** N/A -- this email is transactional and exempt from quiet hours (XBR-30); it sends the instant confirmation is given, regardless of the hour, since the deletion it records begins processing immediately.

## Content Definition

**Email:**
- **Subject:** Your Clientroom account is being permanently deleted
- **Body:**
  Hi {nadia_first_name},

  You confirmed permanent deletion of your Clientroom account and all its data on {confirmation_date}. This action cannot be undone.
- **CTA:** None -- feature-overview.md's Non-Goals establish no undo or restore path once confirmation is given; this email's purpose is to give Nadia a clear, permanent record of exactly when she confirmed, not to offer a way to stop or reverse what she deliberately confirmed.

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {nadia_first_name} | Freelancer Account -- name (first name portion) | Nadia | Greeting renders as "Hi," |
| {confirmation_date} | Derived -- the timestamp FEAT-24.SPEC-002 captures when Delete My Account is tapped with a valid confirmation | October 4, 2026, 3:14 PM | Never empty -- this notification's own trigger condition requires a captured confirmation timestamp to exist |

## Delivery Rules

**Batching:** None -- a Freelancer Account can be confirmed for deletion at most once (FEAT-24.SPEC-004 treats any further attempt as no-action or as a discarded redundant trigger), so there is never more than one instance of this email per account.
**Deduplication:** At most one email per confirmed-deletion event. FEAT-24.SPEC-004's own concurrency handling (only one confirmed instruction ever proceeds past its trigger check) is the deduplication boundary -- a second, redundant confirmation attempt for the same account never produces a second email.
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001).
**Expiry:** This email does not expire in the sense of becoming pointless to send late: it is a record of a moment that already happened and remains true regardless of delay. Unlike other transactional emails in this product, a final failure after all retries has no screen to surface a delivery warning on, since the account this email is about will, by that point, likely already be deleted (FEAT-24.SPEC-004 proceeds regardless of this email's own delivery outcome) -- delivery failure here is simply accepted as unrecoverable rather than retried indefinitely or surfaced anywhere.

## Edge Cases

- **FEAT-24.SPEC-004's cascade fails and reverts the account to Active after this email has already been sent** -- The email already sent is not retracted or followed by a correction; it accurately reported the confirmation Nadia gave at that moment. She sees the reverted-account error directly on FEAT-24.SPEC-002, and no separate "never mind" email is sent, since a second, contradicting email would confuse rather than clarify, and the account remains intact for her to try again.
- **Every retry fails and the account has since been fully deleted** -- No further action is taken; unlike other transactional emails in this product, there is no ongoing screen or account state left to surface the delivery failure on, since deletion proceeds regardless of whether this email itself was ever delivered.
- **Two deletion confirmations for the same account are submitted close together (a double-tap or two open tabs)** -- Only the first reaching FEAT-24.SPEC-004 produces this email, mirroring that automation's own concurrency handling; the second is discarded before it would trigger a duplicate.
- **Quiet hours colliding with expiry** -- N/A, since this email is transactional and exempt from quiet hours, and has no expiry cutoff to collide with in the first place.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-24.SPEC-002 (Account Deletion Screen) | Triggered by (inbound) | The confirmation action fires this notification |
| FEAT-24.SPEC-004 (Account Deletion Processing) | References (outbound) | The cascade this email precedes and runs in parallel with, without gating it |
| FEAT-24.SPEC-007 (Export & Deletion Access Rules) | References (inbound) | Confines the recipient to Nadia alone |
| FEAT-14.SPEC-001 (Transactional Email Delivery) | References (outbound) | Underlying delivery and retry capability this notification is sent through |

## Analytics and Success Signals

- **account_deletion_final_warning_email_delivered** () -- N/A -- no success-metrics.md metric is connected to Data Export & Account Deletion; retained as the audit-relevant record that Nadia's final warning was actually delivered before her account's irreversible removal completed.
- **account_deletion_final_warning_email_delivery_failed** (retry_count) -- N/A -- same reason; retained so an undelivered final warning stays observable even though nothing further can be done about it once the account is gone.

## Acceptance Criteria

**FEAT-24.SPEC-009-AC-01:** Given Nadia gives explicit confirmation on FEAT-24.SPEC-002, when this notification fires, then she receives an email with subject "Your Clientroom account is being permanently deleted" carrying the exact confirmation timestamp.

**FEAT-24.SPEC-009-AC-02:** Given Nadia opens this email, when she looks for a way to undo or cancel the deletion, then no CTA or link of any kind offers one.

**FEAT-24.SPEC-009-AC-03:** Given Nadia has no way to opt out of this email, when her notification preferences are checked, then no preference control exists for it and it always sends.

**FEAT-24.SPEC-009-AC-04:** Given confirmation is given at any hour, when this notification fires, then it sends immediately regardless of Nadia's configured quiet hours.

**FEAT-24.SPEC-009-AC-05:** Given delivery fails on the first attempt, when the retry logic runs, then it retries up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`.

**FEAT-24.SPEC-009-AC-06:** Given every retry fails and the account has since been fully deleted, when the final retry would otherwise be attempted, then no further action is taken and no delivery warning is surfaced anywhere.

**FEAT-24.SPEC-009-AC-07:** Given this email has already been sent, when FEAT-24.SPEC-004's cascade subsequently fails and reverts the account to Active, then no correction or "never mind" email is sent.

**FEAT-24.SPEC-009-AC-08:** Given two deletion confirmations for the same account are submitted close together, when both reach FEAT-24.SPEC-004, then only the first produces this email.

**FEAT-24.SPEC-009-AC-09:** Given Owen, Priya, or Dana has no access to this entity, when Nadia confirms deletion, then none of them receives any copy of this email.

**FEAT-24.SPEC-009-AC-10:** Given this email is delivered, when Nadia reads it, then it names the exact date and time she confirmed, with no ambiguity about which action it refers to.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on -- no preference exists) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 4 | 4 |

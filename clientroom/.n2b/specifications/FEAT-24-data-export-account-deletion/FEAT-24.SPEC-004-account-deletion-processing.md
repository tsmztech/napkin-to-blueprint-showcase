---
document_type: spec
spec_type: automation
spec_id: FEAT-24.SPEC-004
spec_name: Account Deletion Processing
spec_slug: account-deletion-processing
parent_feature: FEAT-24
parent_feature_name: Data Export & Account Deletion
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 18
---

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

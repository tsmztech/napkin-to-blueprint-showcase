---
document_type: spec
spec_type: automation
spec_id: FEAT-18.SPEC-009
spec_name: Contact Removal & Data Erasure
spec_slug: contact-removal-data-erasure
parent_feature: FEAT-18
parent_feature_name: Client Contact Management & Roles
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Automation Spec: Contact Removal & Data Erasure

## Overview

**Name:** Contact Removal & Data Erasure
**ID:** FEAT-18.SPEC-009
**Type:** Automation
**Purpose:** Ends a removed contact's access immediately, erases their email address and other contact details, and retains their evidentiary records under their name only.
**Parent Feature:** FEAT-18 -- Client Contact Management & Roles

## Scope and Non-Goals

**In Scope:**
- Setting the contact's status to Removed and erasing their email address and every other contact detail on the record, while the retained row keeps the contact's name as the actor label for evidence
- Ending the contact's ability to sign in or act in the client portal, immediately
- Preserving every evidentiary record (acceptances, approvals, comments, payments) the contact created, unaltered, still referencing the retained Client Contact row and displaying the contact's name only
- Writing the removal event to the activity trail

**Non-Goals:**
- The confirmation dialog and last-Primary block shown before this automation fires -- owned by FEAT-18.SPEC-003 (Remove Client Contact) and FEAT-18.SPEC-006 (Primary Contact Requirement Rule); this automation begins only once a removal has already been confirmed and cleared
- Restoring a removed contact -- excluded per the feature's Entity-Lifecycle Coverage Matrix: because this automation's erasure is the mechanism that satisfies the erasure guarantee (XBR-27), the original personal details are genuinely gone and cannot be restored; a returning contact is entered as a brand-new Client Contact record
- Purging the contact's evidentiary records (acceptances, approvals) -- excluded per scope-boundaries.md (SC-24) and ASMP-25's record-immutability guarantee; those records are retained permanently under the account's own retention terms, with no automatic purge window except where FEAT-24 account deletion applies
- Notifying the removed contact that they were removed -- product-features.md's Communications field for this feature names only the invitation email (FEAT-18.SPEC-010) and the counterpart alert (FEAT-18.SPEC-011); no removal notice to the removed party is defined, consistent with the immediate access-revocation intent

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Removal confirmed | FEAT-18.SPEC-003 (Remove Client Contact) | Fires once Nadia confirms removal and the commit-time last-Primary re-check (FEAT-18.SPEC-006) passes | Client Contact reference, its current name, email, role, and status |

## Processing Logic

1. Receive the confirmed removal request for one Client Contact record from FEAT-18.SPEC-003, after the last-Primary re-check has already passed.
2. Re-verify the contact's current status is not already Removed (guards against a duplicate trigger; see Edge Cases).
3. Set the contact's status to Removed.
4. Erase the contact's email field and every other contact detail on the record (for example last sign-in and any sign-in identifiers) -- overwrite each with the product's standard erased-field representation, so no trace of the original values remains. The name field is the one field deliberately kept: it is the label under which the contact's evidence stays attributed (XBR-27).
5. Leave every reference to this contact -- as the actor on their own past acceptances (FEAT-03), approvals (FEAT-08), comments (FEAT-07), and payments (FEAT-10) -- unchanged. One model applies: those evidentiary records reference the retained Client Contact row (status Removed) and show the contact's name only. Their email address is not shown anywhere in retained evidence, because it no longer exists on the row and was never copied into it.
6. Immediately revoke the contact's ability to sign in or act: any currently open client-portal session for this contact is ended, and any future magic-link sign-in attempt for the erased email no longer resolves to a recognized contact (FEAT-05, XBR-28).
7. Write an append-only Activity Log Entry recording the removal event, referencing the retained Client Contact row and carrying the contact's name only -- never the email address (FEAT-13, XBR-05). The trail write does not gate the outcome; see Edge Cases.
8. Return control to FEAT-18.SPEC-003 with the completed outcome.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Removal completed | The contact was not already Removed, and the last-Primary re-check already passed before this automation fired | status set to Removed; email and other contact details erased (name retained as evidence label); access revoked immediately; one Activity Log Entry queued for write (retried independently if it fails) | Nadia sees "Contact removed" and returns to FEAT-18.SPEC-001, where the contact no longer appears | FEAT-18.SPEC-001, FEAT-18.SPEC-003, FEAT-05, FEAT-13 |
| Already removed (idempotent no-op) | The contact's status was already Removed when this automation received the trigger (e.g., a rapid duplicate submission) | None -- no further change is made | Nadia sees the same "Contact removed" success outcome, since the requested end state already holds | FEAT-18.SPEC-001, FEAT-18.SPEC-003 |
| Removal failure | The automation cannot complete the status change or field erasure for a processing reason | No partial change is left in place -- the contact's status, name, and email remain exactly as they were before the attempt (a trail-write failure alone is not a removal failure; see Edge Cases) | Nadia sees the error state on FEAT-18.SPEC-003: "Couldn't remove this contact. Try again." with a Retry button | FEAT-18.SPEC-003 |

## Data Model

**Reads:** Client Contact -- current status, name, email, role, for the contact being removed.
**Creates:** Activity Log Entry -- event_type "contact removed," actor (Nadia), occurred_at, affected_record (the Client Contact), with the contact's name only; no email address is captured (FEAT-13 responsibility, referenced here as the trail-write outcome).
**Updates:** Client Contact -- status set to Removed; email and all other contact details erased; name retained.
**Deletes:** None -- this automation erases specific fields on the existing record and never deletes the Client Contact row itself, since the row must remain referenceable as the actor on past evidentiary records (dependency map, Client Contact -- Delete/Archive: "status set to Removed rather than the row being physically dropped").

## Business Rules

- XBR-27: a contact's erasure request ends their access immediately and removes their contact details, while acceptances and approvals they gave remain on the record under their name. Only the name persists in retained evidence and in the trail entry; no email or other contact detail persists anywhere (feature-dependency-map.md, XBR-27 Authority: FEAT-18).
- Erasure applies identically whether the removal's trigger was a routine client-side departure or an explicit data-subject erasure request (feature-overview.md, Key Capabilities); this automation makes no distinction between the two once FEAT-18.SPEC-003 hands off a confirmed removal.
- Access revocation and field erasure happen together, in the same automation run -- there is no intermediate state where access is revoked but personal details remain, or vice versa.
- No cascade to the Client or Project records -- removing a contact never affects the client company's own record, its projects, or any other contact at that company (dependency map, Client Contact -- Delete/Archive: "No cascade to Client or Project").
- This automation is non-reversible by design: it has no undo path, consistent with the product's decision that a removed contact is re-added as a new record rather than restored.

## Edge Cases

- **The contact is currently signed in to the client portal when this automation runs** -- Their open session is ended immediately; any in-flight action they attempt after this automation completes is rejected by FEAT-05's access rules, since the contact record they authenticated against no longer resolves to a recognized, non-Removed contact.
- **The contact's email bounces or the erased email is later reused by a different person entirely** -- No conflict: an erased email is available for reuse in the per-client uniqueness check (FEAT-18.SPEC-005), since the Removed contact's email field no longer holds a live value to collide with.
- **A concurrent read of this contact's row is in progress on FEAT-18.SPEC-001 when this automation commits** -- The list screen is a snapshot (per FEAT-18.SPEC-001); the removed contact continues to display until the list is next reloaded, at which point it no longer appears, since the underlying record's status is now Removed.
- **Concurrent trigger firing (two removal confirmations for the same contact submitted in quick succession, e.g., a rapid duplicate tap that reached FEAT-18.SPEC-003 twice)** -- The first to commit produces the Removal completed outcome; the second finds the contact already Removed and produces the idempotent Already removed outcome, with no duplicate Activity Log Entry written and no error shown to Nadia.
- **A trigger fires while a previous run for the same contact is still in flight** -- FEAT-18.SPEC-003's Save-equivalent action is disabled while a removal is in progress (mirroring the pattern used for saves elsewhere in this feature), so a second run for the same record cannot start until the first completes; runs for different contacts proceed independently.
- **This automation's Activity Log Entry write fails even though the status change and erasure succeeded** -- Per XBR-05 and FEAT-13's own write-retry behavior, the trail write is retried independently of this automation's own outcome; the removal itself is not rolled back or held pending the trail write, since access revocation must never wait on a secondary record. This is not a Removal failure outcome: Nadia sees success.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-18.SPEC-003 (Remove Client Contact) | Triggered by (inbound) | Confirmed removal fires this automation |
| FEAT-18.SPEC-006 (Primary Contact Requirement Rule) | References (inbound) | The last-Primary re-check that must already have passed before this automation is triggered |
| FEAT-18.SPEC-001 (Client Contact List) | Affects (outbound) | The removed contact no longer appears once this automation completes |
| FEAT-05 (Client Portal Access) | Affects (outbound) | Access revocation ends any open session and blocks future sign-in for the erased contact |
| FEAT-13 (Immutable Activity & Audit Trail) | Affects (outbound) | Writes the append-only removal entry (contact name only) |
| FEAT-03 (Proposal Acceptance), FEAT-08 (Milestone Approval), FEAT-07 (Deliverable Review & Feedback), FEAT-10 (Invoice Payment Processing) | References (outbound) | Each feature's own evidentiary records continue to reference the retained contact row and display the contact's name only, unaffected by this automation |
| FEAT-24 (Data Export & Account Deletion) | References (outbound) | Full account deletion removes remaining client contacts' personal data through its own process, subject to the same evidence-retention boundary this automation establishes |

## Analytics and Success Signals

- **contact_removed** (role: primary / reviewer, trigger_context: routine / erasure_request) -- N/A -- no success-metrics.md metric is connected to Client Contact Management & Roles; retained so removal volume and its erasure-versus-routine mix are observable
- **contact_removal_idempotent_noop** (-- ) -- N/A -- no success-metrics.md metric is connected to this feature; retained so duplicate-submission handling is observable rather than silent
- **contact_removal_failed** (reason: processing_error) -- N/A -- no success-metrics.md metric is connected to this feature

## Acceptance Criteria

**FEAT-18.SPEC-009-AC-01:** Given Nadia confirms removing a Reviewer contact and the trigger fires, when this automation runs, then the contact's status is set to Removed, their email and other contact details are erased while their name is retained, and an Activity Log Entry carrying the name only is written.

**FEAT-18.SPEC-009-AC-02:** Given the removed contact previously accepted a proposal, when their acceptance record is viewed afterward, then it still shows their name as the accepting party via the retained Client Contact row, shows no email address for them, and is otherwise unaffected by the erasure.

**FEAT-18.SPEC-009-AC-03:** Given the removed contact is currently signed in to the client portal when this automation runs, when they next attempt any action, then it is rejected because their contact record no longer resolves as recognized.

**FEAT-18.SPEC-009-AC-04:** Given this automation completes successfully, when FEAT-18.SPEC-001 is next reloaded, then the removed contact no longer appears in the list.

**FEAT-18.SPEC-009-AC-05:** Given the removed contact's erased email is later entered for a new contact at the same client, when the uniqueness check runs (FEAT-18.SPEC-005), then no conflict is found.

**FEAT-18.SPEC-009-AC-06:** Given a removal is triggered a second time for a contact already Removed (a duplicate rapid submission), when this automation receives the second trigger, then it makes no further change and produces the same success outcome shown for the first.

**FEAT-18.SPEC-009-AC-07:** Given this automation cannot complete the status change or field erasure for a processing reason, when the failure occurs, then no partial change is left in place, and Nadia sees "Couldn't remove this contact. Try again." with a Retry button.

**FEAT-18.SPEC-009-AC-08:** Given a client's other contacts and its own Client and Project records, when a contact is removed, then none of those other records are affected.

**FEAT-18.SPEC-009-AC-09:** Given the removed contact left comments on a deliverable before removal, when those comments are viewed afterward, then they remain visible and attributed to the contact's name, with no email address shown.

**FEAT-18.SPEC-009-AC-10:** Given the removal's Activity Log Entry write fails independently of the status change and erasure succeeding, when this occurs, then the removal itself is not rolled back, Nadia still sees "Contact removed", and the trail write is retried per FEAT-13's own retry behavior.

**FEAT-18.SPEC-009-AC-11:** Given two removal confirmations for the same contact fire in quick succession, when both reach this automation, then the first produces the Removal completed outcome and the second produces the idempotent Already removed outcome, with no duplicate trail entry.

**FEAT-18.SPEC-009-AC-12:** Given a removal is triggered by an explicit data-subject erasure request rather than a routine departure, when this automation runs, then it behaves identically to the routine case -- erasing the email and other contact details and preserving evidentiary records the same way.

**FEAT-18.SPEC-009-AC-13:** Given a removal is in progress for one contact, when a second, unrelated removal is triggered for a different contact at the same client at the same time, then both proceed independently without queuing behind each other.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 (completed, idempotent no-op, failure -- trail-write failure handled as an independent retry edge case) | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

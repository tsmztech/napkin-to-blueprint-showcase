---
document_type: spec
spec_type: automation
spec_id: FEAT-13.SPEC-004
spec_name: Client Deletion Execution
spec_slug: client-deletion-execution
parent_feature: FEAT-13
parent_feature_name: Client Record Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Automation Spec: Client Deletion Execution

## Overview

**Name:** Client Deletion Execution
**ID:** FEAT-13.SPEC-004
**Type:** Automation
**Purpose:** Hard-deletes the client's contact details and private note, cascades to remove Messaging Consent, and retains de-identified financial and timeline history.
**Parent Feature:** FEAT-13 -- Client Record Management

## Scope and Non-Goals

**In Scope:**
- Re-validating deletion eligibility at the moment of execution (not just at screen load)
- Permanently removing the Client record's name, phone, email, and private_note
- Cascade-deleting the client's Messaging Consent record(s)
- De-identifying the client's Booking, Deposit Transaction, and Activity Event history rather than deleting it
- Logging the client_record_deleted event

**Non-Goals:**
- Determining eligibility (the upcoming-booking block) -- governed by FEAT-13.SPEC-006 (Deletion Eligibility & Retention Rule); this automation re-checks but does not define that rule
- Collecting the Pro's confirmation -- handled by FEAT-13.SPEC-003 (Client Deletion Confirmation); this automation only executes once confirmation has been given
- Cancelling any booking -- if an upcoming booking exists, this automation refuses rather than cancelling on the Pro's behalf; cancellation is a separate, explicit action through FEAT-30 (Pro Booking Management)
- Deleting Booking, Deposit Transaction, or Activity Event records outright -- excluded per scope-boundaries.md SC-22: full history is retained for as long as the account exists, with only de-identification (not deletion) applied to a deleted client's records

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Pro confirms deletion | FEAT-13.SPEC-003 (Client Deletion Confirmation) | Fires when the Pro taps "Delete client" with the confirmation checkbox checked | Client ID; the eligibility result the confirmation screen last displayed |

## Processing Logic

1. Receive the client ID from the triggering confirmation screen, and re-check delete-access per FEAT-13.SPEC-005 (the requester is the Pro who owns this client); if not, refuse and change nothing.
2. Re-run the deletion eligibility check (FEAT-13.SPEC-006): read the client's bookings and determine whether any upcoming booking exists.
3. If an upcoming booking now exists (created or confirmed after the confirmation screen's own check), stop processing and return the Blocked outcome -- no data is changed.
4. If no upcoming booking exists, proceed:
   a. Permanently remove the Client record's name, phone, email, and private_note fields.
   b. Cascade-delete the client's Messaging Consent record(s) for every channel, except where evidence of a previously granted or revoked consent must be retained under applicable law (per FEAT-13.SPEC-006's retention rule) -- in that case, retain only the minimum evidence required, stripped of the phone number and any other identifying field not itself the required evidence.
   c. Mark the client's Booking, Deposit Transaction, and Activity Event records as de-identified: replace the client reference with a de-identified marker, leaving service, timing, amount, and outcome fields intact for dispute/audit purposes (per scope-boundaries.md SC-22 and XBR-19).
   d. Log the client_record_deleted event.
5. Signal the triggering screen (FEAT-13.SPEC-003) that deletion completed successfully.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Deletion succeeded | No upcoming booking at execution time | Client contact details and private_note permanently removed; Messaging Consent cascade-deleted (except retained legal-evidence remnants); Booking, Deposit Transaction, and Activity Event records de-identified but retained | Toast "Client deleted"; Pro returned to the screen they arrived at before opening the client record | FEAT-13.SPEC-003 (triggering screen), FEAT-12 / FEAT-24 (destination screens, which no longer list this client) |
| Blocked at execution (booking appeared since the confirmation screen's check) | An upcoming booking now exists that did not exist (or was not confirmed) when FEAT-13.SPEC-003 last checked | None -- no data changed | FEAT-13.SPEC-003 shows the Blocked panel with the newly discovered booking's date, time, and service, per its own refresh behavior | FEAT-13.SPEC-003 |
| Deletion failed (processing error) | The write fails partway or the automation cannot complete | None -- the operation is all-or-nothing; no partial deletion is left behind | FEAT-13.SPEC-003 shows "Could not delete this client. Check your connection and try again." with a Retry option | FEAT-13.SPEC-003 |

## Data Model

**Reads:** Client record (for the ID and current field values); Booking (to re-check eligibility and to identify records needing de-identification); Messaging Consent (to identify records to cascade-delete or retain as evidence).
**Creates:** None.
**Updates:** Booking, Deposit Transaction, and Activity Event records -- client reference replaced with a de-identified marker; all other fields (service, timing, amount, outcome) unchanged.
**Deletes:** Client record -- name, phone, email, private_note permanently removed (the record itself is retired, per the dependency map's Client entity Lifecycle: "Deleted by FEAT-13"). Messaging Consent -- cascade-deleted, except any minimum evidence retained where law requires (per FEAT-13.SPEC-006).

## Business Rules

- XBR-19: deletion is refused while an upcoming booking exists; removes contact details, notes, and consent; retains financial and timeline records only in de-identified form; a later booking creates a new, unconnected Client record.
- Delete-access is re-enforced at execution time per FEAT-13.SPEC-005 (Client Field Validation & Access Rules): only the Pro who owns the client may execute; a request from any other role or Pro is refused and changes nothing.
- This automation is all-or-nothing: it either completes every step in Processing Logic or changes nothing, so a client record is never left in a partially deleted state.
- Deletion is irreversible; no restore or undo path exists for the Client entity itself, per the Brief's Non-Goals.
- A later booking from the same person (matched or not to the deleted client's former identity) creates a new, unconnected Client record -- this automation never resurrects the deleted record, per the dependency map's Contention note for the Client entity.

## Edge Cases

- **The client has no Messaging Consent record at all (e.g., they always declined texts)** -- Step 4b has nothing to cascade-delete; processing continues normally to step 4c.
- **The client has zero Booking history (a data inconsistency scenario)** -- Step 4c has nothing to de-identify; processing continues normally, and client_record_deleted is still logged.
- **A card-issuer dispute is open on one of this client's past Deposit Transactions at the moment of deletion** -- The Deposit Transaction is de-identified like any other retained record; its Disputed status and evidence remain intact and usable for the dispute process, per XBR-22, with only the client reference removed.
- **Concurrent trigger firing (the Pro somehow confirms deletion for the same client from two devices at effectively the same time)** -- The first confirmed run to reach step 4 completes the deletion; the second run's re-validation at step 2 finds the client record already gone and returns a failure equivalent to "This client's record no longer exists," surfaced by FEAT-13.SPEC-003 as its own concurrent-edit refusal.
- **A trigger fires while a previous run for the same client is still in flight** -- A second confirmation attempt for the same client cannot start while the first is executing: FEAT-13.SPEC-003's "Delete client" button is disabled and the screen non-interactive during the Deleting state. Runs for different clients proceed independently.
- **A concurrent edit (e.g., a private-note save from FEAT-13.SPEC-001 on another device) is attempted on this client after deletion completes** -- Per the dependency map's Contention note ("once deleted any concurrent edit is refused with refresh"), that edit is refused with a message prompting the Pro to refresh, as specified in FEAT-13.SPEC-001's and FEAT-13.SPEC-002's own edge cases.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-13.SPEC-003 (Client Deletion Confirmation) | Triggered by (inbound) | Fires when the Pro confirms deletion |
| FEAT-13.SPEC-003 (Client Deletion Confirmation) | Affects (outbound) | Returns the success, blocked, or failure outcome to the confirmation screen |
| FEAT-13.SPEC-005 (Client Field Validation & Access Rules) | References (inbound) | Delete-access authorization (Pro only, own clients only) re-enforced at execution time, consistent with FEAT-13.SPEC-003 |
| FEAT-13.SPEC-006 (Deletion Eligibility & Retention Rule) | References (inbound) | Eligibility re-check and retention/de-identification rules applied during execution |
| FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) | Affects (outbound) | De-identified Deposit Transaction and Activity Event records remain available there for dispute/audit purposes, per XBR-19 |
| FEAT-14 (Messaging Consent Management) | Affects (outbound) | Messaging Consent records are cascade-deleted as part of this automation |
| FEAT-06 (Client Booking Identity) | Affects (outbound) | The client's Access Links, tied to the deleted record, cease to resolve to any client once the record is gone |

## Analytics and Success Signals

- **client_record_deleted** (had_retained_consent_evidence: boolean, booking_history_count) -- N/A -- no success-metrics.md metric names this feature directly; this event is the completion signal for the Brief's compliance-adjacent deletion-on-request obligation rather than a metric-connected signal, and it is the only event this automation emits worth measuring
- **client_deletion_failed** (reason: blocked_by_new_booking / processing_error) -- N/A -- same rationale as client_record_deleted; this event measures how often the all-or-nothing guarantee is exercised rather than feeding a named success metric

## Acceptance Criteria

**FEAT-13.SPEC-004-AC-01:** Given Talia has confirmed deletion for a client with no upcoming booking, when the automation runs, then the client's name, phone, email, and private note are permanently removed and the client_record_deleted event is logged.

**FEAT-13.SPEC-004-AC-02:** Given the deleted client had an active Messaging Consent record, when the automation runs, then that consent record is cascade-deleted along with the client's contact details.

**FEAT-13.SPEC-004-AC-03:** Given the deleted client had three past bookings and one deposit transaction, when the automation runs, then those Booking and Deposit Transaction records are de-identified (client reference removed) but retained with their service, timing, amount, and outcome fields intact.

**FEAT-13.SPEC-004-AC-04:** Given a new upcoming booking is created for this client between FEAT-13.SPEC-003's last check and this automation's execution, when the automation re-validates eligibility, then it stops without changing any data and returns the Blocked outcome to the confirmation screen.

**FEAT-13.SPEC-004-AC-05:** Given the automation's write fails partway through processing, when the failure occurs, then no partial deletion is left behind and FEAT-13.SPEC-003 shows "Could not delete this client. Check your connection and try again."

**FEAT-13.SPEC-004-AC-06:** Given a card-issuer dispute is open on one of the client's past deposit transactions, when the automation de-identifies that record, then the Disputed status and its evidence remain intact with only the client reference removed.

**FEAT-13.SPEC-004-AC-07:** Given Talia confirms deletion for the same client from two devices at effectively the same time, when both runs execute, then the first to complete succeeds and the second finds the record already gone, returning a "client no longer exists" failure.

**FEAT-13.SPEC-004-AC-08:** Given a deletion is already in flight for a client, when a second confirmation attempt is made for the same client before the first completes, then it cannot start -- FEAT-13.SPEC-003's Delete action is disabled during the in-flight run.

**FEAT-13.SPEC-004-AC-09:** Given a client with no Messaging Consent record on file is deleted, when the automation runs, then step 4b has nothing to cascade-delete and processing completes normally through to client_record_deleted.

**FEAT-13.SPEC-004-AC-10:** Given a client with zero booking history (a data inconsistency scenario) is deleted, when the automation runs, then step 4c has nothing to de-identify and client_record_deleted is still logged.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 (success, blocked, failure) | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |

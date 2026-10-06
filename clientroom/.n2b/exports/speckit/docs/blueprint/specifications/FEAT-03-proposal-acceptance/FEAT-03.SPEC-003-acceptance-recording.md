---
document_type: spec
spec_type: automation
spec_id: FEAT-03.SPEC-003
spec_name: Acceptance Recording
spec_slug: acceptance-recording
parent_feature: FEAT-03
parent_feature_name: Proposal Acceptance
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Automation Spec: Acceptance Recording

## Overview

**Name:** Acceptance Recording
**ID:** FEAT-03.SPEC-003
**Type:** Automation
**Purpose:** Writes the immutable acceptance record on the Proposal when Owen accepts, and fires the downstream deposit-invoice and audit-trail effects.
**Parent Feature:** FEAT-03 -- Proposal Acceptance

## Scope and Non-Goals

**In Scope:**
- Validating that the proposal is current (not voided) and unaccepted at the moment of write
- Writing the Proposal's `status`, `accepted_at`, and `accepted_by` fields exactly once
- Firing the deposit-invoice trigger to FEAT-09 when the Payment Schedule includes a deposit
- Firing the audit-trail entry to FEAT-13
- Firing the confirmation notification (FEAT-03.SPEC-006)

**Non-Goals:**
- Generating or sending the deposit invoice itself -- owned entirely by FEAT-09 (Invoice Generation & Sending); this automation only fires the trigger per XBR-01, using the Payment Schedule as it stood at the moment of acceptance.
- Composing or sending the confirmation email's content -- owned by FEAT-03.SPEC-006 (Acceptance Confirmation Notification), which this automation only triggers.
- Legally binding e-signature capture -- deferred per scope-boundaries.md's Deferral note; the timestamped Accept click, written by this automation, is the launch default, and e-signature (FEAT-26) extends this record in v1 without changing this automation's scope.
- Voiding or editing the proposal -- owned by FEAT-02 (Proposal Creation & Sending); this automation only reads the proposal's current status to determine eligibility.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|-------------------|
| Owen taps Accept | FEAT-03.SPEC-001 (Proposal Review & Accept) | Fires after FEAT-03.SPEC-005's eligibility check passes (proposal is Sent, not yet Accepted, not Voided, and the actor is a Primary contact at the owning client) | Proposal reference, accepting Client Contact's identity, current timestamp |

## Processing Logic

1. Receive the proposal reference and the accepting Client Contact's identity from FEAT-03.SPEC-001, after FEAT-03.SPEC-005's eligibility check has passed on that screen.
2. Re-check eligibility at write time against the Proposal's current state: `status` must still be `Sent` (not `Accepted`, not `Voided`). This re-check is the authoritative one -- the screen-time check in step 1 only gates the user's initial tap.
3. If the re-check fails because the proposal is already `Accepted`, stop and return the "already accepted" outcome (no write performed).
4. If the re-check fails because the proposal is `Voided`, stop and return the "voided -- redirect" outcome (no write performed).
5. If the re-check passes, write the acceptance in a single, atomic step: set `status` to `Accepted`, `accepted_at` to the current timestamp, and `accepted_by` to the accepting Client Contact's reference. This write is exactly-once: the atomic step itself is what prevents two concurrent passes of step 2-5 from both succeeding.
6. Read the Project's Payment Schedule as it stood at this exact moment (dependency map, Payment Schedule Contention: "a trigger uses the schedule as it stood at the moment of the triggering action").
7. If the Payment Schedule's structure includes a deposit, fire the deposit-invoice trigger to FEAT-09 (XBR-01), passing the Payment Schedule's deposit terms as read in step 6.
8. If the Payment Schedule's structure does not include a deposit, take no invoicing action.
9. Fire the audit-trail entry to FEAT-13 (XBR-05) with event type "proposal accepted," the accepting contact as actor, the timestamp from step 5, and the Proposal as the affected record.
10. Fire FEAT-03.SPEC-006 (Acceptance Confirmation Notification) to notify Owen and Nadia.
11. Return the success outcome to FEAT-03.SPEC-001, which displays the "Accepted on {date}" marker.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|-----------------|-------------------|
| Acceptance recorded, no deposit due | Write succeeds; Payment Schedule has no deposit trigger | Proposal `status` set to Accepted, `accepted_at` and `accepted_by` written | FEAT-03.SPEC-001 shows the "Accepted on {date}" marker | FEAT-03.SPEC-001, FEAT-03.SPEC-006, FEAT-13 |
| Acceptance recorded, deposit invoice triggered | Write succeeds; Payment Schedule includes a deposit | Proposal `status` set to Accepted, `accepted_at` and `accepted_by` written; deposit-invoice trigger fired to FEAT-09 | FEAT-03.SPEC-001 shows the "Accepted on {date}" marker; the deposit invoice appears shortly after via FEAT-09's own notification | FEAT-03.SPEC-001, FEAT-03.SPEC-006, FEAT-09, FEAT-13 |
| Already accepted | Write-time re-check finds `status` already `Accepted` (a concurrent acceptance won the race, or the client re-submitted) | None -- no duplicate write | FEAT-03.SPEC-001 shows "This proposal has already been accepted" | FEAT-03.SPEC-001 |
| Voided -- redirect | Write-time re-check finds `status` is `Voided` (Nadia edited and re-sent between screen load and this write) | None | FEAT-03.SPEC-001 redirects Owen to the current proposal version | FEAT-03.SPEC-001 |
| Write failure (connectivity or processing error) | The write itself does not complete (e.g., a connectivity drop mid-flight) | None -- the write either fully completes or leaves no partial acceptance record | FEAT-03.SPEC-001 shows an inline error with a retry option; retrying re-runs this automation from step 2 | FEAT-03.SPEC-001 |

## Data Model

**Reads:** Proposal -- `status`, `sent_at` (to confirm eligibility). Payment Schedule -- `structure`, `deposit_amount` (read as it stands at the exact moment of acceptance). Client Contact -- the accepting contact's identity and `role`.
**Creates:** None directly -- the deposit-invoice trigger causes FEAT-09 to create an Invoice; the audit-trail trigger causes FEAT-13 to create an Activity Log Entry. Neither record is created by this automation itself.
**Updates:** Proposal -- `status` (Sent to Accepted), `accepted_at`, `accepted_by`. Written exactly once; never altered afterward (XBR-04).
**Deletes:** None.

## Business Rules

- XBR-01: Accepting a proposal immediately generates and sends a deposit invoice when the Payment Schedule includes a deposit, using the schedule as it stood at acceptance. This automation owns firing that trigger; FEAT-09 owns the invoice itself.
- XBR-04: The acceptance record is never silently altered once written -- `accepted_at` and `accepted_by` are set exactly once and are never updated by any later process.
- XBR-05: Acceptance writes an append-only Activity Log Entry with actor and timestamp.
- The Proposal Contention resolution (dependency map): acceptance is recorded exactly once; two Primary contacts accepting at the same moment yield one acceptance, and the second sees "already accepted" -- enforced by this automation's atomic write in Processing Logic step 5, not by the triggering screen.
- The write-time eligibility re-check (step 2) is authoritative over the screen-time check performed by FEAT-03.SPEC-005 on FEAT-03.SPEC-001 -- the screen check only prevents an obviously stale tap; this automation's own re-check is what actually guarantees exactly-once acceptance.

## Edge Cases

- **Two Primary contacts at the same client tap Accept within the same instant** -- Concurrent trigger firing: both invocations reach step 2 near-simultaneously, but the atomic write in step 5 admits only one. The first to complete the atomic write succeeds; the second's re-check (step 2) then finds `status` already `Accepted` and returns the "already accepted" outcome. Neither invocation blocks the other; there is no queuing.
- **Owen taps Accept, the automation begins, and he taps Accept again before the first run finishes (e.g., a slow connection)** -- Trigger fires while a previous run is in flight: FEAT-03.SPEC-001 disables the Accept control while the automation is running (screen-level debounce), so a second automation run for the same proposal from the same session cannot start until the first completes. If it did reach this automation regardless, step 2's re-check on the second run would find the first run's write already applied (once it commits) and return "already accepted," or would race the still-in-flight first run under the same exactly-once write guarantee as the concurrent-trigger case.
- **Nadia edits and re-sends the proposal between Owen's tap and this automation's write** -- The write-time re-check (step 2) is the authority here, not the screen-time check: if the void completes before this automation's re-check runs, the re-check finds `status: Voided` and returns "voided -- redirect" with no acceptance recorded.
- **The Payment Schedule is being adjusted by Nadia at the exact moment of acceptance** -- Per the dependency map's Payment Schedule Contention, this automation reads the schedule as it stood at the moment of acceptance (step 6); a schedule edit saved afterward is dated and applies to later triggers only, never retroactively to this acceptance's deposit determination.
- **The deposit-invoice trigger to FEAT-09 fails to fire after the acceptance write has already succeeded** -- The acceptance record itself is not rolled back (it is already the evidentiary, immutable record per XBR-04); the deposit-invoice trigger is retried by FEAT-09's own retry handling. Owen still sees "Accepted on {date}" -- the deposit invoice's own appearance is FEAT-09's concern, not this automation's failure path.
- **The audit-trail trigger to FEAT-13 fails to fire** -- Non-blocking: the acceptance record and any deposit-invoice trigger already fired are unaffected; the audit-trail write is retried by FEAT-13's own handling, consistent with FEAT-13's append-only, never-silently-lost design intent.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|-------------------|--------------|
| FEAT-03.SPEC-001 (Proposal Review & Accept) | Triggered by (inbound) | Accept button tap, after eligibility passes |
| FEAT-03.SPEC-005 (Acceptance & Access Rules) | References (inbound) | Eligibility, accept-once, and voided-proposal rules this automation re-checks and enforces at write time |
| FEAT-03.SPEC-006 (Acceptance Confirmation Notification) | Triggers (outbound) | Fires the confirmation email to Owen and Nadia once acceptance is written |
| FEAT-09 (Invoice Generation & Sending) | Triggers (outbound) | Fires the deposit-invoice trigger (XBR-01) when the schedule includes a deposit |
| FEAT-13 (Immutable Activity & Audit Trail) | Triggers (outbound) | Fires the append-only trail entry for the acceptance event (XBR-05) |

## Analytics and Success Signals

- **proposal_accepted** (proposal reference, deposit invoice triggered: yes/no, time elapsed since sent_at) -- supports success-metrics.md: "Time to Proposal Acceptance"
- **deposit_invoice_auto_generated** (proposal reference, deposit amount reference) -- N/A -- no success metric in success-metrics.md is connected to deposit-invoice generation itself; this event is recorded here as the trigger-side signal, with the invoice's own lifecycle measured under FEAT-09's metrics
- **proposal_accept_write_failed** (failure reason: connectivity / concurrent-write-lost) -- N/A -- no connected success metric measures write failures; recorded as diagnostic-only exhaust from this automation's write path

## Acceptance Criteria

**FEAT-03.SPEC-003-AC-01:** Given Owen taps Accept on a proposal in Sent status with a Payment Schedule that includes no deposit, when this automation runs, then the Proposal's `status` becomes Accepted with `accepted_at` and `accepted_by` set, and no invoice trigger fires.

**FEAT-03.SPEC-003-AC-02:** Given Owen taps Accept on a proposal in Sent status with a Payment Schedule that includes a deposit, when this automation runs, then the acceptance is recorded and the deposit-invoice trigger fires to FEAT-09 (XBR-01), using the schedule as it stood at that moment.

**FEAT-03.SPEC-003-AC-03:** Given the acceptance write succeeds, when this automation completes, then it fires FEAT-03.SPEC-006 to notify Owen and Nadia, and fires the FEAT-13 audit-trail entry for the acceptance event.

**FEAT-03.SPEC-003-AC-04:** Given two Primary contacts at the same client both tap Accept on the same proposal at effectively the same moment, when this automation's write-time re-check runs for each, then exactly one acceptance is recorded and the other invocation returns "already accepted."

**FEAT-03.SPEC-003-AC-05:** Given Owen taps Accept on a proposal that Nadia voids by editing and re-sending before this automation's write-time re-check runs, when the re-check finds `status: Voided`, then no acceptance is recorded and FEAT-03.SPEC-001 redirects Owen to the current version.

**FEAT-03.SPEC-003-AC-06:** Given Owen taps Accept and connectivity drops before the write completes, when the failure occurs, then no partial acceptance record is left and FEAT-03.SPEC-001 shows a retry option.

**FEAT-03.SPEC-003-AC-07:** Given Owen retries Accept after a failed write, when this automation re-runs, then it re-checks eligibility from the current Proposal state rather than assuming the prior attempt's context still holds.

**FEAT-03.SPEC-003-AC-08:** Given Nadia adjusts the Payment Schedule at the exact moment Owen's acceptance is being written, when this automation reads the schedule in step 6, then it uses the schedule as it stood at the moment of acceptance, and Nadia's adjustment applies only to later triggers.

**FEAT-03.SPEC-003-AC-09:** Given the acceptance write succeeds but the deposit-invoice trigger to FEAT-09 fails to fire, when this automation completes, then the acceptance record remains valid and unaffected, and the invoice trigger is retried by FEAT-09's own handling.

**FEAT-03.SPEC-003-AC-10:** Given the acceptance is recorded, when the analytics signal is emitted, then a proposal_accepted event carries the proposal reference, whether a deposit invoice was triggered, and the time elapsed since the proposal was sent.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 5 (no deposit, deposit triggered, already accepted, voided-redirect, write failure) | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 (concurrent tap, run-in-flight, void-race, schedule-race, invoice-trigger-failure, audit-trigger-failure) | 6 |

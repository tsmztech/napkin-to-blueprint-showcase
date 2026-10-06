---
document_type: spec
spec_type: automation
spec_id: FEAT-26.SPEC-002
spec_name: Signature Recording
spec_slug: signature-recording
parent_feature: FEAT-26
parent_feature_name: Legally Binding E-Signature for Proposals
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

# Automation Spec: Signature Recording

## Overview

**Name:** Signature Recording
**ID:** FEAT-26.SPEC-002
**Type:** Automation
**Purpose:** Validates Owen's signature submission, submits it for electronic-signature attestation, and writes the immutable signature record alongside FEAT-03's acceptance fields so the signed acceptance still fires the deposit invoice and audit-trail entry.
**Parent Feature:** FEAT-26 -- Legally Binding E-Signature for Proposals

## Scope and Non-Goals

**In Scope:**
- Re-checking signing eligibility at write time (signing-enabled, not-yet-signed, not-voided) and the actor's authorization at write time (signing contact's role still Primary and status still Active)
- Submitting the signature data (full legal name, timestamp) to the electronic-signature attestation capability (FEAT-26.SPEC-005) and receiving its attestation confirmation
- Writing the Proposal's signature record (signer identity, signature data, timestamp) and, in the same atomic step, the `status`, `accepted_at`, and `accepted_by` fields FEAT-03.SPEC-003 owns
- Firing FEAT-03.SPEC-003's downstream effects for the signed acceptance: the deposit-invoice trigger to FEAT-09 (XBR-01) and the audit-trail entry to FEAT-13 (XBR-05)
- Firing FEAT-26.SPEC-004 (Signed-Copy Confirmation Notification) in addition to FEAT-03.SPEC-006's standard acceptance confirmation

**Non-Goals:**
- Generating or sending the deposit invoice itself -- owned entirely by FEAT-09 (Invoice Generation & Sending); this automation only fires the trigger per XBR-01, exactly as FEAT-03.SPEC-003 does for a plain acceptance.
- Composing or sending either confirmation email's content -- FEAT-03.SPEC-006 and FEAT-26.SPEC-004 own their own content; this automation only triggers both.
- Determining whether the proposal is signing-enabled or who may sign it -- owned by FEAT-26.SPEC-003 (E-Signature Opt-In & Signing Access Rules), which this automation re-checks and enforces at write time rather than re-deriving.
- Choosing or configuring the electronic-signature attestation vendor -- vendor selection is a Stage 4 architecture decision (FEAT-26.SPEC-005); this automation only defines the functional exchange with whatever capability is selected.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|-------------------|
| Owen taps Sign | FEAT-26.SPEC-001 (Signature Signing Step) | Fires after FEAT-26.SPEC-003's eligibility check passes on that screen (proposal is Sent, signing-enabled, not yet signed, not Voided, and the actor is a Primary contact at the owning client) | Proposal reference, signing Client Contact's identity, entered full legal name, current timestamp |

## Processing Logic

1. Receive the proposal reference, the signing Client Contact's identity, and the entered full legal name from FEAT-26.SPEC-001, after FEAT-26.SPEC-003's eligibility check has passed on that screen.
2. Re-check eligibility and authorization at write time against current state: (a) the Proposal's `status` must still be `Sent`, its signing-enabled flag must still be set, and no signature record must yet exist; and (b) the signing Client Contact's current `role` must still be Primary and `status` must still be Active, at the client that owns the proposal (FEAT-26.SPEC-003 Authorization Rules; XBR-08, XBR-27). If check (b) fails, stop immediately and return the "denied -- not authorized" outcome: no write is performed and nothing is submitted to attestation. Check (b) is evaluated before the outcomes in steps 3 and 4 so an unauthorized actor learns nothing about the proposal's signed or voided state. This re-check is the authoritative one -- the screen-time check in step 1 only gates the user's initial tap.
3. If the re-check fails because a signature record already exists, stop and return the "already signed" outcome (no write performed).
4. If the re-check fails because the proposal is `Voided`, stop and return the "voided -- redirect" outcome (no write performed).
5. If the re-check passes, submit the signature data (the entered full legal name, the signing Client Contact's reference, and the current timestamp) to the electronic-signature attestation capability (FEAT-26.SPEC-005) for attestation.
6. If the attestation capability confirms the submission, proceed to step 7. If it rejects the submission or the request fails, stop and return the "attestation failed" outcome (no write performed).
7. Write the signature and acceptance in a single, atomic step: set the signature record's signer identity, signature data (the entered full legal name), and timestamp; and set the Proposal's `status` to `Accepted`, `accepted_at` to the current timestamp, and `accepted_by` to the signing Client Contact's reference. This write is exactly-once: the atomic step itself is what prevents two concurrent passes of steps 2-7 from both succeeding.
8. Read the Project's Payment Schedule as it stood at this exact moment (dependency map, Payment Schedule Contention: "a trigger uses the schedule as it stood at the moment of the triggering action").
9. If the Payment Schedule's structure includes a deposit, fire the deposit-invoice trigger to FEAT-09 (XBR-01), passing the Payment Schedule's deposit terms as read in step 8, exactly as FEAT-03.SPEC-003 does for a plain acceptance.
10. If the Payment Schedule's structure does not include a deposit, take no invoicing action.
11. Fire the audit-trail entry to FEAT-13 (XBR-05) with event type "proposal signed," the signing contact as actor, the timestamp from step 7, and the Proposal as the affected record.
12. Fire FEAT-03.SPEC-006 (Acceptance Confirmation Notification) and FEAT-26.SPEC-004 (Signed-Copy Confirmation Notification) to notify Owen and Nadia.
13. Return the success outcome to FEAT-26.SPEC-001, which displays the "Signed on {date}" marker.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|-----------------|-------------------|
| Signature recorded, no deposit due | Attestation confirmed; write succeeds; Payment Schedule has no deposit trigger | Proposal signature record written; `status` set to Accepted, `accepted_at` and `accepted_by` written | FEAT-26.SPEC-001 shows the "Signed on {date}" marker | FEAT-26.SPEC-001, FEAT-03.SPEC-006, FEAT-26.SPEC-004, FEAT-13 |
| Signature recorded, deposit invoice triggered | Attestation confirmed; write succeeds; Payment Schedule includes a deposit | Proposal signature record written; `status` set to Accepted, `accepted_at` and `accepted_by` written; deposit-invoice trigger fired to FEAT-09 | FEAT-26.SPEC-001 shows the "Signed on {date}" marker; the deposit invoice appears shortly after via FEAT-09's own notification | FEAT-26.SPEC-001, FEAT-03.SPEC-006, FEAT-26.SPEC-004, FEAT-09, FEAT-13 |
| Denied -- not authorized | Write-time check (step 2b) finds the signing contact's role is no longer Primary (e.g., changed to Reviewer) or status is no longer Active (e.g., Removed) | None -- no attestation submission, no write | FEAT-26.SPEC-001 shows the Not authorized state: "You no longer have permission to sign this proposal. If you think this is a mistake, ask the person who sent it." with a "Go to portal home" link; no proposal content is shown | FEAT-26.SPEC-001 |
| Already signed | Write-time re-check finds a signature record already exists (a concurrent signature won the race, or the client re-submitted) | None -- no duplicate write | FEAT-26.SPEC-001 shows "This proposal has already been signed" | FEAT-26.SPEC-001 |
| Voided -- redirect | Write-time re-check finds `status` is `Voided` (Nadia edited and re-sent between screen load and this write) | None | FEAT-26.SPEC-001 redirects Owen to the current proposal version | FEAT-26.SPEC-001 |
| Attestation failed | The electronic-signature attestation capability rejects the submission or the request itself fails | None -- no partial signature record is written | FEAT-26.SPEC-001 shows "We couldn't record your signature. Check your connection and try again." with the entered full legal name preserved | FEAT-26.SPEC-001 |
| Write failure (connectivity or processing error, after attestation is confirmed) | The write itself does not complete after the attestation capability has already confirmed | None -- the write either fully completes or leaves no partial signature record | FEAT-26.SPEC-001 shows the same inline error with a retry option; retrying re-runs this automation from step 2, re-submitting to attestation only if step 5 has not already succeeded for a record that now exists | FEAT-26.SPEC-001 |

## Data Model

**Reads:** Proposal -- `status`, `sent_at`, signing-enabled flag, existing signature record (to confirm eligibility). Payment Schedule -- `structure`, `deposit_amount` (read as it stands at the exact moment of signing). Client Contact -- the signing contact's identity, `role`, and `status` (re-read at write time for the authorization check).
**Creates:** None directly -- the deposit-invoice trigger causes FEAT-09 to create an Invoice; the audit-trail trigger causes FEAT-13 to create an Activity Log Entry. Neither record is created by this automation itself.
**Updates:** Proposal -- signature record (signer identity, signature data, timestamp), `status` (Sent to Accepted), `accepted_at`, `accepted_by`. Written exactly once; never altered afterward (XBR-04).
**Deletes:** None.

## Business Rules

- XBR-34: When enabled for a proposal, a legally binding e-signature replaces the plain Accept click with the same Primary-only access and the same immutability. This automation is the write that carries out that replacement.
- XBR-01: Signing a proposal immediately generates and sends a deposit invoice when the Payment Schedule includes a deposit, using the schedule as it stood at signing -- identical to FEAT-03.SPEC-003's rule for a plain acceptance.
- XBR-04: The signature and acceptance record are never silently altered once written -- the signature record, `accepted_at`, and `accepted_by` are set exactly once and are never updated by any later process.
- XBR-05: Signing writes an append-only Activity Log Entry with actor and timestamp.
- The Proposal Contention resolution (dependency map): signing is recorded exactly once; a second signing attempt after the first succeeds is denied with "This proposal has already been signed" -- enforced by this automation's atomic write in Processing Logic step 7, not by the triggering screen.
- The write-time eligibility re-check (step 2) is authoritative over the screen-time check performed by FEAT-26.SPEC-003 on FEAT-26.SPEC-001 -- the screen check only prevents an obviously stale tap; this automation's own re-check is what actually guarantees exactly-once signing.
- XBR-08, XBR-27: Only a Primary contact in Active status may sign. This automation re-checks the actor's current role and status at write time (step 2b) -- a contact whose role changed or who was Removed after the screen loaded is denied with the "denied -- not authorized" outcome, and no signature or acceptance is written for them.
- The signature is not written until the electronic-signature attestation capability confirms it (step 6) -- a rejected or failed attestation never leaves a partial signature record.

## Edge Cases

- **Owen taps Sign, and a second signing attempt for the same proposal is submitted before the first completes (e.g., a slow connection and a retried tap)** -- Concurrent trigger firing: both invocations reach step 2 near-simultaneously, but the atomic write in step 7 admits only one. The first to complete the atomic write succeeds; the second's re-check (step 2) then finds a signature record already exists and returns the "already signed" outcome. Neither invocation blocks the other; there is no queuing.
- **Owen taps Sign, this automation begins, and the screen submits a retry before the first run finishes** -- Trigger fires while a previous run is in flight: FEAT-26.SPEC-001 disables the Sign control while the automation is running (screen-level debounce), so a second automation run for the same proposal from the same session cannot start until the first completes. If it did reach this automation regardless, step 2's re-check on the second run would find the first run's write already applied (once it commits) and return "already signed," or would race the still-in-flight first run under the same exactly-once write guarantee as the concurrent-trigger case.
- **Owen's role is changed to Reviewer, or his status to Removed, after the signing step loaded but before this automation's write-time check** -- Step 2b finds the role or status no longer qualifies and returns "denied -- not authorized"; no attestation submission and no write occur, and FEAT-26.SPEC-001 shows the Not authorized state. If the change lands after step 2b has passed, the atomic write in step 7 still proceeds for that in-flight action, since authorization was valid at the authoritative check.
- **Nadia edits and re-sends the proposal between Owen's tap and this automation's write** -- The write-time re-check (step 2) is the authority here, not the screen-time check: if the void completes before this automation's re-check runs, the re-check finds `status: Voided` and returns "voided -- redirect" with no signature recorded.
- **The electronic-signature attestation capability confirms the submission but the atomic write then fails (e.g., a connectivity drop immediately after step 6)** -- No partial signature record is left; the acceptance record and signature remain unwritten until a retry re-submits and succeeds. Owen sees the same retry-capable error as an outright attestation rejection.
- **The Payment Schedule is being adjusted by Nadia at the exact moment of signing** -- Per the dependency map's Payment Schedule Contention, this automation reads the schedule as it stood at the moment of signing (step 8); a schedule edit saved afterward is dated and applies to later triggers only, never retroactively to this signature's deposit determination.
- **The deposit-invoice trigger to FEAT-09 fails to fire after the signature write has already succeeded** -- The signature and acceptance record are not rolled back (they are already the evidentiary, immutable record per XBR-04); the deposit-invoice trigger is retried by FEAT-09's own retry handling. Owen still sees "Signed on {date}" -- the deposit invoice's own appearance is FEAT-09's concern, not this automation's failure path.
- **The audit-trail trigger to FEAT-13 fails to fire** -- Non-blocking: the signature record and any deposit-invoice trigger already fired are unaffected; the audit-trail write is retried by FEAT-13's own handling.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|-------------------|--------------|
| FEAT-26.SPEC-001 (Signature Signing Step) | Triggered by (inbound) | Sign button tap, after eligibility passes |
| FEAT-26.SPEC-003 (E-Signature Opt-In & Signing Access Rules) | References (inbound) | Eligibility, sign-once, and voided-proposal rules this automation re-checks and enforces at write time |
| FEAT-26.SPEC-005 (Electronic-Signature Attestation Capability) | Triggers (outbound) | Submits the signature data for attestation before the record is written |
| FEAT-03.SPEC-003 (Acceptance Recording) | References (outbound) | This automation performs the same acceptance write FEAT-03.SPEC-003 owns, extended with the signature record, per XBR-34 |
| FEAT-03.SPEC-006 (Acceptance Confirmation Notification) | Triggers (outbound) | Fires the standard confirmation email to Owen and Nadia once the signature is written |
| FEAT-26.SPEC-004 (Signed-Copy Confirmation Notification) | Triggers (outbound) | Fires the signed-copy confirmation email to Owen and Nadia in addition to the standard confirmation |
| FEAT-09 (Invoice Generation & Sending) | Triggers (outbound) | Fires the deposit-invoice trigger (XBR-01) when the schedule includes a deposit |
| FEAT-13 (Immutable Activity & Audit Trail) | Triggers (outbound) | Fires the append-only trail entry for the signed-acceptance event (XBR-05) |

## Analytics and Success Signals

- **proposal_signed** (proposal reference, deposit invoice triggered: yes/no, time elapsed since sent_at) -- N/A -- no Stage 2 metric in success-metrics.md is connected to FEAT-26; this event is the product-defined signal named in product-features.md's Signals field (proposal_signed), retained so signing volume is observable even without a connected metric
- **esignature_attestation_confirmed** (proposal reference) -- N/A -- no connected success metric measures attestation outcomes specifically; retained as the trigger-side signal for the electronic-signature capability's own reliability, mirroring FEAT-03.SPEC-003's deposit_invoice_auto_generated event
- **signature_write_failed** (failure reason: attestation-rejected / attestation-timeout / connectivity / concurrent-write-lost / not-authorized) -- N/A -- no connected success metric measures write failures; recorded as diagnostic-only exhaust from this automation's write path, mirroring FEAT-03.SPEC-003's proposal_accept_write_failed event

## Acceptance Criteria

**FEAT-26.SPEC-002-AC-01:** Given Owen submits his signature on a proposal in Sent status with a Payment Schedule that includes no deposit, when the attestation capability confirms the submission and this automation runs, then the signature record is written, the Proposal's `status` becomes Accepted with `accepted_at` and `accepted_by` set, and no invoice trigger fires.

**FEAT-26.SPEC-002-AC-02:** Given Owen submits his signature on a proposal in Sent status with a Payment Schedule that includes a deposit, when this automation runs, then the signature and acceptance are recorded and the deposit-invoice trigger fires to FEAT-09 (XBR-01), using the schedule as it stood at that moment.

**FEAT-26.SPEC-002-AC-03:** Given the signature write succeeds, when this automation completes, then it fires FEAT-03.SPEC-006 and FEAT-26.SPEC-004 to notify Owen and Nadia, and fires the FEAT-13 audit-trail entry for the signed-acceptance event.

**FEAT-26.SPEC-002-AC-04:** Given two signing attempts for the same proposal reach this automation at effectively the same moment, when the write-time re-check runs for each, then exactly one signature is recorded and the other invocation returns "already signed."

**FEAT-26.SPEC-002-AC-05:** Given Owen submits his signature on a proposal that Nadia voids by editing and re-sending before this automation's write-time re-check runs, when the re-check finds `status: Voided`, then no signature is recorded and FEAT-26.SPEC-001 redirects Owen to the current version.

**FEAT-26.SPEC-002-AC-06:** Given Owen submits his signature, when the electronic-signature attestation capability rejects the submission, then no signature record is written and FEAT-26.SPEC-001 shows a retry-capable error with the entered full legal name preserved.

**FEAT-26.SPEC-002-AC-07:** Given the attestation capability confirms Owen's submission but connectivity drops before the atomic write completes, when the failure occurs, then no partial signature record is left and FEAT-26.SPEC-001 shows a retry option.

**FEAT-26.SPEC-002-AC-08:** Given Owen retries signing after a failed write, when this automation re-runs, then it re-checks eligibility from the current Proposal state rather than assuming the prior attempt's context still holds.

**FEAT-26.SPEC-002-AC-09:** Given Nadia adjusts the Payment Schedule at the exact moment Owen's signature is being written, when this automation reads the schedule in step 8, then it uses the schedule as it stood at the moment of signing, and Nadia's adjustment applies only to later triggers.

**FEAT-26.SPEC-002-AC-10:** Given the signature write succeeds but the deposit-invoice trigger to FEAT-09 fails to fire, when this automation completes, then the signature and acceptance record remain valid and unaffected, and the invoice trigger is retried by FEAT-09's own handling.

**FEAT-26.SPEC-002-AC-11:** Given the audit-trail trigger to FEAT-13 fails to fire after the signature write succeeds, when this automation completes, then the signature record is unaffected and the trail write is retried by FEAT-13's own handling.

**FEAT-26.SPEC-002-AC-12:** Given the signature is recorded, when the analytics signal is emitted, then a proposal_signed event carries the proposal reference, whether a deposit invoice was triggered, and the time elapsed since the proposal was sent.

**FEAT-26.SPEC-002-AC-13:** Given Owen's role has been changed to Reviewer after the signing step loaded, when he taps Sign and this automation's write-time check runs, then it returns "denied -- not authorized", no attestation submission and no signature or acceptance write occurs, and FEAT-26.SPEC-001 shows "You no longer have permission to sign this proposal. If you think this is a mistake, ask the person who sent it."

**FEAT-26.SPEC-002-AC-14:** Given Owen's status has been set to Removed after the signing step loaded, when he taps Sign, then this automation returns "denied -- not authorized" before evaluating signed or voided state, the Proposal's `status`, signature record, `accepted_at`, and `accepted_by` are unchanged, and no invoice trigger, audit-trail entry for a signature, or confirmation notification fires.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 7 (no deposit, deposit triggered, denied-not-authorized, already signed, voided-redirect, attestation failed, write failure) | 7 |
| Business Rules | 8 | 8 |
| Edge Cases | 8 (concurrent tap, run-in-flight, unauthorized-actor, void-race, attestation-confirmed-then-write-fails, schedule-race, invoice-trigger-failure, audit-trigger-failure) | 8 |

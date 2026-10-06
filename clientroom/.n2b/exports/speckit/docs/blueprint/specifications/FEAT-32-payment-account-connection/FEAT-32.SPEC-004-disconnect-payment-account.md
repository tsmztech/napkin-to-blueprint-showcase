---
document_type: spec
spec_type: automation
spec_id: FEAT-32.SPEC-004
spec_name: Disconnect Payment Account
spec_slug: disconnect-payment-account
parent_feature: FEAT-32
parent_feature_name: Payment Account Connection
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Automation Spec: Disconnect Payment Account

## Overview

**Name:** Disconnect Payment Account
**ID:** FEAT-32.SPEC-004
**Type:** Automation
**Purpose:** Removes Nadia's connection reference on her explicit disconnect action, without cancelling any payment already submitted to the processor.
**Parent Feature:** FEAT-32 -- Payment Account Connection

## Scope and Non-Goals

**In Scope:**
- Removing the Payment Account Connection record (hard delete, no restore path) when Nadia confirms disconnect from FEAT-32.SPEC-001
- Removing the same record when account deletion (FEAT-24) reaches its payment-disconnect step
- Leaving any payment already submitted to the processor unaffected

**Non-Goals:**
- Showing the disconnect warning and collecting Nadia's confirmation -- owned by FEAT-32.SPEC-001 (Payment Connection Screen); this automation only executes once that confirmation, gated by FEAT-32.SPEC-005, is already given.
- The account-deletion warning sequence itself (unpaid invoices, pending approvals) -- owned entirely by FEAT-24 (Data Export & Account Deletion); this automation only performs the payment-account-specific removal step within that sequence.
- Cancelling or reversing a payment already submitted to the processor -- excluded per feature-overview.md's Side-Effect Inventory: an in-flight payment is not cancelled by a disconnect; that payment's own outcome is owned entirely by FEAT-10.SPEC-003.
- Restoring a disconnected account without repeating the connect flow -- excluded per feature-overview.md's Non-Goals: the record holds no historical value beyond the live reference, so reconnecting always runs FEAT-32.SPEC-002's Create operation again.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nadia confirms Disconnect | FEAT-32.SPEC-001 (Payment Connection Screen) | Fires only after FEAT-32.SPEC-005's authorization passes (Nadia, her own account) and she has confirmed the explicit disconnect warning | The current `processor_account_reference` to be removed |
| Account deletion reaches the payment-disconnect step | FEAT-24.SPEC-004 (Account Deletion Processing, FEAT-24) | Fires when FEAT-24's own warn-then-remove sequence reaches the payment-account step, which is that cascade's commit point (FEAT-24.SPEC-004 step 5; XBR-33); no separate warning re-confirmation is required here, since FEAT-24's own sequence has already warned about disconnecting the payment account | The freelancer account's existing Payment Account Connection record, if any |

## Processing Logic

1. Receive the disconnect instruction, noting its source: Nadia's own confirmed action (FEAT-32.SPEC-001) or account-deletion processing (FEAT-24.SPEC-004).
2. Check whether a Payment Account Connection record currently exists for this freelancer account. If none exists, stop -- there is nothing to remove (Edge Cases: idempotent no-op).
3. Set `status` to Disconnected as the record's terminal value, then remove the record entirely: `processor_account_reference`, `status`, and `available_payment_methods` are all deleted together in the same operation -- Disconnected is never a value a live, readable record persists in; it names the outcome of this step, not a lingering state. There is no partial removal and no retained history beyond this point.
4. Any payment already submitted to the processor before this step is left entirely unaffected; this automation makes no request to the payment-processing capability at all.
5. Confirm completion to the triggering source: FEAT-32.SPEC-001 (for Nadia's own action) or FEAT-24.SPEC-004 (for account deletion), so each can proceed with its own next step.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Disconnected (Nadia's own action) | Nadia's confirmed disconnect is processed and a record existed | Payment Account Connection record deleted in full | FEAT-32.SPEC-001 returns to the Empty state with "Payment account disconnected." | FEAT-32.SPEC-001, FEAT-09 (pay-link fallback), FEAT-10 (pay-screen fallback) |
| Disconnected (account deletion) | FEAT-24's sequence reaches this step and a record existed | Payment Account Connection record deleted in full | No independent feedback from this automation -- FEAT-24 owns the account-deletion confirmation the freelancer sees | FEAT-24 |
| No-action (already disconnected) | No Payment Account Connection record exists when either trigger fires | None | FEAT-32.SPEC-001 (if the trigger was Nadia's) already shows the Empty state -- nothing changes; FEAT-24 proceeds with its sequence unaffected | FEAT-32.SPEC-001, FEAT-24 |
| Failure | The deletion write itself fails (an internal processing error) | None -- record retained unchanged | FEAT-32.SPEC-001 shows "We couldn't disconnect your account. Try again." and the Disconnect action remains available for Nadia to retry; for the account-deletion trigger, FEAT-24's own sequence is informed the step did not complete and it retries per its own process | FEAT-32.SPEC-001, FEAT-24 |

## Data Model

**Reads:** Payment Account Connection -- checks for the record's existence before attempting removal.
**Creates:** None.
**Updates:** None.
**Deletes:** Payment Account Connection -- the entire record (`processor_account_reference`, `status`, `available_payment_methods`) is removed together; no restore path exists, per the entity's hard-delete lifecycle (feature-dependency-map.md, Entity: Payment Account Connection).

## Business Rules

- Disconnect never cancels a payment already submitted to the processor (feature-overview.md, Side-Effect Inventory); this automation makes no request to the payment-processing capability.
- Once removed, any pay link opened afterward shows the "temporarily unavailable" fallback owned by FEAT-09.SPEC-009 and FEAT-10 (XBR-19) -- this automation itself renders no such copy.
- FEAT-32.SPEC-005's contention rule (processor-authoritative, in-flight-payment protection) governs the fact that disconnect is never blocked by an in-flight payment; this automation always proceeds once triggered.
- Reconnecting after a disconnect always runs FEAT-32.SPEC-002's Create operation again -- there is no resume or restore of the removed record.
- XBR-33: account deletion's own warn-then-remove sequence owns the warning shown for this step when triggered by FEAT-24; this automation performs the removal itself without re-warning.

## Edge Cases

- **Concurrent trigger firing (Nadia confirms Disconnect at the same moment FEAT-24's sequence reaches its own disconnect step, e.g. she starts account deletion right after disconnecting)** -- Whichever trigger's removal reaches step 2 first finds the record and removes it; the second trigger then finds no record (per step 2) and takes the no-action outcome. Neither trigger errors on finding nothing to remove.
- **Trigger fires while a previous run is still in flight (Nadia double-taps Disconnect, or the confirmation dialog is submitted twice)** -- FEAT-32.SPEC-001 disables the Disconnect button while the first request is processing, preventing a genuine second in-flight run from the same source; if a second request somehow reaches this automation, it is the no-action outcome described above (idempotent).
- **A disconnect is issued while a client's payment is already submitted to the processor** -- Per FEAT-32.SPEC-005's contention rule, the disconnect is not blocked and proceeds exactly as in the standard outcome; the in-flight payment continues to its own outcome (FEAT-10.SPEC-003) entirely independent of this removal.
- **A FEAT-32.SPEC-003 status-report event is in flight for this connection at the same moment it is disconnected** -- The disconnect's removal takes precedence: once the record is deleted, the status-report event finds nothing to update and is discarded (FEAT-32.SPEC-003's own edge-case handling), rather than recreating a partial record.
- **Account deletion is later cancelled or reverted before finalizing (a scenario FEAT-24 itself would define)** -- Outside this spec's scope: this automation performs an unconditional removal once FEAT-24's own sequence reaches this specific step; any decision to not reach that step at all is entirely FEAT-24's to make before triggering this automation.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-32.SPEC-001 (Payment Connection Screen) | Triggered by (inbound) | Nadia's confirmed disconnect (after the warning dialog) fires this automation |
| FEAT-32.SPEC-001 (Payment Connection Screen) | Affects (outbound) | Returns to the Empty state on success, or shows the failure message |
| FEAT-32.SPEC-005 (Payment Connection Authorization & Validation Rules) | References (inbound) | Authorization and the in-flight-payment contention rule gate and shape this automation's behavior |
| FEAT-24 (Data Export & Account Deletion) | Triggered by (inbound) | Account deletion's warn-then-remove sequence fires this automation as its payment-disconnect step (XBR-33) |
| FEAT-32.SPEC-003 (Connection Status Sync) | References (outbound) | A status-report event arriving after this automation's removal finds no record, per that spec's own edge-case handling |
| FEAT-09 (Invoice Generation & Sending) | Affects (outbound) | Open invoices' pay links switch to the fallback experience once this automation removes the connection |
| FEAT-10 (Invoice Payment Processing) | Affects (outbound) | The pay screen switches to the "temporarily unavailable" fallback once this automation removes the connection |

## Analytics and Success Signals

- **payment_account_disconnected** (trigger_source: nadia_manual / account_deletion) -- N/A -- no Stage 2 metric measures disconnect frequency directly; retained per product-features.md's named signal for observability of connection churn against "Payment Readiness Before First Invoice."
- **payment_account_disconnect_failed** (trigger_source, retry_attempted: yes/no) -- N/A -- no Stage 2 metric measures this automation's own failure rate; retained as a standard reliability signal.

## Acceptance Criteria

**FEAT-32.SPEC-004-AC-01:** Given Nadia has confirmed the disconnect warning on FEAT-32.SPEC-001, when this automation runs, then the Payment Account Connection record is removed in full and FEAT-32.SPEC-001 returns to the Empty state with "Payment account disconnected."

**FEAT-32.SPEC-004-AC-02:** Given FEAT-24's account-deletion sequence reaches its payment-disconnect step, when this automation runs, then the Payment Account Connection record is removed in full without a separate warning shown by this spec.

**FEAT-32.SPEC-004-AC-03:** Given no Payment Account Connection record exists when Nadia's disconnect is (redundantly) triggered, when this automation runs, then nothing changes and FEAT-32.SPEC-001 remains on the Empty state.

**FEAT-32.SPEC-004-AC-04:** Given a client's payment is already submitted to the processor, when Nadia disconnects, then the disconnect proceeds and the in-flight payment is entirely unaffected.

**FEAT-32.SPEC-004-AC-05:** Given the removal write itself fails, when the failure occurs, then FEAT-32.SPEC-001 shows "We couldn't disconnect your account. Try again." and the record is retained unchanged.

**FEAT-32.SPEC-004-AC-06:** Given Nadia double-taps Disconnect, when the second tap reaches this automation while the first is still processing, then the second tap produces the no-action outcome without error.

**FEAT-32.SPEC-004-AC-07:** Given a FEAT-32.SPEC-003 status-report event is in flight for the same connection, when this automation completes the removal first, then the status-report event later finds no record and is discarded.

**FEAT-32.SPEC-004-AC-08:** Given Nadia reconnects after a disconnect, when she does so, then FEAT-32.SPEC-002's Create operation runs again rather than restoring any prior record.

**FEAT-32.SPEC-004-AC-09:** Given account deletion and Nadia's own manual disconnect are triggered at effectively the same time, when both reach this automation, then whichever reaches step 2 first performs the removal and the other takes the no-action outcome.

**FEAT-32.SPEC-004-AC-10:** Given the connection is removed by this automation, when an open invoice's pay link is opened afterward, then it shows the fallback experience owned by FEAT-09.SPEC-009 and FEAT-10, not a working payment form.

**FEAT-32.SPEC-004-AC-11:** Given this automation is triggered by FEAT-24, when it completes, then FEAT-24's own sequence proceeds to its next step informed that the payment-account removal succeeded.

**FEAT-32.SPEC-004-AC-12:** Given this automation's write fails when triggered by FEAT-24, when the failure occurs, then FEAT-24's sequence is informed the step did not complete and retries per its own process, rather than this automation silently reporting success.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 | 2 |
| Outcome Paths | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |

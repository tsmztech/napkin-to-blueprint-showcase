---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-32.SPEC-005
spec_name: Payment Connection Authorization & Validation Rules
spec_slug: payment-connection-authorization-validation-rules
parent_feature: FEAT-32
parent_feature_name: Payment Account Connection
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 19
acceptance_criteria_count: 21
---

# Logic/Rule Spec: Payment Connection Authorization & Validation Rules

## Overview

**Name:** Payment Connection Authorization & Validation Rules
**ID:** FEAT-32.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs the one-account-per-freelancer limit, who may connect/reconnect/disconnect versus view only, the disconnect warning, and the processor-authoritative contention rule that protects an in-flight payment from a concurrent disconnect.
**Parent Feature:** FEAT-32 -- Payment Account Connection
**Governed Entity:** Payment Account Connection

## Scope and Non-Goals

**In Scope:**
- Field-level rules for the Payment Account Connection record's fields
- The one-connected-account-per-freelancer limit
- Authorization for Connect, View, Reconnect, Disconnect, and Start over (the exit from a stalled hand-off) for every role in the Access Matrix
- The exact disconnect-warning wording and when it gates the disconnect action
- The processor-authoritative contention rule protecting an in-flight payment from a concurrent disconnect

**Non-Goals:**
- Executing the connect/reconnect hand-off -- owned by FEAT-32.SPEC-002 (Payment Account Connection & Status Reporting); this spec defines only who may initiate it and under what condition.
- Applying a reported status change -- owned by FEAT-32.SPEC-003 (Connection Status Sync); this spec governs authorization and the contention rule, not the status-application logic itself.
- Executing the removal -- owned by FEAT-32.SPEC-004 (Disconnect Payment Account); this spec defines the gate (authorization and the acknowledged warning) that must pass before that automation runs.
- The exact client-facing "temporarily unavailable" pay-link copy -- owned by FEAT-09.SPEC-009 and FEAT-10; this spec states the underlying rule (an in-flight payment is protected and a disconnected account's pay links stop working), not the rendered wording on those screens.

## Governed Entity

**Entity:** Payment Account Connection
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| processor_account_reference | text (reference) | A reference to the freelancer's linked processor account; never card numbers or bank credentials; present only when a connection exists |
| status | enum | Not connected (represented by the absence of a record), Connecting (interim -- a first-connect hand-off is in progress and no readiness has yet been reported), Connected ("Ready to accept payments"), Needs attention (with the specific reason), or Disconnected (terminal outcome name only; never persisted) |
| available_payment_methods | derived (enum set) | Which of card and/or bank transfer are currently available on this freelancer's pay links |
| handoff_started_at | timestamp | When Nadia's latest Continue started a Connect or Reconnect hand-off; present only while that hand-off is in progress (also the marker that drives the Connecting screen state during a Reconnect, when `status` remains Needs attention); cleared when an outcome is applied |
| last_event_reported_at | timestamp | The report time of the most recently applied processor event, or the hand-off start time when a hand-off begins; used by FEAT-32.SPEC-003 to discard stale and duplicate events |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-32.SPEC-001 | Payment Connection Screen | Authorization on screen entry (which actions render for the viewer) and on each action attempt; the disconnect-warning dialog is shown before Disconnect proceeds |
| FEAT-32.SPEC-002 | Payment Account Connection & Status Reporting | Authorization check (Nadia, her own account) and the one-account limit check before a hand-off is initiated |
| FEAT-32.SPEC-003 | Connection Status Sync | Applies the processor-authoritative rule when reconciling a reported status against the current record |
| FEAT-32.SPEC-004 | Disconnect Payment Account | Executes only once the disconnect-warning acknowledgment and authorization gate here have passed; applies the in-flight-payment contention rule (the disconnect is never blocked by it) |
| FEAT-31.SPEC-002 (Operator Support Session Console, FEAT-31) | Dana's read-only connection view | Enforces the status-only visibility rule -- the processor account reference and any credential-adjacent detail are never rendered in her view |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| processor_account_reference | No validation beyond data type -- entirely supplied by the payment-processing capability once a hand-off confirms readiness; never entered or edited directly by any user | Always | -- | -- | -- |
| status | Must be one of Not connected (no record), Connecting, Connected, Needs attention, Disconnected (terminal, never persisted); Connecting is set only by FEAT-32.SPEC-002 when it creates the record for a first connect, and Connected/Needs attention only by FEAT-32.SPEC-003 (status reports); the record is removed by FEAT-32.SPEC-004 (disconnect) or by FEAT-32.SPEC-003 (failed first connect) -- never directly editable by any user | Always | On every hand-off start, processor-reported event, or disconnect action | N/A -- not a user-facing field; there is no direct-edit path to produce an invalid value | No (system-enforced, not user validation) |
| handoff_started_at | Set only by FEAT-32.SPEC-002 when Nadia confirms Continue in the consent notice; cleared only by FEAT-32.SPEC-003 when it applies any outcome; never entered by a user | Always | On Continue and on every applied outcome | N/A -- system-set | No (system-enforced) |
| last_event_reported_at | Set only by FEAT-32.SPEC-002 (to the hand-off start time) and FEAT-32.SPEC-003 (to an applied event's report time); never moves backward | Always | On every hand-off start and applied event | N/A -- system-set | No (system-enforced) |
| available_payment_methods | No user-facing validation -- fully derived from the payment-processing capability's latest report; never set by direct user input | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Needs attention implies no available methods | status, available_payment_methods | When status is Needs attention, available_payment_methods is set to none, regardless of what was available immediately before the restriction was reported | N/A -- system-derived, not a user-facing validation failure |
| Connected implies at least one available method | status, available_payment_methods | A readiness-confirmed event that reports zero available methods is treated the same as a restriction: FEAT-32.SPEC-003 (Processing Logic step 3) applies Needs attention with `available_payment_methods` none and the product-defined reason "Your payment account has no payment methods turned on yet. Turn on card or bank transfer in your payment account, then reconnect.", since "Connected" with nothing payable is not a meaningful ready state and the reason is never empty | N/A -- system-derived, not a user-facing validation failure; the reason text above is what Nadia sees on FEAT-32.SPEC-001 and in FEAT-32.SPEC-006's alert email |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Connect (create) | Nadia (Freelancer) | Always, provided no Payment Account Connection record currently exists for her account (the one-account-per-freelancer limit) | -- |
| Connect (create) | Nadia (Freelancer) | When a record already exists in any status (including interim Connecting) | Connect is not shown at all -- FEAT-32.SPEC-001 offers Reconnect and/or Disconnect as the status allows (none while Connecting); this is a structural absence, not a denial with an error message |
| Connect (create) | Owen (Client Primary Contact), Priya (Client Reviewer Contact) | Never | The action does not exist anywhere in the client portal -- no Payment Account settings surface is reachable by either role |
| Connect (create) | Dana (Support Operator) | Never | The Connect control is never shown inside Dana's read-only support session (FEAT-31); she has no path to initiate a connection |
| View status (Connected / Needs attention / Not connected, and available payment methods) | Nadia (Freelancer) | Always, her own account | -- |
| View status | Owen (Client Primary Contact), Priya (Client Reviewer Contact) | Never, directly | Neither role has any view of this entity; each experiences its effect only indirectly, as pay-link availability on their own company's invoices (FEAT-09), which is outside this entity entirely |
| View status (status field only) | Dana (Support Operator) | Always, inside a logged support session (FEAT-31), status only | -- |
| View processor_account_reference or any credential-adjacent detail | Dana (Support Operator) | Never | Dana's support-session view renders status only; the reference is never displayed to her, per feature-dependency-map.md's Data Sensitivity note ("the operator sees status only") |
| Reconnect | Nadia (Freelancer) | Always, when the record exists in Needs attention status. Reconnect is never offered when no record exists (post-disconnect or never-connected) -- that case offers Connect instead -- nor while Connecting or Connected | Not shown outside Needs attention -- a structural absence on FEAT-32.SPEC-001, not an error message |
| Reconnect | Owen, Priya, Dana | Never | Same as Connect -- no surface exists for these roles |
| Disconnect | Nadia (Freelancer) | Her own account, when the record is in Connected or Needs attention status, provided she has acknowledged the explicit disconnect warning (Business Rules, below). Not offered while Connecting or when no record exists | If she dismisses or cancels the warning instead of confirming, the disconnect does not proceed and the connection is unchanged; outside Connected and Needs attention the control is not shown (structural absence, no error message) |
| Disconnect | Owen, Priya, Dana | Never | Same as Connect and Reconnect -- no disconnect surface exists for these roles |
| Start over (abandon a stalled hand-off) | Nadia (Freelancer) | Her own account, only while a hand-off is in progress (`status` Connecting, or `handoff_started_at` present) and `handoff_started_at` is older than platform parameter: `payment-connect-handoff-timeout`; she must confirm the "Start over" dialog. Allowed even though Connect, Reconnect, and Disconnect are all hidden while Connecting | Before the threshold has elapsed, or with no hand-off in progress, the control is not shown (structural absence, no error message); if the request still arrives (for example from a stale tab) FEAT-32.SPEC-003 makes no change and the screen shows the state the record holds. Cancelling the dialog changes nothing |
| Start over | Owen, Priya, Dana | Never | Same as Connect -- no surface exists for these roles |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| status | The absence of a Payment Account Connection record represents the Not connected state; for a first connect, FEAT-32.SPEC-002 creates the record with `status` Connecting when Nadia confirms Continue; thereafter `status` is set only by FEAT-32.SPEC-003 (reported events) and the record is removed by FEAT-32.SPEC-003 (failed or abandoned first connect) or FEAT-32.SPEC-004 (disconnect) | On every hand-off, status report, or disconnect | No |
| handoff_started_at | Set to the Continue time by FEAT-32.SPEC-002 on every Connect or Reconnect hand-off; cleared by FEAT-32.SPEC-003 when any outcome is applied | On hand-off start and on every applied outcome | No |
| last_event_reported_at | Initialised to the hand-off start time by FEAT-32.SPEC-002; advanced by FEAT-32.SPEC-003 to each applied event's report time | On hand-off start and on every applied event | No |
| available_payment_methods | Derived entirely from the payment-processing capability's most recently applied report (FEAT-32.SPEC-003); cleared to none whenever status is Needs attention | On every applied status update | No |
| processor_account_reference | Supplied by the payment-processing capability once a hand-off's readiness is confirmed; removed entirely on disconnect | On Connect/Reconnect confirmation; removed on Disconnect | No |

## Business Rules

- One connected payment account per freelancer at a time (product-features.md, Validation & Limits) -- enforced structurally: FEAT-32.SPEC-002 never initiates a second Create while a record exists, and FEAT-32.SPEC-001 only ever offers Connect when none exists.
- **Connecting is an interim, persisted status:** for a first connect, FEAT-32.SPEC-002 creates the record with `status` Connecting (no `processor_account_reference` yet) when Nadia confirms Continue; this is what lets the screen show Connecting after she navigates away and returns. FEAT-32.SPEC-003 replaces it with Connected or Needs attention, or deletes the record if the first attempt fails or is abandoned, so no half-created record survives a failed first attempt. For a Reconnect the existing record keeps its `status` and only `handoff_started_at` is set.
- XBR-19: with no connected account, invoices still issue with fallback instructions for paying Nadia directly; a Needs attention connection shows the pay screen's "temporarily unavailable" fallback; disconnecting warns that open invoices lose their pay links until reconnection.
- **Disconnect Warning:** Before a disconnect proceeds, Nadia is shown the exact dialog: "If you disconnect, your open invoices will lose their pay links until you connect a payment account again. Any payment already submitted to your processor will not be affected." with "Disconnect" and "Cancel" options. Disconnect proceeds only on "Disconnect"; "Cancel" leaves the connection unchanged.
- **Processor-authoritative contention rule:** A disconnect issued while a client's payment is already submitted to the processor is never blocked by that in-flight payment, and the in-flight payment is never cancelled by the disconnect (feature-dependency-map.md, Entity: Payment Account Connection, Contention). Once the disconnect completes, any pay link opened afterward reflects the now-removed connection, per the fallback rule FEAT-09.SPEC-009 and FEAT-10 own (XBR-19); this spec defines the underlying protection, not the rendered pay-screen copy.
- **Stalled hand-off exit:** Connecting is never a dead end. Once `handoff_started_at` is older than platform parameter: `payment-connect-handoff-timeout`, Nadia alone may confirm "Start over" on FEAT-32.SPEC-001; FEAT-32.SPEC-003 then applies its abandoned-hand-off handling (first connect: interim record deleted; Reconnect: only `handoff_started_at` cleared). Start over is the only action allowed while Connecting, and only past the threshold; Connect, Reconnect, and Disconnect stay unavailable in that status.
- Reconnect always re-runs the full Connect hand-off (FEAT-32.SPEC-002) rather than resuming a partial or prior connection -- there is no partial-state resume, consistent with the entity's hard-delete-on-disconnect lifecycle.
- Processor-reported status is authoritative over any other party's expectation of the connection's state (feature-dependency-map.md, Contention) -- applied by FEAT-32.SPEC-003.

## Edge Cases

- **Nadia attempts to disconnect twice in rapid succession (double-tap)** -- The second attempt, once the first has already removed the reference, finds nothing left to remove and is a no-op (FEAT-32.SPEC-004's idempotent handling); no error is shown for the second attempt.
- **The one-account limit at its exact boundary** -- Nadia has zero records: Connect is offered (and Reconnect is not). Nadia has exactly one record, in any status: Connect is never offered, only Reconnect and/or Disconnect as the status allows (neither while Connecting). No value between zero and one is possible, since the entity carries no count field and no list view exists (product-features.md, Validation & Limits).
- **Dana's read-only support session is open on the connection status at the exact moment Nadia disconnects** -- Dana's session reflects the new state (Not connected) on her next refresh; her session never continues showing a reference or status for a record that no longer exists.
- **A disconnect is confirmed while a client's payment is mid-submission to the processor** -- The in-flight payment proceeds to its own outcome entirely independent of the disconnect (FEAT-10.SPEC-003); the connection reference is removed regardless, and any pay link opened even moments later reflects the removed connection.
- **Nadia cancels the disconnect warning dialog** -- No data changes; the connection remains exactly as it was, and FEAT-32.SPEC-004 is never triggered.
- **A restriction event is applied (Needs attention) while Nadia is mid-way through confirming a disconnect** -- The disconnect, once confirmed, still proceeds and removes the record regardless of the just-applied Needs attention status; her own explicit disconnect is not blocked by an intervening status change.

## Acceptance Criteria

**FEAT-32.SPEC-005-AC-01:** Given Nadia has no existing Payment Account Connection record, when she attempts to Connect, then the action is allowed.

**FEAT-32.SPEC-005-AC-02:** Given Nadia already has a Payment Account Connection record in any status, when she views FEAT-32.SPEC-001, then no Connect action is shown -- only Reconnect (Needs attention) and/or Disconnect (Connected or Needs attention), and neither while Connecting.

**FEAT-32.SPEC-005-AC-03:** Given Owen (Client Primary Contact) is signed in, when he looks for any way to connect, view, reconnect, or disconnect a payment account, then no such action or view exists anywhere in his portal.

**FEAT-32.SPEC-005-AC-04:** Given Priya (Client Reviewer Contact) is signed in, when she looks for any way to interact with this entity, then no such action or view exists anywhere in her portal.

**FEAT-32.SPEC-005-AC-05:** Given Dana (Support Operator) is inside a logged support session on Nadia's account, when she views the payment connection, then she sees the status only, never the processor account reference or any credential-adjacent detail.

**FEAT-32.SPEC-005-AC-06:** Given Dana is inside a logged support session, when she looks for a Connect, Reconnect, or Disconnect control, then none is shown -- her session is view-only.

**FEAT-32.SPEC-005-AC-07:** Given Nadia's connection is in Needs attention status, when she attempts to Reconnect, then the action is allowed.

**FEAT-32.SPEC-005-AC-08:** Given Nadia has no connection record (post-disconnect or never connected), when she views FEAT-32.SPEC-001, then only Connect is offered and no Reconnect control appears.

**FEAT-32.SPEC-005-AC-09:** Given Nadia's connection is Connected or Needs attention, when she initiates Disconnect, then the warning dialog "If you disconnect, your open invoices will lose their pay links until you connect a payment account again. Any payment already submitted to your processor will not be affected." appears before anything is removed.

**FEAT-32.SPEC-005-AC-10:** Given the disconnect warning dialog is shown, when Nadia taps "Cancel," then the connection is unchanged and no removal occurs.

**FEAT-32.SPEC-005-AC-11:** Given the disconnect warning dialog is shown, when Nadia taps "Disconnect," then the removal proceeds via FEAT-32.SPEC-004.

**FEAT-32.SPEC-005-AC-12:** Given a client's payment is already submitted to the processor, when Nadia confirms a disconnect, then the disconnect is not blocked and the in-flight payment is not cancelled.

**FEAT-32.SPEC-005-AC-13:** Given the connection is removed by disconnect, when a pay link is opened afterward, then it reflects the now-disconnected state per the fallback rule owned by FEAT-09.SPEC-009 and FEAT-10.

**FEAT-32.SPEC-005-AC-14:** Given a readiness-confirmed event reports zero available payment methods, when FEAT-32.SPEC-003 evaluates it (Processing Logic step 3), then the outcome is Needs attention rather than Connected, `available_payment_methods` is none, and the reason is "Your payment account has no payment methods turned on yet. Turn on card or bank transfer in your payment account, then reconnect."

**FEAT-32.SPEC-005-AC-15:** Given the connection's status is set to Needs attention, when the update is applied, then `available_payment_methods` is cleared to none regardless of what was available immediately before.

**FEAT-32.SPEC-005-AC-16:** Given Nadia double-taps Disconnect, when the second tap reaches processing after the first has already removed the reference, then it is a no-op with no error shown.

**FEAT-32.SPEC-005-AC-17:** Given no field on this entity is ever directly editable by a user, when any party attempts to set `status`, `processor_account_reference`, or `available_payment_methods` directly, then no such input path exists -- all three are set only through FEAT-32.SPEC-002, SPEC-003, or SPEC-004.

**FEAT-32.SPEC-005-AC-18:** Given a restriction event (Needs attention) is applied while Nadia is mid-way through confirming a disconnect, when she then confirms the disconnect, then the removal still proceeds regardless of the just-applied Needs attention status.

**FEAT-32.SPEC-005-AC-19:** Given Nadia's hand-off is in progress and `handoff_started_at` is older than platform parameter: `payment-connect-handoff-timeout` with no outcome applied, when she views FEAT-32.SPEC-001, then "Start over" is allowed (and Connect, Reconnect, and Disconnect remain hidden), and confirming it lets FEAT-32.SPEC-003 apply the abandoned-hand-off handling.

**FEAT-32.SPEC-005-AC-20:** Given a hand-off is in progress but `handoff_started_at` is not yet older than platform parameter: `payment-connect-handoff-timeout`, or no hand-off is in progress, when Nadia views FEAT-32.SPEC-001, then no "Start over" control is shown, and a "Start over" request that arrives anyway changes nothing.

**FEAT-32.SPEC-005-AC-21:** Given Owen, Priya, or Dana is signed in, when any of them looks for a "Start over" control, then none exists on any surface available to them.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 14 | 14 |
| Defaults/Derivations | 5 | 5 |
| Business Rules | 8 | 8 |
| Edge Cases | 6 | 6 |

---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-05.SPEC-009
spec_name: Policy Acknowledgment Capture & Integrity Check
spec_slug: policy-acknowledgment-capture-integrity-check
parent_feature: FEAT-05
parent_feature_name: Public Booking Page & Booking Flow
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 7
acceptance_criteria_count: 14
---

# Logic/Rule Spec: Policy Acknowledgment Capture & Integrity Check

## Overview

**Name:** Policy Acknowledgment Capture & Integrity Check
**ID:** FEAT-05.SPEC-009
**Type:** Logic/Rule
**Purpose:** Computes this booking's exact deposit amount and cancellation cut-off, captures the acknowledged cancellation policy version and wording into the in-progress checkout (carried onto the Booking when FEAT-05.SPEC-006 creates it), and re-validates that version still matches at payment time.
**Parent Feature:** FEAT-05 -- Public Booking Page & Booking Flow
**Governed Entity:** Booking (policy acknowledgment fields: price_agreed, deposit_amount, policy_version, acknowledgment timestamp, acknowledged wording)

## Scope and Non-Goals

**In Scope:**
- Computing the exact deposit amount for this booking from the Service's deposit_rule and the Pro's account currency
- Computing the exact cancellation cut-off time for this booking from the current Cancellation Policy's window_hours and the booking's start_time
- Recording the acknowledged Cancellation Policy version, its exact plain-language wording, and the acknowledgment timestamp into the in-progress checkout the moment the client checks the acknowledgment box; FEAT-05.SPEC-006 carries them onto the Booking when it creates it
- Re-validating, when the client advances into the payment step (Acknowledge & continue on FEAT-05.SPEC-004, before the checkout hold and hand-off to FEAT-07.SPEC-001), that the acknowledged version still matches the Pro's current Cancellation Policy version

**Non-Goals:**
- Defining or versioning the Cancellation Policy itself -- owned by FEAT-09 (Cancellation & No-Show Policy Engine); this spec only reads the current version and records which one was shown
- Displaying the policy block or the acknowledgment checkbox -- owned by FEAT-05.SPEC-004, which calls this spec's computation and capture logic but owns the screen presentation
- Defining or editing the Service's deposit_rule -- owned by FEAT-01 (Service & Pricing Management); this spec only reads it to compute the exact amount
- Determining the deposit's eventual outcome (refunded, kept, forfeited) once the appointment's cancellation window opens or closes -- owned by FEAT-09 and FEAT-11; this spec only captures what was agreed at booking time, not what happens afterward

## Governed Entity

**Entity:** Booking (policy acknowledgment fields)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| price_agreed | number | The service price fixed at booking time, in the Pro's account currency |
| deposit_amount | number | The exact deposit computed for this booking, fixed at booking time |
| policy_version | reference | The specific Cancellation Policy version shown and acknowledged for this booking |
| acknowledgment_timestamp (part of policy_version field per the dependency map: "with acknowledgment time") | date/time | The exact moment the client checked the acknowledgment box |
| acknowledged_wording (recorded alongside policy_version for dispute evidence, per Non-Functional Notes) | text | The exact plain-language wording shown at the moment of acknowledgment |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|---------------------|
| FEAT-05.SPEC-004 | Policy Acknowledgment & Deposit Checkout | Computation runs on screen load (to display the amount and cut-off); capture runs the instant the acknowledgment checkbox is checked; the integrity re-check runs when the client taps Acknowledge & continue, before the checkout hold is requested and the client is navigated to FEAT-07.SPEC-001 |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| deposit_amount | Must be computed, not entered -- derived exactly once from the Service's current deposit_rule and price at the moment the client reaches this screen; never editable by the client | Always | On screen load (computation), re-verified not to have changed at payment time as part of the Service's own contention rule (owned by FEAT-01, referenced here) | Not applicable -- this is a system-derived value with no client-facing error state | Yes (computation must complete before Pay is offered) |
| policy_version | Must reference an existing Cancellation Policy version at the moment of acknowledgment | Always | When the acknowledgment checkbox is checked | Not applicable -- if no Cancellation Policy exists at all, the booking page itself would not be live (XBR-26 requires the policy to be set before go-live) | Yes |
| acknowledgment_timestamp | No validation beyond data type -- always system-derived, never user-entered | Always | -- | -- | -- |
| acknowledged_wording | No validation beyond data type -- captured verbatim from the current wording at the moment of acknowledgment, never user-entered | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|-----------------|
| Version integrity at continue | policy_version (as recorded at acknowledgment), the Pro's current Cancellation Policy version | The recorded policy_version must still equal the Pro's current Cancellation Policy version at the instant the client taps Acknowledge & continue into the payment step; if the Pro's current version has advanced since acknowledgment, the check fails | "The cancellation policy has changed since you agreed to it. Please review the updated terms." |
| Deposit-price consistency | deposit_amount, price_agreed, Service.deposit_rule | deposit_amount must equal the value produced by applying the Service's current deposit_rule to price_agreed at the moment of computation (fixed form: the flat amount; percentage form: price_agreed x percentage / 100); the two are never computed independently or allowed to diverge | Not applicable -- this is an internal computation consistency rule with no client-facing error path of its own |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Trigger the deposit/cut-off computation and acknowledgment capture | The Client (Riley) | Only for their own in-progress Booking | -- |
| Trigger the computation and capture in preview mode | The Pro (Talia) | Always, but the result is never persisted to a real Booking | -- |
| Read the recorded acknowledgment (version, wording, timestamp) on a Booking | The Pro (Talia) | Always, for her own bookings (dispute evidence) | -- |
| Edit or delete a recorded acknowledgment once captured | The Client (Riley), The Pro (Talia) | Never -- the recorded acknowledgment is immutable once written, forming dispute evidence (Non-Functional Notes) | Not applicable -- no edit or delete control exists for this data anywhere in the product |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| price_agreed | Copied from the Service's current price at the moment the client reaches FEAT-05.SPEC-004 | On computation (screen load) | No |
| deposit_amount | Fixed form: the Service's deposit_rule flat amount. Percentage form: price_agreed x (deposit_rule percentage / 100), in the Pro's account currency | On computation (screen load) | No |
| Cancellation cut-off (displayed, not itself a stored Booking field -- derived for display from policy_version and the Booking's start_time) | Booking.start_time minus the acknowledged Cancellation Policy's window_hours | On computation (screen load) and whenever the display refreshes after a version change | No |
| policy_version | The Pro's current Cancellation Policy version at the moment the client checks the acknowledgment checkbox | On acknowledgment (checkbox checked) | No |
| acknowledged_wording | The Cancellation Policy's plain_language_wording at that same moment, captured verbatim | On acknowledgment (checkbox checked) | No |
| acknowledgment_timestamp | The current date and time | On acknowledgment (checkbox checked) | No |

## Business Rules

- The deposit is computed once, exactly, from the service's rule in the Pro's account currency, cannot be altered by the client, and is charged once per booking (XBR-05).
- Every booking is governed by the cancellation policy version shown and acknowledged at booking; policy edits never change existing bookings once the acknowledgment is captured (XBR-08).
- If the Pro's current Cancellation Policy version changes between the client's acknowledgment and the moment the client taps Acknowledge & continue, the client is refused with refresh and asked to acknowledge the current wording -- existing bookings keep their own already-recorded version regardless (dependency map's Cancellation Policy contention note).
- The recorded acknowledgment (version, wording, timestamp) forms dispute evidence and must be retained with the Booking for as long as the Booking exists (Non-Functional Notes; SC-22).
- This spec's derivations run identically in preview mode, so the Pro reviews the exact computation her own settings would produce -- but the result is discarded rather than persisted to a real Booking.

## Edge Cases

- **The Pro edits the Cancellation Policy's wording without changing the substantive window_hours, creating a new version anyway (per FEAT-09's every-edit-creates-a-new-version rule)** -- The integrity check still fails against the acknowledged version, since versions are compared by identity, not by substantive difference; the client must re-acknowledge even a wording-only change.
- **The client acknowledges, the Pro edits the policy, and the client re-acknowledges the new version, all before payment** -- The second acknowledgment overwrites the first with the new version, wording, and timestamp; only the most recent acknowledgment is carried onto the Booking when FEAT-05.SPEC-006 creates it.
- **The client continues at the exact same version that was acknowledged, with no changes in between** -- The integrity check passes and the client advances to the payment step without any re-acknowledgment prompt.
- **The Service's deposit_rule is edited by the Pro after the client reaches FEAT-05.SPEC-004 but before payment** -- Per the Service entity's contention rule (dependency map), the client pays the amount shown when they acknowledged the policy; a service archived (not merely edited) before payment instead refuses the client with a refresh, per FEAT-05.SPEC-004's Business Rules.
- **The computed cancellation cut-off falls at a boundary moment (e.g., exactly at the window_hours mark)** -- The cut-off is computed as an exact timestamp (start_time minus window_hours to the minute); a cancellation exactly at that timestamp is evaluated by FEAT-09's own boundary rule, not re-derived here -- this spec only displays and records the computed value.
- **Two different clients acknowledge the same current policy version for two different bookings at the same time** -- Each Booking records its own independent copy of the version, wording, and timestamp; there is no shared or contended record between them.

## Acceptance Criteria

**FEAT-05.SPEC-009-AC-01:** Given Riley reaches FEAT-05.SPEC-004 for a fixed-amount-deposit service, when the screen loads, then the exact deposit amount shown equals the Service's fixed deposit_rule amount.

**FEAT-05.SPEC-009-AC-02:** Given Riley reaches FEAT-05.SPEC-004 for a percentage-deposit service, when the screen loads, then the exact deposit amount shown equals price_agreed multiplied by the deposit percentage, in the Pro's account currency.

**FEAT-05.SPEC-009-AC-03:** Given Riley reaches FEAT-05.SPEC-004, when the cancellation cut-off is displayed, then it equals the booking's start time minus the current Cancellation Policy's window_hours, shown as an exact date and time.

**FEAT-05.SPEC-009-AC-04:** Given Riley checks the acknowledgment checkbox, when the check registers, then the in-progress checkout's policy_version, acknowledged_wording, and acknowledgment_timestamp (carried onto the Booking when FEAT-05.SPEC-006 creates it) are set to the current Cancellation Policy version, its exact wording, and the current moment.

**FEAT-05.SPEC-009-AC-05:** Given Riley has acknowledged one policy version, when the Pro edits the Cancellation Policy before Riley continues, then Riley's Acknowledge & continue attempt fails the integrity check and she sees "The cancellation policy has changed since you agreed to it. Please review the updated terms."

**FEAT-05.SPEC-009-AC-06:** Given Riley's Acknowledge & continue attempt fails the integrity check, when she reviews and re-checks the acknowledgment box against the refreshed wording, then the in-progress checkout's policy_version, acknowledged_wording, and acknowledgment_timestamp are overwritten with the new version's values.

**FEAT-05.SPEC-009-AC-07:** Given Riley acknowledges the current policy version and no change occurs before she continues, when she taps Acknowledge & continue, then the integrity check passes and she advances to the payment step without a re-acknowledgment prompt.

**FEAT-05.SPEC-009-AC-08:** Given a completed Booking's policy_version, acknowledged_wording, and acknowledgment_timestamp, when Talia (the Pro) views the Booking's record, then all three values are visible exactly as recorded.

**FEAT-05.SPEC-009-AC-09:** Given a completed Booking's recorded acknowledgment, when any role attempts to edit or delete it, then no such control exists anywhere in the product.

**FEAT-05.SPEC-009-AC-10:** Given Talia previews her own booking page and reaches FEAT-05.SPEC-004, when she checks the acknowledgment checkbox, then the identical computation and capture logic runs, but no real Booking record is persisted.

**FEAT-05.SPEC-009-AC-11:** Given the Pro edits only the Cancellation Policy's wording (not the window_hours) after Riley acknowledges, when Riley taps Acknowledge & continue, then the integrity check still fails, since versions are compared by identity, not by substantive difference.

**FEAT-05.SPEC-009-AC-12:** Given the Service's deposit_rule is edited by the Pro after Riley reaches FEAT-05.SPEC-004 but the service is not archived, when Riley continues to payment, then she is charged the amount shown when she acknowledged the policy, not the newly edited amount.

**FEAT-05.SPEC-009-AC-13:** Given two different clients acknowledge the same current Cancellation Policy version at the same time for two different bookings, when both acknowledgments are recorded, then each Booking holds its own independent copy of the version, wording, and timestamp.

**FEAT-05.SPEC-009-AC-14:** Given a Booking has an acknowledgment recorded, when the Booking is later referenced for a deposit dispute (FEAT-16), then the recorded version, wording, and timestamp remain available and unchanged from the moment of acknowledgment.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 4 | 4 |
| Defaults/Derivations | 6 | 6 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

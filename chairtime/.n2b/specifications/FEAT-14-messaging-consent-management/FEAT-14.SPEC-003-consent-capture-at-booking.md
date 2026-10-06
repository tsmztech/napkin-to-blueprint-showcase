---
document_type: spec
spec_type: automation
spec_id: FEAT-14.SPEC-003
spec_name: Consent Capture at Booking
spec_slug: consent-capture-at-booking
parent_feature: FEAT-14
parent_feature_name: Messaging Consent Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Automation Spec: Consent Capture at Booking

## Overview

**Name:** Consent Capture at Booking
**ID:** FEAT-14.SPEC-003
**Type:** Automation
**Purpose:** Records the client's opt-in or opt-out choice made at booking as a Messaging Consent record, with state, timestamp, channel, and the exact consent wording shown, creating the record on a client's first booking with a Pro and updating it on a later booking if the choice changes.
**Parent Feature:** FEAT-14 -- Messaging Consent Management

## Scope and Non-Goals

**In Scope:**
- Creating the Messaging Consent record on a client's first booking with a given Pro
- Updating the existing Messaging Consent record when a returning client makes a different choice on a later booking
- Capturing the exact consent wording shown at the moment of the choice, as compliance evidence

**Non-Goals:**
- The opt-in checkbox and its never-pre-checked default -- owned by FEAT-05.SPEC-003 (Client Details & Consent); this automation begins once that screen's booking submission carries the client's choice, and never renders the checkbox itself.
- Processing a revoke via opt-out link or STOP reply -- owned by FEAT-14.SPEC-004; this automation only ever writes Granted (a fresh opt-in) or the initial Revoked state (an opt-out at booking), never a mid-relationship revoke.
- Resolving a race between this automation's write and a concurrent STOP reply or re-grant -- owned by FEAT-14.SPEC-006; this automation always fires from a single, sequential booking submission and has no concurrent-trigger case of its own beyond the general two-run-in-flight edge case below.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Client submits a booking with an opt-in/opt-out choice (first booking with this Pro) | FEAT-05.SPEC-003 (Client Details & Consent), where the choice is captured, and FEAT-05.SPEC-004 (Policy Acknowledgment & Deposit Checkout), where the booking is submitted | Fires on a successful booking submission where no Messaging Consent record yet exists for this Client-Pro relationship | The client's opt-in/opt-out choice, the exact consent wording shown on FEAT-05.SPEC-003, the client's phone number, the current timestamp |
| Client submits a later booking with a different opt-in/opt-out choice than their existing record | FEAT-05.SPEC-003 (Client Details & Consent), where the new choice is captured, and FEAT-05.SPEC-004 (Policy Acknowledgment & Deposit Checkout), where the booking is submitted | Fires on a successful booking submission where a Messaging Consent record already exists for this Client-Pro relationship and the newly submitted choice differs from its current state | The client's new choice, the exact consent wording shown at this booking, the client's phone number, the current timestamp, the existing record's prior state |

## Processing Logic

1. Receive the booking submission's opt-in/opt-out choice, the exact consent wording text shown on FEAT-05.SPEC-003 at that moment, the client's phone number, and the timestamp of submission.
2. Check whether a Messaging Consent record already exists for this Client-Pro relationship.
3. If no record exists: create one with channel set to text, state set to Granted (if the client opted in) or Revoked (if the client opted out), timestamp set to the submission time, consent_wording set to the exact text shown, and phone_number set to the client's phone number at booking.
4. If a record already exists: compare the newly submitted choice to the record's current state.
   - If the choice matches the current state, leave the record unchanged (no-op; the booking's own acknowledgment is still logged by FEAT-16, but this automation makes no write).
   - If the choice differs, update the record: set state to Granted or Revoked per the new choice, timestamp to the current submission time, consent_wording to the wording shown at this booking, and phone_number to the client's current phone number.
5. Confirm the write completed before allowing FEAT-05's booking confirmation step to proceed to FEAT-08's confirmation-message channel selection.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| First-booking creation | No prior Messaging Consent record exists for this relationship | Messaging Consent record created with the submitted choice | None distinct from the booking confirmation itself -- consent capture is silent and folded into the booking flow | FEAT-05.SPEC-005 (Booking Confirmation), FEAT-08.SPEC-011 (reads the resulting state for the confirmation message's channel) |
| Later-booking update (choice changed) | A record exists and the new choice differs from its current state | Messaging Consent record's state, timestamp, consent_wording, and phone_number updated | None distinct from the booking confirmation itself | FEAT-05.SPEC-005, FEAT-08.SPEC-011, FEAT-14.SPEC-001 (the client's next view of their status reflects the change) |
| Later-booking no-op (choice unchanged) | A record exists and the new choice matches its current state | None | None distinct from the booking confirmation itself | -- |
| Write failure | The consent write cannot complete (a processing error, not a validation failure -- the choice itself is always a simple binary value) | No Messaging Consent record is created or updated | The booking submission itself is not blocked or rolled back by a consent-write failure alone; the booking proceeds, and the client's channel defaults to the safer no-text state (email) until the write can be confirmed, per this feature's own Error-state discipline | FEAT-08.SPEC-011 (defaults to email when consent state is unconfirmed) |

## Data Model

**Creates:** Messaging Consent -- channel, state, timestamp, consent_wording, phone_number, on a client's first booking with a Pro.
**Reads:** Client -- to identify the owning Client-Pro relationship and current phone_number (read-only; this automation never writes the Client entity).
**Updates:** Messaging Consent -- state, timestamp, consent_wording, phone_number, on a later booking with a changed choice.
**Deletes:** None.

## Business Rules

- XBR-15: no text is sent without active texting consent; this automation is the sole creation path for that consent record, and its write must complete before any message tied to the same booking selects a channel (FEAT-08.SPEC-011).
- Consent capture must never be pre-checked at the source screen (FEAT-05.SPEC-003); this automation records exactly the choice the client made, never a default assumption.
- The consent_wording captured is the literal text shown to the client at that specific booking, not a generic or later-edited version of the disclosure -- consistent with ASMP-24's evidentiary requirement.
- A client booking with a second Pro creates an entirely separate, unconnected Messaging Consent record (scope-boundaries SC-04); this automation never reuses or looks up a consent record from a different Pro relationship.
- A phone number change invalidates the existing record before this automation would next run for that relationship (FEAT-14.SPEC-008); if this automation fires for a booking made under a newly changed number with no fresh consent yet on file, it treats the booking as if capturing consent for the first time under that number.

## Edge Cases

- **Client submits a booking with a phone number that differs from any existing Messaging Consent record's phone_number for this same Pro relationship (a very recent number change)** -- Per FEAT-14.SPEC-008, the prior record was already invalidated by the number change; this automation treats the submission as fresh consent capture for the new number and updates the record's phone_number, state, timestamp, and consent_wording accordingly.
- **The exact consent wording shown includes dynamic content (e.g., the Pro's display name)** -- The wording is captured exactly as rendered at that moment, including any such substitution, since the evidentiary requirement is what the client actually saw, not a template.
- **A booking is submitted and then immediately cancelled before payment completes** -- If FEAT-05's checkout re-validation (FEAT-05.SPEC-006) rejects the booking before this automation's trigger condition (a successful booking submission) is met, this automation never fires; a rejected or abandoned checkout never produces or updates a Messaging Consent record.
- **Concurrent trigger firing (two devices submit a booking for the same Client-Pro relationship at effectively the same time -- not realistically possible for the same client, but a data-consistency case worth stating)** -- The Client entity's own contention rule (phone-number match resolves to a single record) applies first; whichever booking submission commits first at the Client/Booking level is the one this automation processes, and the second submission's consent write, if it differs, is handled as a later-booking update against the just-created record, following the standard update path in Processing Logic step 4.
- **Trigger fires while a previous run for the same relationship is still in flight** -- A second submission for the same Client-Pro relationship cannot be in flight at the same time in practice, since FEAT-05's checkout flow serializes one booking submission at a time per client session; if it were to occur, the second run's read of "does a record exist" would wait for the first run's write to commit, and then proceed as a later-booking update rather than a duplicate creation.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-05.SPEC-003 (Client Details & Consent) | Triggered by (inbound) | Source of the opt-in/opt-out choice and the exact consent wording shown |
| FEAT-05.SPEC-004 (Policy Acknowledgment & Deposit Checkout) | Triggered by (inbound) | Booking submission is the moment this automation fires |
| FEAT-05.SPEC-005 (Booking Confirmation) | Affects (outbound) | The confirmation step proceeds only after this automation's write completes |
| FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule) | Affects (outbound) | Reads the resulting Messaging Consent state to choose the confirmation message's channel |
| FEAT-14.SPEC-008 (Phone Number Change Consent Invalidation Rule) | References (inbound) | Governs the state this automation finds when a phone number changed since the client's last booking |
| FEAT-14.SPEC-001 (Consent & Preferences) | Affects (outbound) | The client's next view of their status reflects any change this automation makes |

## Analytics and Success Signals

- **consent_captured** (outcome: created / updated / no_op; choice: opt_in / opt_out) -- N/A -- no metric in success-metrics.md names Messaging Consent Management as its Connected Feature or references consent capture; retained as an operational signal so opt-in/opt-out volume at booking is observable.
- **consent_write_failed** (booking reference) -- N/A -- same reason as above; retained so a silent consent-capture failure is never invisible, consistent with this feature's own Error-state discipline defaulting to no-text on doubt.

## Acceptance Criteria

**FEAT-14.SPEC-003-AC-01:** Given Riley books with Talia for the first time and checks the opt-in box, when her booking submission succeeds, then a Messaging Consent record is created with state Granted, the exact wording shown, her phone number, and the submission timestamp.

**FEAT-14.SPEC-003-AC-02:** Given Riley books with Talia for the first time and leaves the opt-in box unchecked, when her booking submission succeeds, then a Messaging Consent record is created with state Revoked and the same wording/timestamp/phone_number capture.

**FEAT-14.SPEC-003-AC-03:** Given Riley already has an active (Granted) Messaging Consent record with Talia, when she books again and leaves her choice unchanged, then no write occurs to the existing record.

**FEAT-14.SPEC-003-AC-04:** Given Riley's existing Messaging Consent record with Talia is Revoked, when she books again and checks the opt-in box this time, then the record is updated to Granted with a fresh timestamp and wording capture.

**FEAT-14.SPEC-003-AC-05:** Given Riley books with a second Pro, Jordan, for the first time, when her booking submission succeeds, then a wholly separate Messaging Consent record is created for the Riley-Jordan relationship, unconnected to her Riley-Talia record.

**FEAT-14.SPEC-003-AC-06:** Given Riley's booking submission is rejected by checkout re-validation before payment completes, when the rejection occurs, then no Messaging Consent record is created or updated.

**FEAT-14.SPEC-003-AC-07:** Given the consent write fails due to a processing error after a successful booking, when the failure occurs, then the booking itself still completes and the client's channel defaults to email until the write is confirmed.

**FEAT-14.SPEC-003-AC-08:** Given Riley's phone number changed since her last booking with Talia and her prior consent was invalidated per FEAT-14.SPEC-008, when she submits a new booking with a fresh opt-in choice, then this automation treats it as first-time capture under the new number.

**FEAT-14.SPEC-003-AC-09:** Given the consent wording shown to Riley at this specific booking includes Talia's display name, when the record is created or updated, then the consent_wording field stores that exact rendered text.

**FEAT-14.SPEC-003-AC-10:** Given this automation's write for Riley's booking has not yet committed, when FEAT-08.SPEC-011 evaluates the channel for the resulting confirmation message, then it waits for the write to complete before selecting a channel, per the sequencing in Processing Logic step 5.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (first booking, later booking with changed choice) | 2 |
| Outcome Paths | 4 (creation, update, no-op, write failure) | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |

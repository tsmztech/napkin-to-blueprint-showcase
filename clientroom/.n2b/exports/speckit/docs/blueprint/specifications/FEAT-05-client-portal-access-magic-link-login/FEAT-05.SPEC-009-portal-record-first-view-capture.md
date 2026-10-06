---
document_type: spec
spec_type: automation
spec_id: FEAT-05.SPEC-009
spec_name: Portal Record First-View Capture
spec_slug: portal-record-first-view-capture
parent_feature: FEAT-05
parent_feature_name: Client Portal Access (Magic-Link Login)
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Automation Spec: Portal Record First-View Capture

## Overview

**Name:** Portal Record First-View Capture
**ID:** FEAT-05.SPEC-009
**Type:** Automation
**Purpose:** Detects a client contact's first view of a proposal, deliverable, or invoice reached through the portal and hands the timestamped event to FEAT-13's audit trail.
**Parent Feature:** FEAT-05 -- Client Portal Access (Magic-Link Login)

## Scope and Non-Goals

**In Scope:**
- Detecting the first time a portal session opens a proposal, deliverable, or invoice screen, whichever feature owns that screen
- Capturing the timestamp of that first view exactly once per record per contact
- Handing the timestamped event to FEAT-13, which writes the append-only trail entry
- Idempotency: a re-view of an already-first-viewed record produces no further event

**Non-Goals:**
- Writing the Activity Log Entry itself -- owned by FEAT-13; this automation only detects the moment and hands off the event.
- Displaying the resulting `first_client_view_at` field (e.g., on the Deliverable) -- owned by the record's own feature (FEAT-03, FEAT-06/FEAT-07, FEAT-09/FEAT-10), which exposes the field this automation's event ultimately feeds.
- Detecting views that happen outside a portal session (e.g., the freelancer's own view of her sent proposal) -- this automation is scoped to the portal session FEAT-05 owns; a freelancer's own screens have no client "first view" to detect.
- Any retention or purge decision for first-view data -- excluded per this feature's own Non-Goals: `last_sign_in` and issued-link data carry no retention decision of this feature's own, and first-view events are FEAT-13's evidentiary record, governed by FEAT-13's own retention (life of the account, per the dependency map).

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Contact opens the Proposal Review & Accept screen | FEAT-03 (Proposal Acceptance) | Fires when a portal session opens that screen for a specific Proposal | Client Contact reference, Proposal reference, current timestamp |
| Contact opens a deliverable review screen | FEAT-06 (Deliverable Upload & Sharing) / FEAT-07 (Deliverable Review & Feedback) | Fires when a portal session opens that screen for a specific Deliverable | Client Contact reference, Deliverable reference, current timestamp |
| Contact opens the invoice list or pay-invoice screen | FEAT-09 (Invoice Generation & Sending) / FEAT-10 (Invoice Payment Processing) | Fires when a portal session opens that screen for a specific Invoice | Client Contact reference, Invoice reference, current timestamp |

## Processing Logic

1. Receive the opening event from the triggering screen: which portal session opened it, which record (Proposal, Deliverable, or Invoice) it opened, and the current timestamp.
2. Check whether a first-view event already exists for that specific record and that specific Client Contact.
3. If a first-view event already exists, take no further action (No-Action outcome).
4. If no first-view event exists for that record/contact pair, capture the current timestamp as the first-view moment.
5. Hand the timestamped event to FEAT-13 (XBR-05): event_type "first client view," actor the Client Contact, occurred_at the captured timestamp, affected_record the specific Proposal, Deliverable, or Invoice, project the record's owning project.
6. The record's own owning feature (FEAT-03, FEAT-06/FEAT-07, or FEAT-09/FEAT-10) reads this event to populate its own first-view field (e.g., Deliverable's `first_client_view_at`) through its own data flow -- this automation's responsibility ends at the FEAT-13 hand-off.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| First view captured | No prior first-view event exists for this record/contact pair | A timestamped first-view event is created and handed to FEAT-13 | None visible to the contact -- the capture is silent and does not alter the screen they are viewing | FEAT-13 (trail entry); the owning record's feature (FEAT-03, FEAT-06/FEAT-07, FEAT-09/FEAT-10), which later exposes the resulting field |
| No action (already viewed) | A first-view event already exists for this record/contact pair | None | None visible -- identical to a normal view of an already-viewed record | None beyond the triggering screen, which renders normally either way |
| Failure (hand-off to FEAT-13 fails) | The event hand-off cannot be delivered | No first-view event is durably recorded | None visible to the contact -- the screen renders normally regardless of the trail write's outcome, since a client contact must never see the platform's internal audit mechanics | FEAT-13 (retried per its own delivery guarantee) |

## Data Model

**Reads:** Client Contact -- identity of the portal session's contact; Proposal, Deliverable, or Invoice -- the reference to the record being viewed and its `first_client_view_at`-equivalent state (to check whether a first view already exists).
**Creates:** First-view event (feature-internal, not a Domain Entity Inventory record) -- one per record per contact, handed to FEAT-13.
**Updates:** None directly by this automation -- the owning record's `first_client_view_at`-equivalent field (e.g., Deliverable's `first_client_view_at`) is written by that record's own feature (FEAT-06) once it consumes this automation's event, per the Entity-Lifecycle Coverage Matrix's note that this feature "never writes directly" to that field.
**Deletes:** None.

## Business Rules

- XBR-05: this first-view event is one of the record-worthy events that always writes an append-only trail entry with actor and timestamp; it is owned by FEAT-13, this automation is the source that detects and hands it off.
- Idempotency: at most one first-view event exists per record per contact, ever -- a contact re-viewing the same record any number of times after the first produces no further event, no matter how the feature's own screen re-renders on subsequent visits.
- This automation runs on behalf of whichever feature's screen the client contact opened (FEAT-03, FEAT-06, FEAT-07, FEAT-09, or FEAT-10); it is owned by FEAT-05 because first view is inherently a property of the portal session FEAT-05 owns, not of the record's own feature, per the Feature Breakdown Brief's Discovery Rationale.
- The evidence this event produces is what the "Pointing to the Record in a Scope Dispute" journey relies on to settle a dispute; the timestamp captured is therefore the moment the record's screen opened within the portal session, not any later moment (e.g., scrolling, dwelling, or closing the screen).

## Edge Cases

- **Contact opens the same record twice in the same session, moments apart** -- The second open finds the first-view event already exists (step 2) and takes No-Action; only the first open's timestamp is ever recorded.
- **Concurrent trigger firing (the same contact opens the same record from two devices or tabs at effectively the same time)** -- Whichever open reaches step 2 first captures the first-view event; the second finds it already exists and takes No-Action. At most one first-view event per record per contact is ever created, regardless of how many near-simultaneous opens occur.
- **Trigger fires while a previous run for the same record/contact pair is still in flight** -- The existence check (step 2) and the event creation (steps 4-5) are treated as a single atomic step for a given record/contact pair, so a second trigger arriving before the first completes waits for that step to resolve and then correctly finds the event already captured.
- **Two different contacts (Owen and Priya) open the same Deliverable** -- Each contact's first view is tracked independently, since the first-view event is scoped to a record/contact pair, not to the record alone; both Owen's and Priya's first views are captured separately.
- **The underlying record is removed or voided between a prior first view and a later re-view attempt** -- The idempotency check in step 2 still finds the existing first-view event for that record/contact pair (the event is immutable and never deleted alongside the record, consistent with FEAT-13's Activity Log Entry never being edited or deleted except by FEAT-24 account deletion); no new event is captured, since a first view, once recorded, is a historical fact independent of the record's current state.
- **Hand-off to FEAT-13 fails after the timestamp is captured** -- The screen the contact is viewing renders normally regardless; FEAT-13's own delivery guarantee retries the hand-off, and until it succeeds, a subsequent open of the same record is treated as a new candidate for first-view capture only if FEAT-13 confirms no entry was ultimately recorded, preventing a permanently lost first-view record.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-03 (Proposal Acceptance) | Triggered by (inbound) | Opening the Proposal Review & Accept screen fires this automation |
| FEAT-06 (Deliverable Upload & Sharing) / FEAT-07 (Deliverable Review & Feedback) | Triggered by (inbound) | Opening a deliverable review screen fires this automation |
| FEAT-09 (Invoice Generation & Sending) / FEAT-10 (Invoice Payment Processing) | Triggered by (inbound) | Opening the invoice list or pay-invoice screen fires this automation |
| FEAT-13 (Immutable Activity & Audit Trail) | Affects (outbound) | Receives the timestamped first-view event and writes the append-only trail entry (XBR-05) |
| FEAT-05.SPEC-003 (Portal Home) | Triggered by (inbound) | Navigating from a Portal Home waiting item into a proposal, deliverable, or invoice screen is the entry path that leads to this automation's trigger |

## Analytics and Success Signals

- **first_view_captured** (record_type: proposal / deliverable / invoice) -- N/A -- no Stage 2 metric in success-metrics.md is connected to this feature's first-view detection specifically; the two metrics connected to FEAT-05 (Client Portal Login Success, Client Portal Mobile Responsiveness) measure the sign-in flow and page responsiveness, not record-viewing behavior. This event is retained because it is the operational signal that the evidentiary hand-off to FEAT-13 (relied on by the scope-dispute journey) is firing as expected.

## Acceptance Criteria

**FEAT-05.SPEC-009-AC-01:** Given Owen opens a proposal waiting on him for the first time through his portal session, when the screen opens, then a first-view event is captured and handed to FEAT-13 with his identity, the timestamp, and the proposal reference.

**FEAT-05.SPEC-009-AC-02:** Given Priya has already had a first-view event captured for a specific deliverable, when she opens that same deliverable again, then no new event is captured.

**FEAT-05.SPEC-009-AC-03:** Given Owen opens the same invoice twice in quick succession from two open tabs, when both opens are processed, then exactly one first-view event exists for that invoice and Owen's contact record.

**FEAT-05.SPEC-009-AC-04:** Given both Owen and Priya open the same deliverable for the first time, each from their own portal session, when both views occur, then two separate first-view events are captured, one per contact.

**FEAT-05.SPEC-009-AC-05:** Given the hand-off to FEAT-13 fails after Owen's first view of an invoice is captured, when the failure occurs, then Owen's invoice screen still renders normally, and FEAT-13's own retry eventually completes the trail write.

**FEAT-05.SPEC-009-AC-06:** Given a deliverable is later superseded by a new version after Priya's first view of the original was captured, when she opens the superseded version again, then no new first-view event is captured, since her first-view event for that deliverable already exists.

**FEAT-05.SPEC-009-AC-07:** Given Owen opens a proposal through the portal for the first time, when the first-view event is captured, then the timestamp recorded is the moment the screen opened, not any later moment such as scrolling or closing the screen.

**FEAT-05.SPEC-009-AC-08:** Given this automation fires from FEAT-03, FEAT-06, FEAT-07, FEAT-09, or FEAT-10's own screens, when any of those triggers fire, then the same detection and idempotency logic applies uniformly regardless of which feature's screen triggered it.

**FEAT-05.SPEC-009-AC-09:** Given Owen's first-view event for a proposal is already recorded, when he later re-opens that proposal after it has been voided and re-sent as a new Proposal record (FEAT-02's void-and-resend), then the new Proposal record is a distinct record from the voided one, so his open of the new record is evaluated as its own first-view candidate, independent of the voided proposal's already-captured event.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 | 3 |
| Outcome Paths | 3 | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |

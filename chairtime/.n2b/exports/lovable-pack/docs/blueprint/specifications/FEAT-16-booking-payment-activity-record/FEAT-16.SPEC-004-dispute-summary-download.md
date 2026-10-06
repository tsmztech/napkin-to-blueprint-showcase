---
document_type: spec
spec_type: automation
spec_id: FEAT-16.SPEC-004
spec_name: Dispute Summary Download
spec_slug: dispute-summary-download
parent_feature: FEAT-16
parent_feature_name: Booking & Payment Activity Record
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

# Automation Spec: Dispute Summary Download

## Overview

**Name:** Dispute Summary Download
**ID:** FEAT-16.SPEC-004
**Type:** Automation
**Purpose:** Assembles a plain-language, shareable summary of a disputed booking's timeline -- policy shown and acknowledged, booking time, messages sent, and no-show mark -- and hands it to the Pro as a downloadable file to submit as evidence with the payment processor.
**Parent Feature:** FEAT-16 -- Booking & Payment Activity Record

## Scope and Non-Goals

**In Scope:**
- Assembling a plain-language summary from the disputed booking's existing Activity Event, Booking, Deposit Transaction, Cancellation Policy, and Message data
- Producing that summary as a file the Pro can download to her own device
- Restricting this action to bookings that currently carry the Disputed overlay

**Non-Goals:**
- Submitting the summary to the payment processor on the Pro's behalf -- excluded per feature-overview.md's Key Capabilities ("to use as evidence with the payment processor") and SC-17; the Pro submits it herself through the processor's own channel
- Deciding the dispute's outcome -- excluded per SC-17: this automation only assembles a factual record, it never argues a position or renders a verdict
- Detecting or flagging the dispute itself -- owned by FEAT-16.SPEC-003 (Card-Issuer Dispute Integration), which sets the Disputed overlay this automation checks for
- Including the Pro's private client notes in the summary -- excluded per XBR-24 and FEAT-16.SPEC-005: the summary is built for external submission to the payment processor, so it carries strictly less than even Support's internal view, which itself already excludes private notes
- Generating a summary for a non-disputed booking -- this automation has no trigger path that reaches a booking without an active Disputed overlay, since its only entry point (FEAT-16.SPEC-001) hides the download action for non-disputed bookings

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Pro requests the evidence summary | FEAT-16.SPEC-001 (Booking Activity Timeline) | The Pro taps "Download evidence summary" on a booking whose Deposit Transaction currently carries the Disputed overlay | Booking reference |

## Processing Logic

1. Confirm the referenced Booking's Deposit Transaction currently carries the Disputed overlay (set by FEAT-16.SPEC-003). If it does not, refuse the request (see Outcome Definitions).
2. Read the booking's full ordered Activity Event history (the same data FEAT-16.SPEC-001 renders): the policy version and wording shown and its acknowledgment timestamp, the appointment time, every message sent and its delivery outcome (including any gap per XBR-17), any cancellation/reschedule event, the no-show mark (if present) and its timestamp, and the deposit outcome including the dispute event itself.
3. Assemble this data into a plain-language summary document, organized chronologically, using the same event descriptions the timeline screen already shows -- no new wording or interpretation is introduced beyond what the timeline already states as fact.
4. Exclude the Pro's private client notes from the assembled summary entirely, per XBR-24 and FEAT-16.SPEC-005.
5. Render the assembled summary as a downloadable file and hand it to the Pro's device.
6. Since this automation assembles from data already loaded by the timeline (feature-overview.md's Responsiveness note: "assembles from already-loaded timeline data"), the assembly completes near-instantaneously rather than as a long-running export.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Summary produced | The booking's Deposit Transaction carries the Disputed overlay at request time | None -- this automation is read-only against product data; it produces a file, it does not write one | The file downloads to Talia's device; FEAT-16.SPEC-001 shows a brief confirmation that the download started | FEAT-16.SPEC-001 (Booking Activity Timeline) |
| Request refused (not disputed) | The booking's Deposit Transaction does not carry the Disputed overlay at request time (e.g., the overlay was somehow cleared or the request is stale) | None | Talia sees: "This booking is no longer flagged as disputed. A summary is only available for disputed bookings." and no file is produced | FEAT-16.SPEC-001 |
| Assembly failure | The automation cannot read one or more required data elements (e.g., a transient read failure) | None | Talia sees: "The evidence summary could not be prepared right now. Try again in a moment." with a retry option; no partial or corrupted file is ever handed to her device | FEAT-16.SPEC-001 |

## Data Model

**Reads:** Activity Event (event_type, time, actor, details) for the booking; Booking (service, start_time, state); Deposit Transaction (status including Disputed overlay, outcome_reason, timestamps); Cancellation Policy (version, plain_language_wording); Message (type, channel, send time, delivery_status) -- all read-only, identical in source to what FEAT-16.SPEC-001 already renders.
**Creates:** A downloadable summary file, handed to the Pro's device; this file is not itself a product entity and is not stored by the product beyond the download hand-off.
**Updates:** None.
**Deletes:** None.

## Business Rules

- XBR-22: the plain timeline summary is made available to submit as evidence once a card-issuer dispute has flagged the booking; this automation is exactly that mechanism.
- The summary never includes the Pro's private client notes, per XBR-24 and FEAT-16.SPEC-005, regardless of how much detail the notes might otherwise add to the Pro's case.
- The summary is available only for a booking whose Deposit Transaction currently carries the Disputed overlay -- there is no path to request one for a non-disputed booking.
- The summary states facts only, drawn verbatim from the same Activity Event history the timeline already shows the Pro; this automation introduces no new characterization, argument, or recommendation, consistent with SC-17 (Chairtime never rules on or manages the dispute process itself).
- The Pro alone submits the downloaded file to the payment processor; no automated hand-over exists, per feature-overview.md's own Non-Goals.

## Edge Cases

- **The dispute is resolved (won, lost, or withdrawn) between the Pro's first and second download of the summary** -- The Disputed overlay remains set (per FEAT-16.SPEC-003's Data Exchanged: the concluded outcome is added alongside the overlay, which is not cleared), so a second download remains available and reflects the same underlying timeline plus the now-visible concluded-dispute entry.
- **Two rapid taps on "Download evidence summary" on the same device (trigger fires while a previous run is in flight)** -- The second tap while the first assembly is in flight is ignored (the action shows a brief in-progress state); only one file is handed to the device per request.
- **Talia requests the summary from two signed-in devices at effectively the same time (concurrent trigger firing)** -- Each device's request runs its own independent assembly against the same read-only source data; both succeed independently and each device receives its own downloaded file, since assembly never writes shared state that the two runs could conflict over.
- **A gap exists in the timeline (e.g., a failed-then-fallback message)** -- The gap is included in the summary exactly as the timeline shows it, per XBR-17; the summary never presents a falsely clean record.
- **The booking has very few events (e.g., created, policy acknowledged, deposit paid, then immediately disputed with no messages or no-show mark)** -- The summary includes exactly the events that exist; it is not padded with placeholder sections for events that never occurred.
- **The Pro requests the summary while offline** -- The action is disabled with "This action needs a connection." per FEAT-16.SPEC-001's Offline/Degraded state, since assembly and hand-off require connectivity even though the underlying data was already loaded.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-16.SPEC-001 (Booking Activity Timeline) | Triggered by (inbound) | "Download evidence summary" initiates this automation |
| FEAT-16.SPEC-003 (Card-Issuer Dispute Integration) | References (inbound) | The Disputed overlay this automation checks for is set there |
| FEAT-16.SPEC-002 (Activity Event Recording) | References (inbound) | The Activity Event history this automation assembles from is written there |
| FEAT-16.SPEC-005 (Activity Record Immutability & Visibility Rules) | Governed by | Rule spec that governs this automation: its Enforced By table names this spec for the private-notes exclusion on assembly (XBR-24), and its Authorization Rules define who may request a summary |

## Analytics and Success Signals

- **dispute_summary_downloaded** (booking reference) -- N/A -- no success-metrics.md metric is connected to FEAT-16; this signal (named in feature-overview.md's Signals field) is retained for operational observability of how often the record is actually used as evidence.
- **dispute_summary_assembly_failed** (reason) -- N/A -- no connected success-metrics.md metric; retained to observe whether the evidence path is reliable when Talia needs it most.

## Acceptance Criteria

**FEAT-16.SPEC-004-AC-01:** Given Talia is viewing a disputed booking's timeline, when she taps "Download evidence summary," then a plain-language file downloads to her device containing the policy shown and acknowledged, the booking time, messages sent, and the no-show mark (if any).

**FEAT-16.SPEC-004-AC-02:** Given a booking's timeline includes a message-delivery gap, when Talia downloads the summary, then the gap appears in the summary exactly as it appears on the timeline.

**FEAT-16.SPEC-004-AC-03:** Given the summary is assembled, when Talia opens the downloaded file, then it contains no reference to her private client notes about that client.

**FEAT-16.SPEC-004-AC-04:** Given a booking's Deposit Transaction does not carry the Disputed overlay, when a request for its evidence summary somehow reaches this automation, then it is refused with "This booking is no longer flagged as disputed. A summary is only available for disputed bookings." and no file is produced.

**FEAT-16.SPEC-004-AC-05:** Given the automation cannot read the booking's required data at request time, when the assembly fails, then Talia sees "The evidence summary could not be prepared right now. Try again in a moment." with a retry option, and no partial file is produced.

**FEAT-16.SPEC-004-AC-06:** Given Talia downloads a disputed booking's summary and the dispute later resolves as "lost," when she downloads it again, then the file still assembles, now also reflecting the concluded-dispute entry alongside the original facts.

**FEAT-16.SPEC-004-AC-07:** Given Talia taps "Download evidence summary" twice rapidly, when the second tap registers while the first is still assembling, then it is ignored and exactly one file is handed to her device.

**FEAT-16.SPEC-004-AC-08:** Given a disputed booking has only a handful of events (created, policy acknowledged, deposit paid, disputed), when Talia downloads the summary, then it contains exactly those events with no placeholder sections for events that never occurred.

**FEAT-16.SPEC-004-AC-09:** Given Talia loses connectivity while viewing a disputed booking's timeline, when she looks for the download action, then it is disabled with "This action needs a connection."

**FEAT-16.SPEC-004-AC-10:** Given Talia has downloaded the summary, when she looks for a way to send it to the payment processor from within the product, then no such feature exists -- she must submit it herself through the processor's own channel.

**FEAT-16.SPEC-004-AC-11:** Given the assembled summary states the sequence of events, when Talia reads it, then it contains no argument, recommendation, or verdict about who is right -- only the recorded facts.

**FEAT-16.SPEC-004-AC-12:** Given a booking's cancellation policy version and acknowledgment time are part of its timeline, when Talia downloads the summary, then that exact version and timestamp appear in the file.

**FEAT-16.SPEC-004-AC-13:** Given the disputed booking includes a no-show mark, when Talia downloads the summary, then the no-show mark and its timestamp appear as one of the summarized facts.

**FEAT-16.SPEC-004-AC-14:** Given the summary is assembled from data already loaded by the timeline screen, when Talia taps download, then the file is produced near-instantaneously rather than showing a long-running export progress state.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

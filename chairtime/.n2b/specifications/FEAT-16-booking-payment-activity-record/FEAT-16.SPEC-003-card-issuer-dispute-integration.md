---
document_type: spec
spec_type: integration
spec_id: FEAT-16.SPEC-003
spec_name: Card-Issuer Dispute Integration
spec_slug: card-issuer-dispute-integration
parent_feature: FEAT-16
parent_feature_name: Booking & Payment Activity Record
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Integration Spec: Card-Issuer Dispute Integration

## Overview

**Name:** Card-Issuer Dispute Integration
**ID:** FEAT-16.SPEC-003
**Type:** Integration
**Purpose:** Receives an inbound card-issuer dispute notice from the payment-processing capability, flags the affected booking, sets the Deposit Transaction's Disputed overlay without erasing its underlying outcome, and records the dispute as an Activity Event.
**Parent Feature:** FEAT-16 -- Booking & Payment Activity Record

## Scope and Non-Goals

**In Scope:**
- Receiving the payment-processing capability's card-issuer dispute notice for a Deposit Transaction
- Setting the Disputed overlay on the affected Deposit Transaction, additive to (never replacing) its existing outcome
- Handing off the dispute for the Pro notification and dashboard flag (owned elsewhere, cited below)
- Recording the dispute event through FEAT-16.SPEC-002's append-only mechanism
- Degradation behavior when the payment-processing capability is slow, down, or rejects, for every screen this integration affects
- Disclosure of what dispute metadata is read from the capability

**Non-Goals:**
- Deciding or adjudicating the dispute -- excluded per SC-17: Chairtime never rules on who is right between the Pro and the client; this spec supplies the record and the flag, nothing more
- Reading or storing card data of any kind -- excluded per SC-11: this spec reads only dispute metadata and outcome flags from the payment-processing capability, which alone owns card data
- Sending the Pro's dispute notification -- owned by FEAT-08.SPEC-006 (Pro Attention Alert), which this spec hands off to per feature-overview.md's own Communications field ("A Pro notification (via FEAT-08)")
- Feeding the dashboard's attention flag -- owned by FEAT-12.SPEC-002 / FEAT-12.SPEC-005 (Attention List / Attention Flag Aggregation), which consume the Disputed overlay this spec sets
- Assembling or delivering the downloadable evidence summary -- owned by FEAT-16.SPEC-004 (Dispute Summary Download)
- Any outbound submission of evidence to the payment processor on the Pro's behalf -- excluded per feature-overview.md's own Non-Goals and SC-17; the Pro submits the downloaded summary herself through the processor's own channel

## Capability Category

**Category:** Payment processing -- card-issuer dispute notifications
**Dependency Source:** ASMP-31 -- "Payment-processing capability ... notify the product of card-issuer disputes" (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Payment processing — card-issuer dispute notifications (ASMP-31)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-16, FEAT-12)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision; BRIEF.md records no user mandate for a specific payment processor.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Talia sees a booking flagged the moment a client raises a card-issuer dispute | See a booking flagged when the client raises a dispute with their card issuer | FEAT-12.SPEC-002 / FEAT-12.SPEC-005 (Attention List / Attention Flag Aggregation), FEAT-16.SPEC-001 (Booking Activity Timeline) |
| Talia is notified of the dispute without checking the dashboard | See a booking flagged... | FEAT-08.SPEC-006 (Pro Attention Alert) |
| The disputed booking's Deposit Transaction carries a Disputed marker alongside its existing outcome | See a booking flagged... | FEAT-16.SPEC-001, FEAT-28 (money list, read-only reflection) |
| The dispute is recorded as a permanent, immutable fact on the booking's timeline | Reference this record when responding to a client's dispute | FEAT-16.SPEC-002 (Activity Event Recording), FEAT-16.SPEC-001 |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Deposit Transaction reference | Deposit Transaction -- processor transaction reference (already held from FEAT-07.SPEC-005's original charge) | The capability's own dispute process needs to be tied to the original charge (this reference already exists from the original charge and is not newly sent for this integration -- listed here as context data already known to the capability) | Ties the dispute back to the exact charge in the capability's own systems |

No new outbound data is sent by this integration beyond what the original deposit charge (FEAT-07.SPEC-005) already shared with the capability. This spec is inbound-only: the payment-processing capability reports the dispute to the product; the product sends nothing further to initiate or advance the dispute itself. Booking details, client contact information, Pro Account details, and every other product entity never leave the product through this integration.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Dispute notice (deposit reference, dispute reason category, dispute timestamp) | The payment-processing capability reports a card-issuer dispute against a captured deposit | Deposit Transaction -- status overlay set to Disputed; outcome_reason and timestamps updated to include the dispute event; Booking -- flagged for dashboard display |
| Dispute outcome metadata (won / lost / withdrawn), if reported later by the capability | The capability reports the dispute process concluding | Deposit Transaction -- outcome_reason updated to reflect the concluded dispute status; the original Captured/Forfeited/Refunded outcome is never erased |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Card-issuer dispute notice received | The payment-processing capability reports a client dispute against a captured deposit | Deposit Transaction's Disputed overlay set (additive, never replacing the underlying Captured/Forfeited/Refunded outcome); Booking flagged; a "disputed" Activity Event is written via FEAT-16.SPEC-002 | Talia's dashboard shows the dispute flag (FEAT-12.SPEC-002/FEAT-12.SPEC-005); Talia receives a Pro notification (FEAT-08.SPEC-006); the booking's timeline (FEAT-16.SPEC-001) shows the dispute banner and download entry point | FEAT-16.SPEC-002 (Activity Event Recording), FEAT-16.SPEC-001 (Booking Activity Timeline), FEAT-12.SPEC-002, FEAT-12.SPEC-005, FEAT-08.SPEC-006 |
| Dispute process concludes (won / lost / withdrawn) | The capability reports the dispute process has ended | Deposit Transaction's outcome_reason updated with the concluded dispute status alongside the retained Disputed overlay and original outcome; a corresponding Activity Event is written via FEAT-16.SPEC-002 | The booking's timeline (FEAT-16.SPEC-001) shows the concluded dispute event; no separate Pro notification is defined for this event beyond the existing dashboard flag update, since the founder has no further action to take once the processor's own process has concluded | FEAT-16.SPEC-002, FEAT-16.SPEC-001, FEAT-12.SPEC-002, FEAT-12.SPEC-005 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-12.SPEC-002 / FEAT-12.SPEC-005 (Attention List / Attention Flag Aggregation) | N/A -- this integration is purely inbound; there is no Talia-initiated request from the dashboard that can be "slow" toward the capability | If the capability's dispute-notification channel is unavailable, no new dispute notices arrive during the outage; the dashboard shows exactly the dispute flags it already knows about, with no error state, since a missing notification looks identical to "no dispute has occurred yet" from the product's perspective -- there is no in-progress request to fail visibly | N/A -- the dashboard never sends a request this capability could reject; it only reflects notices already received |
| FEAT-16.SPEC-001 (Booking Activity Timeline) | N/A -- same reasoning: this integration never initiates a request from this screen | Same as above: a dispute notice delayed by an outage simply has not arrived yet; the timeline shows no dispute banner until the notice lands, with no error state | N/A -- no outbound request from this screen to reject |

## Consent and Disclosure

- **No client-facing or Pro-facing consent moment exists for this integration** -- the payment-processing capability's own terms (accepted once, when the Pro connected her payout account through FEAT-28) already cover dispute-notification reporting as part of standard card-processing service; this spec introduces no new outbound data element requiring a fresh disclosure, since the "Leaves the product" section above confirms no new data is sent to enable this integration beyond what the original deposit charge already shared.
- **What is never shared or requested** -- this integration never reads or requests card data, cardholder identity details, or any product entity (Booking, Client, Message) beyond the already-known Deposit Transaction reference; SC-11 keeps card data entirely with the payment processor.

## Edge Cases

- **A dispute notice arrives for a Deposit Transaction that has since been de-identified (client deletion processed, per XBR-19)** -- The Disputed overlay is applied to the retained, de-identified financial record; the dashboard flag and Pro notification still fire, since the dispute is a financial fact independent of the client's contact details having been removed.
- **The same dispute notice is delivered twice** -- The second delivery changes nothing: the Deposit Transaction's Disputed overlay is already set, the dispute Activity Event already exists, and no duplicate notification or duplicate timeline entry is created.
- **A "dispute concluded" event arrives before the original "dispute notice" event (out-of-order delivery)** -- The product holds the concluded-outcome data and applies it only once the original dispute notice is also received and the Disputed overlay is set; if the original notice never arrives, the concluded event alone is insufficient to flag a booking that was never shown as disputed, and this out-of-order case is flagged as a data inconsistency for support to investigate rather than silently applied.
- **A dispute is raised on a deposit that was already fully refunded** -- The Disputed overlay is set alongside the existing Refunded outcome per the entity's Contention rule (a Disputed overlay never erases the underlying outcome); Talia sees both facts on the timeline: the refund and the dispute.
- **The payment-processing capability is down when the dispute notice would otherwise have arrived** -- The notice simply has not been delivered yet; once the capability's channel recovers, the notice arrives and is processed normally with its own original dispute timestamp, not the recovery time.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-16.SPEC-002 (Activity Event Recording) | Triggers (outbound, external-event source) | The dispute notice is the external-event trigger for the "disputed" Activity Event write |
| FEAT-16.SPEC-001 (Booking Activity Timeline) | Affects (outbound) | The dispute banner and flagged entry reflect this spec's write |
| FEAT-16.SPEC-005 (Activity Record Immutability & Visibility Rules) | Governed by | Rule spec listing this spec in its Enforced By table: the "disputed" Activity Event this spec hands to FEAT-16.SPEC-002 is subject to the same append-only, immutable, non-editable guarantee, and the Disputed overlay is scoped to Deposit Transaction, not Activity Event |
| FEAT-16.SPEC-004 (Dispute Summary Download) | References (inbound) | That spec's download entry is available only once this spec has flagged the booking |
| FEAT-08.SPEC-006 (Pro Attention Alert) | Triggers (outbound) | The dispute notice hands off to this notification |
| FEAT-12.SPEC-002 (Attention List) | Affects (outbound) | Consumes the dispute flag for dashboard display |
| FEAT-12.SPEC-005 (Attention Flag Aggregation) | Affects (outbound) | Aggregates the dispute flag among other attention items |
| FEAT-07.SPEC-005 (Card Deposit Charge & Payout Routing) | References (inbound) | The original deposit charge this dispute is raised against |

## Analytics and Success Signals

- **card_dispute_flagged** (booking reference) -- N/A -- no success-metrics.md metric is connected to FEAT-16; this signal (named in feature-overview.md's Signals field) is retained for operational observability of dispute frequency, since Chairtime never adjudicates disputes and has no target rate to measure them against.
- **card_dispute_concluded** (outcome: won / lost / withdrawn) -- N/A -- no connected success-metrics.md metric; retained purely for operational visibility into dispute resolution outcomes over time.

## Acceptance Criteria

**FEAT-16.SPEC-003-AC-01:** Given a client raises a card-issuer dispute against a booking's captured deposit, when the payment-processing capability reports the dispute, then the Deposit Transaction's Disputed overlay is set without changing its existing Captured/Forfeited/Refunded outcome.

**FEAT-16.SPEC-003-AC-02:** Given a dispute notice is received, when the overlay is set, then a "disputed" Activity Event is written for the affected booking through FEAT-16.SPEC-002.

**FEAT-16.SPEC-003-AC-03:** Given a dispute notice is received, when the flag is applied, then Talia's dashboard attention list (FEAT-12.SPEC-002/FEAT-12.SPEC-005) shows the dispute flag and Talia receives a Pro notification via FEAT-08.SPEC-006.

**FEAT-16.SPEC-003-AC-04:** Given the payment-processing capability later reports the dispute concluded as "lost," when the event arrives, then the Deposit Transaction's outcome_reason is updated to reflect the concluded status while the Disputed overlay and original outcome remain visible.

**FEAT-16.SPEC-003-AC-05:** Given the payment-processing capability's dispute-notification channel is down, when Talia views her dashboard during the outage, then it shows exactly the dispute flags already known, with no error state suggesting something is broken.

**FEAT-16.SPEC-003-AC-06:** Given the same dispute notice is delivered twice by the capability, when the second delivery is processed, then no duplicate Activity Event or duplicate Pro notification is created.

**FEAT-16.SPEC-003-AC-07:** Given a dispute is raised on a deposit that was already refunded, when the notice is processed, then Talia's timeline (FEAT-16.SPEC-001) shows both the refund and the dispute as separate, coexisting facts.

**FEAT-16.SPEC-003-AC-08:** Given a dispute notice arrives for a client whose record has since been de-identified per XBR-19, when the notice is processed, then the Disputed overlay is still applied to the retained financial record and the dashboard flag still fires.

**FEAT-16.SPEC-003-AC-09:** Given Talia asks what data this integration reads, when she reviews the disclosure, then it states plainly that only dispute metadata and outcome flags are read, never card data, per SC-11.

**FEAT-16.SPEC-003-AC-10:** Given a "dispute concluded" event arrives before its corresponding original dispute notice, when the out-of-order delivery is detected, then the concluded outcome is held rather than silently applied to a booking never shown as disputed.

**FEAT-16.SPEC-003-AC-11:** Given Talia wants to know who is right in a dispute, when she looks to Chairtime for a ruling, then no such feature exists anywhere in this spec or its connected specs -- only the record and the flag are provided, per SC-17.

**FEAT-16.SPEC-003-AC-12:** Given the payment-processing capability's channel was down and then recovers, when a delayed dispute notice finally arrives, then it is processed with its own original dispute timestamp, not the time of recovery.

**FEAT-16.SPEC-003-AC-13:** Given no new outbound data element is introduced by this integration beyond the original deposit charge's own transaction reference, when a downstream reviewer checks Consent and Disclosure, then no new disclosure moment is required, and this is stated explicitly rather than left unaddressed.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 4 | 4 |
| Inbound Events | 2 | 2 |
| Degradation Paths | 2 (2 screens; N/A cells justified) | 2 |
| Consent and Disclosure | 2 | 2 |
| Edge Cases | 5 | 5 |

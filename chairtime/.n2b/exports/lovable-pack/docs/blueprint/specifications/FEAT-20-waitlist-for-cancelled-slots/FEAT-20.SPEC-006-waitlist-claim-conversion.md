---
document_type: spec
spec_type: automation
spec_id: FEAT-20.SPEC-006
spec_name: Waitlist Claim Conversion
spec_slug: waitlist-claim-conversion
parent_feature: FEAT-20
parent_feature_name: Waitlist for Cancelled Slots
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Automation Spec: Waitlist Claim Conversion

## Overview

**Name:** Waitlist Claim Conversion
**ID:** FEAT-20.SPEC-006
**Type:** Automation
**Purpose:** When a notified client completes the ordinary booking flow for the matching slot, converts their entry to Converted and leaves the other notified entries untouched.
**Parent Feature:** FEAT-20 -- Waitlist for Cancelled Slots

## Scope and Non-Goals

**In Scope:**
- Recognizing that a completed Booking corresponds to a Notified Waitlist Entry's claimed slot
- Transitioning that entry to Converted
- Confirming the other Notified entries for the same freed slot (if any) are left unconverted and unexpired by this event

**Non-Goals:**
- Creating the Booking itself, or handling the deposit payment -- owned entirely by FEAT-05 (Public Booking Page & Booking Flow); this automation reacts to a booking FEAT-05 already completed, it never creates one
- Resolving which of several notified clients wins a contested slot -- that resolution is FEAT-03.SPEC-005's first-to-pay-wins mechanism (XBR-01), applied by the ordinary booking flow itself; this automation only records the outcome for the winning client's Waitlist Entry once FEAT-05 reports the booking complete
- Expiring the other, still-Requested notified entries -- owned by FEAT-20.SPEC-007; this automation only converts the one entry whose client won the slot

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A booking completes via the claim link | FEAT-05 (Public Booking Page & Booking Flow), reached via the claim link on FEAT-20.SPEC-008 | Fires when FEAT-05's booking flow reports a completed Booking whose service, date, and start time match a Notified Waitlist Entry's claimed slot for the same Client | Booking reference, service, date, start time, Client reference |

## Processing Logic

1. Receive the completed-Booking signal from FEAT-05, carrying the Client reference and the booked service/date/start time.
2. Find the requesting Client's Waitlist Entry in state Notified whose matched slot (service, date, start time, as set by FEAT-20.SPEC-005) equals the completed Booking's service, date, and start time.
3. If found, confirm the entry's claim_deadline has not yet passed at the moment the Booking completed (re-checked here as the authoritative gate, even though FEAT-05's own slot re-validation already confirmed the slot was bookable at payment time).
4. If the window check passes, transition that Waitlist Entry's state to Converted.
5. Take no action on any other Notified entry for the same freed slot -- they remain Notified, each still governed by its own independent claim_deadline, per FEAT-20.SPEC-004's contention rule that the losing entries remain eligible for the next opportunity.
6. If no matching Notified entry is found for this Client and this slot (the booking was an ordinary booking unrelated to any waitlist claim), take no action -- this is the expected outcome for the overwhelming majority of bookings, which never touch this automation at all.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Claim converted | A completed Booking matches a Notified entry for the same Client, within its claim window | Waitlist Entry: state -> Converted | The client's entry shows "Booked" on FEAT-20.SPEC-002 on next load; no separate notification is sent beyond the ordinary booking confirmation (FEAT-08.SPEC-001), since the client is already looking at the confirmation | FEAT-20.SPEC-002 |
| No matching entry found | The completed Booking does not correspond to any Notified entry for this Client (an ordinary booking, unrelated to the waitlist) | None | None -- this is the expected outcome for ordinary bookings | -- |
| Claim window had already lapsed at booking completion | A matching Notified entry exists, but its claim_deadline passed before the Booking completed | No conversion -- the entry is left exactly as FEAT-20.SPEC-007's own expiry processing will handle it | The client still keeps the Booking she just completed (it was validated as bookable through the ordinary, general-availability path, not the priority window); her waitlist entry's own fate is governed separately by FEAT-20.SPEC-007 | FEAT-20.SPEC-007 |
| Automation failure (the conversion write itself fails) | A transient failure prevents the state write from completing | No change to the Waitlist Entry | The Booking itself is unaffected and remains completed; the entry may still show as Notified until the next successful processing pass or until FEAT-20.SPEC-007's expiry naturally resolves it | FEAT-20.SPEC-002, FEAT-20.SPEC-007 |

## Data Model

**Reads:** Waitlist Entry -- state, service, start_date/end_date, claim_deadline, scoped to the Client who completed the Booking; Booking -- service, date, start_time, Client reference, to confirm the match.
**Creates:** None.
**Updates:** Waitlist Entry -- state (Notified -> Converted), for the one matched entry only.
**Deletes:** None.

## Business Rules

- XBR-01 governs the underlying contention this automation observes the outcome of: the first client to complete payment wins a contested slot; this automation records that outcome for the winner's Waitlist Entry, it does not itself decide the winner.
- Only the client whose completed Booking matches a Notified entry's exact slot receives a conversion -- every other Notified entry for the same freed slot is left untouched by this event, per FEAT-20.SPEC-004's rule that losing entries remain eligible for the next opportunity, never automatically expired or deleted by someone else's successful claim.
- A Converted entry is terminal and retained indefinitely per this Brief's Non-Goals (no automatic purge); it is never re-activated or re-matched.
- This automation performs no notification of its own -- the client already sees the ordinary booking confirmation (FEAT-08.SPEC-001) from completing the booking flow; a redundant "you claimed it" message would be noise on top of a confirmation she is already looking at.
- Platform Operator (Support) can view a Converted entry and the Booking it produced through FEAT-19's read-only account view, per XBR-24, useful for a dispute or confusion around who claimed a given opening; Support never performs or reverses a conversion.

## Edge Cases

- **Two notified clients race to claim the same freed slot** -- Whichever completes payment first triggers this automation's conversion for her own entry; the second client's payment attempt is refused by FEAT-03.SPEC-005's contention resolution before it ever reaches this automation, so only one conversion is ever produced per freed slot.
- **A client completes a booking for the same service and date through the ordinary booking flow, coincidentally, without ever having been notified (she never joined the waitlist for this slot)** -- No matching Notified entry exists for her, so this automation takes no action; this is simply an ordinary booking.
- **A client holds a Notified entry for one freed slot but books a completely different time for the same service through the general booking flow** -- The completed Booking's date/time does not match the Notified entry's claimed slot, so no conversion occurs; her Notified entry remains active and subject to its own claim window and eventual expiry.
- **The claim window lapses between the client tapping "Claim now" and completing payment** -- FEAT-05's own slot re-validation at checkout (mirroring FEAT-03.SPEC-006's re-validation pattern) determines whether the slot is still reachable through the priority path at that moment; if her window lapsed first, this automation's window check in Processing Logic step 3 also fails, and no conversion is recorded -- her entry is instead handled by FEAT-20.SPEC-007's expiry.
- **Concurrent trigger firing (two different clients each complete a claim for two different freed slots at the same time)** -- Each conversion runs independently; neither is delayed by the other.
- **Trigger fires while a previous run is in flight for the same Client and entry** -- Not possible in practice: a given Waitlist Entry converts at most once (state moves from Notified to the terminal Converted and never back), so a second completed-Booking signal for an already-Converted entry finds no eligible Notified entry to match and takes no action.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-05 (Public Booking Page & Booking Flow) | Triggered by (inbound) | A completed Booking through the claim link fires this automation |
| FEAT-20.SPEC-008 (Waitlist Opening Notification) | References (inbound) | The claim link this automation's trigger originates from is the one that notification carries |
| FEAT-20.SPEC-004 (Waitlist Priority & Claim Window Rule) | References (outbound) | Governs the window check and the losing entries' contention resolution |
| FEAT-20.SPEC-002 (My Waitlists) | Affects (outbound) | The Converted status is what that screen's next load reflects |
| FEAT-20.SPEC-007 (Waitlist Entry Expiry) | References (outbound) | Governs what happens to an entry whose window lapsed before this automation could convert it |
| FEAT-08.SPEC-001 (Booking Confirmation Message) | References (outbound) | Supplies the client's booking confirmation; this automation triggers no separate message of its own |

## Analytics and Success Signals

- **waitlist_converted_to_booking** (service ID, time from notification to conversion) -- N/A -- no success-metrics.md metric directly measures waitlist conversion; retained per the Brief's Non-Functional Notes naming waitlist_converted_to_booking as one of the four Stage 2 signals this feature's transitions must emit.
- **waitlist_claim_window_lapsed_before_conversion** (service ID) -- N/A -- no Stage 2 metric measures near-miss claims; retained for operational visibility into how often a claim attempt arrives just after its window closes.

## Acceptance Criteria

**FEAT-20.SPEC-006-AC-01:** Given Riley taps the claim link on her opening notification and completes the booking flow within her claim window, when this automation processes the completed Booking, then her Waitlist Entry transitions to Converted.

**FEAT-20.SPEC-006-AC-02:** Given Riley's entry converts, when she next views FEAT-20.SPEC-002, then it shows "Booked" for that entry.

**FEAT-20.SPEC-006-AC-03:** Given two clients were both Notified for the same freed slot and Riley completes payment first, when this automation processes her completed Booking, then only her entry converts and the other client's entry remains Notified, unconverted.

**FEAT-20.SPEC-006-AC-04:** Given Riley completes an ordinary booking unrelated to any waitlist entry, when this automation evaluates it, then no matching Notified entry is found and no action is taken.

**FEAT-20.SPEC-006-AC-05:** Given Riley holds a Notified entry for one slot but books a different time for the same service, when this automation evaluates the completed Booking, then it does not match her Notified entry, and that entry remains active.

**FEAT-20.SPEC-006-AC-06:** Given Riley's claim window lapses before her payment completes, when this automation checks the window at completion, then no conversion is recorded and her entry is instead handled by FEAT-20.SPEC-007.

**FEAT-20.SPEC-006-AC-07:** Given a second notified client's payment attempt is refused by FEAT-03.SPEC-005's contention resolution after the first client already won the slot, then this automation is never triggered for the second client's failed attempt.

**FEAT-20.SPEC-006-AC-08:** Given Riley's entry converts, when the conversion completes, then no separate waitlist-specific notification is sent -- only the ordinary booking confirmation (FEAT-08.SPEC-001) she already sees.

**FEAT-20.SPEC-006-AC-09:** Given the conversion write itself fails due to a transient error, when the failure occurs, then Riley's completed Booking is unaffected and her entry's state is resolved by the next successful pass or by FEAT-20.SPEC-007's expiry.

**FEAT-20.SPEC-006-AC-10:** Given two different clients each complete a claim for two different freed slots at effectively the same time, when this automation processes both, then each conversion runs independently.

**FEAT-20.SPEC-006-AC-11:** Given an entry has already converted, when a second completed-Booking signal somehow arrives referencing the same entry, then no eligible Notified entry is found and no further action is taken.

**FEAT-20.SPEC-006-AC-12:** Given Riley's Notified entry converts, when the entry is later inspected, then it is retained indefinitely as a terminal, historical record per this Brief's Non-Goals.

**FEAT-20.SPEC-006-AC-13:** Given a Converted entry exists, when anyone looks for a way to re-activate or re-match it, then no such control exists anywhere in the product.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

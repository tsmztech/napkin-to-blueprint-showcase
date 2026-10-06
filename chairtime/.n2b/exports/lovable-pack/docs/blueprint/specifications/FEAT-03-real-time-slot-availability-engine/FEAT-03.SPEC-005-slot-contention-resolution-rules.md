---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-03.SPEC-005
spec_name: Slot Contention Resolution Rules
spec_slug: slot-contention-resolution-rules
parent_feature: FEAT-03
parent_feature_name: Real-Time Slot Availability Engine
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-26
rule_count: 10
acceptance_criteria_count: 11
---

# Logic/Rule Spec: Slot Contention Resolution Rules

## Overview

**Name:** Slot Contention Resolution Rules
**ID:** FEAT-03.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs how a contested slot -- two clients attempting the same time, or a client colliding with a Pro-side change -- resolves: the first committed action wins, and every other attempt sees a plain re-pick message, never a payment error.
**Parent Feature:** FEAT-03 -- Real-Time Slot Availability Engine
**Governed Entity:** Slot Hold, plus its contention against Booking and Time Block occupancy

## Scope and Non-Goals

**In Scope:**
- The tie-break rule for two clients (or a client and a Pro-created deposit-request hold) contending for the same slot at effectively the same time
- The tie-break rule for a client's in-progress checkout colliding with a Pro-side change (a new Time Block, a Pro-side booking) committed first
- The exact "just taken" experience every losing attempt receives
- Authorization for who can trigger a contention outcome and who is shown what

**Non-Goals:**
- Creating the Slot Hold that becomes contended -- owned by FEAT-03.SPEC-002 (checkout) and FEAT-03.SPEC-007 (Pro-created deposit request); this spec governs only the tie-break outcome, not hold creation itself
- Expiring an uncontested hold that simply times out -- owned by FEAT-03.SPEC-003 and FEAT-03.SPEC-007; this spec applies only when two committed actions actually collide
- The waitlist's 30-minute priority window -- excluded per this Brief's Non-Goals: XBR-28 assigns that timing rule to FEAT-20 (Waitlist for Cancelled Slots); this spec supplies only the underlying free/not-free slot truth and generic tie-break mechanism FEAT-20 builds on
- Deciding deposit refund or forfeiture outcomes for a losing or bumped booking -- owned by FEAT-09 (Cancellation & No-Show Policy Engine); this spec governs only which attempt wins the slot, not the money consequence of losing one

## Governed Entity

**Entity:** Slot Hold, contended against Booking and Time Block occupancy
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|--------------|
| service | reference | The Service the contended hold or booking is for |
| start_time / duration | date/time, number | The specific contested span |
| owning_actor | enum (Client checkout, Pro deposit request, Pro-side change) | Which actor's action is attempting to claim or occupy the span |
| commit_timestamp | date/time | The exact moment the action was committed (hold created, or Time Block/Booking written) |
| state | enum (Active, Expired, Consumed) | Slot Hold's own lifecycle state, per the Entity-Lifecycle Coverage Matrix |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|--------------------|
| FEAT-03.SPEC-002 | Slot Hold Creation & Checkout Reservation | At the moment a checkout hold-creation attempt finds an existing conflicting hold or occupancy |
| FEAT-03.SPEC-003 | Slot Hold Expiration | When an expiring hold races against a completing payment for the same slot |
| FEAT-03.SPEC-007 | Pro-Created Deposit Request Hold & Expiration | At the moment a Pro-created hold-creation attempt finds an existing conflicting hold or occupancy |
| FEAT-20.SPEC-004 | Waitlist Priority Claim Window Rule | Builds the waitlist priority window on this tie-break: when two notified clients claim the same freed slot, the first to complete payment wins under this rule |
| FEAT-05 | Public Booking Page & Booking Flow | Displays the "just taken" message and refreshed list to a losing client |
| FEAT-10 | Client-Initiated Cancel/Reschedule | Displays the same contention outcome when a reschedule's new-time attempt is contended |
| FEAT-17 | Manual Time Blocking | A Time Block committed first against an in-progress checkout produces the Pro-side-change contention outcome |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| commit_timestamp | The action with the earliest commit_timestamp for a given contested span is the winner; every later action for the same span is refused | Always, whenever two or more actions target the same or overlapping span | At the instant a second (or later) action attempts to commit against an already-committed span | "That time was just taken. Here are the current available times." (client-facing); the Pro sees the conflict on their dashboard if the Pro-side action loses (rare, since a Pro-side change ordinarily wins against a client) | Yes |
| owning_actor | No validation beyond data type -- contention resolution applies identically regardless of which actor type is involved | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| First-committed-wins | commit_timestamp, owning_actor | Whichever action (checkout hold creation, Pro-created hold creation, or a Pro-side Time Block/Booking write) commits first for a given span wins it; every other action targeting the same or overlapping span at that point is refused, regardless of actor type | "That time was just taken. Here are the current available times." |
| Checkout-hold vs. Pro-side-change collision | commit_timestamp (hold), commit_timestamp (Time Block/Booking) | If a Pro-side change commits before the client's payment completes, the client's in-progress checkout is refused even though a hold already exists on the slot, per this Brief's Time Block Contention note ("first committed wins... a block committed first removes the slot and the client's confirmation is refused with a refreshed slot list") | "That time was just taken. Here are the current available times." |
| Hold-expiry-vs-payment-completion race | commit_timestamp (payment completion), expiry timestamp (hold) | If payment completes before the hold's expiry is processed, the hold is Consumed, not Expired -- payment completion counts as the earlier "commit" for this race (FEAT-03.SPEC-003) | -- (no error; this is the winning path) |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|----------------------------------------------|
| Trigger a contention check | The Client (Riley), the Pro (Talia) | Whenever either commits an action (hold creation, Time Block, booking) against a span another action already occupies | -- |
| Receive the "just taken" message | The Client (Riley) | Only the losing attempt in a client-vs-client or client-vs-Pro-side-change contention | -- (this is the denied behavior itself) |
| See the conflicting Pro-side change that caused a client's loss | The Pro (Talia) | Always, on her own dashboard, if her own action lost a rare Pro-vs-Pro-created-hold race | Flagged for her explicit choice per XBR-11 (setup changes never silently cancel a confirmed booking) |
| View another client's contention outcome or identity | The Client (Riley) | Never | Riley sees only "that time was just taken," never who took it or any detail about the other client |
| View another client's contention outcome or identity | Platform Operator (Support) | View-only, for troubleshooting a specific Pro's reported conflict; never shown another client's identity beyond what troubleshooting requires | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|----------------------|
| commit_timestamp | Set automatically to the exact system time the action is written (hold created, or Time Block/Booking committed) | On every hold creation, Time Block creation, or Booking write | No -- never client- or Pro-editable |
| winner determination | Derived by comparing commit_timestamp across every action targeting the same or overlapping span; earliest wins | Whenever a contention is detected | No |

## Business Rules

- A time is offered, held, or booked only if it passes the live slot check; the first client to complete payment wins a contested slot and the other sees a plain "just taken" message, never a payment error (XBR-01) -- this is the rule this spec formalizes as the tie-break itself.
- Slot holds are time-limited and release automatically; a race between an expiring hold and a completing payment resolves in the completing payment's favor if payment commits first (XBR-02, FEAT-03.SPEC-003).
- A Time Block committed first removes the slot and refuses a contended client confirmation with a refreshed slot list; a client's confirmation committed first makes the block a conflict the Pro must explicitly decide (cancel, reschedule, or keep as an exception), per this Brief's Time Block Contention note and XBR-11 -- this spec's tie-break rule underlies both directions.
- Contention resolution never merges two conflicting actions into a combined or partial outcome -- exactly one action wins a given span, and every other action for that span is refused outright.
- The losing party is always shown a refreshed live slot list immediately alongside the "just taken" message, never left on a page showing the now-stale slot as still available.

## Edge Cases

- **Two clients' checkout holds are created within the same millisecond for the same slot** -- Exactly one hold-creation write succeeds (the underlying single-Pro-Account data store enforces this); the other is refused as a contention loss even though both attempts appeared simultaneous to their respective clients.
- **A client's payment completes and a Pro-created deposit-request hold is attempted on the same slot at nearly the same time** -- Whichever action's commit_timestamp is earlier wins; if the client's payment committed first, the Pro's attempt to create a deposit-request hold on that slot fails with the Pro seeing the slot is no longer available.
- **A Pro's Time Block is committed for a span where a client's checkout hold already exists (Active, unexpired)** -- The existing hold committed first, so it wins: the Time Block creation is refused or flagged to the Pro as a conflict requiring her explicit choice (FEAT-17, XBR-11), not silently applied over an active hold.
- **A losing client retries the exact same slot immediately after seeing "just taken"** -- The retry is evaluated as a brand-new contention check against current data; if the slot is still occupied, the same refusal recurs; if it has since freed (e.g., the winning hold itself later expired), the retry can succeed.
- **Contention outcome must be determined but the underlying data store cannot confirm which action committed first (a rare consistency failure)** -- The system defaults to refusing both contending actions rather than guessing a winner, and both parties see a refreshed live list; neither is shown as confirmed until a clean, unambiguous single-winner commit succeeds.

## Acceptance Criteria

**FEAT-03.SPEC-005-AC-01:** Given Riley and a second client both attempt to hold the same slot at effectively the same time, when the hold-creation attempts are evaluated, then exactly one succeeds (the earliest commit_timestamp) and the other sees "That time was just taken. Here are the current available times."

**FEAT-03.SPEC-005-AC-02:** Given Riley's checkout hold is Active on a slot, when Talia attempts to place a Time Block over that same span, then the Time Block attempt is refused or flagged to Talia as a conflict requiring her explicit choice, since Riley's hold committed first.

**FEAT-03.SPEC-005-AC-03:** Given Talia commits a Time Block over a span before any client has begun checkout on it, when a client subsequently attempts to hold that span, then the hold-creation attempt is refused with the "just taken" message, since the block committed first.

**FEAT-03.SPEC-005-AC-04:** Given Riley's checkout hold's timeout is about to elapse at the same moment Riley's payment completes, when both are evaluated, then the hold is Consumed by the completed payment, not expired, because payment completion is treated as the earlier commit.

**FEAT-03.SPEC-005-AC-05:** Given a losing client sees the "just taken" message, when the message displays, then Riley also sees a refreshed live slot list immediately, never a stale page still showing the taken slot as available.

**FEAT-03.SPEC-005-AC-06:** Given Riley loses a contention, when Riley checks whether they can see who took the slot, then no other client's identity or detail is ever shown -- only the plain "just taken" message.

**FEAT-03.SPEC-005-AC-07:** Given Talia's own Pro-created deposit-request hold attempt loses a rare race against a client's just-completed payment, when Talia views her dashboard, then she sees the slot is no longer available for that deposit request, with no ambiguity about the outcome.

**FEAT-03.SPEC-005-AC-08:** Given Riley retries the same slot immediately after losing a contention, when the retry is evaluated and the slot is still occupied, then Riley sees the same "just taken" refusal.

**FEAT-03.SPEC-005-AC-09:** Given Riley retries the same slot after the winning hold has since expired, when the retry is evaluated, then Riley's new attempt can succeed since the slot is now free.

**FEAT-03.SPEC-005-AC-10:** Given a rare data-consistency failure prevents determining which of two contending actions committed first, when the contention check runs, then both actions are refused and both parties see a refreshed live list rather than either being shown as confirmed.

**FEAT-03.SPEC-005-AC-11:** Given Riley is rescheduling an existing booking and the new time becomes contended by another client mid-flow, when the contention resolves against Riley, then Riley sees the "just taken" message and a refreshed live list, with the original booking left untouched.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 2 | 2 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |

---
document_type: feature-overview
feature_number: FEAT-11
feature_name: No-Show Marking & Deposit Forfeiture
feature_slug: no-show-marking-deposit-forfeiture
priority_tier: Core
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 4
screen_count: 1
automation_count: 2
logic_rule_count: 1
integration_count: 0
notification_count: 0
---

# Feature Breakdown Brief: No-Show Marking & Deposit Forfeiture

## Summary

**Feature:** No-Show Marking & Deposit Forfeiture
**ID:** FEAT-11
**Description:** The Pro marks a booking as a no-show when a client fails to appear, and the deposit is forfeited to the Pro automatically under the agreed policy -- with no manual chasing, invoicing, or renegotiation required.
**Priority:** Core
**Phase:** MVP
**Type:** User-Facing
**Rationale:** This is the founder's headline promise verbatim: "the pro never chases a no-show again," and BRIEF.md's Success Criteria states "Pros say 'I haven't had an unpaid no-show since I switched.'" The mechanism must be a single, low-effort action for the Pro. [RESEARCH-INFORMED: deposit-based no-show protection is consistently credited with materially reducing no-shows across three competitors (3 sources, HIGH confidence); because the deposit was already captured at booking, marking a no-show moves no money -- it only makes the deposit non-refundable, so there is nothing left to chase]

**Key Capabilities:**
- Mark a past-due booking as a no-show in one tap from the daily schedule
- See the deposit automatically reflected as kept, with no separate invoicing step
- Reverse a mistaken no-show mark (e.g., the client did show up) within a short grace period
- Choose goodwill instead -- refund the deposit in full for a genuine emergency through Pro Booking Management (FEAT-30) rather than marking a no-show [AUDIT-ADDED: 1 -- reversal path: BRIEF.md lets the Pro "refund within policy", which the no-show flow did not connect to]

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-11.SPEC-001 | No-Show Mark & Undo Prompt | Screen | The Pro | One-tap prompt reached from a past-due booking row that marks a no-show, offers "refund as goodwill instead" (to FEAT-30), or -- within the 24-hour grace window -- offers Undo, reflecting the current deposit outcome inline |
| FEAT-11.SPEC-002 | No-Show Marking & Deposit Forfeiture | Automation | The Pro, The Client, Platform Operator (Support) | On confirmation, transitions the Booking to No-Show and the Deposit Transaction to Forfeited in one step, deriving the outcome from the booking's acknowledged policy version |
| FEAT-11.SPEC-003 | No-Show Mark Undo | Automation | The Pro, The Client, Platform Operator (Support) | Within the 24-hour grace window, reverses a no-show mark -- restoring the Booking to Completed and the Deposit Transaction to its prior Captured status |
| FEAT-11.SPEC-004 | No-Show Marking Window & Authorization Rules | Logic/Rule | The Pro, Platform Operator (Support) | Governs who may mark or undo a no-show (the Pro, on their own bookings only), the eligible marking window (after start time, before the 7-day auto-completion), and the fixed 24-hour undo grace window |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Mark a past-due booking as a no-show in one tap from the daily schedule | FEAT-11.SPEC-001, FEAT-11.SPEC-002, FEAT-11.SPEC-004 | The prompt captures the one tap; the window/authorization rule confirms eligibility; the automation writes the Booking and Deposit Transaction transitions atomically | Phase 2 (Explicit) |
| See the deposit automatically reflected as kept, with no separate invoicing step | FEAT-11.SPEC-001, FEAT-11.SPEC-002 | The automation writes the Forfeited outcome the instant the mark is confirmed; the prompt reflects it inline with no separate invoicing screen or step | Phase 2 (Explicit) |
| Reverse a mistaken no-show mark within a short grace period | FEAT-11.SPEC-001, FEAT-11.SPEC-003, FEAT-11.SPEC-004 | The prompt offers Undo only while the rule spec's 24-hour window holds; the undo automation reverts both the Booking and Deposit Transaction cleanly | Phase 2 (Explicit) |
| Choose goodwill instead -- refund the deposit in full through Pro Booking Management (FEAT-30) rather than marking a no-show | FEAT-11.SPEC-001 | The prompt offers a navigation choice to FEAT-30's goodwill refund flow as an alternative to confirming the no-show mark; the refund logic itself is FEAT-30's own | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-11.SPEC-002 | No-Show Marking & Deposit Forfeiture | Phase 4 (Trigger-Response) | Marking a no-show is a cross-entity side-effect -- one confirmed tap must atomically transition both the Booking and the Deposit Transaction, and a failed write must be retried and flagged rather than silently dropped (States field) -- too consequential to leave inline in the prompt |
| FEAT-11.SPEC-003 | No-Show Mark Undo | Phase 4 (Trigger-Response) / Phase 6 (Failure Analysis) | The Alternate flow's undo path is itself a cross-entity reversal (Booking and Deposit Transaction both revert) with its own eligibility window and failure handling -- distinct enough from the forward marking automation to warrant its own spec rather than a shared one |
| FEAT-11.SPEC-004 | No-Show Marking Window & Authorization Rules | Phase 5 (Rule Discovery) | The Validation & Limits field names three interacting conditions (start-time-passed lower bound, auto-completion upper bound, fixed 24-hour undo window) plus an ownership authorization rule (Pro, own bookings only) -- past the inline-validation threshold and shared across the prompt and both automations |

## Entity-Lifecycle Coverage Matrix

**Entity: Deposit Transaction**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A | Owned by FEAT-07 (Deposit Payment at Booking) -- the transaction already exists, Captured, before this feature ever acts | This feature only ever transitions an existing transaction |
| Read (single) | FEAT-11.SPEC-002, FEAT-11.SPEC-003 | Read internally to confirm current status (Captured, to forfeit; Forfeited, to undo) before writing a new transition | Display of transaction status to the Pro, Client, or Support is owned by FEAT-16 and FEAT-28, not a screen this feature produces |
| Read (list) | N/A | This feature produces no list view of Deposit Transactions | The money list is FEAT-28's screen; the activity record is FEAT-16's |
| Update | FEAT-11.SPEC-002 (Captured -> Forfeited), FEAT-11.SPEC-003 (Forfeited -> Captured, undo) | Each write is the sole, terminal-per-attempt transition for this feature; a failed write is retried and flagged, never silently dropped (States field) | Any later Disputed overlay or goodwill refund is owned by FEAT-16's dispute notice handling and FEAT-30, respectively -- not this feature |
| Delete/Archive | N/A | No delete path exists for a financial record | Hard delete never occurs; per SC-22 the record is retained for the life of the account and de-identified (never removed) only after client deletion or account closure -- recorded as an explicit non-goal below |
| State Transition | FEAT-11.SPEC-002, FEAT-11.SPEC-003 | Captured -> Forfeited on marking; Forfeited -> Captured on undo within the 24-hour grace window (FEAT-11.SPEC-004) | A deposit can be refunded only once overall (dependency map's Contention rule); this feature's transitions never touch a Refunded or Disputed transaction |

**Booking (narrow update, not fully managed by this feature):**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A | Owned by FEAT-05, FEAT-30, FEAT-21 | Not this feature's concern |
| Read (single) | FEAT-11.SPEC-001, FEAT-11.SPEC-002, FEAT-11.SPEC-003, FEAT-11.SPEC-004 | Reads the booking's start time, current state, and policy version to display the prompt and to validate the marking/undo window | -- |
| Read (list) | N/A | The past-due booking row this feature is invoked from is FEAT-12's dashboard list | -- |
| Update | FEAT-11.SPEC-002 (Awaiting Outcome -> No-Show), FEAT-11.SPEC-003 (No-Show -> Completed, undo) | The only two Booking transitions this feature ever writes | Every transition is validated against the current state per the dependency map's Contention rule (e.g., a completed or already-cancelled booking cannot be marked) |
| Delete/Archive | N/A | Bookings are kept for the life of the account (SC-22); no delete path exists in the product at all | Recorded as an explicit non-goal below, consistent with every other feature that touches Booking |
| State Transition | FEAT-11.SPEC-002, FEAT-11.SPEC-003, governed by FEAT-11.SPEC-004 | The two transitions above, bounded by the eligibility window rule spec | All other Booking states and transitions (Pending Payment, Confirmed, Cancelled, Rescheduled, Expired, and auto-completion) are owned by FEAT-05, FEAT-07, FEAT-10, FEAT-12, FEAT-30 |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Cancellation Policy | FEAT-11.SPEC-002 | The forfeiture outcome is derived from the policy version the booking's client acknowledged (XBR-08, XBR-09), owned and versioned by FEAT-09 -- this feature never edits policy terms, only reads the binary no-show outcome it defines |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Pro taps "no-show" on a past-due booking row | Open the mark/undo prompt, showing "Mark no-show?" (not yet marked) or "Undo no-show?" (already marked, within the grace window) | Inline in triggering screen (opens the prompt) | FEAT-11.SPEC-001 (invoked from FEAT-12) |
| Pro confirms "mark no-show" | Validate the marking window and Pro ownership, then atomically transition the Booking to No-Show and the Deposit Transaction to Forfeited | Standalone Logic/Rule, then Standalone Automation | FEAT-11.SPEC-004, FEAT-11.SPEC-002 |
| Booking marked no-show / deposit forfeited | Write the no-show mark and forfeiture events to the append-only activity record | Cross-feature -- owned by FEAT-16 (XBR-21) | FEAT-16 responsibility |
| Booking marked no-show / deposit forfeited | Feed the no-show rate and forfeited-revenue aggregates | Cross-feature | FEAT-25 responsibility |
| Pro confirms "undo" within the 24-hour grace window | Validate the window has not elapsed, then atomically transition the Booking back to Completed and the Deposit Transaction back to Captured | Standalone Logic/Rule, then Standalone Automation | FEAT-11.SPEC-004, FEAT-11.SPEC-003 |
| The 24-hour undo grace window elapses | The prompt stops offering Undo; the no-show mark becomes permanent | Inline in triggering screen, governed by the rule spec | FEAT-11.SPEC-001, FEAT-11.SPEC-004 |
| A forfeiture or undo write fails (connectivity or processing error) | Retry the write and flag it for the Pro rather than silently dropping it, since money is at stake (States field) | Inline failure handling within the triggering automation | FEAT-11.SPEC-002 / FEAT-11.SPEC-003 |
| Pro taps "refund as goodwill instead" from the prompt | Navigate to Pro Booking Management's goodwill refund flow instead of marking a no-show | Cross-feature -- navigation and refund logic owned by FEAT-30 | FEAT-30 responsibility |
| Booking is marked no-show | No separate message is sent to the client; the deposit-kept outcome is visible only in the client's own booking history | Inline in triggering screen (no delivery rules -- the Phase 4 inline-communication exception) | N/A (notification_count: 0 by design) |

The feature's Communications field is explicit and deliberate: the deposit outcome is "visible in the client's own booking history rather than triggering a separate confrontational notification," and the Pro's action is "deliberately low-friction and silent toward the client." This is a same-screen-visibility disposition with no channel, audience, or delivery rule of its own, so it stays inline per Phase 4's inline-communication exception rather than becoming a standalone Notification spec. The field's own [CHALLENGED] annotation flags a possible future neutral outcome notice as worth revisiting against the Policy Clarity at Booking metric, but records that the original silent disposition is retained for this decomposition -- the Analyst elaborates the Stage 2 decision as given rather than overriding it. `notification_count: 0` and `integration_count: 0` are both intentional: the assumptions-constraints slice states plainly that marking a no-show "moves no money" and requires no external call of its own, since the deposit was already captured by FEAT-07 through payment processing (ASMP-31); this feature is not listed on any External Touchpoints row.

## Shared Context

**Shared Entities:**
- Deposit Transaction -- read by SPEC-002 and SPEC-003 to confirm current status before writing; transitioned Captured -> Forfeited by SPEC-002 and reversed by SPEC-003. Fields touched: status, outcome_reason/timestamps.
- Booking (narrow slice) -- read by SPEC-001, SPEC-002, SPEC-003, and SPEC-004 for start_time, state, and policy_version; updated only by SPEC-002 and SPEC-003, and only for the No-Show <-> Completed transition pair.

**Shared UI Patterns:**
- Single prompt surface -- SPEC-001 is the one screen for both marking and undoing; it toggles between "Mark no-show?" and "Undo no-show?" based on the Booking's current state and the elapsed time since marking, rather than presenting two separate screens. Spec Writers should describe both states of this one prompt consistently.

**Shared Validation:**
- SPEC-004 defines all eligibility, window, and authorization rules (marking window bounds, 24-hour undo window, Pro-owns-this-booking check). SPEC-001, SPEC-002, and SPEC-003 all reference it rather than duplicating the checks.

## Internal Dependency Map

```
SPEC-001 (No-Show Mark & Undo Prompt) -> [Pro confirms "mark no-show"] -> SPEC-004 (Window & Authorization Rules) -> [eligible] -> SPEC-002 (No-Show Marking & Deposit Forfeiture)
SPEC-002 (No-Show Marking & Deposit Forfeiture) -> [Booking + Deposit Transaction updated] -> SPEC-001 (reflects kept outcome, now offers Undo)
SPEC-001 (No-Show Mark & Undo Prompt) -> [Pro confirms "undo"] -> SPEC-004 (Window & Authorization Rules) -> [within 24-hour grace window] -> SPEC-003 (No-Show Mark Undo)
SPEC-003 (No-Show Mark Undo) -> [Booking + Deposit Transaction reverted] -> SPEC-001 (reflects restored prior state)
SPEC-001 (No-Show Mark & Undo Prompt) -> [Pro taps "refund as goodwill instead"] -> FEAT-30 (Pro Booking Management)
```

**Default Entry:** SPEC-001 (No-Show Mark & Undo Prompt) -- reached only from a specific past-due booking row on FEAT-12's dashboard; this feature has no standalone entry point of its own (per its Interactions field: "invoked from Pro Daily Schedule Dashboard").

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-11.SPEC-001 | Inbound | FEAT-12 (Pro Daily Schedule Dashboard) | Pro taps "no-show" on a past-due booking row, opening the prompt | Booking's appointment start time has passed |
| FEAT-11.SPEC-001 | Outbound | FEAT-30 (Pro Booking Management) | Pro chooses a full goodwill refund instead of marking a no-show | Pro taps "refund as goodwill instead" from the prompt |
| FEAT-11.SPEC-002 | Inbound | FEAT-09 (Cancellation & No-Show Policy Engine) | Forfeiture outcome is derived from the booking's acknowledged policy version | Booking is marked no-show |
| FEAT-11.SPEC-002 | Inbound | FEAT-07 (Deposit Payment at Booking) | The Deposit Transaction this feature forfeits was created and captured at booking | Booking is marked no-show |
| FEAT-11.SPEC-002 | Outbound | FEAT-16 (Booking & Payment Activity Record) | No-show mark and forfeiture events are written to the append-only activity record, later used as dispute evidence | booking_marked_no_show / deposit_forfeited signals fire |
| FEAT-11.SPEC-003 | Outbound | FEAT-16 (Booking & Payment Activity Record) | Undo event is written to the activity record | no_show_mark_undone signal fires |
| FEAT-11.SPEC-002 | Outbound | FEAT-25 (Booking & Revenue Insights) | Forfeited deposit feeds no-show rate and revenue aggregates | Booking is marked no-show |

## Non-Functional Notes

**Data volumes / growth:** No-show marks apply to a subset of a Pro's 20-40 bookings a week (SC-19); volume tracks Booking volume directly and stays well within a solo pro's scale over multiple years of history (ASMP-22). No separate growth pattern applies to this feature specifically.

**Responsiveness:** The mark/undo action is a single, instantaneous local-feeling action once eligible (States field: "Loading: N/A -- instantaneous local action"); however, ASMP-27 requires connectivity for any action that books, pays, cancels, refunds, or marks a no-show, so the prompt must say plainly when connectivity is missing rather than silently queue the write, and a failed write is retried and flagged rather than dropped (States field, Error).

**Data sensitivity / privacy:** Both entities this feature updates are personal, financial records tied to an identifiable client (Booking, Deposit Transaction) -- visible only to the Pro (Full access, own bookings only) and, view-only, to Platform Operator (Support) for dispute troubleshooting; the Client sees only the resulting deposit-kept outcome in their own booking history, never the mark/undo action itself (Access field; Access Matrix, Cancellation & No-Show Handling row).

**Compliance flags:** N/A -- no compliance regime beyond the general financial-record retention rule (SC-22) applies specifically to this feature; no card data is touched here, since the deposit was already captured by FEAT-07 through payment processing (ASMP-31) before this feature ever acts.

## Non-Goals

- **A full goodwill refund as an alternative to a no-show mark** -- Owned by Pro Booking Management (FEAT-30), per this feature's own Key Capabilities line and Interactions field ("alternative path through Pro Booking Management (FEAT-30)"); this feature only offers the navigation choice, never the refund logic itself.
- **Partial refunds or tiered forfeiture** -- Excluded per SC-18: deposit outcomes are binary in v1; a no-show always keeps the full deposit already captured, never a percentage or schedule.
- **Client-side marking or in-app dispute of a no-show** -- Excluded per the Access field and the Access Matrix's Cancellation & No-Show Handling row: the Client's access is Own-only visibility of the outcome; a client who disagrees contacts the Pro directly, with any escalation running through FEAT-16's activity record and, ultimately, the card issuer's own dispute process (SC-17).
- **Support acting on the Pro's behalf** -- Excluded per SC-05: Platform Operator (Support) has View-only access for dispute troubleshooting; support cannot mark, unmark, or refund on the Pro's account.
- **Chairtime adjudicating no-show disputes** -- Excluded per SC-17: the product provides the trustworthy record (FEAT-16) and lets the Pro decide goodwill (FEAT-30); it never rules on who is right between Pro and client.
- **A neutral outcome notice sent to the client at the moment of forfeiture** -- The Communications field deliberately keeps the outcome silent, visible only in the client's own booking history; its own [CHALLENGED] annotation flags this as worth revisiting against the Policy Clarity at Booking metric, but the Stage 2 decision is retained as-is in this decomposition rather than overridden by the Analyst.
- **Deletion or purge of the Booking or Deposit Transaction this feature updates** -- Excluded per SC-22: full booking, client, and deposit history is retained for the life of the account for dispute evidence and insights; only de-identified financial records survive a client deletion or account closure, and no automatic purge applies before that.
- **Auto-completing a past-due booking** -- Owned by FEAT-12 per XBR-12; this feature only bounds its own marking window against that auto-completion (7 days after the appointment), it does not perform the completion itself.
- **Re-charging the client's card after a no-show** -- Excluded per SC-13: no card is kept on file for later charges; the deposit was already captured at booking (FEAT-07), so marking a no-show moves no money and needs no new charge.

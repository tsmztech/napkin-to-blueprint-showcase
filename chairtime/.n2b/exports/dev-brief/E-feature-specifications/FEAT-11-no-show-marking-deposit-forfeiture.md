# FEAT-11 — No-Show Marking & Deposit Forfeiture

This chapter covers No-Show Marking & Deposit Forfeiture (FEAT-11), a Core-tier feature. It carries 4 specifications carrying 54 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-11.SPEC-001 | No-Show Mark & Undo Prompt | screen | 15 |
| FEAT-11.SPEC-002 | No-Show Marking & Deposit Forfeiture | automation | 11 |
| FEAT-11.SPEC-003 | No-Show Mark Undo | automation | 11 |
| FEAT-11.SPEC-004 | No-Show Marking Window & Authorization Rules | logic-rule | 17 |

The feature breakdown brief follows, then every specification in full.


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



# Screen Spec: No-Show Mark & Undo Prompt

## Overview

**Name:** No-Show Mark & Undo Prompt
**ID:** FEAT-11.SPEC-001
**Type:** Screen
**Purpose:** Talia reaches this one prompt from a past-due booking row to mark a no-show, choose a goodwill refund instead, or -- within the 24-hour grace window (platform parameter: `no-show-undo-grace-window-hours`) -- undo a mark she already made, seeing the current deposit outcome reflected inline the instant she confirms.
**Parent Feature:** FEAT-11 -- No-Show Marking & Deposit Forfeiture

## Scope and Non-Goals

**In Scope:**
- The single prompt surface that toggles between a "Mark no-show?" state (booking not yet marked) and an "Undo no-show?" state (booking marked, undo grace window still open)
- Confirming a no-show mark, which hands off to FEAT-11.SPEC-002 to write the Booking and Deposit Transaction transitions
- Confirming an undo, which hands off to FEAT-11.SPEC-003 to reverse those transitions
- Offering "Refund as goodwill instead" as a navigation choice out to Pro Booking Management (FEAT-30)
- Reflecting the deposit outcome (kept, or restored) inline the instant the triggering automation completes, with no separate invoicing step
- Showing when the undo window has elapsed, so the mark is understood as permanent

**Non-Goals:**
- Performing the no-show marking window and ownership checks themselves -- owned by FEAT-11.SPEC-004 (Logic/Rule); this screen only reflects the eligibility outcome the rule spec returns
- Writing the Booking or Deposit Transaction state transitions directly -- owned by FEAT-11.SPEC-002 (mark) and FEAT-11.SPEC-003 (undo); this screen only triggers them and displays their result
- The goodwill refund flow itself -- owned entirely by Pro Booking Management (FEAT-30), per this feature's Key Capabilities and Interactions field; this screen only offers the navigation choice
- Sending any notice to the Client -- excluded per the feature's Communications field, which keeps the deposit outcome "visible in the client's own booking history rather than triggering a separate confrontational notification"; this is a same-screen-visibility disposition with no channel or delivery rule of its own

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-12 (Pro Daily Schedule Dashboard) -- past-due booking row | Talia taps "no-show" on a booking row whose appointment start time has passed | The Booking's identity, current state, start_time, client name, and policy_version |

This feature has no standalone entry point of its own -- per the Brief's Internal Dependency Map, the only way in is a specific past-due booking row on FEAT-12's dashboard.

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen, for her own bookings only | Mark no-show, undo within the grace window, choose goodwill refund instead | If Talia somehow reaches this prompt for a booking she does not own (e.g. a stale deep link), the prompt does not open; she sees "This booking could not be found." and returns to her dashboard |
| The Client (Riley) | No -- this screen is never reachable from any Client-facing surface | No | Riley has no path to this prompt at all; her Cancellation & No-Show Handling access is Own-only visibility of the resulting outcome in her own booking history (FEAT-16), never this action screen |
| Platform Operator (Support) | No -- Support's View access to Cancellation & No-Show Handling is served through Platform Support Read-Only Access (FEAT-19)'s own read-only screens, not this prompt | No | Support has no path to this prompt; a booking's no-show state and forfeiture outcome are visible to Support only through FEAT-19 |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen; no in-progress prompt context is preserved since the prompt cannot open without an authenticated Pro session |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- if the prompt was open with a choice not yet confirmed, no partial write exists (this action is one atomic confirm), so nothing is lost or replayed after re-authentication; Talia returns to the dashboard and re-opens the prompt from the booking row |

## Layout and Content

**Header:** Prompt title that reflects the current state -- "Mark no-show?" when the Booking is not yet marked, or "Undo no-show?" when it is marked and the undo grace window is still open. A close control (X, top-right) dismisses the prompt without any change.

**Body (Mark no-show state):**
- The booking's client name and appointment time, for confirmation of which booking is being acted on
- A single line stating the deposit outcome that will result: the deposit is kept in full once marked (no percentage, no invoicing step)
- Three actions, stacked: "Mark no-show" (primary), "Refund as goodwill instead" (secondary -- navigates to FEAT-30), and "Cancel" (dismisses the prompt)

**Body (Undo no-show state):**
- The same booking identity line
- A line confirming the current outcome: the deposit is kept, marked as a no-show at the timestamp it was marked
- A line stating how much of the undo grace window remains (e.g., expressed as remaining hours), computed against the fixed grace window governed by FEAT-11.SPEC-004
- Two actions, stacked: "Undo no-show" (primary) and "Keep as no-show" (secondary -- dismisses the prompt, no change)

**Body (grace window elapsed):**
- The same booking identity and outcome line
- A line stating the mark is now permanent: "This no-show mark can no longer be undone."
- One action: "Close" (dismisses the prompt)

**Footer:** None -- all actions live in the body.

### Responsive Behavior

- **Compact breakpoint:** The prompt renders as a full-width bottom sheet with the layout described above, stacked vertically.
- **Medium size class and above:** The prompt renders as a centered modal dialog capped at a consistent platform-wide dialog width (exact value is the design layer's decision); content and action order are unchanged from the compact layout -- uniform scaling, no structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Close (X) | Tap | Dismiss the prompt, no change | Prompt closes | Return to FEAT-12's dashboard, booking row unchanged |
| "Mark no-show" button | Tap | Re-validate eligibility via FEAT-11.SPEC-004, then trigger FEAT-11.SPEC-002 | Button shows a brief loading state | On success: prompt switches in place to the Undo state showing the kept-deposit outcome. On ineligibility (window closed since the prompt opened): prompt shows the exact denied message from FEAT-11.SPEC-004 and offers only "Close" |
| "Refund as goodwill instead" link | Tap | Navigate to Pro Booking Management's goodwill refund flow (FEAT-30), carrying this Booking's identity | Prompt closes | FEAT-30's refund screen opens, pre-scoped to this booking |
| "Cancel" button (Mark state) | Tap | Dismiss the prompt, no change | Prompt closes | Return to FEAT-12's dashboard, booking row unchanged |
| "Undo no-show" button | Tap | Re-validate the grace window via FEAT-11.SPEC-004, then trigger FEAT-11.SPEC-003 | Button shows a brief loading state | On success: prompt switches in place to the Mark state, reflecting the booking as Completed again with the deposit restored to Captured. On ineligibility (window elapsed since the prompt opened): prompt shows "This no-show mark can no longer be undone." and offers only "Close" |
| "Keep as no-show" button (Undo state) | Tap | Dismiss the prompt, no change | Prompt closes | Return to FEAT-12's dashboard, booking row still shows no-show/kept |
| "Close" button (elapsed state) | Tap | Dismiss the prompt, no change | Prompt closes | Return to FEAT-12's dashboard |

### Accessibility Notes

- **Focus order:** Close (X) -> title -> booking identity line -> outcome line -> primary action -> secondary action -> tertiary action (Cancel, when present).
- **State-change announcements:** When the prompt switches from Mark to Undo (or vice versa) after a successful confirm, the new title and outcome line are announced to assistive technology as a live-region update, since the content changes in place without a full screen navigation.
- **Error announcements:** An ineligibility message (window closed, session expired) is announced when it appears and moves focus to the message text.
- **Keyboard alternatives:** Every action on this prompt is reachable by keyboard; there are no pointer-only gestures. Escape closes the prompt with the same effect as the Close/Cancel control.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | The prompt shell (header, close control) renders immediately with a skeleton placeholder in place of the booking identity, outcome line, and action row | Talia taps "no-show" on a dashboard row and the prompt opens, triggering the fresh Booking and Deposit Transaction read described in Data Model Reads | The fresh read resolves -- the prompt renders directly into whichever of Mark, Undo, or Grace window elapsed the resolved state and outcome timestamp dictate |
| Empty | N/A -- this screen only ever opens against one specific past-due booking carried from the triggering dashboard row (per Entry Points); it is never a list or collection view and has no zero-items case to render | N/A | N/A |
| Mark (default) | "Mark no-show?" title, booking identity, kept-deposit-outcome preview, three actions | Prompt opens for a booking not yet marked no-show, within the eligible marking window | Talia confirms the mark, cancels, or closes |
| Undo (within grace window) | "Undo no-show?" title, booking identity, current kept-deposit outcome, remaining grace-window time, two actions | Prompt opens for a booking already marked no-show, within the undo grace window | Talia confirms the undo, keeps the mark, or closes |
| Grace window elapsed | "Undo no-show?" title suppressed in favor of a permanent-outcome message; one Close action | Prompt opens for a booking marked no-show whose undo grace window has elapsed | Talia closes the prompt |
| Confirming | Primary action button shows a loading state; other actions disabled | Talia taps "Mark no-show" or "Undo no-show" | The triggered automation (FEAT-11.SPEC-002 or FEAT-11.SPEC-003) returns an outcome |
| Error | Inline message describing the exact ineligibility reason, with only a Close action remaining | The re-validation on confirm (FEAT-11.SPEC-004) finds the booking no longer eligible, or the triggered automation reports a write failure that could not complete after retry | Talia closes the prompt and returns to the dashboard, where the booking row reflects the current true state |
| Offline/Degraded | Banner "You're offline. Marking or undoing a no-show needs a connection." replaces the action row; the outcome preview remains visible but no action can be confirmed | Connectivity is lost while the prompt is open | Connectivity is restored -- the action row reappears and the prompt re-validates eligibility before allowing a confirm |

## Validation Rules

Validation governed by FEAT-11.SPEC-004 (No-Show Marking Window & Authorization Rules). See that spec for the marking-window bounds, the fixed undo grace window, and the Pro-ownership check. This screen re-validates on prompt open and again on confirm, and surfaces the exact denied message that spec defines.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Close, Cancel, or Keep as no-show | Returns to the triggering booking row | FEAT-12 (Pro Daily Schedule Dashboard) |
| Successful mark or undo confirm | Prompt stays open, switched to the resulting state (no navigation) | -- |
| "Refund as goodwill instead" | Pro Booking Management's goodwill refund flow, scoped to this booking | FEAT-30 (Pro Booking Management) |

## Data Model

**Creates:** None.
**Reads:** Booking -- client name, start_time, state, policy_version (to determine which prompt state to show and to preview the deposit outcome). Deposit Transaction -- status and outcome_reason/timestamps (to show the current kept-outcome and compute remaining grace-window time).
**Updates:** None directly -- all Booking and Deposit Transaction transitions are written by FEAT-11.SPEC-002 and FEAT-11.SPEC-003, which this screen triggers.
**Deletes:** None.

## Business Rules

- The prompt shows exactly one of its three states (Mark, Undo, or Grace window elapsed) at a time, determined entirely by the Booking's state and the Deposit Transaction's outcome timestamp evaluated against FEAT-11.SPEC-004's window rules -- never a manual toggle.
- Eligibility is re-checked at prompt open and again at confirm (FEAT-11.SPEC-004), so a window that closes while the prompt sits open is caught before any write is attempted.
- A failed mark or undo write is retried by the triggering automation and, if it still cannot complete, flagged to Talia rather than silently dropped (per the Brief's Side-Effect Inventory) -- this screen surfaces that flag as the Error state rather than pretending the action succeeded.
- XBR-12 bounds this screen's Undo state to a fixed 24-hour window (platform parameter: `no-show-undo-grace-window-hours`) and its Mark state to the span between the booking's start_time and its 7-day auto-completion boundary (platform parameter: `booking-auto-completion-window-days`, owned by FEAT-12).

## Edge Cases

- **Talia taps "Mark no-show" twice rapidly** -- The second tap is ignored while the first confirm is in flight (button in loading state); only one write is attempted.
- **The undo grace window elapses while the prompt is open in the Undo state** -- The prompt's remaining-time line reaches zero and the prompt switches in place to the Grace window elapsed state without requiring Talia to close and reopen it.
- **Talia reopens the prompt for a booking that was marked no-show, then undone, from a different device in the meantime** -- Concurrent-edit conflict: the Booking and Deposit Transaction are read fresh on every prompt open, so the prompt reflects the current true state (Mark, not Undo) rather than a stale cached one; no separate conflict dialog is needed since this screen never holds an editable draft, only a fresh read-then-confirm action. Resolution consistent with the dependency map's reject-with-refresh Contention rule for Booking and Deposit Transaction: if Talia's confirm targets a state that has already changed (e.g., she confirms "Mark no-show" on a booking that was just cancelled by an expiring hold or another action), the write is rejected and she sees "This booking's state has changed. " followed by the exact current state, with no partial write applied.
- **Talia taps "Mark no-show" then immediately closes the prompt before the confirm completes** -- The confirm already in flight completes regardless of the prompt being closed; the next time Talia opens this booking's prompt (or views the dashboard), it reflects the outcome of that completed write.
- **Network failure during confirm** -- Error state: "Could not complete this action. Check your connection and try again." with the booking's true state re-read and re-displayed; no partial Booking or Deposit Transaction change is left behind (FEAT-11.SPEC-002 / FEAT-11.SPEC-003 own atomicity).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-12 (Pro Daily Schedule Dashboard) | Navigation (inbound) | Talia arrives here by tapping "no-show" on a past-due booking row |
| FEAT-11.SPEC-004 (No-Show Marking Window & Authorization Rules) | References (inbound) | Governs which prompt state shows, the window checks re-run on open and confirm, and the exact denied messages |
| FEAT-11.SPEC-002 (No-Show Marking & Deposit Forfeiture) | Triggers (outbound) | "Mark no-show" confirm triggers this automation |
| FEAT-11.SPEC-003 (No-Show Mark Undo) | Triggers (outbound) | "Undo no-show" confirm triggers this automation |
| FEAT-30 (Pro Booking Management) | Navigation (outbound) | "Refund as goodwill instead" hands off to FEAT-30's goodwill refund flow |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| no_show_prompt_opened | prompt_state (mark / undo / elapsed) | Prompt opens from the dashboard | supports success-metrics.md: "No-Show Recovery Rate" |
| no_show_marked | time_since_start_time | Talia confirms "Mark no-show" and the write succeeds | supports success-metrics.md: "No-Show Recovery Rate" |
| no_show_mark_undone | time_since_marked | Talia confirms "Undo no-show" and the write succeeds | supports success-metrics.md: "No-Show Recovery Rate" |
| no_show_goodwill_redirect | -- | Talia taps "Refund as goodwill instead" | N/A -- goodwill refund correctness is measured by FEAT-30's own success-metrics.md connection ("Pro Change Correctness"), not by this feature's No-Show Recovery Rate, which measures the no-show path specifically |
| no_show_action_failed | action (mark / undo), reason (window_closed / write_failed / offline) | The confirm is rejected by re-validation or the triggered automation reports an unrecoverable failure | supports success-metrics.md: "No-Show Recovery Rate" (a rate target of zero manual chasing requires visibility into every case the automatic path could not complete cleanly) |

## Acceptance Criteria

**FEAT-11.SPEC-001-AC-01:** Given Talia is viewing her dashboard (FEAT-12) and a booking's appointment start time has passed with no outcome marked yet, when she taps "no-show" on that row, then this prompt opens in the Mark state showing the client's name, appointment time, and the kept-deposit outcome preview.

**FEAT-11.SPEC-001-AC-02:** Given Talia is on the Mark state of this prompt, when she taps "Mark no-show," then FEAT-11.SPEC-004 re-validates eligibility, FEAT-11.SPEC-002 runs, and on success the prompt switches in place to the Undo state showing the deposit as kept.

**FEAT-11.SPEC-001-AC-03:** Given Talia is on the Mark state of this prompt, when she taps "Refund as goodwill instead," then the prompt closes and she is taken to Pro Booking Management's (FEAT-30) goodwill refund flow scoped to this booking.

**FEAT-11.SPEC-001-AC-04:** Given Talia is on the Mark state of this prompt, when she taps "Cancel," then the prompt closes with no change and she returns to the dashboard.

**FEAT-11.SPEC-001-AC-05:** Given Talia marked a booking as a no-show 3 hours ago, when she taps "no-show" on that row again, then this prompt opens in the Undo state showing the kept outcome and roughly 21 hours remaining in the undo grace window (platform parameter: `no-show-undo-grace-window-hours`).

**FEAT-11.SPEC-001-AC-06:** Given Talia is on the Undo state of this prompt, when she taps "Undo no-show," then FEAT-11.SPEC-004 re-validates the grace window, FEAT-11.SPEC-003 runs, and on success the prompt switches in place to the Mark state, reflecting the booking as Completed again with the deposit restored.

**FEAT-11.SPEC-001-AC-07:** Given Talia is on the Undo state of this prompt, when she taps "Keep as no-show," then the prompt closes with no change.

**FEAT-11.SPEC-001-AC-08:** Given Talia marked a booking as a no-show more than 24 hours ago (platform parameter: `no-show-undo-grace-window-hours` has elapsed), when she taps "no-show" on that row, then this prompt opens in the Grace window elapsed state showing "This no-show mark can no longer be undone." with only a Close action.

**FEAT-11.SPEC-001-AC-09:** Given Talia is on the Undo state with 10 minutes remaining in the grace window, when the window elapses while the prompt stays open, then the prompt switches in place to the Grace window elapsed state without requiring her to reopen it.

**FEAT-11.SPEC-001-AC-10:** Given Talia taps "Mark no-show" and the booking's state changed since the prompt opened (e.g., it was cancelled through another action in the meantime), when the re-validation in FEAT-11.SPEC-004 runs, then no write occurs and Talia sees the exact denied message stating the booking's current state.

**FEAT-11.SPEC-001-AC-11:** Given Talia loses connectivity while this prompt is open, when she looks at the action row, then it is replaced by the banner "You're offline. Marking or undoing a no-show needs a connection." and no action can be confirmed until connectivity returns.

**FEAT-11.SPEC-001-AC-12:** Given Talia taps "Mark no-show" and the write cannot complete after the automation's retry, when the failure is reported back to this prompt, then she sees an error state describing the failure rather than a silent success, and the booking's true state is re-read and shown.

**FEAT-11.SPEC-001-AC-13:** Given Talia taps "Mark no-show" twice in rapid succession, when the second tap registers, then it is ignored while the first confirm is in flight, and only one write is attempted.

**FEAT-11.SPEC-001-AC-14:** Given Riley (the Client) has no path that reaches this prompt, when she views her own booking history after a no-show mark, then she sees only the resulting deposit-kept outcome there (FEAT-16), never this prompt or its actions.

**FEAT-11.SPEC-001-AC-15:** Given Talia taps "no-show" on a past-due booking row, when the prompt opens and the fresh Booking and Deposit Transaction read has not yet resolved, then the prompt shows the Loading state (header and close control visible, skeleton in place of the booking identity, outcome line, and action row) rather than an empty or blank surface, and it renders directly into the Mark, Undo, or Grace window elapsed state once the read resolves.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 7 | 7 |
| States | 8 (loading, empty, mark, undo, elapsed, confirming, error, offline) | 8 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Automation Spec: No-Show Marking & Deposit Forfeiture

## Overview

**Name:** No-Show Marking & Deposit Forfeiture
**ID:** FEAT-11.SPEC-002
**Type:** Automation
**Purpose:** On Talia's confirmed mark, atomically transitions the Booking to No-Show and its Deposit Transaction to Forfeited in one step, deriving the forfeiture outcome from the booking's acknowledged cancellation policy version, with no separate invoicing step and no manual chasing.
**Parent Feature:** FEAT-11 -- No-Show Marking & Deposit Forfeiture

## Scope and Non-Goals

**In Scope:**
- Re-validating marking eligibility immediately before writing (window and ownership, per FEAT-11.SPEC-004)
- Atomically transitioning the Booking (Awaiting Outcome -> No-Show) and its Deposit Transaction (Captured -> Forfeited) as one committed step
- Deriving the forfeiture outcome from the Cancellation Policy version the booking's client acknowledged
- Retrying a failed write and flagging it to Talia rather than silently dropping it, since money is at stake
- Feeding the resulting event to the activity record (FEAT-16.SPEC-002) and revenue insights (FEAT-25.SPEC-004)
- Notifying the Booking-to-Calendar Sync automation (FEAT-04.SPEC-005) that the booking is now marked no-show, so it can record its explicit no-action decision for the Pro's connected calendar

**Non-Goals:**
- Deciding the marking window or ownership eligibility itself -- owned by FEAT-11.SPEC-004 (Logic/Rule); this automation only re-checks the outcome that rule spec defines immediately before writing
- Reversing a no-show mark -- owned by FEAT-11.SPEC-003 (No-Show Mark Undo), a distinct automation with its own trigger, window, and outcome set
- Charging or re-charging the client's card -- excluded per SC-13: no card is kept on file for later charges, and the deposit was already captured at booking by FEAT-07, so this automation moves no money, it only changes the deposit's disposition
- Writing the append-only activity record entry itself -- owned by FEAT-16 (XBR-21); this automation only emits the event FEAT-16 consumes
- Changing anything on the Pro's personal calendar -- owned by FEAT-04.SPEC-005 (Booking-to-Calendar Sync), which receives the no-show event and decides no calendar action is needed because the appointment already occurred; this automation only emits the event
- Notifying the Client -- excluded per the feature's Communications field, which keeps the outcome visible only in the Client's own booking history rather than triggering a separate notice

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Talia confirms "Mark no-show" | FEAT-11.SPEC-001 (No-Show Mark & Undo Prompt) | The prompt's re-validation via FEAT-11.SPEC-004 has already passed at the moment the tap registers | Booking identity, start_time, state, policy_version, and the Deposit Transaction identity and current status |

## Processing Logic

1. Receive the confirmed mark request from FEAT-11.SPEC-001, carrying the Booking's identity.
2. Re-check the marking window and Pro-ownership eligibility per FEAT-11.SPEC-004: the Booking's start_time has passed, the current time is before the auto-completion boundary (platform parameter: `booking-auto-completion-window-days`, owned by FEAT-12), and the requesting Pro owns this booking.
3. If eligibility fails at this re-check (the window closed or the state changed between prompt open and confirm), stop and return the exact denied reason to FEAT-11.SPEC-001 without writing anything.
4. Read the Deposit Transaction linked to this Booking and confirm its current status is Captured. If it is any other status (already Forfeited, Refunded, Disputed, or in a refund cycle), stop and report the current status back to FEAT-11.SPEC-001 -- a deposit is refunded or forfeited only once (dependency map's Contention rule for Deposit Transaction).
5. Read the Booking's acknowledged policy_version to confirm the no-show outcome it defines is "deposit kept" (XBR-09's binary rule: inside the window or a no-show always keeps the deposit).
6. Atomically transition the Booking's state from Awaiting Outcome to No-Show and the Deposit Transaction's status from Captured to Forfeited, recording the outcome_reason as this no-show mark and the outcome timestamp as the current time. Both writes commit together or neither does -- there is no state where one succeeds and the other does not.
7. On successful commit, emit the no-show marked and deposit forfeited events for the activity record (FEAT-16.SPEC-002), revenue insights (FEAT-25.SPEC-004), and the Booking-to-Calendar Sync automation (FEAT-04.SPEC-005, which confirms no calendar action is needed for a no-show and takes none), and return the success outcome to FEAT-11.SPEC-001.
8. If the atomic write cannot complete (connectivity or processing error), retry automatically; if it still cannot complete, flag the failure back to FEAT-11.SPEC-001 for Talia to see, leaving the Booking and Deposit Transaction in their prior, consistent state (Awaiting Outcome / Captured) -- never a partial transition.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Marked successfully | Eligibility re-check passes, Deposit Transaction was Captured, atomic write commits | Booking: Awaiting Outcome -> No-Show. Deposit Transaction: Captured -> Forfeited, outcome_reason and timestamp set | Prompt switches in place to the Undo state showing the deposit as kept | FEAT-11.SPEC-001 (result), FEAT-16.SPEC-002 (activity record), FEAT-25.SPEC-004 (revenue aggregates), FEAT-04.SPEC-005 (calendar no-action confirmation) |
| Marking window closed since prompt opened | Re-check finds the current time now past the auto-completion boundary, or the booking's state changed | None | Prompt shows the exact denied message from FEAT-11.SPEC-004 ("This booking has already auto-completed and can no longer be marked as a no-show." or the current-state message) and offers only Close | FEAT-11.SPEC-001 |
| Deposit Transaction not in a forfeitable state | Deposit Transaction status is not Captured at the moment of the check (already Forfeited, Refunded, Disputed, or Refund in Progress) | None | Prompt shows the current deposit status and does not attempt the mark | FEAT-11.SPEC-001 |
| Write failure, resolved on retry | Atomic write fails once, succeeds on automatic retry | Same as "Marked successfully," delayed by the retry interval | Prompt's loading state extends briefly through the retry, then resolves to the success view | FEAT-11.SPEC-001, FEAT-16.SPEC-002, FEAT-25.SPEC-004, FEAT-04.SPEC-005 |
| Write failure, unresolved | Atomic write fails and remains unresolved after automatic retry | None -- Booking and Deposit Transaction remain in their prior state | Prompt shows an error state flagging the failure to Talia, with the booking's true (unmarked) state re-read and displayed | FEAT-11.SPEC-001 |

## Data Model

**Reads:** Booking -- start_time, state, policy_version, owning Pro Account. Deposit Transaction -- status. Cancellation Policy -- the version referenced by the Booking's policy_version, specifically its inside_window_outcome / no-show outcome (binary: deposit kept).
**Creates:** None.
**Updates:** Booking -- state (Awaiting Outcome -> No-Show). Deposit Transaction -- status (Captured -> Forfeited), outcome_reason, outcome timestamp.
**Deletes:** None -- financial and booking records are retained for the life of the account (SC-22); this automation never removes a record.

## Business Rules

- XBR-08: the forfeiture outcome is derived from the policy version the booking's client acknowledged at booking time -- a later edit to the Pro's cancellation policy never changes an existing booking's outcome.
- XBR-09: deposit outcomes are binary -- a no-show always keeps the full deposit already captured, never a percentage or schedule (SC-18).
- XBR-12: a booking can be marked no-show only after its start_time has passed and before its 7-day auto-completion boundary (platform parameter: `booking-auto-completion-window-days`, owned by FEAT-12); this automation re-enforces that window at write time even though FEAT-11.SPEC-001 already checked it, since time may have advanced between prompt open and confirm.
- The Booking and Deposit Transaction transitions are always written together, atomically -- this automation never leaves the Booking marked No-Show with the Deposit Transaction still Captured, or vice versa.
- A deposit can be forfeited only once overall (dependency map's Contention rule for Deposit Transaction) -- this automation refuses to act on a Deposit Transaction that is not currently Captured.
- This automation moves no money: the deposit was already captured by FEAT-07 through payment processing at booking time (ASMP-31); marking a no-show only changes the deposit's recorded disposition.

## Edge Cases

- **The booking's start_time has not yet passed when the confirm arrives** -- Cannot occur through the normal path: FEAT-11.SPEC-001 only offers this prompt from a past-due booking row. If reached anyway (e.g., a stale client-side state), the re-check in step 2 rejects it with the standard "not yet eligible" denial from FEAT-11.SPEC-004.
- **The Cancellation Policy version referenced by the booking has since been superseded by a newer edit** -- The booking's own acknowledged policy_version is read, never the Pro's current policy (XBR-08); a newer version never applies retroactively to this booking.
- **A card-issuer dispute notice arrives for this Deposit Transaction between prompt open and confirm** -- The Deposit Transaction's Disputed overlay does not itself change its underlying status from Captured (per the dependency map: "a Disputed overlay never erases the underlying outcome"), so the forfeiture proceeds normally if still otherwise eligible; the Disputed overlay and the Forfeited outcome coexist and both remain visible to Support and Talia through FEAT-16.
- **Concurrent trigger firing (Talia taps "Mark no-show" from two devices signed into the same account at effectively the same time)** -- The first commit to reach the atomic write wins; the second re-check (step 4) finds the Deposit Transaction already Forfeited and reports the "not in a forfeitable state" outcome rather than attempting a second forfeiture, per the dependency map's reject-with-refresh Contention resolution.
- **Trigger fires while a previous run is in flight for the same booking** -- The prompt's Confirming state (FEAT-11.SPEC-001) disables the action button while the first run is in progress, so a second run for the same booking cannot start from the same session; a run from a different session for the same booking is handled by the concurrent-trigger-firing case above.
- **The write partially applies before a processing error interrupts it** -- The atomic commit guarantees both the Booking and Deposit Transaction transitions succeed together or neither is retained; a retry after an interrupted attempt re-runs the full atomic write from the last confirmed state, never resuming from a half-applied one.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-11.SPEC-001 (No-Show Mark & Undo Prompt) | Triggered by (inbound) | Talia's "Mark no-show" confirm fires this automation |
| FEAT-11.SPEC-004 (No-Show Marking Window & Authorization Rules) | References (inbound) | Supplies the marking-window and ownership eligibility this automation re-checks before writing |
| FEAT-11.SPEC-003 (No-Show Mark Undo) | References (outbound) | The counterpart automation that can later reverse this transition within the grace window |
| FEAT-09 (Cancellation & No-Show Policy Engine) | References (inbound) | Supplies the acknowledged policy version's no-show outcome this automation reads |
| FEAT-07 (Deposit Payment at Booking) | References (inbound) | Created and captured the Deposit Transaction this automation forfeits |
| FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) | Affects (outbound) | Consumes the no-show marked and deposit forfeited events for the append-only record |
| FEAT-25.SPEC-004 (Historical Aggregate Maintenance) -- within FEAT-25 (Booking & Revenue Insights) | Affects (outbound) | Consumes the forfeited deposit for the no-show rate and revenue aggregates |
| FEAT-04.SPEC-005 (Booking-to-Calendar Sync) -- within FEAT-04 (Two-Way Calendar Sync) | Affects (outbound) | Receives the booking-marked-no-show event and confirms no calendar action is needed (the appointment already occurred); this automation takes no calendar action itself |

## Analytics and Success Signals

- **no_show_marked** (time_since_start_time, retry_occurred: true/false) -- supports success-metrics.md: "No-Show Recovery Rate"
- **no_show_forfeiture_write_failed** (reason: connectivity / processing_error) -- supports success-metrics.md: "No-Show Recovery Rate" (the target is zero instances requiring manual chasing, so every unresolved write failure is exactly the gap this metric must surface)
- **no_show_marking_denied** (reason: window_closed / not_owner / deposit_not_forfeitable) -- supports success-metrics.md: "No-Show Recovery Rate" (denials on re-check are the cases the automatic path could not complete cleanly on the first attempt)

## Acceptance Criteria

**FEAT-11.SPEC-002-AC-01:** Given Talia confirms "Mark no-show" on a booking that is past its start_time, before its auto-completion boundary, and owned by her, when the automation runs, then the Booking transitions to No-Show and the Deposit Transaction transitions to Forfeited in one atomic write.

**FEAT-11.SPEC-002-AC-02:** Given the atomic write in FEAT-11.SPEC-002-AC-01 commits successfully, when it completes, then the prompt (FEAT-11.SPEC-001) switches in place to the Undo state showing the deposit as kept, with no separate invoicing step.

**FEAT-11.SPEC-002-AC-03:** Given Talia confirms "Mark no-show" but the booking's auto-completion boundary (platform parameter: `booking-auto-completion-window-days`) has passed since the prompt opened, when the automation re-checks eligibility, then no write occurs and the exact denied message from FEAT-11.SPEC-004 is returned.

**FEAT-11.SPEC-002-AC-04:** Given Talia confirms "Mark no-show" on a booking whose Deposit Transaction is already Forfeited (e.g., marked from another session moments earlier), when the automation reads the Deposit Transaction's status, then no second forfeiture is attempted and the current status is reported back.

**FEAT-11.SPEC-002-AC-05:** Given the atomic write fails once due to a connectivity error, when the automation retries automatically, then the Booking and Deposit Transaction transition successfully on retry and Talia sees the success outcome after a brief extended loading state.

**FEAT-11.SPEC-002-AC-06:** Given the atomic write fails and remains unresolved after automatic retry, when the failure is reported, then the Booking and Deposit Transaction remain in their prior state (Awaiting Outcome / Captured) and Talia sees an error state flagging the failure rather than a false success.

**FEAT-11.SPEC-002-AC-07:** Given a booking's acknowledged policy_version defines the no-show outcome as deposit kept, when this automation derives the forfeiture outcome, then it applies that acknowledged version's outcome even if the Pro's current cancellation policy has since been edited to a newer version.

**FEAT-11.SPEC-002-AC-08:** Given this Deposit Transaction has an open card-issuer dispute (Disputed overlay) at the moment Talia confirms the mark, when the automation runs and the deposit is otherwise still Captured, then the forfeiture proceeds and the Disputed overlay remains visible alongside the new Forfeited outcome.

**FEAT-11.SPEC-002-AC-09:** Given Talia confirms "Mark no-show" from two devices for the same booking at effectively the same time, when both requests reach the automation, then only the first commit succeeds and the second is rejected with the current (already Forfeited) status.

**FEAT-11.SPEC-002-AC-10:** Given this automation's write is in flight for a booking, when a second "Mark no-show" confirm for the same booking arrives from the same session before the first completes, then the prompt's disabled Confirming state (FEAT-11.SPEC-001) prevents a second run from starting.

**FEAT-11.SPEC-002-AC-11:** Given this automation successfully marks a booking as a no-show, when the transition commits, then the no_show_marked event is emitted and the activity record (FEAT-16.SPEC-002), revenue insights (FEAT-25.SPEC-004), and the Booking-to-Calendar Sync automation (FEAT-04.SPEC-005, which takes no calendar action for a no-show) receive the resulting no-show and forfeiture data.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 5 (marked, window closed, deposit not forfeitable, retry-resolved failure, unresolved failure) | 5 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |



# Automation Spec: No-Show Mark Undo

## Overview

**Name:** No-Show Mark Undo
**ID:** FEAT-11.SPEC-003
**Type:** Automation
**Purpose:** Within the 24-hour grace window (platform parameter: `no-show-undo-grace-window-hours`), atomically reverses a no-show mark -- restoring the Booking to Completed and the Deposit Transaction to its prior Captured status -- when Talia confirms the mistake was hers.
**Parent Feature:** FEAT-11 -- No-Show Marking & Deposit Forfeiture

## Scope and Non-Goals

**In Scope:**
- Re-validating the undo grace window and ownership immediately before writing (per FEAT-11.SPEC-004)
- Atomically transitioning the Booking (No-Show -> Completed) and its Deposit Transaction (Forfeited -> Captured) as one committed step
- Retrying a failed undo write and flagging it to Talia rather than silently dropping it, since money is at stake
- Feeding the resulting reversal event to the activity record (FEAT-16.SPEC-002) and revenue insights (FEAT-25.SPEC-004), which reverses only the saved-from-no-shows aggregate contribution

**Non-Goals:**
- Deciding the undo grace window or ownership eligibility itself -- owned by FEAT-11.SPEC-004 (Logic/Rule); this automation only re-checks the outcome that rule spec defines immediately before writing
- Marking a booking as a no-show in the first place -- owned by FEAT-11.SPEC-002 (No-Show Marking & Deposit Forfeiture), the forward automation this one reverses
- Issuing a goodwill refund -- excluded per this feature's Key Capabilities; a goodwill refund is a distinct, separate action available only through Pro Booking Management (FEAT-30) and is never a side effect of this undo
- Undoing an undo (re-marking after reversal is a fresh "Mark no-show" action) -- excluded because once reversed, the Booking returns to Completed and the standard marking window and eligibility in FEAT-11.SPEC-004 govern any subsequent mark attempt from first principles, not a special "re-undo" path
- Extending or restarting the grace window on a failed or retried undo attempt -- excluded per FEAT-11.SPEC-004: the window is measured from the original Forfeited outcome timestamp and is never reset by an intervening attempt

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Talia confirms "Undo no-show" | FEAT-11.SPEC-001 (No-Show Mark & Undo Prompt) | The prompt's re-validation via FEAT-11.SPEC-004 has already passed at the moment the tap registers, and the Booking is currently No-Show | Booking identity, state, and the Deposit Transaction identity, current status, and Forfeited outcome timestamp |

## Processing Logic

1. Receive the confirmed undo request from FEAT-11.SPEC-001, carrying the Booking's identity.
2. Re-check ownership and the undo grace window per FEAT-11.SPEC-004: the requesting Pro owns this booking, and the current time is within the fixed grace window (platform parameter: `no-show-undo-grace-window-hours`) measured from the Deposit Transaction's Forfeited outcome timestamp.
3. If eligibility fails at this re-check (the window elapsed between prompt open and confirm), stop and return the exact denied reason ("This no-show mark can no longer be undone.") to FEAT-11.SPEC-001 without writing anything.
4. Read the Booking's current state and confirm it is still No-Show, and read the Deposit Transaction's current status and confirm it is still Forfeited. If either has already changed (e.g., a goodwill refund was issued through FEAT-30 in the meantime), stop and report the current state back to FEAT-11.SPEC-001 -- this automation only reverses its own counterpart transition, never any other outcome.
5. Atomically transition the Booking's state from No-Show back to Completed and the Deposit Transaction's status from Forfeited back to Captured, clearing the no-show outcome_reason and timestamp back to the state they held immediately before the original mark. Both writes commit together or neither does.
6. On successful commit, emit the no-show mark undone event for the activity record (FEAT-16.SPEC-002) and revenue insights (FEAT-25.SPEC-004), and return the success outcome to FEAT-11.SPEC-001.
7. If the atomic write cannot complete (connectivity or processing error), retry automatically; if it still cannot complete, flag the failure back to FEAT-11.SPEC-001 for Talia to see, leaving the Booking and Deposit Transaction in their prior, consistent state (No-Show / Forfeited) -- never a partial reversal.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Undo successful | Re-check passes (window open, Pro owns booking), Booking still No-Show, Deposit Transaction still Forfeited, atomic write commits | Booking: No-Show -> Completed. Deposit Transaction: Forfeited -> Captured, outcome_reason/timestamp cleared | Prompt switches in place to the Mark state, reflecting the booking as Completed and the deposit as restored | FEAT-11.SPEC-001 (result), FEAT-16.SPEC-002 (activity record), FEAT-25.SPEC-004 (aggregate reversal) |
| Grace window elapsed since prompt opened | Re-check finds the current time now past the grace-window boundary | None | Prompt shows "This no-show mark can no longer be undone." and offers only Close | FEAT-11.SPEC-001 |
| Booking or Deposit Transaction state already changed | Booking is no longer No-Show, or Deposit Transaction is no longer Forfeited (e.g., a goodwill refund already issued) | None | Prompt shows the current state and does not attempt the undo | FEAT-11.SPEC-001 |
| Write failure, resolved on retry | Atomic write fails once, succeeds on automatic retry | Same as "Undo successful," delayed by the retry interval | Prompt's loading state extends briefly through the retry, then resolves to the success view | FEAT-11.SPEC-001, FEAT-16.SPEC-002, FEAT-25.SPEC-004 |
| Write failure, unresolved | Atomic write fails and remains unresolved after automatic retry | None -- Booking and Deposit Transaction remain in their prior state | Prompt shows an error state flagging the failure to Talia, with the booking's true (still-marked) state re-read and displayed | FEAT-11.SPEC-001 |

## Data Model

**Reads:** Booking -- state, owning Pro Account. Deposit Transaction -- status, outcome_reason/timestamps.
**Creates:** None.
**Updates:** Booking -- state (No-Show -> Completed). Deposit Transaction -- status (Forfeited -> Captured), outcome_reason and timestamp cleared back to their pre-mark values.
**Deletes:** None -- financial and booking records are retained for the life of the account (SC-22); this automation never removes a record, it only reverses a transition.

## Business Rules

- XBR-12: an undo is available only within a fixed 24-hour window (platform parameter: `no-show-undo-grace-window-hours`) measured from the original Forfeited outcome timestamp, never extended by a failed or retried attempt.
- The Booking and Deposit Transaction transitions are always reversed together, atomically -- this automation never leaves the Booking Completed with the Deposit Transaction still Forfeited, or vice versa.
- This automation reverses only its own counterpart forward transition (FEAT-11.SPEC-002's mark); it never acts on a Booking or Deposit Transaction whose state changed through a different path (cancellation, goodwill refund, dispute) in the meantime.
- This automation moves no money in either direction: the deposit remains the same captured funds throughout, and reversing the mark only restores its recorded disposition -- no re-authorization or new charge occurs.
- Undoing does not reopen the original marking window: once reversed, the Booking is Completed, and any later no-show determination for this same booking is a fresh FEAT-11.SPEC-002 mark attempt, governed by FEAT-11.SPEC-004 from first principles (not a special re-undo case).

## Edge Cases

- **A goodwill refund is issued through FEAT-30 between the mark and the undo attempt** -- The Deposit Transaction is no longer Forfeited (it has moved to a refund outcome), so step 4's re-check finds the state already changed and reports it; this automation never overwrites a refund outcome back to Captured.
- **The undo grace window elapses by seconds while the confirm request is in flight** -- The re-check in step 2 uses the time at the moment the automation evaluates it, not the time the prompt was opened; a confirm that arrives after the boundary is denied even if the prompt showed time remaining moments earlier.
- **Concurrent trigger firing (Talia taps "Undo no-show" from two devices at effectively the same time)** -- The first commit to reach the atomic write wins; the second re-check (step 4) finds the Booking already Completed and the Deposit Transaction already Captured, and reports the current state rather than attempting a second reversal, per the dependency map's reject-with-refresh Contention resolution.
- **Trigger fires while a previous run is in flight for the same booking** -- The prompt's Confirming state (FEAT-11.SPEC-001) disables the action button while the first run is in progress, so a second run for the same booking cannot start from the same session; a run from a different session is handled by the concurrent-trigger-firing case above.
- **The write partially applies before a processing error interrupts it** -- The atomic commit guarantees both the Booking and Deposit Transaction reversal succeed together or neither is retained; a retry after an interrupted attempt re-runs the full atomic reversal from the last confirmed (No-Show / Forfeited) state, never resuming from a half-applied one.
- **Talia undoes, then immediately wants to re-mark the same booking as a no-show** -- Because undo does not reopen the original window (per Business Rules above), this is simply a fresh FEAT-11.SPEC-002 attempt; it succeeds only if the booking is still within the marking window bounds FEAT-11.SPEC-004 defines at that later moment.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-11.SPEC-001 (No-Show Mark & Undo Prompt) | Triggered by (inbound) | Talia's "Undo no-show" confirm fires this automation |
| FEAT-11.SPEC-004 (No-Show Marking Window & Authorization Rules) | References (inbound) | Supplies the undo grace-window and ownership eligibility this automation re-checks before writing |
| FEAT-11.SPEC-002 (No-Show Marking & Deposit Forfeiture) | References (inbound) | The forward automation whose transition this one reverses |
| FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) | Affects (outbound) | Consumes the no-show mark undone event for the append-only record |
| FEAT-25.SPEC-004 (Historical Aggregate Maintenance) -- within FEAT-25 (Booking & Revenue Insights) | Affects (outbound) | Consumes the no-show mark undone event to reverse only the saved-from-no-shows aggregate contribution; the booking count is untouched |

## Analytics and Success Signals

- **no_show_mark_undone** (time_since_marked, retry_occurred: true/false) -- supports success-metrics.md: "No-Show Recovery Rate" (a clean, complete reversal keeps the automatic outcome trustworthy when Talia catches her own mistake, rather than requiring a manual correction)
- **no_show_undo_write_failed** (reason: connectivity / processing_error) -- supports success-metrics.md: "No-Show Recovery Rate"
- **no_show_undo_denied** (reason: window_elapsed / not_owner / state_already_changed) -- supports success-metrics.md: "No-Show Recovery Rate"

## Acceptance Criteria

**FEAT-11.SPEC-003-AC-01:** Given Talia confirms "Undo no-show" on a booking marked no-show 3 hours ago, within the 24-hour grace window and owned by her, when the automation runs, then the Booking transitions back to Completed and the Deposit Transaction transitions back to Captured in one atomic write.

**FEAT-11.SPEC-003-AC-02:** Given the atomic write in FEAT-11.SPEC-003-AC-01 commits successfully, when it completes, then the prompt (FEAT-11.SPEC-001) switches in place to the Mark state, reflecting the booking as Completed with the deposit restored.

**FEAT-11.SPEC-003-AC-03:** Given Talia confirms "Undo no-show" but the 24-hour grace window (platform parameter: `no-show-undo-grace-window-hours`) has elapsed since the prompt opened, when the automation re-checks eligibility, then no write occurs and "This no-show mark can no longer be undone." is returned.

**FEAT-11.SPEC-003-AC-04:** Given a goodwill refund has already been issued for this booking's Deposit Transaction through FEAT-30 since it was marked no-show, when Talia confirms "Undo no-show," then the automation finds the Deposit Transaction is no longer Forfeited, makes no write, and reports the current state.

**FEAT-11.SPEC-003-AC-05:** Given the atomic reversal write fails once due to a connectivity error, when the automation retries automatically, then the Booking and Deposit Transaction transition back successfully on retry and Talia sees the success outcome after a brief extended loading state.

**FEAT-11.SPEC-003-AC-06:** Given the atomic reversal write fails and remains unresolved after automatic retry, when the failure is reported, then the Booking and Deposit Transaction remain in their prior state (No-Show / Forfeited) and Talia sees an error state flagging the failure rather than a false success.

**FEAT-11.SPEC-003-AC-07:** Given the undo grace window has exactly 5 seconds remaining when Talia's confirm request reaches the automation, when the automation evaluates the current time against the boundary, then the outcome is governed by the time of evaluation, not the time the prompt displayed.

**FEAT-11.SPEC-003-AC-08:** Given Talia confirms "Undo no-show" from two devices for the same booking at effectively the same time, when both requests reach the automation, then only the first commit succeeds and the second is rejected with the current (already Completed / Captured) state.

**FEAT-11.SPEC-003-AC-09:** Given this automation's write is in flight for a booking, when a second "Undo no-show" confirm for the same booking arrives from the same session before the first completes, then the prompt's disabled Confirming state (FEAT-11.SPEC-001) prevents a second run from starting.

**FEAT-11.SPEC-003-AC-10:** Given this automation successfully undoes a no-show mark, when the reversal commits, then the no_show_mark_undone event is emitted the activity record (FEAT-16.SPEC-002) receives the resulting reversal data, and revenue insights (FEAT-25.SPEC-004) reverses only the saved-from-no-shows contribution.

**FEAT-11.SPEC-003-AC-11:** Given Talia successfully undoes a no-show mark and the booking returns to Completed, when she later reconsiders and taps "no-show" on the same booking row again, then FEAT-11.SPEC-002 evaluates the attempt as a fresh mark against FEAT-11.SPEC-004's window rules, not as a special re-undo case.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 5 (undo successful, window elapsed, state already changed, retry-resolved failure, unresolved failure) | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: No-Show Marking Window & Authorization Rules

## Overview

**Name:** No-Show Marking Window & Authorization Rules
**ID:** FEAT-11.SPEC-004
**Type:** Logic/Rule
**Purpose:** Governs who may mark or undo a no-show (the Pro, on their own bookings only), the eligible marking window (after the appointment start time, before the booking auto-completes), and the 24-hour undo grace window (platform parameter: `no-show-undo-grace-window-hours`), shared by the prompt and both automations rather than duplicated in each.
**Parent Feature:** FEAT-11 -- No-Show Marking & Deposit Forfeiture
**Governed Entity:** Booking (the no-show marking and undo transition, and the Deposit Transaction status it depends on)

## Scope and Non-Goals

**In Scope:**
- The marking-window bounds (lower: start_time has passed; upper: before the booking's auto-completion boundary)
- The fixed 24-hour undo grace window, measured from the Deposit Transaction's Forfeited outcome timestamp
- Authorization for the "mark no-show" and "undo no-show" actions, by role
- Ownership: a Pro may only mark or undo bookings they own
- The cross-entity condition that a mark or undo is only valid against a Deposit Transaction in the matching status (Captured to mark, Forfeited to undo)

**Non-Goals:**
- Writing the Booking or Deposit Transaction transitions themselves -- owned by FEAT-11.SPEC-002 (mark) and FEAT-11.SPEC-003 (undo); this spec only defines whether the attempt is eligible
- The prompt's layout and interaction handling -- owned by FEAT-11.SPEC-001 (Screen); this spec is referenced by it, not embedded in it
- Governing any other Booking state transition (Pending Payment, Confirmed, Cancelled, Rescheduled, Expired (unpaid), or the auto-completion itself) -- excluded per the Brief's Entity-Lifecycle Coverage Matrix: those transitions and their windows are owned by FEAT-05, FEAT-07, FEAT-10, FEAT-12, and FEAT-30
- Deciding the forfeiture outcome's substance (deposit kept vs. refunded) -- owned by the Cancellation Policy Engine (FEAT-09) and read as a binary, acknowledged fact by FEAT-11.SPEC-002; this spec governs only the window and authorization for the action, not the financial outcome it produces

## Governed Entity

**Entity:** Booking (narrow slice relevant to the no-show marking and undo transition), with the Deposit Transaction's status read as a cross-entity precondition.
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| service | text | Service booked -- fixed at booking, not evaluated by this spec |
| start_time | date/time | Appointment start time, in the Pro's timezone -- the lower bound of the marking window |
| duration | number | Appointment duration -- not evaluated by this spec |
| client | reference | The booked Client -- not evaluated by this spec beyond confirming a booking exists |
| price_agreed / deposit_amount | number | Fixed at booking -- not evaluated by this spec (the amount itself is FEAT-07's and FEAT-09's concern) |
| policy_version | reference | The acknowledged Cancellation Policy version -- read by FEAT-11.SPEC-002 for the forfeiture outcome, not evaluated as an eligibility condition here |
| state | enum | Pending Payment \| Confirmed \| Awaiting Outcome \| Completed \| No-Show \| Cancelled by Client \| Cancelled by Pro \| Rescheduled \| Expired (unpaid) -- the marking window is only open while state is Awaiting Outcome, and the undo window only while state is No-Show |
| attendance_reply | text | Reminder reply -- not evaluated by this spec |
| balance_due | number (derived) | Not evaluated by this spec |
| source | enum | Booking origin -- not evaluated by this spec |
| cancellation / reschedule timestamps and optional private Pro reason | text/date | Not evaluated by this spec -- a cancelled or rescheduled booking cannot be in Awaiting Outcome or No-Show state, so these fields are already excluded by the state condition |
| owning Pro Account | reference | The Pro who created this booking -- the ownership condition for both actions |

**Cross-entity precondition (Deposit Transaction):**

| Field | Data Type | Description |
|-------|-----------|-------------|
| status | enum | Authorized \| Captured \| Applied \| Refunded \| Refund in Progress \| Forfeited \| Disputed -- must be Captured for a mark attempt to be eligible, and Forfeited for an undo attempt to be eligible |
| outcome_reason / timestamps | text/date | The Forfeited outcome timestamp anchors the 24-hour undo grace window |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-11.SPEC-001 | No-Show Mark & Undo Prompt | On prompt open (determines which of the three prompt states to show) and again on confirm (re-validates before triggering the automation) |
| FEAT-11.SPEC-002 | No-Show Marking & Deposit Forfeiture | Re-checks the marking window, ownership, and Deposit Transaction status immediately before the atomic write |
| FEAT-11.SPEC-003 | No-Show Mark Undo | Re-checks the undo grace window, ownership, and Deposit Transaction status immediately before the atomic write |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| Booking.start_time | Marking window lower bound: the current time must be at or after this value | Mark action only | On prompt open and again on confirm | "This booking can be marked no-show only after its appointment time has passed." | Yes |
| Booking.start_time (via auto-completion boundary) | Marking window upper bound: the current time must be before start_time + platform parameter: `booking-auto-completion-window-days` (boundary owned by FEAT-12) | Mark action only | On prompt open and again on confirm | "This booking has already auto-completed and can no longer be marked as a no-show." | Yes |
| Booking.state | Must be Awaiting Outcome | Mark action only | On prompt open and again on confirm | "This booking's state has changed and can no longer be marked as a no-show." | Yes |
| Booking.state | Must be No-Show | Undo action only | On prompt open and again on confirm | "This booking is not currently marked as a no-show." | Yes |
| Deposit Transaction.status | Must be Captured | Mark action only | On confirm (re-checked by FEAT-11.SPEC-002 immediately before write) | "This booking's deposit is not in a state that can be forfeited." | Yes |
| Deposit Transaction.status | Must be Forfeited | Undo action only | On confirm (re-checked by FEAT-11.SPEC-003 immediately before write) | "This booking's deposit is not in a state that can be restored." | Yes |
| Deposit Transaction.outcome_reason/timestamps | Undo grace window: the current time must be within the fixed span (platform parameter: `no-show-undo-grace-window-hours`) of the Forfeited outcome timestamp | Undo action only | On prompt open (whether Undo is offered at all) and again on confirm | "This no-show mark can no longer be undone." | Yes |
| owning Pro Account | Must match the requesting Pro's account | Both actions | On prompt open and again on confirm | "This booking could not be found." (the prompt does not reveal another Pro's booking exists at all -- per user-persona.md's "No role can ever see another pro's data") | Yes |
| service, duration, client, price_agreed/deposit_amount, policy_version, attendance_reply, balance_due, source, cancellation/reschedule timestamps | No validation beyond data type | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Marking window is fully bounded | Booking.start_time, auto-completion boundary (platform parameter: `booking-auto-completion-window-days`) | The marking window is open only in the span from start_time (inclusive) to the auto-completion boundary (exclusive); outside either edge, the mark action is unavailable | "This booking can be marked no-show only after its appointment time has passed." (below the window) / "This booking has already auto-completed and can no longer be marked as a no-show." (past the window) |
| Undo window never outlasts auto-completion | Deposit Transaction's Forfeited outcome_timestamp + platform parameter: `no-show-undo-grace-window-hours`, Booking.start_time + platform parameter: `booking-auto-completion-window-days` | The fixed 24-hour undo grace window is always shorter than the 7-day auto-completion boundary measured from the same appointment, so an undo attempt can never collide with the booking auto-completing out from under it; the two boundaries are evaluated independently and never need reconciling against each other | N/A -- this rule states a designed non-collision, not a validation failure |
| Action must match current cross-entity state pair | Booking.state, Deposit Transaction.status | Mark requires (Awaiting Outcome, Captured); undo requires (No-Show, Forfeited). Any other pairing observed at confirm time means the state has already moved (through cancellation, a goodwill refund, or a dispute) and the action is refused | "This booking's state has changed. {current state and deposit status shown}." |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Mark booking as no-show | The Pro | Only bookings the requesting Pro owns, and only within the marking window (Booking.state is Awaiting Outcome, Deposit Transaction.status is Captured) | If the window or state condition fails: the exact message from Field Validation Rules above. If the Pro does not own the booking: "This booking could not be found." (the booking is never revealed to exist for another Pro's account) |
| Mark booking as no-show | The Client | Never | The prompt (FEAT-11.SPEC-001) is not reachable from any Client-facing surface -- there is no control to deny; the Client's Cancellation & No-Show Handling access is Own-only visibility of the resulting outcome, never this action |
| Mark booking as no-show | Platform Operator (Support) | Never | No mark control is shown anywhere in Support's read-only surfaces (FEAT-19); Support's View access to Cancellation & No-Show Handling never includes an action control |
| Undo a no-show mark | The Pro | Only bookings the requesting Pro owns, and only within the fixed grace window (Booking.state is No-Show, Deposit Transaction.status is Forfeited, current time within platform parameter: `no-show-undo-grace-window-hours` of the Forfeited outcome timestamp) | If the window or state condition fails: the exact message from Field Validation Rules above. If the Pro does not own the booking: "This booking could not be found." |
| Undo a no-show mark | The Client | Never | Same as above -- no reachable surface exists for the Client |
| Undo a no-show mark | Platform Operator (Support) | Never | Same as above -- no action control exists on any Support surface; SC-05 excludes Support from acting on the Pro's behalf |
| View the resulting deposit-kept or restored outcome (not this action itself) | The Pro | Always, for their own bookings | -- |
| View the resulting deposit-kept or restored outcome (not this action itself) | The Client | Only in their own booking history, own bookings only (FEAT-16) | -- |
| View the resulting deposit-kept or restored outcome (not this action itself) | Platform Operator (Support) | View-only, for dispute troubleshooting, through FEAT-19 | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Marking window upper bound | Booking.start_time + platform parameter: `booking-auto-completion-window-days` | Computed on every eligibility check (prompt open and confirm) | No -- one value for every Pro (7 days per XBR-12), owned by FEAT-12 |
| Undo grace window boundary | Deposit Transaction's Forfeited outcome_timestamp + platform parameter: `no-show-undo-grace-window-hours` | Computed on every eligibility check (prompt open and confirm) | No -- one value for every Pro and every booking (24 hours per XBR-12) |
| Prompt state selection (Mark / Undo / Grace window elapsed) | Derived entirely from Booking.state, Deposit Transaction.status, and the two window boundaries above -- never a stored field or a manual toggle | On every prompt open | No |

## Business Rules

- XBR-12: a booking can be marked no-show or completed only after its start time; it auto-completes 7 days after the appointment (platform parameter: `booking-auto-completion-window-days`); a no-show mark can be undone for 24 hours (platform parameter: `no-show-undo-grace-window-hours`); a completed or no-show booking can no longer be cancelled or rescheduled.
- XBR-08: the forfeiture outcome this window and authorization govern is derived from the policy version the booking's client acknowledged at booking -- this spec does not decide the outcome itself, only whether the action that produces it is currently eligible.
- Ownership is the sole authorization gate for both actions -- there is no additional role tier among Pros; every Pro Account has identical authority over its own bookings, and none over any other Pro's.
- The Deposit Transaction status condition (Captured for mark, Forfeited for undo) exists because a deposit can be forfeited or refunded only once overall (dependency map's Contention rule) -- these rules prevent this feature's actions from firing against a deposit that has already moved to a different outcome through another path (a client cancellation refund via FEAT-09, or a goodwill refund via FEAT-30).
- The marking window's lower bound (start_time passed) and the undo window's anchor (the Forfeited outcome timestamp, not the original start_time) are deliberately distinct measurements -- the undo window is about how recently Talia acted, not about the appointment's timing.

## Edge Cases

- **Talia's confirm arrives at the exact instant start_time passes** -- The lower bound is inclusive (current time "at or after" start_time), so a confirm timestamped exactly at start_time is eligible.
- **Talia's confirm arrives at the exact instant the auto-completion boundary is reached** -- The upper bound is exclusive ("before" the boundary), so a confirm timestamped exactly at the boundary is denied with the auto-completed message.
- **The undo grace window has exactly 0 seconds remaining** -- Treated as elapsed; the boundary is exclusive in Talia's favor only up to and not including the exact expiry instant, so a confirm timestamped at or after the boundary is denied.
- **A goodwill refund is issued through FEAT-30 while the booking is still in Awaiting Outcome (before any no-show mark)** -- The Deposit Transaction's status moves away from Captured, so a subsequent mark attempt fails the "must be Captured" condition and is denied with the deposit-status message, even though the marking window itself may still be open.
- **The Pro's account ownership of a booking changes** -- Cannot occur in this product: a Booking belongs to exactly one Pro Account for its entire lifecycle (dependency map's Booking Relationships), so no ownership-transfer edge case exists for this entity.
- **Two eligibility checks (prompt open and confirm) disagree because time passed between them** -- The confirm-time check is authoritative; if the window closed between open and confirm, the confirm is denied even though the prompt initially showed the action as available (per FEAT-11.SPEC-001's re-validation-on-confirm behavior).
- **A booking's state and its Deposit Transaction's status momentarily disagree with the expected pairing (e.g., mid-write from a concurrent action)** -- The cross-field rule "Action must match current cross-entity state pair" catches this at confirm time and denies with the current-state message rather than allowing an action against an inconsistent pairing.

## Acceptance Criteria

**FEAT-11.SPEC-004-AC-01:** Given Talia views a booking whose start_time has just passed, when she opens this booking's prompt, then the mark action is eligible and no denial message appears.

**FEAT-11.SPEC-004-AC-02:** Given Talia views a booking whose start_time has not yet arrived, when she attempts to mark it no-show (a state unreachable through the normal dashboard path), then the eligibility check denies with "This booking can be marked no-show only after its appointment time has passed."

**FEAT-11.SPEC-004-AC-03:** Given a booking's auto-completion boundary (platform parameter: `booking-auto-completion-window-days` after start_time) has passed, when Talia attempts to mark it no-show, then the eligibility check denies with "This booking has already auto-completed and can no longer be marked as a no-show."

**FEAT-11.SPEC-004-AC-04:** Given a booking is in Awaiting Outcome state with its Deposit Transaction Captured, when Talia's mark confirm reaches the eligibility check, then both conditions pass and the mark proceeds to FEAT-11.SPEC-002.

**FEAT-11.SPEC-004-AC-05:** Given a booking's Deposit Transaction is no longer Captured (e.g., already Forfeited from an earlier mark), when Talia attempts to mark it no-show again, then the eligibility check denies with "This booking's deposit is not in a state that can be forfeited."

**FEAT-11.SPEC-004-AC-06:** Given Talia marked a booking as a no-show 3 hours ago, when she opens the prompt, then the undo action is eligible because the current time is within the 24-hour grace window (platform parameter: `no-show-undo-grace-window-hours`) of the Forfeited outcome timestamp.

**FEAT-11.SPEC-004-AC-07:** Given Talia marked a booking as a no-show more than 24 hours ago, when she opens the prompt, then the undo action is not offered and the eligibility check denies with "This no-show mark can no longer be undone."

**FEAT-11.SPEC-004-AC-08:** Given a booking's Deposit Transaction is no longer Forfeited (e.g., a goodwill refund has since been issued through FEAT-30), when Talia attempts to undo the no-show mark, then the eligibility check denies with "This booking's deposit is not in a state that can be restored."

**FEAT-11.SPEC-004-AC-09:** Given Talia (the Pro) attempts to mark or undo a no-show on a booking she owns, when the ownership check runs, then it passes and the action-specific window checks proceed.

**FEAT-11.SPEC-004-AC-10:** Given Talia attempts to reach the mark/undo prompt for a booking that belongs to a different Pro Account (e.g., a stale or manipulated link), when the ownership check runs, then it denies with "This booking could not be found." and never reveals that the booking exists.

**FEAT-11.SPEC-004-AC-11:** Given Riley (the Client) has no path to this prompt, when the product's screens are reviewed for a mark or undo control reachable to her, then none exists -- her Cancellation & No-Show Handling access remains Own-only visibility of the resulting outcome.

**FEAT-11.SPEC-004-AC-12:** Given Platform Operator (Support) is viewing this booking through FEAT-19's read-only surfaces, when Support looks for a mark or undo control, then none is shown -- Support's access to Cancellation & No-Show Handling is View-only.

**FEAT-11.SPEC-004-AC-13:** Given a booking's start_time has passed but is not yet at the auto-completion boundary, when the marking window is evaluated, then it is found open (both the lower and upper bound conditions are satisfied).

**FEAT-11.SPEC-004-AC-14:** Given a confirm request for a mark action arrives at the exact instant the auto-completion boundary is reached, when the eligibility check evaluates the upper bound, then it is treated as past the window (the boundary is exclusive) and the action is denied.

**FEAT-11.SPEC-004-AC-15:** Given a confirm request for an undo action arrives at the exact instant the 24-hour grace boundary is reached, when the eligibility check evaluates the window, then it is treated as elapsed and the action is denied.

**FEAT-11.SPEC-004-AC-16:** Given the Booking's state and the Deposit Transaction's status do not match the expected pairing for the requested action at confirm time (e.g., the booking is No-Show but the deposit is already Refunded through another path), when the cross-field consistency rule evaluates the pairing, then the action is denied with the current-state message rather than proceeding against an inconsistent pairing.

**FEAT-11.SPEC-004-AC-17:** Given the marking window and the undo window are both computed against the same appointment, when their boundaries are compared, then the fixed 24-hour undo window always falls before the 7-day auto-completion boundary, so the two never need to be reconciled against each other for the same booking.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 8 | 8 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 9 | 9 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |


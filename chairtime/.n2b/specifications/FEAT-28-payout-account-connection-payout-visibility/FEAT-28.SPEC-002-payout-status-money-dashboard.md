---
document_type: spec
spec_type: screen
spec_id: FEAT-28.SPEC-002
spec_name: Payout Status & Money Dashboard
spec_slug: payout-status-money-dashboard
parent_feature: FEAT-28
parent_feature_name: Payout Account Connection & Payout Visibility
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 22
---

# Screen Spec: Payout Status & Money Dashboard

## Overview

**Name:** Payout Status & Money Dashboard
**ID:** FEAT-28.SPEC-002
**Type:** Screen
**Purpose:** Talia's ongoing view of her payout account's status (with an action-required banner and resolution path when needed) and the money list of deposits, refunds, processor fees, and payouts, in all its data states.
**Parent Feature:** FEAT-28 -- Payout Account Connection & Payout Visibility

## Scope and Non-Goals

**In Scope:**
- The status banner (pending / active / action required), consistent with the treatment first shown by FEAT-28.SPEC-001
- The money list: deposits received, refunds sent, processor fees, and upcoming and past payouts, assembled by FEAT-28.SPEC-005
- Every data state the money list can be in: empty (no bookings yet), loading, error (with last-loaded figures and retry), and offline/degraded
- A direct resolution link from the action-required banner into the processor's own flow

**Non-Goals:**
- The initial connection handoff itself -- owned by FEAT-28.SPEC-001; this screen is reached only once a Payout Account already exists
- Composing or deriving the money list's rows and net figure -- owned by FEAT-28.SPEC-005 (Money List Composition & Net Calculation); this screen only displays what that spec assembles
- Editing bank or identity details -- excluded per SC-05 and the Data Notes field: this screen never collects or shows bank/identity details, only status and money figures
- Support taking any action on a Pro's payout account -- excluded per SC-05: Support's access here is View-only, per the Access Matrix

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-12 (Pro Daily Schedule Dashboard), navigation | Talia taps the money list navigation entry | None -- opens to the default (most recent) view |
| FEAT-12 (Pro Daily Schedule Dashboard), attention list | Talia taps a money-related attention item (refund in progress / payout action required) | The specific attention condition, so the relevant banner or list entry is highlighted on load |
| FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) (Platform Support Read-Only Access) | Support opens a Pro account after a help request | Read-only context; Support's view omits any action affordance |
| FEAT-28.SPEC-007 (Payout Status Notification) | Talia taps "View payouts" or "Resolve now" in a payout status notification | None -- opens to the default view; for "Resolve now" the action-required item is surfaced for FEAT-28.SPEC-006's resolution flow |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen (status banner and full money list) | Tap the resolution link on the action-required banner, retry a failed load | -- |
| Platform Operator (Support) | Status banner and money list, exactly as the Pro sees them | None -- no resolution link, no retry action; view is read-only per ASMP-20 and XBR-24 | Attempting to tap the resolution link (if somehow present) has no effect; the interface shows no such control for Support in the first place |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29, XBR-29) |
| Expired session | No | No | "Your session has expired. Sign in to continue." -- the most recently loaded money list is not shown until re-authentication succeeds, since it reflects financial data |

## Layout and Content

**Header:** Screen title "Payouts" with a back control to FEAT-12, consistent with the setup wizard's status treatment carried forward from FEAT-28.SPEC-001.

**Body, top region -- Status banner:**
- Shown when status is Verification Pending: "Verification pending -- we'll let you know as soon as it's ready."
- Shown when status is Action Required: a prominent banner "Your payout account needs attention" with a one-line reason drawn from the capability's report and a "Resolve now" action.
- Not shown when status is Active (no banner; the money list occupies the full body).

**Body, main region -- Money list:**
- A reverse-chronological list of rows, each one of: a deposit (client name, amount, processor fee, net, date, status), a refund (client name, amount, date, status including "in progress"), a processor payout to the bank (amount, date, status: upcoming or completed).
- A summary strip above the list showing net amount received for the current period (this month by default), per FEAT-28.SPEC-005's derivation.
- Each row shows a running status consistent with the Shared UI Pattern "money list row" from the Brief's Shared Context.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Status banner and summary strip stack above the single-column money list, full width.
- **Medium size class and above:** The summary strip and money list remain single-column but are capped at a consistent platform-wide content width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back control | Tap | Navigate to FEAT-12 (Pro Daily Schedule Dashboard) | Screen closes | Returns to the schedule dashboard |
| "Resolve now" (action-required banner) | Tap (Pro only) | Navigate into the processor's own resolution flow via FEAT-28.SPEC-006 | Screen transitions to the external flow | The capability's own resolution surface opens |
| Money list row | Tap | Expands the row in place to show the full detail already summarized (no navigation to another spec) | Row expands | Additional detail (e.g., the full outcome_reason for a refund, or the payout's arrival estimate) becomes visible |
| Retry (Error state) | Tap (Pro and Support) | Re-attempts loading the money list | List returns to Loading | Success shows the refreshed list; repeated failure keeps the Error state with the same last-loaded figures |
| Summary strip period selector | Tap | Switches the net-figure period (e.g., this month / last 90 days) | Summary strip recalculates | Updated net figure and label shown, per FEAT-28.SPEC-005 |

### Accessibility Notes

- **Focus order:** Back control -> status banner (when present) -> "Resolve now" (when present) -> summary strip period selector -> money list rows in displayed order -> retry action (when present).
- **Dynamic announcements:** A newly appearing action-required banner is announced to assistive technology; a successful retry that refreshes the list announces "Money list updated."
- **Keyboard alternatives:** Row expansion and the period selector are reachable and operable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Active, populated | No status banner; summary strip and full money list shown | Payout Account status is Active and at least one money-list entry exists | Status changes to Action Required, or the list becomes empty (never occurs once populated, per SC-22 retention) |
| Verification Pending | Informational banner shown; money list area shows the Empty (no bookings yet) sub-state, since a deposit cannot be taken before Active (XBR-06) | Payout Account status is Verification Pending | Status changes to Active or Action Required |
| Action Required | Prominent banner with "Resolve now"; money list below remains visible and populated as normal -- existing and new bookings continue per the Brief's Alternate flow | The payment-processing capability reports the account needs action | Talia resolves the flag through FEAT-28.SPEC-006 and the capability reports Active |
| Empty (no bookings yet) | Money list area shows "Your deposits will appear here after your first booking." instead of a zero-filled table | No Deposit Transaction exists yet for this Pro Account | The first deposit is captured (FEAT-07) |
| Loading | A brief in-place indicator over the money list area; nothing in it appears tappable before real data has loaded (ASMP-27) | Screen opens, or a retry is triggered | Data loads successfully or the load fails |
| Error | The last-loaded money list figures remain visible, with the time they were loaded and a "Retry" action | The money list cannot be retrieved on this load attempt | Talia or Support taps Retry and the load succeeds |
| Offline/Degraded | The most recently loaded money list stays viewable, read-only; a banner reads "You're offline -- showing the last money list we loaded." No action (resolution link, retry) is available while offline | Connectivity is lost while this screen is open, or the screen is opened without connectivity | Connectivity is restored and the list refreshes automatically |

## Validation Rules

N/A -- this screen has no user-entered fields; all data is read-only. Composition and derivation rules for the money list are governed by FEAT-28.SPEC-005; eligibility and status rules are governed by FEAT-28.SPEC-004.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back control tap | FEAT-12.SPEC-001 (Today's & Upcoming Schedule) | FEAT-12 (Pro Daily Schedule Dashboard) |
| "Resolve now" tap | FEAT-28.SPEC-006 (Payout Account Connection & Verification) | -- |

## Data Model

**Creates:** None.
**Reads:** Payout Account -- status, payout_schedule, recent payouts; Deposit Transaction -- amount, currency, status, processor_fee, outcome_reason, timestamps; Balance Payment -- amount, tip, state (from v1, once FEAT-22 exists). All composed into the displayed list by FEAT-28.SPEC-005.
**Updates:** None -- this screen never writes to the Payout Account or any Deposit Transaction; resolution happens entirely inside the processor's own flow (FEAT-28.SPEC-006).
**Deletes:** None.

## Business Rules

- XBR-06 / XBR-26: while status is Verification Pending or Action Required, existing and new bookings continue exactly as the Brief's Alternate flow describes; this screen never implies bookings have stopped.
- FEAT-28.SPEC-004 governs that the only deduction ever shown on a deposit row is the processor's own card fee -- Chairtime's own fee is always zero and is never rendered as a line item (XBR-07).
- FEAT-28.SPEC-005 governs money list composition and the net-per-period derivation; this screen is its only display consumer, per the Brief's Shared Validation section.
- FEAT-19.SPEC-004 (Support Session Scope & Access Rules) is the rule spec for Support-session exclusions; this screen enforces it on render by never showing bank or identity details and offering Support no resolution or retry controls.
- Support's View access (Access Matrix, Payouts) never exposes bank or identity details on this screen, since those are never held by the product in the first place (ASMP-31).

## Edge Cases

- **Talia opens the screen with no connectivity at all** -- The Offline/Degraded state shows the most recently loaded money list if one was ever cached on this device, or the Error state's "cannot be retrieved, retry" wording if none exists yet.
- **The action-required banner's condition resolves while Talia is viewing the screen** -- The banner disappears and the screen transitions to the Active, populated state without requiring a manual refresh, since FEAT-28.SPEC-003 updates the Payout Account status the moment the capability reports the change.
- **A refund shown as "in progress" completes while Talia is viewing the list** -- The row updates in place to its completed status the next time the list refreshes (on this screen's own refresh cadence); it is not required to update instantaneously mid-view.
- **Support opens this screen while the Pro is also viewing it** -- No conflict exists since neither role writes to any entity from this screen; both see independent, read-only snapshots (dependency map's Payout Account Contention note: the processor-reported status is authoritative for both).
- **Talia taps Retry while offline** -- Retry has no effect while genuinely offline; the offline banner's own wording already states that connectivity is required, and no request is sent.
- **The money list has grown to multiple years of history (per ASMP-22)** -- The list remains equally responsive; older entries are available by scrolling further, and the summary strip's period selector lets Talia narrow to a shorter window without waiting on the full history to load.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-28.SPEC-003 (Payout Account Status Processing) | References (inbound) | Supplies the status this screen's banner reflects |
| FEAT-28.SPEC-004 (Payout Account Eligibility & Constraints) | References (inbound) | Governs the zero-Chairtime-fee display rule and status meaning |
| FEAT-28.SPEC-005 (Money List Composition & Net Calculation) | References (inbound) | Supplies the assembled money list and net-per-period figure this screen displays |
| FEAT-28.SPEC-006 (Payout Account Connection & Verification) | Navigation (outbound) | "Resolve now" hands the Pro into the same processor flow |
| FEAT-12 (Pro Daily Schedule Dashboard) | Navigation (inbound / outbound) | Entry point via navigation or attention list; back control returns here |
| FEAT-19 (Platform Support Read-Only Access) | Navigation (inbound) | Support's read-only entry point after a help request |
| FEAT-19.SPEC-004 (Support Session Scope & Access Rules) | Enforces (inbound) | That rule's Support-specific exclusions (no bank or identity details, read-only scoping) are enforced on this screen's render, jointly with this spec's own scoping |
| FEAT-25 (Booking & Revenue Insights) | Affects (outbound) | Money list and payout data feed that feature's insights, once it exists |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| money_list_viewed | status at time of view (active / verification_pending / action_required) | Screen opens successfully | supports success-metrics.md: "Payout Transparency" |
| payout_account_action_required | reason category | The Action Required banner is shown | supports success-metrics.md: "Payout Transparency" |
| money_list_load_failed | -- | The money list fails to load, showing the Error state | N/A -- no Stage 2 metric measures load-failure frequency directly; retained so the correctness bar (ASMP-26) is observable for this screen's data-fetching path |
| money_list_row_expanded | row type (deposit / refund / payout) | Talia or Support expands a row | supports success-metrics.md: "Payout Transparency" |

## Acceptance Criteria

**FEAT-28.SPEC-002-AC-01:** Given Talia's Payout Account is Active and has at least one deposit, when she opens this screen, then she sees the summary strip and full money list with no status banner.

**FEAT-28.SPEC-002-AC-02:** Given Talia's Payout Account is Verification Pending, when she opens this screen, then she sees the pending banner and the money list area shows the Empty (no bookings yet) message, since no deposit could be taken yet (XBR-06).

**FEAT-28.SPEC-002-AC-03:** Given Talia's Payout Account is Action Required, when she opens this screen, then she sees the prominent banner with a plain reason and a "Resolve now" action, and her existing money list remains fully visible below it.

**FEAT-28.SPEC-002-AC-04:** Given Talia taps "Resolve now", when the tap registers, then she is handed into the processor's own resolution flow (FEAT-28.SPEC-006).

**FEAT-28.SPEC-002-AC-05:** Given Talia has no bookings yet, when she opens this screen, then she sees "Your deposits will appear here after your first booking." instead of a zero-filled table.

**FEAT-28.SPEC-002-AC-06:** Given the money list cannot be retrieved, when the load fails, then the last-loaded figures remain visible with the time they were loaded and a "Retry" action.

**FEAT-28.SPEC-002-AC-07:** Given Talia loses connectivity while viewing a previously loaded money list, when connectivity drops, then the most recently loaded list stays viewable read-only with the offline banner shown.

**FEAT-28.SPEC-002-AC-08:** Given a deposit row is displayed, when Talia looks at its deduction, then only the processor's own card fee is shown -- never a Chairtime fee line, since Chairtime's fee is always zero (XBR-07).

**FEAT-28.SPEC-002-AC-09:** Given Talia taps a deposit row, when the tap registers, then the row expands in place to show its full detail without leaving this screen.

**FEAT-28.SPEC-002-AC-10:** Given Platform Operator (Support) opens this screen for a Pro after a help request, when the banner is Action Required, then Support sees the same banner and reason Talia sees but no "Resolve now" action, since resolution is Pro-only per SC-05; Support's Retry action for a failed load remains available, since retrying a read is not an action on the Pro's account.

**FEAT-28.SPEC-002-AC-11:** Given the Action Required condition resolves while Talia is viewing this screen, when FEAT-28.SPEC-003 records the change, then the banner disappears without a manual refresh.

**FEAT-28.SPEC-002-AC-12:** Given a refund shows as "in progress," when it completes, then the row updates to its completed status on the screen's next refresh.

**FEAT-28.SPEC-002-AC-13:** Given Talia opens this screen with no connectivity and no cached list on this device, when the load is attempted, then the Error state's "cannot be retrieved" wording with Retry is shown rather than the offline banner.

**FEAT-28.SPEC-002-AC-14:** Given Talia is offline and viewing the cached list, when she taps Retry, then no request is sent and the offline state remains unchanged.

**FEAT-28.SPEC-002-AC-15:** Given an unauthenticated visitor attempts to reach this screen, when the attempt is made, then they are redirected to the Pro sign-in screen (FEAT-29).

**FEAT-28.SPEC-002-AC-16:** Given Talia's session expires while viewing this screen, when she next interacts with it, then she sees "Your session has expired. Sign in to continue." and the money list is not shown until she re-authenticates.

**FEAT-28.SPEC-002-AC-17:** Given Talia is on FEAT-12's attention list and taps the payout action-required item, when this screen opens, then the Action Required banner is already visible and, where applicable, highlighted.

**FEAT-28.SPEC-002-AC-18:** Given Talia changes the summary strip's period selector, when the new period is chosen, then the net figure recalculates per FEAT-28.SPEC-005 and the money list itself is unaffected (it always shows full history).

**FEAT-28.SPEC-002-AC-19:** Given Talia's money list has multiple years of history, when she scrolls further back, then older entries continue to load responsively.

**FEAT-28.SPEC-002-AC-20:** Given a money-list load fails and Talia taps Retry, when the retry succeeds, then the refreshed list replaces the last-loaded figures and the "loaded at" timestamp updates.

**FEAT-28.SPEC-002-AC-21:** Given Support views this screen, when Support looks for bank or identity details anywhere on it, then none are present -- only status and money-list figures, consistent with ASMP-20.

**FEAT-28.SPEC-002-AC-22:** Given Talia's Payout Account transitions from Active to Action Required while she is on FEAT-12 rather than this screen, when she next opens this screen (via the attention list), then the Action Required banner and its reason are already reflected.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 7 (active/populated, verification pending, action required, empty, loading, error, offline) | 7 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |

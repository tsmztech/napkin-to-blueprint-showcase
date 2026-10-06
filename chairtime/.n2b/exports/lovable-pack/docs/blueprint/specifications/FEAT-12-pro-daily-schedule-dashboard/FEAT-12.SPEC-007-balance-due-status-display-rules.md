---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-12.SPEC-007
spec_name: Balance Due & Status Display Rules
spec_slug: balance-due-status-display-rules
parent_feature: FEAT-12
parent_feature_name: Pro Daily Schedule Dashboard
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 24
acceptance_criteria_count: 15
---

# Logic/Rule Spec: Balance Due & Status Display Rules

## Overview

**Name:** Balance Due & Status Display Rules
**ID:** FEAT-12.SPEC-007
**Type:** Logic/Rule
**Purpose:** Derives the balance-due amount and the paid/unpaid, "I'll be there," and sync-reliability display states shown consistently on every booking row across this feature's three screens.
**Parent Feature:** FEAT-12 -- Pro Daily Schedule Dashboard
**Governed Entity:** Booking (balance_due derivation and attendance_reply display), plus Deposit Transaction (paid/unpaid status display) and Calendar Connection (sync-reliability display) as read-only inputs to the same derived booking-row presentation

## Scope and Non-Goals

**In Scope:**
- Deriving `balance_due` from Booking and Deposit Transaction data (XBR-23)
- Deriving the paid/unpaid badge shown on every booking row from Deposit Transaction status
- Deriving the "I'll be there" attendance display from Booking's `attendance_reply`
- Deriving the per-booking sync-reliability marking from Calendar Connection status
- The exact display conventions (badge wording, never color-alone) so SPEC-001 and SPEC-003 render the shared booking row identically

**Non-Goals:**
- Whether a Booking may be marked Completed -- owned by FEAT-12.SPEC-006 (Booking Completion Rules); this spec only derives what is displayed once a state is reached
- Who may view a booking at all -- owned by FEAT-12.SPEC-008 (Dashboard Access Authorization); this spec assumes the viewer is already authorized and only governs what is shown once visible
- Collecting or recording the deposit or balance payment itself -- owned by FEAT-07 (Deposit Payment at Booking) and FEAT-22 (In-App Balance Payment, v1, not yet built at MVP); this spec only reads their recorded outcomes for display
- Diagnosing or resolving a calendar sync failure -- owned by FEAT-04 (Two-Way Calendar Sync); this spec only reads the reported `status` field to decide how a booking's reliability marking reads

## Governed Entity

**Entity:** Booking (primary), with read-only display inputs from Deposit Transaction and Calendar Connection
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| price_agreed | number | Booking -- price agreed at booking time |
| deposit_amount | number | Booking -- deposit agreed at booking time |
| balance_due | derived | Booking -- price_agreed − deposit_amount − any in-app balance payment |
| attendance_reply | enum | Booking -- "I'll be there" / reschedule requested, from reminders, or no reply yet |
| state | enum | Booking -- current lifecycle state, read here only to decide whether balance/paid display still applies |
| deposit_transaction.status | enum | Deposit Transaction -- Authorized \| Captured \| Applied \| Refunded \| Refund in Progress \| Forfeited \| Disputed |
| deposit_transaction.amount | number | Deposit Transaction -- the captured deposit amount |
| calendar_connection.status | enum | Calendar Connection -- Connected \| Syncing \| Needs Reconnection \| Disconnected |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-12.SPEC-001 | Today's & Upcoming Schedule | On every booking row render, for today's and upcoming bookings |
| FEAT-12.SPEC-003 | Past Bookings Browse | On every booking row render, for past bookings |
| FEAT-12.SPEC-002 | Attention List | On the underlying booking reference shown inside a sync-reliability or dispute attention item |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| price_agreed | No validation beyond data type -- this spec only reads it for derivation | Always | -- | -- | -- |
| deposit_amount | No validation beyond data type -- this spec only reads it for derivation | Always | -- | -- | -- |
| balance_due | Must equal price_agreed − deposit_amount − any in-app balance payment; never displayed as a negative amount | Always (derivation is display-only, not user input) | On every row render | If the derivation would be negative, display as fully settled (see Business Rules) rather than a negative figure | No -- this is a display derivation, not a user-input validation |
| attendance_reply | No validation beyond data type -- this spec only reads it for the display label | Always | -- | -- | -- |
| state | No validation beyond data type -- read only to gate whether balance/paid display still applies (e.g., a Cancelled booking shows no balance-due badge) | Always | -- | -- | -- |
| deposit_transaction.status | No validation beyond data type -- this spec only reads it to select the paid-badge wording | Always | -- | -- | -- |
| deposit_transaction.amount | No validation beyond data type -- this spec only reads it for derivation | Always | -- | -- | -- |
| calendar_connection.status | No validation beyond data type -- this spec only reads it to select the sync-reliability wording | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Balance-due formula | price_agreed, deposit_amount, balance_due | balance_due = price_agreed − deposit_amount − any in-app balance payment (XBR-23); at MVP, with no in-app balance payment capability, balance_due = price_agreed − deposit_amount | N/A -- derived value, no user-facing error |
| Paid-badge wording | deposit_transaction.status, state | If deposit_transaction.status is Captured or Applied, the badge reads "Paid" (with a word, never color alone, per ASMP-28); if Refund in Progress, the badge reads "Refund in progress"; if Refunded, the booking is no longer shown as a balance-due item since the booking itself is Cancelled; if Disputed, the badge reads "Paid" with a separate dispute flag surfaced through the Attention List (FEAT-12.SPEC-005), never replacing the paid badge itself | N/A -- display-only |
| Attendance display | attendance_reply, state | If attendance_reply is "I'll be there," the row shows that label; if it is a reschedule request, the row shows "Asked to reschedule" and links to FEAT-10's context (no direct action taken here); if no reply has been received, the row shows no attendance label at all (absence of a label is intentional, not an error state) | N/A -- display-only |
| Sync-reliability marking | calendar_connection.status | If Connected or Syncing, the booking's reliability marking reads normally (no special marking); if Needs Reconnection or Disconnected, the booking is marked "Reliability uncertain" per FEAT-12.SPEC-005's aggregation, since the calendar connection cannot currently confirm this time is conflict-free | N/A -- display-only |
| Balance-due suppressed on terminal non-completed states | state, balance_due | If state is Cancelled by Client, Cancelled by Pro, No-Show, Rescheduled, or Expired (unpaid), the balance-due figure is not shown as an outstanding amount on the row -- these states have their own status display (owned by FEAT-09, FEAT-10, FEAT-11, FEAT-30) and this spec defers to them rather than showing a stale balance-due badge | N/A -- display-only |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View derived balance-due, paid badge, attendance reply, and sync-reliability marking on own bookings | The Pro | Own bookings only | -- |
| View derived balance-due, paid badge, attendance reply, and sync-reliability marking | Platform Operator (Support) | The one Pro account under active support review, read-only (XBR-24) -- masked the same way as the Pro's own view since none of these derived values are private client-note content | -- |
| View derived balance-due, paid badge, attendance reply, and sync-reliability marking | The Client | Never on this feature's screens | The Client has no access to this dashboard at all (FEAT-12.SPEC-008); their own balance/paid status is shown through FEAT-06 instead |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| balance_due | price_agreed − deposit_amount − any in-app balance payment (XBR-23); floors at zero for display -- never shown as negative | Recomputed on every render | No -- this is a read-only derived display value |
| paid-badge label | Selected from deposit_transaction.status per the Cross-Field Rules table above | Recomputed on every render | No |
| attendance label | Selected from attendance_reply per the Cross-Field Rules table above | Recomputed on every render | No |
| sync-reliability marking | Selected from calendar_connection.status per the Cross-Field Rules table above, sourced through FEAT-12.SPEC-005's aggregation | Recomputed on every render | No |

## Business Rules

- Balance due is always computed, never stored as a separately editable field: price_agreed − deposit_amount − any in-app balance payment (XBR-23). At MVP, with FEAT-22 (In-App Balance Payment) not yet built, the formula simplifies to price_agreed − deposit_amount.
- If the computed balance_due would be zero or negative (e.g., the deposit equals or exceeds the price, or a balance payment already covers the remainder once FEAT-22 exists), the row shows the booking as fully settled rather than a zero or negative figure.
- Paid/unpaid and attention status are never conveyed by color alone -- every badge carries a word alongside any color treatment (ASMP-28 Accessibility baseline).
- A Disputed Deposit Transaction never overwrites or hides the underlying paid status on the booking row; the dispute is a separate flag raised through the Attention List (FEAT-12.SPEC-002, fed by FEAT-12.SPEC-005), consistent with XBR-22's rule that a dispute overlay never erases the underlying outcome.
- Sync-reliability marking never claims false confidence: if the calendar connection cannot currently confirm a time is genuinely free of external conflicts, the booking is marked uncertain rather than shown as reliable (XBR-13; this feature's own Side-Effect Inventory entry "A booking's calendar-sync status is uncertain").
- Messaging Consent (textability) is displayed as a read-only status alongside the booking row, per FEAT-12.SPEC-001 -- this spec does not compute or alter it; it is read directly from Messaging Consent's `state` field.
- Card data is never part of any derived display value on this feature's screens (ASMP-15, SC-11); only amounts and statuses are shown.
- These derivation rules apply identically wherever the shared booking row pattern is used -- FEAT-12.SPEC-001's today/upcoming list and FEAT-12.SPEC-003's past-bookings list -- so a client's balance-due figure and paid badge read the same regardless of which screen shows it.

## Edge Cases

- **Deposit equals the full price (100% deposit rule)** -- balance_due computes to zero; the row shows "Paid in full" rather than a $0 balance-due badge.
- **Booking is Cancelled by Client with the deposit refunded** -- No balance-due badge is shown; the row instead shows the cancellation/refund status owned by FEAT-10/FEAT-09, and this spec's paid badge is not rendered for a cancelled booking.
- **Deposit Transaction is Disputed while the booking is still upcoming** -- The paid badge continues to read "Paid" (the underlying outcome is unchanged); a separate dispute flag appears via the Attention List (FEAT-12.SPEC-002).
- **Calendar Connection status flips from Needs Reconnection back to Connected while the Pro is viewing the schedule** -- The reliability marking updates to reflect the current status on the screen's next refresh (FEAT-12.SPEC-001 defines the refresh behavior); this spec does not itself define polling frequency.
- **Client has not yet replied to a reminder and the appointment is imminent** -- No attendance label is shown; absence of a reply is not treated as a negative signal or rendered as an error state.
- **In-app balance payment exists (post-MVP, FEAT-22) and partially covers the balance** -- balance_due recomputes to price_agreed − deposit_amount − balance payment amount; this spec's formula already accounts for that term, so no rule change is needed when FEAT-22 ships.

## Acceptance Criteria

**FEAT-12.SPEC-007-AC-01:** Given a booking with price_agreed and deposit_amount recorded, when Talia views it on FEAT-12.SPEC-001, then the balance-due figure shown equals price_agreed minus deposit_amount.

**FEAT-12.SPEC-007-AC-02:** Given a booking's deposit equals its full price, when Talia views the row, then it shows "Paid in full" rather than a $0 balance-due badge.

**FEAT-12.SPEC-007-AC-03:** Given a booking's Deposit Transaction status is Captured, when Talia views the row, then the badge reads "Paid" with the word visible alongside any color treatment.

**FEAT-12.SPEC-007-AC-04:** Given a booking's Deposit Transaction status is Refund in Progress, when Talia views the row, then the badge reads "Refund in progress."

**FEAT-12.SPEC-007-AC-05:** Given a booking's Deposit Transaction becomes Disputed, when Talia views the row, then the paid badge still reads "Paid" and a separate dispute flag appears through the Attention List, per XBR-22.

**FEAT-12.SPEC-007-AC-06:** Given a client has tapped "I'll be there" on a reminder, when Talia views that booking's row, then the attendance label shows "I'll be there."

**FEAT-12.SPEC-007-AC-07:** Given a client has not replied to any reminder for a booking, when Talia views that row, then no attendance label is shown.

**FEAT-12.SPEC-007-AC-08:** Given the Pro's Calendar Connection status is Needs Reconnection, when Talia views a booking whose reliability cannot currently be confirmed, then that booking is marked "Reliability uncertain" rather than shown as normally reliable.

**FEAT-12.SPEC-007-AC-09:** Given the Pro's Calendar Connection status is Connected, when Talia views any booking row, then no reliability-uncertain marking appears.

**FEAT-12.SPEC-007-AC-10:** Given a booking is Cancelled by Client, when Talia views its row, then no balance-due badge is shown for it.

**FEAT-12.SPEC-007-AC-11:** Given Talia (the Pro) views her own dashboard, when a booking row renders, then the derived balance, paid, attendance, and reliability values are shown for it, since she is authorized to view her own bookings.

**FEAT-12.SPEC-007-AC-12:** Given Platform Operator (Support) is reviewing a Pro's account, when Support views a booking row, then the same derived balance, paid, attendance, and reliability values are shown as the Pro sees, since none of these values are private client-note content.

**FEAT-12.SPEC-007-AC-13:** Given a Client attempts to reach this dashboard, when the access check runs, then no booking row or derived value is ever shown to them here, since Clients have no access to this feature (FEAT-12.SPEC-008).

**FEAT-12.SPEC-007-AC-14:** Given the same booking appears on both FEAT-12.SPEC-001 (upcoming) and, once past, FEAT-12.SPEC-003 (past bookings), when Talia views it on either screen, then the balance-due figure and paid badge read identically on both.

**FEAT-12.SPEC-007-AC-15:** Given a booking's price_agreed is less than its deposit_amount due to a later price adjustment path that does not exist in this product (hypothetical negative derivation), when the derivation computes a negative value, then the row shows the booking as fully settled rather than a negative balance-due figure.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 8 | 8 |
| Cross-Field Rules | 5 | 5 |
| Authorization Rules | 3 | 3 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 8 | 8 |
| Edge Cases | 6 | 6 |

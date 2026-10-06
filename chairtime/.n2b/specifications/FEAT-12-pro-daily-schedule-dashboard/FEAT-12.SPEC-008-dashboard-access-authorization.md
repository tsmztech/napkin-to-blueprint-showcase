---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-12.SPEC-008
spec_name: Dashboard Access Authorization
spec_slug: dashboard-access-authorization
parent_feature: FEAT-12
parent_feature_name: Pro Daily Schedule Dashboard
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 30
acceptance_criteria_count: 13
---

# Logic/Rule Spec: Dashboard Access Authorization

## Overview

**Name:** Dashboard Access Authorization
**ID:** FEAT-12.SPEC-008
**Type:** Logic/Rule
**Purpose:** Enforces who may open this feature's three screens and what each role sees: the Pro's own schedule in full, Support's masked read-only view for troubleshooting, and a redirect to sign-in for anyone else.
**Parent Feature:** FEAT-12 -- Pro Daily Schedule Dashboard
**Governed Entity:** Booking (the entity every screen in this feature reads), gated together with the Client, Deposit Transaction, Message, Time Block, Calendar Connection, Messaging Consent, and Waitlist Entry fields those screens display alongside it

## Scope and Non-Goals

**In Scope:**
- Who may open FEAT-12.SPEC-001, FEAT-12.SPEC-002, and FEAT-12.SPEC-003, and under what identity conditions
- What the Pro's Full access covers versus Support's View-only access on this feature's three screens
- Exactly which fields Support never sees, regardless of role-level View access (private client notes, sign-in codes, card or bank/identity details)
- The unauthenticated and expired-session experience for every screen in this feature
- Confirming that Support's every view is logged, per XBR-24

**Non-Goals:**
- Whether a specific booking action (mark completed) is allowed once the dashboard is open -- owned by FEAT-12.SPEC-006 (Booking Completion Rules)
- Deriving what a booking row displays once visible -- owned by FEAT-12.SPEC-007 (Balance Due & Status Display Rules)
- The Pro sign-in mechanism itself (one-time code, device trust, new-device alerts) -- owned by FEAT-29 (Pro Sign-In & Account Lifecycle); this spec only enforces that a signed-in Pro identity is required
- Support's own audit-log screen or how support sessions are opened/closed -- owned by FEAT-19 (Platform Support Read-Only Access); this spec only enforces the resulting view restrictions on FEAT-12's three screens

## Governed Entity

**Entity:** Booking (primary), gated together with the read-only entities every screen in this feature displays
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| client (reference), and via it Client.private_note | reference / text | Client linked to the booking; the Pro's private note about that client |
| service, start_time, duration, price_agreed, deposit_amount, state, attendance_reply, balance_due, source, cancellation/reschedule timestamps | mixed | Booking's own fields (see FEAT-12.SPEC-006 for the full list); visible to the Pro and, masked as below, to Support |
| deposit_transaction.status, amount | enum / number | Deposit Transaction fields shown per booking; card data is never part of this entity (ASMP-15) |
| message.delivery_status | enum | Message delivery-failure flags shown per booking |
| time_block.start / end / label | date/time / text | Manual time blocks shown alongside bookings; `label` is Pro-private |
| calendar_connection.status | enum | Sync-health status; never event titles or details |
| messaging_consent.state | enum | Textability status only |
| waitlist_entry (aggregate count only) | number | Aggregate waitlist demand; never individual entries shown to the Pro |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-12.SPEC-001 | Today's & Upcoming Schedule | On screen entry (before any data loads) and on every subsequent data refresh |
| FEAT-12.SPEC-002 | Attention List | On screen entry and on every subsequent data refresh |
| FEAT-12.SPEC-003 | Past Bookings Browse | On screen entry and on every subsequent data refresh |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| client.private_note | Never shown to Platform Operator (Support), regardless of the booking otherwise being visible to Support | When the viewer is Support | On every render that would otherwise include the note preview | The note-preview element is omitted entirely from Support's view (not shown blank, not shown as "hidden" -- simply not present in the layout) | Yes |
| All other Booking, Deposit Transaction, Message, Time Block (except label), Calendar Connection, and Messaging Consent fields listed above | No validation beyond data type -- this spec governs view authorization, not field-level validation of these fields (owned by their originating features) | Always | -- | -- | -- |
| time_block.label | Never shown to Platform Operator (Support) -- Pro-private per the dependency map's Data Sensitivity note for Time Block | When the viewer is Support | On every render | The label is omitted from Support's view of a time block; the block itself (its start/end) still shows as occupied time | Yes |
| Card, bank, or identity details | Never displayed on any of this feature's three screens, for any role | Always | -- | N/A -- these fields are never surfaced here at all (ASMP-15, SC-11); this is a structural exclusion, not a per-role denial | Yes |
| Pro sign-in codes | Never displayed on any of this feature's three screens, for any role including Support | Always | -- | N/A -- sign-in codes are never surfaced here (ASMP-20, XBR-24) | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Support masking is uniform across all three screens | client.private_note, time_block.label, sign-in codes, card/bank/identity details | The same masking rules apply identically whether Support is viewing FEAT-12.SPEC-001, FEAT-12.SPEC-002, or FEAT-12.SPEC-003 -- no screen in this feature exposes to Support what another screen in this feature withholds | N/A -- structural consistency rule, no user-facing error |
| Support access is scoped to one Pro account at a time | (account-level, not a Booking field) | Support's view, when active, is scoped entirely to the single Pro account under active review (XBR-24); this spec never allows a cross-account view | N/A -- enforced by FEAT-19, referenced here |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Open FEAT-12.SPEC-001 (Today's & Upcoming Schedule) | The Pro | Signed in as the Pro, viewing their own account only | -- |
| Open FEAT-12.SPEC-001 | Platform Operator (Support) | Only while an active support session for that specific Pro account is open, following a Pro help request (XBR-24) | -- |
| Open FEAT-12.SPEC-001 | The Client | Never | Redirected to the Pro sign-in screen (FEAT-29), per XBR-29; a client is never told whether the destination account exists |
| Open FEAT-12.SPEC-001 | Unauthenticated visitor | Never | Redirected to the Pro sign-in screen (FEAT-29) |
| Open FEAT-12.SPEC-002 (Attention List) | The Pro | Signed in as the Pro, viewing their own account only | -- |
| Open FEAT-12.SPEC-002 | Platform Operator (Support) | Only during an active support session for that Pro account | -- |
| Open FEAT-12.SPEC-002 | The Client | Never | Redirected to the Pro sign-in screen (FEAT-29) |
| Open FEAT-12.SPEC-002 | Unauthenticated visitor | Never | Redirected to the Pro sign-in screen (FEAT-29) |
| Open FEAT-12.SPEC-003 (Past Bookings Browse) | The Pro | Signed in as the Pro, viewing their own account only | -- |
| Open FEAT-12.SPEC-003 | Platform Operator (Support) | Only during an active support session for that Pro account | -- |
| Open FEAT-12.SPEC-003 | The Client | Never | Redirected to the Pro sign-in screen (FEAT-29) |
| Open FEAT-12.SPEC-003 | Unauthenticated visitor | Never | Redirected to the Pro sign-in screen (FEAT-29) |
| View a client's private note preview on any booking row | The Pro | Own bookings only | -- |
| View a client's private note preview on any booking row | Platform Operator (Support) | Never | The note-preview element is omitted from the row entirely |
| Perform any write action on this feature's three screens (mark completed, resolve an attention item) | Platform Operator (Support) | Never | Every write control is omitted from Support's view; Support's access is strictly View-only (SC-05) |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Viewer role for the current session | Derived from the signed-in identity: Pro (own account), Support (active support session on a named Pro account), or neither | On every screen entry in this feature | No -- the viewer cannot elevate their own role from within the screen |
| Masked-field set applied to the current render | Derived from the viewer role above: Pro sees everything owned by the account; Support sees everything except private client notes, time-block labels, sign-in codes, and card/bank/identity details | Recomputed on every render | No |

## Business Rules

- Every screen in this feature requires a signed-in Pro identity (or an active Support session scoped to that Pro), per XBR-29: "every Pro-facing screen requires a signed-in Pro; anyone else is sent to the Pro sign-in screen."
- A failed sign-in never reveals whether a Pro account exists (XBR-29) -- this spec never distinguishes "no such account" from "wrong code" in any redirect behavior on these three screens.
- Support's access is read-only, one account at a time, used only after a Pro's help request, and never shows private client notes, bank or identity details, or sign-in codes (XBR-24) -- this is enforced identically across all three of this feature's screens.
- Every Support view of this feature's screens is logged in the Pro's visible account activity (XBR-24); the Pro can see when Support looked, through FEAT-19's own audit surface (not rendered inside this feature's screens themselves).
- Support cannot act on the Pro's behalf from this dashboard (SC-05): no write control (mark completed, resolve an attention item, navigate into an editing flow on another feature) is shown to Support.
- Calendar Connection data shown on this feature's screens is limited to sync health only, never event titles or details, for either the Pro or Support (product-features.md FEAT-04 Data Notes).
- Messaging Consent is shown as textability status only; neither the Pro nor Support can override a client's consent from this feature's screens (US texting-consent rules, ASMP-24).
- Waitlist demand shown on FEAT-12.SPEC-002 is an aggregate count only, for the Pro; individual waitlist entries are never surfaced here (Access Matrix: Waitlist = View for the Pro), and this rule applies identically to Support's masked view.

## Edge Cases

- **A Pro's sign-in session expires while a screen in this feature is open** -- The screen's own Expired session row (defined per-screen in FEAT-12.SPEC-001, FEAT-12.SPEC-002, FEAT-12.SPEC-003) applies; this spec's authorization check re-runs on the next data refresh and, finding no valid session, redirects to the Pro sign-in screen (FEAT-29).
- **A support session is closed (or times out) while Support is viewing one of these screens** -- The next authorization check finds no active support session and the screen redirects Support out; no further data loads.
- **Support attempts to reach one of these three screens without an active session tied to a specific Pro help request** -- Access is refused at the authorization check, consistent with XBR-24's "used only after a Pro's help request"; enforcement of when a session may be opened belongs to FEAT-19, but this spec never renders Pro data absent that active session.
- **A Client, signed in through their own client-side access link (FEAT-06), tries to open a Pro-facing URL for this feature** -- A client access link never grants a Pro-facing session; the visitor is treated as unauthenticated for this feature's purposes and redirected to the Pro sign-in screen (FEAT-29), never shown any booking data.
- **The Pro views the dashboard on two signed-in devices at once** -- Both are the same authorized role (the Pro, own account); both see the full, unmasked view. This is a read-scenario, not a write conflict; any write conflict (e.g., both marking the same booking completed) is governed by FEAT-12.SPEC-006, not this spec.
- **A booking's client note is edited by the Pro (FEAT-13) while Support is viewing the same booking's row on this feature** -- Irrelevant to Support, since the private note is never shown to Support in the first place; no visibility change occurs for Support regardless of the underlying edit.

## Acceptance Criteria

**FEAT-12.SPEC-008-AC-01:** Given Talia is signed in as the Pro, when she opens FEAT-12.SPEC-001, then she sees her own schedule in full, with all fields visible.

**FEAT-12.SPEC-008-AC-02:** Given Platform Operator (Support) has an active support session on Talia's account following her help request, when Support opens FEAT-12.SPEC-001, then Support sees the same bookings with the client private-note preview omitted from every row.

**FEAT-12.SPEC-008-AC-03:** Given a visitor is not signed in as any Pro, when they attempt to open FEAT-12.SPEC-001, then they are redirected to the Pro sign-in screen (FEAT-29), per XBR-29.

**FEAT-12.SPEC-008-AC-04:** Given Riley (the Client) attempts to reach FEAT-12.SPEC-001 directly, when the access check runs, then Riley is redirected to the Pro sign-in screen, since Clients have no access to this feature.

**FEAT-12.SPEC-008-AC-05:** Given Platform Operator (Support) is viewing Talia's booking list, when Support looks for a mark-completed or resolve-attention-item control, then none is shown, since Support's access is strictly View-only.

**FEAT-12.SPEC-008-AC-06:** Given Talia's sign-in session expires while FEAT-12.SPEC-002 is open, when the screen's next data refresh runs, then she is redirected to the Pro sign-in screen (FEAT-29).

**FEAT-12.SPEC-008-AC-07:** Given Support's active session on Talia's account is closed, when Support's screen attempts its next refresh, then Support is redirected out and no further Pro data loads.

**FEAT-12.SPEC-008-AC-08:** Given Talia is viewing a booking row on FEAT-12.SPEC-003, when the row renders, then her private note preview for that client is visible to her.

**FEAT-12.SPEC-008-AC-09:** Given Support is viewing the same booking row on FEAT-12.SPEC-003, when the row renders, then no private note preview element appears at all for Support.

**FEAT-12.SPEC-008-AC-10:** Given Support is viewing FEAT-12.SPEC-002, when a time block appears alongside bookings, then its start/end are shown but its private label is omitted.

**FEAT-12.SPEC-008-AC-11:** Given Talia opens FEAT-12.SPEC-002, when the waitlist demand item renders, then only an aggregate count is shown, never individual waitlist entries.

**FEAT-12.SPEC-008-AC-12:** Given a failed sign-in attempt is made against this feature's entry point, when the sign-in fails, then the response never reveals whether a Pro account exists for the attempted identifier, per XBR-29.

**FEAT-12.SPEC-008-AC-13:** Given Talia is signed in on two devices at once, when she opens FEAT-12.SPEC-001 on both, then both show her full, unmasked schedule, since both are the same authorized Pro role.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 15 | 15 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 8 | 8 |
| Edge Cases | 6 | 6 |

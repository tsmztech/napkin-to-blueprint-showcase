---
document_type: spec
spec_type: screen
spec_id: FEAT-19.SPEC-001
spec_name: Pro Account Lookup & Support Session Entry
spec_slug: pro-account-lookup-support-session-entry
parent_feature: FEAT-19
parent_feature_name: Platform Support Read-Only Access
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 20
---

# Screen Spec: Pro Account Lookup & Support Session Entry

## Overview

**Name:** Pro Account Lookup & Support Session Entry
**ID:** FEAT-19.SPEC-001
**Type:** Screen
**Purpose:** Support looks up one specific Pro by request, records a reason or ticket reference, and opens a single-account, structurally read-only session that hands into the already-validated read-only surfaces of other features for services, schedule, bookings, billing status, and disputed booking timelines.
**Parent Feature:** FEAT-19 -- Platform Support Read-Only Access

## Scope and Non-Goals

**In Scope:**
- The lookup form: entering a Pro identifier and a reason/ticket reference, and opening a scoped support session
- The no-match outcome when the lookup finds no Pro
- The session hub: once a session is open, the single set of navigation entries into every other feature's already-validated Support-facing read-only rendering (services, schedule/bookings, client records, billing, payouts, setup progress, time blocks, booking page preview, messages) and into the Support Access Log (FEAT-19.SPEC-003)
- Ending the current session (explicitly, or automatically when a new lookup is submitted)

**Non-Goals:**
- The detailed rendering of services, schedule, bookings, client records, billing, payouts, setup progress, time blocks, the booking page, or messages themselves -- each is owned by its own feature's Spec Writer (FEAT-01.SPEC-003, FEAT-12.SPEC-001/002/003/008, FEAT-13 (all specs), FEAT-15.SPEC-004, FEAT-16.SPEC-001, FEAT-17.SPEC-002, FEAT-18.SPEC-002, FEAT-28.SPEC-002, FEAT-05.SPEC-001-005, FEAT-08's message screens); this spec owns only the lookup, hand-off, and session-boundary behavior, per feature-overview.md's Shared UI Patterns.
- Recording the support-view event -- owned by FEAT-19.SPEC-002 (Support View Logging), which this screen's session-open and disputed-timeline-view actions trigger but do not themselves write.
- The structural no-write rule, one-account-at-a-time scoping, the help-request precondition, and the field-level exclusions -- owned by FEAT-19.SPEC-004 (Support Session Scope & Access Rules), which this screen enforces but does not restate.
- Any write, edit, refund, or sign-in-as action -- excluded per SC-05: no such control exists anywhere in this screen or any screen it hands into for Support.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| External -- Support's own initiative, prompted by a Pro's help request sent through FEAT-27 (Pro Profile & Booking Page Settings) | Support decides to look up an account after receiving a Pro's help request | None -- FEAT-27 owns no support-side view or direct link into this screen; Support opens this screen on their own and enters the lookup manually |
| This screen (re-entry) | Support submits a new lookup while a session is already open | The prior session's Pro Account reference is discarded as the new lookup replaces it (per FEAT-19.SPEC-004's one-account-at-a-time rule) |

This is the only entry point into FEAT-19 (feature-overview.md's Internal Dependency Map, Default Entry), reached solely by Platform Operator (Support), never by the Pro or a Client, and never by navigation from any client- or Pro-facing screen.

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Platform Operator (Support) | Full screen -- lookup form and, once a session is open, the session hub | Submit a lookup, open a hand-off screen, view the Support Access Log, end the session | -- |
| The Pro (Talia) | No | No | No navigation path anywhere in the product reaches this screen for the Pro; this screen exists outside the Pro's own navigation entirely |
| The Client (Riley) | No | No | No navigation path anywhere in the product reaches this screen for a Client; this screen is never linked from any client-facing surface |
| Unauthenticated | No | No | This screen requires an authenticated operator; an unauthenticated visitor sees a plain "this page isn't available" experience, since no client- or Pro-facing sign-in screen applies to it and no operator sign-in screen is itself part of this feature's scope |
| Expired session (operator's own access) | No | No | The screen (and any open support session) closes immediately; nothing is lost because no draftable input exists beyond the lookup form's two fields, which are discarded |

## Layout and Content

**Header:** Screen title "Support: Pro Account Lookup." Once a session is open, the title changes to "Support Session: {Pro Account display_name}" with an "End Session" action (right-aligned).

**Body -- Lookup state (default, no session open):**
- A single-column form with two fields, in order:
  - Pro Account Lookup (text input, required) -- accepts the Pro's booking_link_name, sign_in_email, sign_in_mobile, or display_name, whichever detail the help request included
  - Reason / Ticket Reference (text input, required) -- a short free-text note Support enters describing the help request
- "Open Support Session" action button, below the form
- If a session is already open when this state is reached again (Support returned to submit a fresh lookup), an informational banner above the form: "Opening a new lookup will end your current session with {display_name}."

**Body -- Session Active state (hub, once a lookup succeeds):**
- A Pro Account summary card at the top: display_name, status (Active | Paused | Closing | Closed), and the reason/ticket reference entered for this session
- Below the card, a list of navigation entries, each handing into that feature's already-validated Support-facing read-only rendering:
  - Services -- FEAT-01.SPEC-003
  - Schedule & Bookings -- FEAT-12.SPEC-001, FEAT-12.SPEC-002, FEAT-12.SPEC-003 (gated by FEAT-12.SPEC-008's masked read-only authorization)
  - Client Records -- FEAT-13 (all specs, private note excluded by FEAT-13.SPEC-005)
  - Setup Progress -- FEAT-15.SPEC-004
  - Time Blocks -- FEAT-17.SPEC-002
  - Billing & Subscription -- FEAT-18.SPEC-002
  - Payout Status -- FEAT-28.SPEC-002 (status and money list only)
  - Booking Page Preview -- FEAT-05.SPEC-001 through FEAT-05.SPEC-005
  - Messages -- FEAT-08's message and notification screens
  - Support Access Log -- FEAT-19.SPEC-003
- No edit control, save control, or action button of any kind appears anywhere on this list or the summary card, by design (SC-05)

**Body -- No Match state:** A single plain message in place of the form's result area: "No Pro account matches that lookup." The form's two fields remain filled with the values Support entered, and no session opens.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Single-column form (lookup state) or single-column summary card and stacked navigation list (session state), full width.
- **Medium size class and above:** Form and summary card remain single-column, capped at a consistent platform-wide content width and horizontally centered; the navigation list becomes a two-column grid of entries rather than a single stacked column, with no other structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Pro Account Lookup input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Pro Account Lookup input | Blur (empty) | Triggers field validation via FEAT-19.SPEC-004 | Error state on field | "A Pro account identifier is required." below field |
| Reason / Ticket Reference input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Reason / Ticket Reference input | Blur (empty or whitespace-only) | Triggers field validation via FEAT-19.SPEC-004 | Error state on field | "A reason or ticket reference is required to open a support session." below field |
| Open Support Session button | Tap | 1. Validate both fields via FEAT-19.SPEC-004. 2. If valid, resolve the Pro Account lookup. 3. If a match is found, end any prior session (FEAT-19.SPEC-004) and open a new one, triggering FEAT-19.SPEC-002 (support_view_opened). 4. If no match, show the No Match state. | Button shows loading state during resolution | Success: screen transitions to Session Active state. No match: plain message shown, form fields retained. |
| Open Support Session button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |
| Navigation entry (e.g., Services, Schedule & Bookings) | Tap | Navigate to the corresponding feature's Support-facing read-only screen, carrying the current session's Pro Account reference | Screen transitions to the destination spec | Standard navigation transition |
| Navigation entry -- disputed booking timeline (reached via Schedule & Bookings or Client Records) | Tap | Navigate to FEAT-16.SPEC-001 (Booking Activity Timeline); this view triggers FEAT-19.SPEC-002 a second time for this specific timeline view | Screen transitions to FEAT-16.SPEC-001 | Standard navigation transition |
| Support Access Log entry | Tap | Navigate to FEAT-19.SPEC-003, scoped to the current session's Pro Account | Screen transitions to FEAT-19.SPEC-003 | Standard navigation transition |
| End Session action | Tap | Ends the current session (FEAT-19.SPEC-004) | Screen returns to the Lookup state, fields empty | Screen transitions back to the default lookup form |

### Accessibility Notes

- **Focus order (Lookup state):** Pro Account Lookup input -> Reason/Ticket Reference input -> Open Support Session button.
- **Focus order (Session Active state):** End Session action -> Pro Account summary card -> navigation entries in the order listed -> Support Access Log entry.
- **Validation announcements:** When a field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Session transition announcement:** On a successful lookup, the transition to the session hub is announced ("Support session opened for {display_name}"); on a no-match result, the message "No Pro account matches that lookup" is announced.
- **Keyboard alternatives:** Every action on this screen (including all navigation entries and End Session) is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Lookup (default) | Empty two-field form, Open Support Session button enabled | Screen first opens, or a session was just ended | Support submits a lookup |
| Resolving | Open Support Session button shows a brief loading indicator | Support taps Open Support Session with both fields valid | Lookup resolves (match or no match) -- feature-overview.md's Non-Functional Notes state this view "loads instantly" given its small per-account dataset, so this state is momentary |
| No Match | Plain message "No Pro account matches that lookup." shown below the retained form fields | Lookup resolves to zero matching Pro Accounts | Support edits the lookup field and resubmits |
| Session Active | Pro Account summary card and navigation list rendered | Lookup resolves to exactly one Pro Account | Support taps End Session, or submits a new lookup (which ends this session automatically) |
| Offline/Degraded | N/A -- feature-overview.md's Non-Goals state every state beyond the plain no-match message is explicitly N/A for this feature; it is "used only in a connected context" with no offline exposure defined anywhere in its Stage 2 source | -- | -- |

## Validation Rules

Validation governed by FEAT-19.SPEC-004 (Support Session Scope & Access Rules). See that spec for the Pro Account Lookup and Reason/Ticket Reference field rules, and for the one-account-at-a-time and help-request-precondition rules this screen enforces on open.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Services entry tap | FEAT-01.SPEC-003 (Edit Service, Support read-only) | FEAT-01 |
| Schedule & Bookings entry tap | FEAT-12.SPEC-001 / FEAT-12.SPEC-002 / FEAT-12.SPEC-003 (gated by FEAT-12.SPEC-008) | FEAT-12 |
| Disputed booking timeline tap | FEAT-16.SPEC-001 (Booking Activity Timeline) | FEAT-16 |
| Client Records entry tap | FEAT-13 (all specs, private note excluded by FEAT-13.SPEC-005) | FEAT-13 |
| Setup Progress entry tap | FEAT-15.SPEC-004 | FEAT-15 |
| Time Blocks entry tap | FEAT-17.SPEC-002 (Manage Time Blocks, view-only) | FEAT-17 |
| Billing & Subscription entry tap | FEAT-18.SPEC-002 (Billing & Subscription Management Screen) | FEAT-18 |
| Payout Status entry tap | FEAT-28.SPEC-002 (Payout Status & Money Dashboard) | FEAT-28 |
| Booking Page Preview entry tap | FEAT-05.SPEC-001 through FEAT-05.SPEC-005 | FEAT-05 |
| Messages entry tap | FEAT-08's message and notification screens | FEAT-08 |
| Support Access Log entry tap | FEAT-19.SPEC-003 (Support Access Log) | -- |
| End Session tap | This screen, Lookup state | -- |

## Data Model

**Creates:** None -- a support session is an ephemeral scoping context (per FEAT-19.SPEC-004's Governed Entity), not a persisted entity in the Domain Entity Inventory; it exists only as this screen's own state for the duration of the visit.
**Reads:** Pro Account -- display_name and status, to resolve the lookup and populate the session hub's summary card; the reference is carried into every hand-off spec listed in Navigation Out, each of which reads its own additional fields under its own Access rules.
**Updates:** None -- this screen never writes to the Pro Account or any entity it hands into (SC-05, structural read-only).
**Deletes:** None.

## Business Rules

- XBR-24: Support access is read-only, one account at a time, used only after a Pro's help request, never shows private client notes, bank or identity details or sign-in codes, and every view is logged in the Pro's visible account activity.
- FEAT-19.SPEC-004 governs the structural no-write rule, one-account-at-a-time scoping (opening a new lookup ends the prior session before the new one opens), and the help-request precondition -- this screen enforces those rules but does not restate them.
- FEAT-19.SPEC-002 is triggered on every session open and on every disputed-timeline view reached from this session -- this screen's own success path never waits for that logging to complete (consistent with FEAT-19.SPEC-002's non-blocking design).
- No edit, save, refund, or sign-in-as control exists anywhere in this screen's session hub or in any screen it hands into for Support (SC-05) -- this is a structural absence of capability, not a permission check.

## Edge Cases

- **Support submits a lookup that matches no Pro (mistyped identifier or a closed account)** -- Shows the plain "No Pro account matches that lookup." message; no session opens; a Closed Pro Account (per XBR-20) is treated identically to a non-existent one, since a closed account no longer has an active surface for Support to view.
- **Support looks up a second Pro while a session is already open** -- The prior session ends automatically before the new one opens (FEAT-19.SPEC-004); Support never sees two sessions at once, and the informational banner in the Lookup state's Layout warns of this before submission.
- **Support taps Open Support Session twice rapidly** -- Second tap is ignored while the first resolution is in progress (button in loading state).
- **The looked-up Pro's account status changes while a session is open (e.g., FEAT-29 transitions it to Paused or Closing)** -- The summary card reflects the current status the next time the session hub is opened or refreshed; this screen is a snapshot per view, not live-updating, and since Support never writes to the account, no conflict exists to resolve.
- **Support navigates directly to a hand-off screen's own address without an active session** -- Refused by that screen's own authorization gate (e.g., FEAT-12.SPEC-008), consistent with XBR-24's "used only after" condition; this spec's own Access and Visibility governs only the lookup and hub, and relies on each hand-off spec's own Access rules for its own surface.
- **Support's operator access itself expires while a session is open** -- The session ends immediately per the Access and Visibility table's Expired session row; nothing is lost because this screen holds no unsaved input beyond the two lookup fields, which are simply discarded.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-19.SPEC-002 (Support View Logging) | Triggers (outbound) | Session open, and each disputed-timeline view reached from this session, trigger the logging automation |
| FEAT-19.SPEC-003 (Support Access Log) | Navigation (outbound) | The session hub's log entry navigates here, scoped to the current session's Pro Account |
| FEAT-19.SPEC-004 (Support Session Scope & Access Rules) | References (inbound) | Governs the lookup form's field validation, the one-account-at-a-time rule, and the structural no-write rule this screen enforces |
| FEAT-01.SPEC-003, FEAT-12.SPEC-001/002/003/008, FEAT-13 (all specs), FEAT-15.SPEC-004, FEAT-16.SPEC-001, FEAT-17.SPEC-002, FEAT-18.SPEC-002, FEAT-28.SPEC-002, FEAT-05.SPEC-001-005, FEAT-08's message screens | Navigation (outbound) | Every read-only surface this session hands into; each owns its own rendering and Access rules |
| FEAT-27 (Pro Profile & Booking Page Settings) | References (inbound, external trigger) | The Pro's "send a help request" action is the real-world prompt for Support to open this screen; FEAT-27 carries no direct link into it |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| support_lookup_submitted | reason/ticket reference present (boolean) | Support taps Open Support Session with both fields valid | N/A -- no success-metrics.md metric is connected to FEAT-19; retained for operational observability of how often Support opens the tool relative to help-request volume |
| support_lookup_no_match | -- | The lookup resolves to zero matching Pro Accounts | N/A -- no connected success-metrics.md metric; retained to observe how often Support mistypes or looks up a closed account, informing whether the lookup field needs clearer guidance |
| support_session_opened | Pro Account reference | A lookup resolves to exactly one Pro Account and the session hub renders | N/A -- no connected success-metrics.md metric; retained as the operational counterpart to FEAT-19.SPEC-002's logging, confirming every opened session is also recorded |
| support_session_ended | duration, ended via (explicit End Session / superseded by new lookup / operator access expired) | The session hub is closed by any of the three listed causes | N/A -- no connected success-metrics.md metric; retained to observe typical support-session duration for operational tuning |

## Acceptance Criteria

**FEAT-19.SPEC-001-AC-01:** Given Support is on the lookup form, when they leave the Pro Account Lookup field empty and move to the next field, then the field shows an error state with "A Pro account identifier is required."

**FEAT-19.SPEC-001-AC-02:** Given Support is on the lookup form, when they leave the Reason/Ticket Reference field empty and move away, then the field shows an error state with "A reason or ticket reference is required to open a support session."

**FEAT-19.SPEC-001-AC-03:** Given Support enters a valid Pro Account identifier and a reason, when they tap Open Support Session and the lookup matches exactly one Pro Account, then the screen transitions to the Session Active state showing that Pro's display_name and status.

**FEAT-19.SPEC-001-AC-04:** Given Support enters a mistyped identifier, when they tap Open Support Session and no Pro Account matches, then the plain message "No Pro account matches that lookup." appears and no session opens.

**FEAT-19.SPEC-001-AC-05:** Given Support enters the booking_link_name of a Pro Account whose status is Closed, when they submit the lookup, then the result is the No Match state, identical to a non-existent account.

**FEAT-19.SPEC-001-AC-06:** Given Support has an active session with one Pro, when they submit a fresh lookup for a different Pro, then the prior session ends automatically and the new session opens, without ever showing both at once.

**FEAT-19.SPEC-001-AC-07:** Given Support taps Open Support Session, when the tap registers a second time before resolution completes, then the second tap has no effect and the button remains in its loading state.

**FEAT-19.SPEC-001-AC-08:** Given Support is viewing the session hub, when they tap the Services entry, then they are navigated to FEAT-01.SPEC-003's Support-facing read-only rendering for that Pro's services.

**FEAT-19.SPEC-001-AC-09:** Given Support is viewing a disputed booking within the session hub's hand-off, when they open its timeline, then FEAT-19.SPEC-002 is triggered a second time to log that specific view.

**FEAT-19.SPEC-001-AC-10:** Given Support is viewing the session hub, when they tap the Support Access Log entry, then they are navigated to FEAT-19.SPEC-003 scoped to the current session's Pro Account.

**FEAT-19.SPEC-001-AC-11:** Given Support is viewing the session hub, when they look for any edit, save, refund, or sign-in-as control anywhere on the screen or its navigation targets, then none exists.

**FEAT-19.SPEC-001-AC-12:** Given Support taps End Session, when the action completes, then the screen returns to the empty Lookup state.

**FEAT-19.SPEC-001-AC-13:** Given Support's operator access expires while a session is open, when the expiry occurs, then the session ends immediately with no data loss, since no unsaved input exists beyond the discarded lookup fields.

**FEAT-19.SPEC-001-AC-14:** Given the Pro (Talia) or a Client (Riley) attempts to reach this screen, when the attempt is made through any product navigation, then no path exists anywhere in the product that leads them here.

**FEAT-19.SPEC-001-AC-15:** Given a looked-up Pro's account status changes to Paused while Support's session is open, when Support reopens or refreshes the session hub, then the summary card reflects the current status.

**FEAT-19.SPEC-001-AC-16:** Given Support submits a valid lookup, when the session opens, then FEAT-19.SPEC-002 is triggered to log the support_view_opened event.

**FEAT-19.SPEC-001-AC-17:** Given Support attempts to navigate directly to a hand-off screen's own address without an active session, when the attempt reaches that screen, then it is refused by that screen's own authorization gate (e.g., FEAT-12.SPEC-008).

**FEAT-19.SPEC-001-AC-18:** Given Support is on the lookup form with a session already open, when they view the form before submitting a new lookup, then the informational banner "Opening a new lookup will end your current session with {display_name}." is shown.

**FEAT-19.SPEC-001-AC-19:** Given Support is viewing the session hub, when they tap the Booking Page Preview entry, then they are navigated into the Pro's public booking-page screens (FEAT-05.SPEC-001 through FEAT-05.SPEC-005) in Support's read-only mode.

**FEAT-19.SPEC-001-AC-20:** Given Support is viewing the session hub, when they tap the Payout Status entry, then they are navigated to FEAT-28.SPEC-002 showing status and the money list only, never bank or identity details.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 10 | 10 |
| States | 5 (lookup, resolving, no match, session active, offline/degraded N/A) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |

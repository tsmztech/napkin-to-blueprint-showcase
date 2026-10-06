---
document_type: spec
spec_type: screen
spec_id: FEAT-19.SPEC-003
spec_name: Support Access Log
spec_slug: support-access-log
parent_feature: FEAT-19
parent_feature_name: Platform Support Read-Only Access
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

# Screen Spec: Support Access Log

## Overview

**Name:** Support Access Log
**ID:** FEAT-19.SPEC-003
**Type:** Screen
**Purpose:** Renders the Pro's (and Support's own) account-level list of every past support view -- who, when, and the reason/ticket reference -- distinct from FEAT-16.SPEC-001's per-booking timeline.
**Parent Feature:** FEAT-19 -- Platform Support Read-Only Access

## Scope and Non-Goals

**In Scope:**
- Rendering the full, time-ordered list of support-view entries for one Pro Account
- The Pro's own entry point into this screen from her account settings
- Support's entry point into this screen from within an open session, scoped to that same account
- Firing support_view_log_viewed_by_pro when the Pro is the viewer

**Non-Goals:**
- Writing the entries this screen displays -- owned by FEAT-19.SPEC-002 (Support View Logging), which assembles them, and FEAT-16.SPEC-002 (Activity Event Recording), which is their sole writer.
- The per-booking activity timeline -- owned by FEAT-16.SPEC-001 (Booking Activity Timeline); this screen is the separate account-level "who looked, when" log, not a booking's own history.
- Any edit, delete, or dismiss control on any entry -- excluded per FEAT-16.SPEC-005's append-only, immutable guarantee, which this entity inherits in full; no such control exists anywhere for any role, including the Pro.
- Retention or purge policy for these entries -- excluded per feature-overview.md's Entity-Lifecycle Coverage Matrix, which states this feature introduces no separate retention or purge policy beyond FEAT-16.SPEC-002/FEAT-16.SPEC-005's own lifecycle for Activity Event.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-27 (Pro Profile & Booking Page Settings) | The Pro navigates to "Support Access Log" from her own account settings | Her own Pro Account reference (implicit -- she can only ever view her own log) |
| FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) | Support taps the Support Access Log entry from within an open session | The current session's Pro Account reference, scoping the log to that one account only |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full list, for her own account only | None -- this is a read-only log with no action controls for any role | -- |
| Platform Operator (Support) | Full list, for the one Pro Account currently under an active session only | None | Attempting to view this log for any account other than the one under an active session is refused -- no path renders it, since this screen is only ever reached scoped to the current session's account |
| The Client (Riley) | No | No | No navigation path anywhere in the product reaches this screen for a Client; it is never linked from any client-facing surface |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen when arriving via the Pro's own settings path (XBR-29); a plain "this page isn't available" experience when arriving via the Support path, since no operator sign-in screen is in scope for this feature |
| Expired session | No | No | The Pro's expired session redirects to sign-in per XBR-29, with no unsaved input to preserve (this is a read-only screen); Support's expired operator access ends the entire support session per FEAT-19.SPEC-004, closing this screen along with it |

## Layout and Content

**Header:** Screen title "Support Access Log" with a back action -- returns the Pro to FEAT-27's account settings, or returns Support to FEAT-19.SPEC-001's session hub, depending on entry point.

**Body:** A single-column, time-ordered list of entries, most recent first. Each entry shows:
- The reviewer label -- always "Chairtime Support," since Platform Operator (Support) is this product's only reviewer role
- The date and time of the view
- The reason/ticket reference recorded for that view
- When the entry represents a disputed-booking-timeline view, a reference to which booking's timeline was viewed, alongside the entry's other details

No entry ever includes an edit, delete, or dismiss control (FEAT-16.SPEC-005).

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Single-column list, full width; each entry's date/time, reason/ticket reference, and (where applicable) booking reference stack vertically within the entry.
- **Medium size class and above:** The list remains single-column, capped at a consistent platform-wide content width and horizontally centered; each entry's date/time, reason/ticket reference, and booking reference lay out in a single row rather than stacking, with no other structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back action | Tap | Navigate to FEAT-27 (Pro entry) or FEAT-19.SPEC-001 (Support entry) | Screen closes | Standard navigation transition |
| Entry with a referenced booking | Tap | Navigate to FEAT-16.SPEC-001 (Booking Activity Timeline) for that booking | Screen transitions to FEAT-16.SPEC-001 | Standard navigation transition |
| Entry with no referenced booking (a plain session-open entry) | Tap | No action -- display-only | None | None |
| List (more entries than fit on screen) | Scroll | Loads further entries | Additional entries appear below | Standard scroll behavior |

### Accessibility Notes

- **Focus order:** Back action -> each list entry in displayed (most-recent-first) order.
- **Dynamic content announcements:** When the list finishes its initial load, the entry count is announced (e.g., "12 support views recorded"); when further entries load on scroll, no additional announcement interrupts reading.
- **Keyboard alternatives:** Scrolling and opening a referenced booking's timeline are both reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Populated (default) | Full time-ordered list of entries | The account has at least one recorded support view | -- (remains the resting state while entries exist) |
| Empty | Plain message "No support views recorded for this account yet." in place of the list | The account has zero recorded support views (a Pro who has never had a help request looked into) | An entry is recorded and the screen is next opened or refreshed |
| Loading | N/A -- feature-overview.md's Non-Goals state this feature's per-account dataset is small and "loads instantly"; no dedicated loading state is defined beyond an instantaneous render | -- | -- |
| Error | N/A -- feature-overview.md's Non-Goals state every state beyond the plain no-match message (owned by FEAT-19.SPEC-001) is explicitly N/A for this feature; this read-only log has no failure mode distinct from the Empty state already covering the no-data condition | -- | -- |
| Offline/Degraded | N/A -- per the same Non-Goals statement, this feature is "used only in a connected context" with no offline exposure defined anywhere in its Stage 2 source | -- | -- |

## Validation Rules

N/A -- this screen has no user-entered fields; it is a pure read-only list with nothing to validate.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back action (Pro entry) | FEAT-27 (account settings) | FEAT-27 |
| Back action (Support entry) | FEAT-19.SPEC-001 (session hub) | -- |
| Entry with a referenced booking tap | FEAT-16.SPEC-001 (Booking Activity Timeline) | FEAT-16 |

## Data Model

**Creates:** None -- this screen only reads existing entries.
**Reads:** Activity Event (support-view entries only) -- actor, time, reason/ticket reference, and (when applicable) the referenced booking, filtered to entries belonging to the current Pro Account; assembled by FEAT-19.SPEC-002, written by FEAT-16.SPEC-002 (per the Feature Dependency Map's Activity Event lifecycle).
**Updates:** None -- consistent with FEAT-16.SPEC-005's append-only, immutable guarantee; no role, including the Pro, can alter an entry.
**Deletes:** None.

## Business Rules

- XBR-24: every support view is logged in the Pro's visible account activity -- this screen is the Pro-facing (and Support-facing) surface that fulfills that visibility obligation.
- FEAT-16.SPEC-005 governs the underlying Activity Event's append-only immutability and its private-notes exclusion for Support's own rendering elsewhere; a support-view entry itself never contains a client's private note, so no exclusion rendering applies within this screen's own entries.
- FEAT-19.SPEC-004 governs Support's access scope: this screen is reachable for Support only while a session is Active for the account being viewed, never for any other account.
- This is an account-level log, distinct from FEAT-16.SPEC-001's per-booking timeline -- a support-view entry belongs to the Pro Account even when it records a view of a specific booking's timeline (feature-overview.md's Entity-Lifecycle Coverage Matrix).

## Edge Cases

- **Support attempts to view this log for a Pro account with no active session** -- Refused; this screen has no independent entry point of its own outside an active session (per FEAT-19.SPEC-004), so no such attempt can reach it.
- **The Pro views her own log while a Support session for her account happens to be open at that same moment** -- The list reflects whatever entries existed at the time this screen was opened; it is a snapshot per view, not live-updating, so an entry logged moments earlier may not appear until the Pro reopens or refreshes the screen.
- **A Pro Account accumulates a very large number of entries over years of use** -- feature-overview.md's Non-Functional Notes describe this dataset as small and bounded (it grows only with actual help requests, not ordinary use); the list scrolls to show further entries rather than requiring a dedicated pagination control.
- **Multiple entries are logged within the same session (a session-open entry plus one or more timeline-view entries)** -- Each appears as its own separate row, in the order they were recorded, with no merging.
- **A support-view entry references a booking that has since been deleted or archived** -- Bookings are never hard-deleted (per the Feature Dependency Map's Booking lifecycle: "kept for the life of the account"), so this scenario does not occur; the referenced booking's timeline remains reachable for the life of the account.
- **The account has zero recorded support views** -- The Empty state's plain message is shown; this is a normal, non-error condition for a Pro who has never had a help request looked into.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-19.SPEC-002 (Support View Logging) | References (inbound) | Assembles the events this screen ultimately renders |
| FEAT-16.SPEC-002 (Activity Event Recording) | References (inbound) | The sole writer of the entries this screen reads |
| FEAT-16.SPEC-005 (Activity Record Immutability & Visibility Rules) | References (inbound) | Governs the append-only, immutable guarantee these entries carry |
| FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) | Navigation (inbound) | Support arrives here from the open session's hub |
| FEAT-27 (Pro Profile & Booking Page Settings) | Navigation (inbound) | The Pro arrives here from her own account settings |
| FEAT-16.SPEC-001 (Booking Activity Timeline) | Navigation (outbound) | Tapping an entry with a referenced booking opens that booking's own timeline |
| FEAT-19.SPEC-004 (Support Session Scope & Access Rules) | References (inbound) | Governs Support's access scope to this screen |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| support_view_log_viewed_by_pro | Pro Account reference | The Pro opens this screen for her own account | N/A -- no success-metrics.md metric is connected to FEAT-19; retained per feature-overview.md's Key Capabilities, since the Pro's ability to see who looked and when is itself the trust mechanism the audit-added capability introduced, and observing how often she checks it informs whether that trust mechanism is actually being used |
| support_view_log_viewed_by_support | Pro Account reference | Support opens this screen from within an active session | N/A -- no connected success-metrics.md metric; retained for operational observability of how often Support reviews the log during their own session |

## Acceptance Criteria

**FEAT-19.SPEC-003-AC-01:** Given Talia opens her Support Access Log from account settings, when the screen loads and at least one support view has been recorded, then she sees the full, time-ordered list of entries, most recent first.

**FEAT-19.SPEC-003-AC-02:** Given Talia's account has never had a support view, when she opens this screen, then she sees the plain message "No support views recorded for this account yet."

**FEAT-19.SPEC-003-AC-03:** Given Support has an active session with Talia's account, when they tap the Support Access Log entry, then they see the list scoped to that same account only.

**FEAT-19.SPEC-003-AC-04:** Given an entry represents a disputed-booking-timeline view, when Talia or Support taps it, then they are navigated to FEAT-16.SPEC-001 for that specific booking.

**FEAT-19.SPEC-003-AC-05:** Given an entry represents a plain session-open view with no referenced booking, when Talia or Support taps it, then nothing happens -- the entry is display-only.

**FEAT-19.SPEC-003-AC-06:** Given Talia opens this screen, when the reviewer label is examined for any entry, then it reads "Chairtime Support," never a specific individual's name.

**FEAT-19.SPEC-003-AC-07:** Given Talia or Support views any entry on this screen, when they look for an edit, delete, or dismiss control, then none exists anywhere on the screen.

**FEAT-19.SPEC-003-AC-08:** Given Talia opens her Support Access Log, when the screen finishes loading, then the support_view_log_viewed_by_pro event fires with her Pro Account reference.

**FEAT-19.SPEC-003-AC-09:** Given Support opens the log from within an active session, when the screen loads, then the support_view_log_viewed_by_support event fires, and support_view_log_viewed_by_pro does not.

**FEAT-19.SPEC-003-AC-10:** Given a Pro Account has accumulated more entries than fit on one screen, when Talia scrolls, then further entries load below the visible list.

**FEAT-19.SPEC-003-AC-11:** Given one support session produced both a session-open entry and a disputed-timeline-view entry, when Talia views her log, then both appear as separate rows in the order they were recorded.

**FEAT-19.SPEC-003-AC-12:** Given a support view was logged moments before Talia opens this screen, when the screen renders, then it reflects the entries that existed at the moment the screen was opened, without live-updating thereafter.

**FEAT-19.SPEC-003-AC-13:** Given a Client (Riley) attempts to reach this screen, when the attempt is made through any product navigation, then no path exists anywhere in the product that leads them here.

**FEAT-19.SPEC-003-AC-14:** Given Support attempts to view this log for a Pro account with no currently active session, when the attempt is made, then it is refused, since this screen has no independent entry point outside an active session.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 4 | 4 |
| States | 5 (populated, empty, loading N/A, error N/A, offline/degraded N/A) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |

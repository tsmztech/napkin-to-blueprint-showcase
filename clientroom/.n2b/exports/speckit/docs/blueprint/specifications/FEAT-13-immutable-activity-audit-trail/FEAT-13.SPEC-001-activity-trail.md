---
document_type: spec
spec_type: screen
spec_id: FEAT-13.SPEC-001
spec_name: Activity Trail
spec_slug: activity-trail
parent_feature: FEAT-13
parent_feature_name: Immutable Activity & Audit Trail
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 15
---

# Screen Spec: Activity Trail

## Overview

**Name:** Activity Trail
**ID:** FEAT-13.SPEC-001
**Type:** Screen
**Purpose:** Nadia browses the full, permanent chronological history of everything recorded against a project; Dana views the identical trail read-only while a support session is open.
**Parent Feature:** FEAT-13 -- Immutable Activity & Audit Trail

## Scope and Non-Goals

**In Scope:**
- A reverse-chronological list of every Activity Log Entry for one project
- Opening an entry's affected record from the trail
- Launching the printable/shareable copy of the whole trail or a single entry
- Empty, loading, error, and offline/degraded states
- Dana's read-only view of the identical trail during an open support session, including that session's own entries once written

**Non-Goals:**
- Editing, retracting, or annotating any entry, by any role -- excluded per BRIEF.md's Constraints and governed by FEAT-13.SPEC-004: no entry can ever be edited or deleted once written; this is the feature's defining guarantee, not a missing capability.
- Producing the printable, shareable copy itself -- handled by FEAT-13.SPEC-002 (Printable Record Copy); this screen only launches it.
- Any cross-event trail view for Owen or Priya -- excluded per the Access Matrix (user-persona.md): client contacts have "None" for Activity & Audit Trail; they see only the outcome of their own actions within their own scoped views (e.g., FEAT-03, FEAT-08), never this screen.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01 (Client & Project Management) project view, activity area | Nadia opens a project's activity tab | Project reference; trail loads scoped to that project |
| FEAT-31 (Operator Support Access) support session opened | Dana's open support session grants read-only reach into the freelancer's account | Freelancer account and project context, read-only |
| FEAT-13.SPEC-002 (Printable Record Copy) | Nadia returns after producing or viewing a printable copy | Project reference; trail re-loads at its current state |
| FEAT-31.SPEC-007 (Support Session Opened Notice) | Nadia taps the notice email's "View activity trail" CTA | Freelancer account reference; the trail shows her account's support-session entries |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen, her own projects only | Open any entry, navigate to the affected record, launch the printable copy (full trail or a single entry) | -- |
| Owen (Client Primary Contact) | No | No | The capability is not shown at all -- no tab, link, or navigation path from his portal view reaches this screen; he sees only the outcome of his own actions within his own scoped views (e.g., his acceptance confirmation in FEAT-03, his approval confirmation in FEAT-08). |
| Priya (Client Reviewer Contact) | No | No | Same as Owen -- not shown at all; she sees only the outcome of her own comment activity within her own scoped views. |
| Dana (Support Operator) | Full screen, read-only, only inside an open support session on the freelancer's account (FEAT-31) | None -- no entry is actionable, no Share control is rendered | Outside an open support session, this screen is unreachable, identical to the unauthenticated experience below. |
| Unauthenticated | No | No | Redirected to the sign-in screen; no project context is retained. |
| Expired session | No (trail hidden behind a re-authentication prompt) | No | Dialog "Your session has expired. Sign in to continue." The previously loaded trail (for Nadia) is held in memory and re-displayed once she signs back in -- there is no unsaved input to lose on this read-only screen. |

## Layout and Content

**Header:** A breadcrumb reading "{Project Name} > Activity," the screen title "Activity Trail," and, for Nadia only, a "Share" action button at the top right that opens FEAT-13.SPEC-002 scoped to the whole trail.

**Body:** A single reverse-chronological list of entries, most recent first, with no depth limit within the project's lifetime. Each row shows:
- A plain-language event description derived from `event_type` (e.g., "Milestone 'Homepage design' approved," "Invoice #INV-014 sent," "Deliverable 'Hero video v2' uploaded")
- The actor's name -- "You" for Nadia's own actions, the client contact's name for client-side events, "Dana (operator)" for support-session entries, or "Automatic" for system-timed events such as reminders
- The exact date and time, shown in the viewer's own time zone
- A link to the affected record, where one exists, to view its current detail (e.g., the milestone, invoice, or deliverable)
- For Nadia only, a "Share this record" control that opens FEAT-13.SPEC-002 pre-scoped to that single entry

A lightweight loading indicator (a slim progress bar directly under the header) appears while the current page of entries is still being fetched, in place of a blank screen, for projects with long histories.

### Responsive Behavior
- **Compact breakpoint:** Single-column list, full width. Within each row, the event description, actor, and timestamp stack vertically; the affected-record link and the per-entry Share control sit below as separate tap targets.
- **Medium size class and above:** The list stays single-column, but each row lays its description, actor, timestamp, and controls out along one line. The list is capped at a consistent platform-wide content width and horizontally centered; no structural change beyond width capping.
- **Header Share button:** Remains visible in the header at every size class -- never collapsed into an overflow menu.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Entry row | Tap | Navigates to the affected record's own detail spec (e.g., FEAT-08 milestone detail, FEAT-09 invoice detail, FEAT-06 deliverable detail) | Row briefly highlights before navigation | Standard transition to the destination screen |
| Entry row's affected-record link | Tap | Same destination as the row tap -- an explicit link target | -- | -- |
| Per-entry "Share this record" control (Nadia only) | Tap | Navigates to FEAT-13.SPEC-002, pre-scoped to this single entry | Screen transitions | FEAT-13.SPEC-002 opens showing only this entry |
| Header "Share" button (Nadia only) | Tap | Navigates to FEAT-13.SPEC-002, scoped to the full project trail | Screen transitions | FEAT-13.SPEC-002 opens showing every entry for the project |
| Entry list | Scroll | Loads the next page of older entries | List grows | A lightweight loading indicator appears at the list's end while more entries fetch |
| Breadcrumb "{Project Name}" | Tap | Navigates to FEAT-01 (project view) | Screen closes | Standard transition back to the project |

### Accessibility Notes

- **Focus order:** Breadcrumb -> Share button (Nadia only) -> entry list, row by row: description -> actor -> timestamp -> affected-record link -> per-entry Share control (Nadia only).
- **Dynamic-change announcements:** The loading indicator's appearance, an error banner's appearance, and the arrival of newly fetched entries (both at scroll end and when a fresh entry appears live at the top) are announced to assistive technology.
- **Keyboard alternatives:** Every action on this screen (entry navigation, both Share controls, scroll-triggered loading) is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty | Explanatory message that entries will appear as milestones progress, in place of a list | The project has zero Activity Log Entries | The first entry is written for this project |
| Loading | Slim progress indicator under the header; no entries rendered yet | Screen opens, or the viewer switches to a different project's trail | The first page of entries returns |
| Populated | Full reverse-chronological list as described in Layout and Content | Entries load successfully | Viewer navigates away |
| Error | Error banner at the top of the list ("This trail couldn't be fully refreshed. Showing the most recently loaded activity."); any previously fetched entries remain visible underneath -- nothing is ever removed by a failed request | A refresh or scroll-triggered fetch fails | A later fetch succeeds, or the viewer navigates away |
| Offline/Degraded | The most recently loaded trail remains visible, read-only, under a banner "You're offline -- showing the last loaded activity."; no new entries load and affected-record links are disabled while offline | Connectivity is lost while the screen is open, or the screen is opened while already offline | Connectivity returns and the trail re-fetches |

## Validation Rules

**Option B -- Inline (no user input on this screen):**
This screen accepts no field input; every action is navigation. It reads, and never writes, Activity Log Entry data, which is governed entirely by FEAT-13.SPEC-004 (Entry Immutability, Content & Attribution Rules).

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Entry row or affected-record link tap | The affected record's own detail spec (varies by `event_type`: e.g., FEAT-08 Milestone Detail, FEAT-09.SPEC-002 Invoice Detail, FEAT-06 Deliverable Detail) | Varies |
| Per-entry Share tap | FEAT-13.SPEC-002 (Printable Record Copy), single-entry scope | -- |
| Header Share tap | FEAT-13.SPEC-002 (Printable Record Copy), full-trail scope | -- |
| Breadcrumb tap | FEAT-01 project view | FEAT-01 |
| Nadia points to a disputed entry, then records the outcome | Mark invoice refunded / project cancelled | FEAT-25 (Refund & Cancelled Project Handling) |

## Data Model

**Creates:** None -- this screen never writes an Activity Log Entry.
**Reads:** Activity Log Entry -- `event_type`, `actor`, `occurred_at`, `affected_record`, and `project`, for the full list of entries belonging to the current project, in reverse-chronological order.
**Updates:** None.
**Deletes:** None.

## Business Rules

- XBR-04 and XBR-05: every entry shown here was written append-only by FEAT-13.SPEC-003 and can never have been edited or deleted afterward -- what this screen displays is guaranteed unaltered, per FEAT-13.SPEC-004.
- Visibility of this screen, and of every entry on it, is governed entirely by FEAT-13.SPEC-005 (Activity Trail Access & Visibility Rules); this screen enforces no separate access logic of its own.
- XBR-29: Dana's support-session view of this screen includes that session's own opened/closed entries once they are written, so Nadia's later view of the same trail shows exactly what Dana saw during her session.

## Edge Cases

- **Nadia opens the trail for a project with an unusually long history (hundreds of entries over a long engagement)** -- The list loads in pages as she scrolls; the lightweight loading indicator appears at the list's end rather than blocking the whole screen.
- **An entry's affected record has since changed state (e.g., a reopened milestone, a corrected invoice)** -- The entry's own text and timestamp never change; tapping its link navigates to the affected record's current state, which may show a later status than the entry describes. This is expected: the entry is a historical record, not a live mirror of the affected record.
- **Dana's support session closes while she is viewing the trail** -- The screen becomes immediately unreachable, matching the "outside an open session" unauthorized experience in Access and Visibility.
- **A new entry is written while Nadia (or Dana, mid-session) has the trail open** -- The trail is a live view: the new entry appears at the top of the list without a manual refresh. There is no conflict to resolve, since entries are append-only (per the dependency map's Contention note for Activity Log Entry: "None -- entries are append-only and never edited or deleted by anyone; concurrent writers only append independent entries").
- **A brand-new project's trail is opened before its first entry is written** -- The Empty state renders; it is not treated as an error.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-13.SPEC-002 (Printable Record Copy) | Navigation (outbound) | Nadia launches a shareable copy of the whole trail or a single entry |
| FEAT-13.SPEC-003 (Activity Entry Recording) | References (inbound) | Every entry rendered here was written by this automation |
| FEAT-13.SPEC-004 (Entry Immutability, Content & Attribution Rules) | References (inbound) | Governs what every entry must contain and guarantees it is never altered |
| FEAT-13.SPEC-005 (Activity Trail Access & Visibility Rules) | References (inbound) | Governs exactly who may open this screen and see its entries |
| FEAT-01 (Client & Project Management) | Navigation (inbound) | Entry point via the project's activity area |
| FEAT-31 (Operator Support Access) | Navigation (inbound) | Dana reaches this screen only from an open support session |
| FEAT-25 (Refund & Cancelled Project Handling) | Navigation (outbound) | Nadia records a dispute's outcome after pointing to a disputed entry |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| activity_trail_viewed | project reference, viewer role (Nadia / Dana), entry count shown | The screen finishes loading its first page of entries | supports success-metrics.md: "Dispute Resolution Confidence" |
| activity_entry_opened | event_type, time elapsed since occurred_at | Nadia or Dana taps an entry's affected-record link | supports success-metrics.md: "Dispute Resolution Confidence" |
| activity_trail_empty_state_shown | project reference | The Empty state renders for a brand-new project | N/A -- no success-metrics.md metric measures the empty state itself; retained because product-features.md's Signals field for this feature already names `activity_trail_viewed` as its core viewing signal, and this event distinguishes the zero-entry case in the underlying event data without claiming an additional metric |

## Acceptance Criteria

**FEAT-13.SPEC-001-AC-01:** Given Nadia opens a project with existing activity, when the screen finishes loading, then she sees every entry for that project listed most-recent-first, each with its description, actor, exact timestamp, and affected-record link.

**FEAT-13.SPEC-001-AC-02:** Given Nadia is viewing the trail, when she taps an entry's affected-record link, then she is taken to that record's own detail screen.

**FEAT-13.SPEC-001-AC-03:** Given Nadia is viewing the trail, when she taps the header "Share" button, then FEAT-13.SPEC-002 opens scoped to the full project trail.

**FEAT-13.SPEC-001-AC-04:** Given Nadia is viewing the trail, when she taps "Share this record" on a single entry, then FEAT-13.SPEC-002 opens scoped to that one entry only.

**FEAT-13.SPEC-001-AC-05:** Given Nadia scrolls to the end of the currently loaded entries in a project with a long history, when more entries exist, then the next page loads with a lightweight indicator at the list's end.

**FEAT-13.SPEC-001-AC-06:** Given Nadia taps the breadcrumb, when the tap registers, then she returns to the FEAT-01 project view.

**FEAT-13.SPEC-001-AC-07:** Given Nadia opens a brand-new project with no recorded events, when the screen loads, then she sees the Empty state explaining entries will appear as milestones progress.

**FEAT-13.SPEC-001-AC-08:** Given Nadia has a trail loaded and a refresh fails, when the failure occurs, then an error banner appears while every previously loaded entry remains visible underneath.

**FEAT-13.SPEC-001-AC-09:** Given Nadia loses connectivity while the trail is open, when connectivity drops, then the last-loaded trail remains visible read-only under an offline banner, and affected-record links are disabled until connectivity returns.

**FEAT-13.SPEC-001-AC-10:** Given Owen or Priya is signed into the client portal, when they look for any way to reach the cross-event activity trail, then no tab, link, or navigation path to this screen exists anywhere in their portal view.

**FEAT-13.SPEC-001-AC-11:** Given Dana has an open support session on Nadia's account, when she opens the activity trail, then she sees the same full trail read-only, with no Share control and no actionable entries.

**FEAT-13.SPEC-001-AC-12:** Given Dana's support session closes while she is viewing the trail, when the session ends, then the screen becomes immediately unreachable to her.

**FEAT-13.SPEC-001-AC-13:** Given Dana's support session opens and closes during a session, when Nadia later opens the same trail, then she sees the session's own opened and closed entries in the list, attributed to Dana (operator).

**FEAT-13.SPEC-001-AC-14:** Given an unauthenticated visitor attempts to reach this screen directly, when the request is made, then they are redirected to sign-in with no project context retained.

**FEAT-13.SPEC-001-AC-15:** Given Nadia's session expires while the trail is open, when she next interacts with the screen, then a "Your session has expired. Sign in to continue." dialog appears and, once she signs back in, the same trail re-displays without her having lost any input (none existed to lose).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 4 (empty, error, offline, expired-session behavior) | 4 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |

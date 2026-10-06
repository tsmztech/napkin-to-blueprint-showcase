---
document_type: spec
spec_type: screen
spec_id: FEAT-02.SPEC-003
spec_name: Proposal Detail
spec_slug: proposal-detail
parent_feature: FEAT-02
parent_feature_name: Proposal Creation & Sending
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

# Screen Spec: Proposal Detail

## Overview

**Name:** Proposal Detail
**ID:** FEAT-02.SPEC-003
**Type:** Screen
**Purpose:** Nadia (and, read-only, Dana) views a project's current proposal -- its status, history, and the actions available for that status -- and Dana's support view is limited to status only.
**Parent Feature:** FEAT-02 -- Proposal Creation & Sending

## Scope and Non-Goals

**In Scope:**
- The single current proposal's status, content summary, and available status-driven actions for a project
- The empty state when a project has no proposal yet
- Surfacing a request-changes note (FEAT-03) attached to the proposal
- Entry points into editing, sending-adjacent flows, and reuse

**Non-Goals:**
- A browsable version-history or diff view of voided proposal versions -- intentional omission per the Feature Breakdown Brief's Non-Goals: no Stage 2 depth field or journey step calls for browsing or comparing voided versions; only the current version's status is shown.
- Editing proposal content directly on this screen -- editing happens in FEAT-02.SPEC-001 (Proposal Draft Editor), reached via the Edit action.
- Accepting the proposal or recording a request-changes note -- both are Owen's actions, owned by FEAT-03 (Proposal Acceptance); this screen only displays their resulting state and the note's content.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01 (Client & Project Management), project view proposal area | Nadia opens the proposal area of a project | Project reference |
| FEAT-12 (Freelancer Financial Dashboard) | Empty-dashboard zero-state prompt toward sending a first proposal | Project reference (the freelancer's first project) |
| FEAT-28 (Global Search Across Clients & Projects) | A search result for a proposal | Proposal/project reference |
| FEAT-02.SPEC-005 (Proposal Send) | Send succeeds | Project reference, updated to Sent state |
| FEAT-02.SPEC-006 (Void & Resend) | Prior version voided and new version sent | Project reference, updated to Sent state (new version) |
| FEAT-02.SPEC-009 (Discard Draft) | Draft discarded | Project reference, updated to empty state |
| FEAT-03 (Proposal Acceptance), request-changes email | Nadia opens the change-request email | Proposal reference, with the request-changes note shown |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen: status, content summary, request-changes note, and all status-driven actions | Edit, Send, Discard, Start from Copy (Draft); Edit, Resend (Sent); none (Voided, Accepted -- view only) | -- |
| Owen (Client Primary Contact) | No | No | This screen is Nadia's own workspace view; Owen's equivalent is his own portal proposal view (FEAT-03), not this screen. |
| Priya (Client Reviewer Contact) | No | No | Same as Owen -- Priya has no access to proposal content at all, per the Access Matrix; her portal home shows only the project's stage. |
| Dana (Support Operator) | Status and history only -- no scope description, price, or request-changes note text; no send/edit/discard actions are shown or reachable | None | Reached only inside a logged, read-only support session (FEAT-31); content beyond status is not rendered, so there is nothing further to attempt. |
| Unauthenticated | No | No | Redirected to freelancer sign-in. |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." The screen reloads showing the same proposal once re-authentication succeeds. |

## Layout and Content

**Header:** Screen title "Proposal" with a back arrow (returns to the project view, FEAT-01) at the left. When a proposal exists, a status badge (Draft / Sent / Voided / Accepted) sits beside the title.

**Body (proposal exists):**
- **Status summary card:** current status, sent_at (if sent), accepted_at and accepted_by (if accepted, read from the record FEAT-03 writes)
- **Content summary:** scope description (truncated with "Show more"), price and currency, payment schedule reference summary
- **Request-changes note** (shown only when one exists, attached by FEAT-03): note text, posted date, with a label "Change requested by {Primary Contact name}"
- **Action bar:** status-driven action set (see Business Rules)

**Body (no proposal exists -- empty state):** A prompt: "This project has no proposal yet." with a primary "Draft a Proposal" button.

**Footer:** None -- actions live in the body's action bar.

### Responsive Behavior

- **Compact:** Single-column stacked cards, full width; action bar becomes a bottom-fixed bar.
- **Medium size class and above:** Content remains single-column, capped at a consistent platform-wide width and horizontally centered; action bar remains inline at the top of the body rather than fixed to the bottom.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| "Draft a Proposal" (empty state) | Tap | Navigate to FEAT-02.SPEC-001 (Proposal Draft Editor), blank | Screen closes | Animated transition to the editor |
| "Show more" on scope description | Tap | Expands the truncated text inline | Text expands | Text reflows to show full scope description |
| Edit (Draft or Sent status) | Tap | Navigate to FEAT-02.SPEC-001 in the corresponding mode | Screen closes | Animated transition to the editor |
| Send (Draft status) | Tap | Navigate to FEAT-02.SPEC-002 (Proposal Preview) | Screen closes | Animated transition to Preview |
| Discard (Draft status) | Tap | Confirmation dialog, then triggers FEAT-02.SPEC-009 (Discard Draft) | Confirmation dialog appears | Dialog: "Discard this draft? This cannot be undone." |
| Start from Copy (Draft status, when the Draft is otherwise unsent and blank-equivalent) | Tap | Navigate to FEAT-02.SPEC-004 (Reuse Proposal Picker) | Screen closes | Animated transition to the picker |
| Resend (Sent status) | Tap | Triggers FEAT-02.SPEC-007 (Proposal Resend) | Button shows brief loading state | Toast "Proposal link resent to {Primary Contact name}." |
| Request-changes note | Tap | Expands the full note text if truncated | Text expands | Text reflows |

### Accessibility Notes

- **Focus order:** Back arrow -> status badge (announced) -> content summary -> request-changes note (if present) -> action bar buttons in the order shown.
- **Status announcements:** A status change reached by navigating back to this screen after an action (send, resend, discard, void-and-resend) is announced on load, e.g., "Proposal sent."
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty | Prompt "This project has no proposal yet." with "Draft a Proposal" button | Project has no Proposal record, or its only Draft was just discarded | A Draft is created for the project |
| Draft | Status badge "Draft"; Edit, Send, Discard, and (when eligible) Start from Copy actions shown | A Draft Proposal exists for the project | The Draft is sent or discarded |
| Sent | Status badge "Sent"; sent_at shown; Edit and Resend actions shown; request-changes note shown if present | The proposal transitions to Sent (FEAT-02.SPEC-005 or FEAT-02.SPEC-006) | The proposal is accepted, or edited-and-resent again |
| Voided | Not directly displayed as a distinct screen state | N/A -- this row exists only to document the Proposal entity's status enum (FEAT-02.SPEC-010); this screen queries and always renders the project's single current (non-voided) proposal, so when FEAT-02.SPEC-006 voids a version and creates the new Sent version, the screen's next load reads and shows that new Sent version directly -- the intermediate Voided status is never itself rendered here, even momentarily | N/A -- not a reachable screen state |
| Accepted | Status badge "Accepted"; accepted_at and accepted_by shown; no edit/send/discard/resend actions shown | Owen accepts the proposal (FEAT-03) | N/A -- Accepted is terminal |
| Loading | Skeleton placeholders for the status card and content summary | Screen is opening and proposal data has not yet arrived | Data loads and one of Empty, Draft, Sent, or Accepted renders, per the proposal's current, resolved status |
| Error | Error banner: "Could not load this proposal." with a Retry button | Loading the proposal's data fails | Nadia taps Retry, or connectivity returns and a re-fetch succeeds |
| Offline/Degraded | Banner: "You're offline -- showing the last loaded version of this proposal." Content shown is the last successfully loaded snapshot; Edit/Send/Discard/Resend actions are disabled until connectivity returns | Connectivity is lost while viewing a previously loaded proposal | Connectivity returns and the screen re-fetches |

## Validation Rules

Not applicable -- this screen has no user input fields. Action eligibility (which buttons appear per status) is governed by FEAT-02.SPEC-010 (Proposal Validation & Business Rules).

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | Project view | FEAT-01 (Client & Project Management) |
| "Draft a Proposal" / Edit tap | FEAT-02.SPEC-001 (Proposal Draft Editor) | -- |
| Send tap | FEAT-02.SPEC-002 (Proposal Preview) | -- |
| Start from Copy tap | FEAT-02.SPEC-004 (Reuse Proposal Picker) | -- |
| Discard confirmed | This screen, empty state | -- |
| Resend tap | Stays on this screen (toast confirmation) | -- |

## Data Model

**Creates:** None.
**Reads:** Proposal -- status, scope_description, price, currency, sent_at, accepted_at, accepted_by, copied_from. Comment -- request-changes note text, author, and posted_at when one is attached to the proposal (FEAT-03). Payment Schedule -- summary (FEAT-04).
**Updates:** None directly -- all status transitions happen through the triggered automations (FEAT-02.SPEC-005, 006, 007, 009) and FEAT-03's acceptance flow.
**Deletes:** None directly -- Discard hands off to FEAT-02.SPEC-009.

## Business Rules

- Status-driven action set (FEAT-02.SPEC-010 governs eligibility for each): Draft -- Edit, Send, Discard, Start from Copy; Sent -- Edit, Resend; Voided/Accepted -- view only, no actions.
- "Start from Copy" from this screen behaves identically to reaching FEAT-02.SPEC-004 from any other entry point -- see the Feature Breakdown Brief's Shared UI Patterns.
- The screen always shows the project's single current (non-voided) proposal -- a project has at most one active proposal per the dependency map, so no list or version switcher is needed.
- A request-changes note (XBR-26) is display-only here -- it never alters the proposal, and Reviewer contacts' inability to see proposal content (Access Matrix) has no bearing on this screen, which Reviewers cannot reach at all.

## Edge Cases

- **Nadia opens this screen while the proposal is being voided-and-resent in another session** -- Reject-with-refresh is not applicable to a read-only screen; the screen simply re-fetches on next load or resume and shows the current state, which may differ from what was last shown. No stale-write conflict exists here because this screen performs no writes of its own.
- **The project's proposal was just discarded in another session** -- On next load or resume, the screen re-fetches and shows the Empty state rather than a stale Draft.
- **Owen accepts the proposal while Nadia is viewing this screen** -- On next load or resume, the screen shows Accepted with accepted_at/accepted_by; no live-push update is implied (this is a snapshot view, re-fetched on open/resume).
- **A request-changes note is very long** -- The note is truncated to a preview with "Show more", consistent with the scope description's truncation treatment.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-001 (Proposal Draft Editor) | Navigation (outbound) | "Draft a Proposal" (empty state) and Edit (Draft/Sent) both open the editor |
| FEAT-02.SPEC-002 (Proposal Preview) | Navigation (outbound) | Send (Draft status) opens Preview |
| FEAT-02.SPEC-004 (Reuse Proposal Picker) | Navigation (outbound) | Start from Copy opens the picker |
| FEAT-02.SPEC-005 (Proposal Send) | References (inbound) | Shows the Sent state this automation produces |
| FEAT-02.SPEC-006 (Void & Resend) | References (inbound) | Shows the updated Sent state after a void-and-resend |
| FEAT-02.SPEC-007 (Proposal Resend) | Triggers (outbound) | Resend (Sent status) triggers this automation |
| FEAT-02.SPEC-009 (Discard Draft) | Triggers (outbound) | Discard, after confirmation, triggers this automation |
| FEAT-02.SPEC-010 (Proposal Validation & Business Rules) | References (inbound) | Action-set eligibility per status |
| FEAT-03 (Proposal Acceptance) | References (inbound) | Accepted state and the request-changes note both originate here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| proposal_detail_viewed | current status | Screen finishes loading | N/A -- no Stage 2 metric measures detail-screen views; retained for baseline usage visibility |
| proposal_start_from_copy_selected | -- | Nadia taps "Start from Copy" | supports success-metrics.md: "Proposal Send Speed" (a faster path into a new draft than a blank form) |

## Acceptance Criteria

**FEAT-02.SPEC-003-AC-01:** Given Nadia opens the proposal area of a project with no proposal, when the screen loads, then it shows "This project has no proposal yet." with a "Draft a Proposal" button.

**FEAT-02.SPEC-003-AC-02:** Given Nadia taps "Draft a Proposal", then she is navigated to FEAT-02.SPEC-001 with a blank form.

**FEAT-02.SPEC-003-AC-03:** Given a project has a Draft proposal, when Nadia opens the Detail screen, then she sees the "Draft" status badge with Edit, Send, Discard, and Start from Copy actions.

**FEAT-02.SPEC-003-AC-04:** Given a project has a Sent proposal, when Nadia opens the Detail screen, then she sees the "Sent" status badge, the sent_at timestamp, and Edit and Resend actions (no Discard, no Start from Copy).

**FEAT-02.SPEC-003-AC-05:** Given a project has an Accepted proposal, when Nadia opens the Detail screen, then she sees the "Accepted" status badge with accepted_at and accepted_by, and no action buttons.

**FEAT-02.SPEC-003-AC-06:** Given a Sent proposal has a request-changes note attached, when Nadia opens the Detail screen, then the note's text, author, and posted date are shown.

**FEAT-02.SPEC-003-AC-07:** Given Nadia taps "Resend" on a Sent proposal, then FEAT-02.SPEC-007 is triggered and the toast "Proposal link resent to {Primary Contact name}." appears.

**FEAT-02.SPEC-003-AC-08:** Given Nadia taps "Discard" on a Draft, when she confirms in the dialog, then FEAT-02.SPEC-009 is triggered and the screen shows the empty state afterward.

**FEAT-02.SPEC-003-AC-09:** Given Nadia opens the change-request email from FEAT-03, when she follows the link, then she lands on this screen with the request-changes note visible.

**FEAT-02.SPEC-003-AC-10:** Given Dana (Support Operator) opens this screen inside a logged support session, when she views it, then she sees only the status and history, with no scope description, price, request-changes note text, or action buttons.

**FEAT-02.SPEC-003-AC-11:** Given the screen is loading, then skeleton placeholders appear for the status card and content summary until data arrives.

**FEAT-02.SPEC-003-AC-12:** Given loading the proposal's data fails, then the error banner "Could not load this proposal." appears with a Retry button.

**FEAT-02.SPEC-003-AC-13:** Given Nadia loses connectivity after this screen has loaded, then the banner "You're offline -- showing the last loaded version of this proposal." appears and all action buttons are disabled.

**FEAT-02.SPEC-003-AC-14:** Given the proposal was discarded in another session, when Nadia returns to this screen, then it re-fetches and shows the empty state rather than the stale Draft.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 8 (empty, draft, sent, voided, accepted, loading, error, offline) | 8 |
| Business Rules | 4 | 4 |
| Edge Cases | 4 | 4 |

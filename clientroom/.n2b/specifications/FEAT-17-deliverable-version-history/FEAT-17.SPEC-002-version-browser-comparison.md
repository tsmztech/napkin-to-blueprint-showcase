---
document_type: spec
spec_type: screen
spec_id: FEAT-17.SPEC-002
spec_name: Version Browser & Comparison
spec_slug: version-browser-comparison
parent_feature: FEAT-17
parent_feature_name: Deliverable Version History
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 17
---

# Screen Spec: Version Browser & Comparison

## Overview

**Name:** Version Browser & Comparison
**ID:** FEAT-17.SPEC-002
**Type:** Screen
**Purpose:** Any authorized viewer opens the version selector on a deliverable, browses every round by number, and opens any earlier round alongside the latest.
**Parent Feature:** FEAT-17 -- Deliverable Version History

## Scope and Non-Goals

**In Scope:**
- Listing every Deliverable Version for a deliverable, ordered by round_number, with the latest round labeled
- Opening any single round's file and metadata (round_number, uploaded_at)
- Hiding the version selector entirely when the deliverable has exactly one version
- Showing a brief loading indicator when switching between rounds on a large file
- Serving as the single shared implementation FEAT-07's deliverable view navigates into for any version-specific viewing

**Non-Goals:**
- Uploading a new version -- handled by FEAT-17.SPEC-001 (New Version Upload); this screen is read-only for the Deliverable Version entity
- Automated cross-version content diffing or a side-by-side visual comparison UI -- excluded per the Brief's Non-Goals: the Key Capability "open and compare any round by number" and Stage 2's Data Notes describe comparison as opening any round through this selector, with no diff mechanics or comparison UI described anywhere in Stage 2
- Posting, editing, or retracting comments -- owned entirely by FEAT-07 (Deliverable Review & Feedback); this screen only reads that a round can be commented on and links into FEAT-07's thread for the open round
- Editing or removing an existing version -- excluded per FEAT-17.SPEC-004: each version is immutable once uploaded, and no in-product delete path exists while the account is active

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-07 (Deliverable Review & Feedback, deliverable view) | A viewer opens an earlier version of the deliverable via the deliverable view's version selector (dependency map navigation row) | Deliverable reference; the round_number the viewer was looking at in FEAT-07, if any |
| FEAT-17.SPEC-001 (New Version Upload) | Nadia taps "View versions" after her upload completes | Deliverable reference; the newly created round's round_number, opened by default as latest |
| FEAT-31 (Operator Support Access) | Dana opens a read-only support session and navigates to a deliverable's version history | Deliverable reference, read-only session flag (no download control rendered) |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen, every version of her own deliverables | Open any round; download any round's file | -- |
| Owen (Client Primary Contact) | Every version of his own company's deliverables (Own-only) | Open and view any round of his own company's deliverables | Versions belonging to another client company never appear or resolve; a direct link to another company's deliverable shows "This page isn't part of your portal." per XBR-09 |
| Priya (Client Reviewer Contact) | Every version of her own company's deliverables (Own-only) | Open and view any round of her own company's deliverables | Same as Owen -- versions outside her own client company never appear or resolve |
| Dana (Support Operator) | The version list and each round's metadata (round_number, uploaded_at), inside a logged, read-only support session (FEAT-31) | View only -- no file download control is rendered for any round | A direct attempt to open a version's underlying file is refused with "Downloads are not available in a support session." (ASMP-18, ASMP-23; FEAT-17.SPEC-005) |
| Unauthenticated | No | No | Client contacts: redirected to request a fresh magic link (FEAT-05). Nadia: redirected to the freelancer sign-in screen. |
| Expired session | No | No | Client contacts see the expired/invalid link explanation with a one-tap way to request a fresh link (FEAT-05); Nadia sees "Your session has expired. Sign in to continue." Neither preserves the round that was open -- the screen reopens on latest after re-authentication. |

## Layout and Content

**Header:** Deliverable name, milestone name, and project name (read-only), with a back arrow returning to the entry point per the Navigation Out table.

**Body:**
- **Version selector** -- a horizontally scrollable row of round chips ("Round 1", "Round 2", ... "Round {N} (Latest)"), one per Deliverable Version, ordered by round_number ascending, with the currently open round highlighted. Not rendered at all when the deliverable has exactly one version (Single Version state, below).
- **Round detail panel** -- below the selector, shows the open round's file preview area (where feasible) or a generic file icon with name and size, plus its metadata: "Round {round_number} -- uploaded {uploaded_at}" and, when applicable, "Latest version" label.
- **Comment count indicator** -- display-only badge on the round detail panel showing the number of comments anchored to the open round (per FEAT-17.SPEC-005's anchoring rule), linking into FEAT-07's thread for that round.
- **Download control** -- visible to Nadia, Owen, and Priya only; never rendered for Dana's support session.

### Responsive Behavior

- **Compact breakpoint:** Version selector chips scroll horizontally in a single row; round detail panel stacks full-width below it.
- **Medium size class and above:** Version selector and round detail panel remain in the same relative arrangement; the detail panel's file preview area grows to fill the available width, capped at a consistent platform-wide content width.
- **Comment count indicator:** Uniform scaling, no structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to the entry point per Navigation Out | Screen closes | Returns to the deliverable view, the upload screen's origin, or the support session view |
| Round chip | Tap | Loads the selected round's file and metadata | Selected chip highlighted; round detail panel updates | Brief loading indicator shown while a large file's content loads (Loading state, below) |
| File preview area (round detail panel) | Tap (or keyboard Enter on the focused preview) | Where the file type supports inline preview: images and PDFs open in a zoomable full-width view; video and audio play inline with standard playback controls; other previewable types open static. Unpreviewable types show the generic file icon and this tap does nothing | Preview enlarges or begins playback; no data changes | Standard playback/zoom feedback (browser-native); a second tap or the close control returns to the detail panel |
| Disabled round chip (offline; unopened round) | Tap | No action -- the chip is disabled while offline | None | Chip shows "Available when you reconnect"; it becomes selectable again automatically on reconnection |
| "Try again" (Error state) | Tap | Re-attempts resolving the deliverable and its version list | Full-screen message replaced by the Loading (screen) placeholder | On success, the version selector and detail panel render; on repeated failure, the Error message reappears |
| Comment count indicator | Tap (Nadia, Owen, Priya only) | Navigates into FEAT-07's comment thread, scoped to the open round | Screen navigates away | Opens FEAT-07's deliverable comment thread for this round |
| Download control (Nadia, Owen, Priya only) | Tap | Downloads the open round's file | None on this screen | Standard file-download feedback (browser-native) |

### Accessibility Notes

- **Focus order:** Back arrow -> version selector chips (in round order; disabled chips are announced as unavailable) -> round detail panel content, including the file preview area -> comment count indicator (when visible) -> download control (when visible).
- **Status announcements:** Switching rounds announces the new round's label ("Round {N}, uploaded {date}") to assistive technology once its content finishes loading. The Single Version state's absence of a selector is not separately announced -- the round detail panel alone is the entire interactive surface in that state.
- **Keyboard alternatives:** Round chips are reachable and selectable by keyboard (arrow keys move between chips, Enter/Space selects); the download control has no pointer-only equivalent requiring a mouse.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Single Version | No version selector rendered; round detail panel shows the deliverable's only version directly, with no "Round 1" chip or "Latest" label shown (nothing to distinguish it from) | The deliverable has exactly one Deliverable Version | A second version is created (screen re-renders the selector on next load) |
| Multiple Versions (default) | Version selector with 2+ chips; latest round open by default unless entry-point context specifies another | The deliverable has 2 or more Deliverable Versions | -- |
| Loading (round switch) | Round detail panel shows a brief in-progress indicator in place of the file content | User taps a different round chip on a large file | The selected round's content finishes loading |
| Loading (screen) | Full-panel loading placeholder in place of the version selector and detail panel | Screen first opens while the deliverable and its versions are being resolved | Data resolves, or resolution fails (Error, below) |
| Error | Full-screen message "This deliverable's versions couldn't be loaded." with a "Try again" action | The deliverable or version list fails to resolve | User taps "Try again" and resolution succeeds |
| Offline/Degraded | Banner "You're offline -- showing versions you've already viewed." at top; previously loaded rounds remain viewable read-only; unopened rounds show a disabled chip with "Available when you reconnect" | Connectivity is lost while this screen is open | Connectivity returns and all rounds become selectable again |

## Validation Rules

Validation governed by FEAT-17.SPEC-004 (Version Numbering, Immutability & Retention Rules) for round ordering and the latest-version derivation, and by FEAT-17.SPEC-005 (Version Access & Comment-Anchoring Rules) for who may view or download each round. This screen has no user input beyond selecting a round or a download action.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-07 (deliverable view), FEAT-17.SPEC-001 (post-upload), or the FEAT-31 support session view -- whichever entry point was used | FEAT-07 or FEAT-31, per entry point |
| Comment count indicator tap | FEAT-07 (deliverable comment thread), scoped to the open round | FEAT-07 (Deliverable Review & Feedback) |

## Data Model

**Creates:** None.
**Reads:** Deliverable -- name, milestone, project (header context). Deliverable Version -- round_number, file, uploaded_at for every version of this deliverable (selector and detail panel); the "Latest" label is derived from the highest round_number (FEAT-17.SPEC-004), not read from a stored field. Comment -- count of comments whose target is the open Deliverable Version, for the comment count indicator (FEAT-07 owns Comment; this screen only reads a count).
**Updates:** None.
**Deletes:** None.

## Business Rules

- FEAT-17.SPEC-004 governs round ordering (sequential, no gaps) and which round is labeled "Latest" (the is_latest derivation); this screen never computes its own ordering or latest determination.
- FEAT-17.SPEC-005 governs who may view, open, and download each round; Dana's download control is never rendered regardless of session state (ASMP-18).
- The version selector is hidden entirely when only one version exists (feature's States field), so a first-round deliverable looks identical to a non-versioned one until a second round is uploaded.
- XBR-09: client isolation applies here as everywhere in the portal -- a version belonging to another client's deliverable is never reachable, resolvable, or shown to Owen or Priya.
- This screen is a snapshot per load, not live-updating: a version uploaded by Nadia in another session while a viewer has this screen open does not appear until the screen is reopened or reloaded; the currently open round remains valid and unaffected either way, since versions are immutable and never replaced in place.

## Edge Cases

- **A new version is uploaded while a viewer has this screen open on an older round** -- No live conflict: this screen is a snapshot, not live-updating (Business Rules, above). The round the viewer is looking at remains fully valid and unchanged; the new round simply does not appear in the selector until the screen is reopened. This is not a concurrent-edit conflict because this screen performs no write against Deliverable Version -- the entity's own Contention note states no concurrent modification of a version is possible, since versions are appended, never edited in place.
- **Deliverable has zero versions (upload still in progress or never started)** -- Not reachable through this feature's own entry points, since FEAT-06.SPEC-003 creates round 1 only on full upload completion; a stale link surfaces the Error state.
- **A round's file preview cannot be rendered inline (unsupported file type)** -- The round detail panel falls back to a generic file icon, name, and size, with the download control (where visible) still available.
- **Viewer switches rounds rapidly (taps several chips in quick succession)** -- Only the most recently tapped round's load is shown; earlier in-flight loads for previously tapped rounds are discarded when they resolve.
- **Comment count indicator shows zero** -- Indicator still renders with "0 comments"; tapping it still opens FEAT-07's empty-state thread for that round.
- **Dana's support session ends while a version's content is loading** -- The in-progress load is cancelled and the screen redirects to the support session's closed-session experience, per FEAT-31.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-07 (Deliverable Review & Feedback) | Navigation (inbound/outbound) | Deliverable view's version selector opens directly into this screen; comment count indicator navigates into FEAT-07's thread for the open round |
| FEAT-17.SPEC-001 (New Version Upload) | Navigation (inbound) | Destination of the "View versions" button after a completed upload |
| FEAT-17.SPEC-004 (Version Numbering, Immutability & Retention Rules) | References (inbound) | Round ordering and the "Latest" derivation |
| FEAT-17.SPEC-005 (Version Access & Comment-Anchoring Rules) | References (inbound) | Who may view, open, or download each round; Dana's no-download constraint |
| FEAT-31 (Operator Support Access) | Navigation (inbound) | Entry point for Dana's read-only support session; governs session timing and closure |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| version_browser_opened | deliverable reference, version count, viewer role | Screen loads successfully | supports success-metrics.md: "Client Portal Mobile Responsiveness" (this screen is the client-facing surface the metric's 2-second interactivity target measures for Owen and Priya's version-viewing sessions; for Nadia and Dana this event is retained for feature-level visibility only) |
| version_opened | round_number, whether it is the derived latest, load duration | A viewer selects a round and its content finishes loading | supports success-metrics.md: "Client Portal Mobile Responsiveness" |
| version_compared | count of distinct rounds opened in the current screen session | A viewer opens a second or further round after already viewing one in the same session | N/A -- no Stage 2 metric measures manual round-to-round comparison specifically; retained for feature-level visibility since the Key Capability "open and compare any round by number" has no dedicated Stage 2 metric |

## Acceptance Criteria

**FEAT-17.SPEC-002-AC-01:** Given Owen opens a deliverable with only one version, when the screen loads, then no version selector is shown and the round detail panel shows that version directly.

**FEAT-17.SPEC-002-AC-02:** Given Priya opens a deliverable with 3 versions, when the screen loads, then a version selector with 3 chips appears, Round 3 is highlighted as open, and it is labeled "Latest."

**FEAT-17.SPEC-002-AC-03:** Given Nadia is viewing Round 3 of a large-video deliverable, when she taps the Round 1 chip, then a brief loading indicator appears in the detail panel before Round 1's content displays.

**FEAT-17.SPEC-002-AC-04:** Given Owen is viewing Round 1 of a deliverable and Nadia uploads Round 2 in another session, when Owen's screen is already open, then Round 1 remains fully visible and unchanged, and Round 2 does not appear until Owen reopens or reloads the screen.

**FEAT-17.SPEC-002-AC-05:** Given Dana is inside a read-only support session viewing a deliverable's versions, when she looks for a way to download any round's file, then no download control is rendered for any round.

**FEAT-17.SPEC-002-AC-06:** Given Dana attempts to reach a version's underlying file directly (bypassing the rendered controls) during a support session, when the attempt is made, then it is refused with "Downloads are not available in a support session."

**FEAT-17.SPEC-002-AC-07:** Given Priya taps the comment count indicator on Round 2, when the navigation completes, then she lands in FEAT-07's comment thread scoped to Round 2.

**FEAT-17.SPEC-002-AC-08:** Given Owen attempts to reach a deliverable's version history for a company other than his own, when the request is made, then he sees "This page isn't part of your portal." and no version data is shown.

**FEAT-17.SPEC-002-AC-09:** Given Nadia has just completed an upload via FEAT-17.SPEC-001, when she taps View versions and reaches this screen, then the new round is shown as latest and open by default.

**FEAT-17.SPEC-002-AC-10:** Given a round's file type cannot be previewed inline, when that round is opened, then a generic file icon with name and size is shown in place of a preview, with the download control still available to an authorized viewer.

**FEAT-17.SPEC-002-AC-11:** Given Nadia loses connectivity while this screen is open, when the connection drops, then the offline banner appears, previously viewed rounds remain selectable and viewable, and unopened rounds show as unavailable until reconnection.

**FEAT-17.SPEC-002-AC-12:** Given the deliverable or its version list fails to load, when the screen attempts to open, then "This deliverable's versions couldn't be loaded." appears with a "Try again" action.

**FEAT-17.SPEC-002-AC-13:** Given a viewer taps several round chips in rapid succession, when the loads resolve out of order, then only the most recently tapped round's content is shown.

**FEAT-17.SPEC-002-AC-14:** Given Owen opens a round with zero comments, when the comment count indicator renders, then it shows "0 comments" and remains tappable into FEAT-07's empty thread state for that round.

**FEAT-17.SPEC-002-AC-15:** Given the Error state is showing, when Priya taps "Try again" and resolution succeeds, then the Loading placeholder appears briefly and the version selector and detail panel render with the latest round open.

**FEAT-17.SPEC-002-AC-16:** Given Nadia is offline with an unopened Round 2, when she taps its chip, then nothing happens, the chip reads "Available when you reconnect," and it becomes selectable again once connectivity returns.

**FEAT-17.SPEC-002-AC-17:** Given Owen opens a round whose file is an image, when he taps the preview, then it opens in a zoomable view; and given the round is a video, then tapping plays it inline with playback controls; and given the type is unpreviewable, then tapping the generic icon does nothing.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 7 | 7 |
| States | 6 (single version, multiple versions, loading round, loading screen, error, offline) | 6 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

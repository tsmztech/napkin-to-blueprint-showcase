---
document_type: spec
spec_type: screen
spec_id: FEAT-08.SPEC-001
spec_name: Milestone Review & Approval Screen
spec_slug: milestone-review-approval-screen
parent_feature: FEAT-08
parent_feature_name: Milestone Approval
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 22
---

# Screen Spec: Milestone Review & Approval Screen

## Overview

**Name:** Milestone Review & Approval Screen
**ID:** FEAT-08.SPEC-001
**Type:** Screen
**Purpose:** Owen reviews a milestone's current deliverable and comment thread and approves it with a single, plainly explained, timestamped action; Priya sees the identical status and content with no working Approve control.
**Parent Feature:** FEAT-08 -- Milestone Approval

## Scope and Non-Goals

**In Scope:**
- Displaying the milestone's current Deliverable and its Comment thread for review before a decision
- The Approve control and its confirmation copy, available to Owen only
- Showing the milestone's approval state ("Approved on {date}") once approved
- The loading, error, offline, and stale-refresh states around the Approve action

**Non-Goals:**
- Posting or retracting comments -- handled by Deliverable Review & Feedback (FEAT-07); this screen displays the existing thread read-only and links out for participation.
- Uploading, replacing, or removing the deliverable -- handled by Deliverable Upload & Sharing (FEAT-06); this screen displays the current deliverable read-only.
- Reopening an approved milestone -- handled by FEAT-08.SPEC-002 (Milestone Reopen Screen), a freelancer-only, freelancer-facing screen that this one never exposes to any client contact.
- Defining the exact authorization, eligibility, exactly-once, and concurrency rules behind the Approve action -- owned by FEAT-08.SPEC-006 (Approval Authorization & Eligibility Rules), which this screen enforces and reflects rather than restates.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-07.SPEC-001 (Deliverable Comment Thread) | Owen or Priya finishes reviewing the current deliverable and its comments and navigates to the milestone-level view | The milestone reference; the deliverable and comment data already loaded is reused where possible to avoid a second full load |
| FEAT-07.SPEC-002 (Milestone Comment Thread) | Owen or Priya finishes reviewing milestone-level feedback and navigates to the approval view | The milestone reference |
| FEAT-06.SPEC-006 (Deliverable Ready Notification email) | A client contact opens the "deliverable ready for review" email and signs in via FEAT-05 | The milestone reference, landing directly on this screen after magic-link sign-in |
| FEAT-08.SPEC-005 (Reopen Recording) | A milestone Nadia just reopened returns Owen to an unapproved state on a screen he already had open | The same milestone reference; the screen re-renders in place with the Approve control restored |
| FEAT-08.SPEC-007 (Milestone Approval Confirmation Notification) | Owen or Nadia taps the confirmation email's "View milestone" CTA | The approved milestone reference |
| FEAT-04.SPEC-002 (Milestone Timeline (Client View)) | Owen or Priya taps a milestone row whose deliverable is ready for review (Priya sees no Approve control) | The milestone reference |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Owen (Client Primary Contact) | Full screen, for milestones belonging to his own client company (Own-only) | Approve, once the deliverable and comment thread have fully loaded and connectivity is present, and only while the milestone is not already Approved | -- |
| Priya (Client Reviewer Contact) | Full screen, for milestones belonging to her own client company (Own-only) -- identical deliverable, comment thread, and status display to Owen | None -- no Approve control is rendered for her at all | There is no denial message shown, because the control simply does not exist for this role; the screen otherwise behaves identically to Owen's read-only content |
| Nadia (Freelancer) | Not applicable -- this screen is client-facing only; Nadia never opens this exact screen | None | N/A -- Nadia's equivalent view of the same milestone lives in her own project view (FEAT-01) and the Reopen Screen (FEAT-08.SPEC-002), not this spec |
| Dana (Support Operator) | Full screen content, read-only, inside a logged support session (FEAT-31) -- no file downloads | None | Every action affordance (Approve) is rendered disabled with "Support access is read-only." |
| Unauthenticated | No | No | Redirected to FEAT-05's sign-in request page; after a fresh magic-link sign-in, the contact lands back on this exact milestone |
| Expired session | No | No | FEAT-05's "expired or invalid link" page with a plain explanation and a one-step request for a fresh sign-in link; no in-progress state exists to preserve, since this screen has no draft input |

## Layout and Content

**Header:** The project and milestone name, with the milestone's current status shown as a plain label ("Ready for your review," "Approved on {date}," or "Reopened -- new deliverable pending"). A back control returns to the client portal home (FEAT-05).

**Body, in order from top to bottom:**
- **Deliverable panel:** The milestone's current, Active Deliverable -- its kind (uploaded file or linked external asset), a preview where feasible, and a link to open it in full via FEAT-06's viewing surface. If the deliverable is a linked asset, the panel shows the link and its reachability state as recorded by FEAT-06.
- **Comment thread panel, below the deliverable panel:** The milestone's comment thread (Comment entity, target: milestone), read-only on this screen, in post-time order, each entry showing author name, role (Nadia, Owen, or Priya), and posted time. A "Reply in thread" link navigates out to FEAT-07.SPEC-002 (Milestone Comment Thread) for anyone who wants to post.
- **Decision area, at the bottom of the body (Owen only):** The Approve control and, directly beneath it, the fixed consent line: "This records your approval and issues the next invoice." Neither the control nor the line is shown to Priya.
- **Approved marker (once approved, replaces the Decision area for every role):** "Approved on {date}" with the approving contact's name, shown to Owen, Priya, and Dana alike.

**Footer:** None -- Approve sits inline in the body's decision area, not in a persistent footer, since it is a single deliberate action rather than a recurring form action.

### Responsive Behavior

- **Compact size class:** Single column, full width, in the stacking order described above; the deliverable preview scales to the available width and the comment thread scrolls independently beneath it.
- **Medium size class and above:** The deliverable panel and comment thread panel sit side by side (deliverable left, comments right), each independently scrollable; the decision area (or Approved marker) spans the full width beneath both, so the consent line always reads across the whole screen width regardless of size class.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back control | Tap | Navigate to FEAT-05 (portal home) | Screen closes | Standard navigation transition |
| Deliverable panel "Open" link | Tap | Navigate to FEAT-06's viewing surface for the current deliverable, or open the linked external asset in a new context | This screen remains open behind the navigation | Standard navigation or new-context transition |
| "Reply in thread" link | Tap | Navigate to FEAT-07.SPEC-002 (Milestone Comment Thread) | This screen closes | Standard navigation transition |
| Comment thread entries | Display only | None -- read-only on this screen | None | No interaction; posting happens on FEAT-07.SPEC-002 |
| Approve control (Owen only) | Tap, while enabled | 1. Disable the control and show a brief in-progress state. 2. Trigger FEAT-08.SPEC-003 (Approval Recording & Concurrency Guard). 3. Await the outcome. | Control shows an in-progress state during the request | Success: the Decision area is replaced by the "Approved on {date}" marker. Stale/already-approved: refresh dialog described in States, then the marker. Connectivity/failure: the relevant error/offline state described in States. |
| Approve control (Owen only) | Tap, while disabled (loading or offline) | No action -- the control is inert until its enabling condition is met | None | The control's disabled appearance itself communicates why (loading spinner, or the offline banner already visible above it) |

### Accessibility Notes

- **Focus order:** Back control -> Deliverable panel's Open link -> comment thread entries (in post order) -> "Reply in thread" link -> Approve control (when present) -> Approved marker (when present).
- **Consent line association:** The "This records your approval and issues the next invoice" line is programmatically associated with the Approve control, so assistive technology announces it as part of the control's own description, not as separate incidental text.
- **Dynamic announcements:** A successful approval's transition to the "Approved on {date}" marker is announced to assistive technology as a live region update. The stale-refresh dialog and any error/offline banner are announced immediately when they appear.
- **Keyboard alternatives:** Every action on this screen (navigation links, Approve) is reachable and actionable by keyboard alone; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Empty | N/A -- this view exists only once a deliverable has been uploaded for the milestone (product-features.md, States); a milestone with no deliverable yet has no route into this screen, so there is no empty-state rendering to define here | Never entered -- excluded by the milestone's own lifecycle (Defined -> Deliverable Uploaded before this screen becomes reachable) | N/A |
| Loading | Deliverable and comment thread panels show loading placeholders; the Approve control (if the role would otherwise see one) is disabled | Screen first opens | Deliverable and comment thread both finish loading |
| Ready for review (default, Owen) | Deliverable and comments fully loaded; Approve control enabled with the consent line beneath it | Loading completes for a milestone not yet Approved, viewed by Owen | Owen taps Approve, or the milestone's status changes underneath him (reopen elsewhere, or another approval race) |
| Ready for review (Priya / read-only) | Deliverable and comments fully loaded; no Approve control anywhere in the layout | Loading completes for a milestone not yet Approved, viewed by Priya or Dana | Milestone's status changes to Approved |
| Approving (in progress) | Approve control shows an in-progress state; the rest of the screen remains visible and stable | Owen taps Approve | FEAT-08.SPEC-003 returns an outcome |
| Approved | Decision area replaced by "Approved on {date}" with the approving contact's name, visible to every role that can view this screen | A successful approval outcome is returned, or the screen loads for a milestone already Approved | A subsequent reopen (FEAT-08.SPEC-005) changes status away from Approved |
| Stale refresh (Owen) | A dialog: "This milestone has changed since you opened it. Here's the current version." with an acknowledgment action that reloads the deliverable, comments, and current status | FEAT-08.SPEC-003 returns the stale-attempt or unauthorized-because-already-approved outcome | Owen acknowledges the dialog, and the screen reloads to the live state |
| Error (approval failed) | An error banner above the Decision area: "Something went wrong recording your approval. Try again." with a Retry action; the milestone's displayed status is unchanged | FEAT-08.SPEC-003 returns the write-failure outcome | Owen taps Retry (re-triggers the Approve interaction) or navigates away |
| Load error | An error banner in place of the deliverable and/or comment panel that failed to load, with a Retry action; the Approve control (if applicable) stays disabled until loading succeeds | The deliverable or comment thread fails to load | Owen or Priya taps Retry and loading succeeds |
| Offline/Degraded | A persistent banner: "You're offline. Reconnect to approve this milestone." Already-loaded deliverable and comment content remain visible and readable; the Approve control is disabled for the duration, and no approval is queued for later -- an approval never appears to succeed without a live connection | Connectivity is lost while the screen is open, or the screen loads without connectivity | Connectivity is restored; the Approve control re-enables once the milestone's current state is confirmed fresh |

## Validation Rules

Validation and eligibility governed by FEAT-08.SPEC-006 (Approval Authorization & Eligibility Rules). See that spec for the full role, state, and concurrency conditions gating the Approve action; this screen renders the resulting Access and Visibility and States behavior above.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back control tap | FEAT-05 (portal home) | FEAT-05 -- Client Portal Access |
| Deliverable "Open" link tap | FEAT-06 (deliverable viewing surface) | FEAT-06 -- Deliverable Upload & Sharing |
| "Reply in thread" link tap | FEAT-07.SPEC-002 (Milestone Comment Thread) | FEAT-07 -- Deliverable Review & Feedback |
| Successful approval | Stays on this screen, now showing the Approved state | -- |

## Data Model

**Creates:** None directly -- the approval write itself is performed by FEAT-08.SPEC-003, which this screen triggers.
**Reads:** Milestone -- `name`, `status`, `approved_at`, `approved_by`. Deliverable -- the milestone's current, Active deliverable: `kind`, `file`/`link`, `link_status`. Comment -- the milestone's comment thread: `text`, `author`, `posted_at`, `status` (Posted entries only; Retracted entries are omitted per FEAT-07's own display rule).
**Updates:** None directly -- Milestone `status`/`approved_at`/`approved_by` are updated only by FEAT-08.SPEC-003, triggered from this screen.
**Deletes:** None.

## Business Rules

- The Approve control's eligibility, exactly-once, immutability, and concurrency behavior are governed entirely by FEAT-08.SPEC-006 and enforced by FEAT-08.SPEC-003; this screen only reflects their outcomes.
- XBR-09: an out-of-scope or expired milestone link never reveals another client company's data -- it shows a plain explanation and a fresh-link path, handled by FEAT-05.
- Approving is a permanent, evidentiary act (BRIEF.md, Constraints: record immutability) -- once the Approved marker is shown, there is no client-side action anywhere on this screen that reverses it; only Nadia's reopen (FEAT-08.SPEC-005), on a screen this feature never exposes to a client contact, changes the state again.
- The consent line ("This records your approval and issues the next invoice") is required, verbatim, wherever the Approve control appears -- it is the product's stated accessibility and informed-consent commitment for this irreversible action (product-features.md, States: Accessibility).

## Edge Cases

- **Owen taps Approve twice in rapid succession** -- The control disables on the first tap; the second tap has no effect while disabled, so exactly one approval request is ever sent from this screen (the exactly-once guarantee itself is enforced server-side by FEAT-08.SPEC-003 regardless).
- **The milestone was re-priced or had its deliverable removed by Nadia since this screen loaded, and Owen taps Approve** -- FEAT-08.SPEC-003 refuses the write; this screen shows the stale-refresh dialog and reloads the deliverable, comments, and current status rather than recording the approval against what Owen was actually shown.
- **The milestone is approved by an already-completed request from another of Owen's sessions while this screen is open** -- The Approve tap on this screen is refused as stale (already approved); the stale-refresh dialog reloads to the current "Approved on {date}" state.
- **Owen navigates away mid-approval (Approving state) and returns** -- The screen re-loads fresh on return and reflects whatever outcome the in-flight request ultimately produced (Approved, or still awaiting his action if the request had not yet completed) -- there is no separate draft or unsaved-approval state to restore, since Approve has no intermediate input to lose.
- **Priya opens this screen for a milestone Owen has already approved** -- She sees the "Approved on {date}" marker identically to Owen; nothing about her role changes once approved, since she never had an Approve control to begin with.
- **Nadia reopens the milestone while Owen has this screen open** -- The screen's displayed status is a snapshot from load; Owen sees the reopened state only on his next load or a live-update refresh, at which point the Approve control reappears -- this screen does not silently flip state without a reload, since an approval decision must be made against a state the reviewer has actually seen refreshed.
- **A milestone that Owen already approved is later reopened, and he returns to this screen** -- He sees the Approve control again, exactly as he would for a milestone approved for the first time; the prior approval's history is not shown inline here (it lives in the Activity Log via FEAT-13), only the current cycle's status.
- **Concurrent-edit conflict on the underlying Milestone entity (the shared-entity contention case this screen must cover, per the dependency map's Milestone Contention note)** -- Resolution is reject-with-refresh, identical to the stale-approval edge case above: any mismatch between what Owen was shown and the milestone's live state at the moment he acts refuses the write and reloads him to the current state, rather than recording his decision against data that has since changed.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-07.SPEC-001 (Deliverable Comment Thread) | Navigation (inbound) | Owen or Priya arrives here after reviewing the deliverable and its comments |
| FEAT-07.SPEC-002 (Milestone Comment Thread) | Navigation (inbound / outbound) | Arrival after milestone-level review; "Reply in thread" navigates back out to post |
| FEAT-06 (Deliverable Upload & Sharing) | Navigation (outbound) / References (inbound) | Displays the current deliverable read-only and links to its full viewing surface |
| FEAT-08.SPEC-003 (Approval Recording & Concurrency Guard) | Triggers (outbound) | The Approve interaction fires this automation |
| FEAT-08.SPEC-006 (Approval Authorization & Eligibility Rules) | References (inbound) | Governs whether the Approve control renders and is actionable |
| FEAT-08.SPEC-005 (Reopen Recording) | References (inbound) | A successful reopen returns this screen to its unapproved, Approve-enabled state |
| FEAT-05 (Client Portal Access) | Navigation (inbound / outbound) | Entry after magic-link sign-in; exit via the back control |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| milestone_review_screen_viewed | role (Owen / Priya / Dana), milestone status at view time | Screen finishes loading | supports success-metrics.md: "Milestone Approval Turnaround" |
| milestone_approve_tapped | time since deliverable marked ready for review | Owen taps an enabled Approve control | supports success-metrics.md: "Milestone Approval Turnaround" |
| milestone_approval_confirmed_on_screen | time from tap to confirmed outcome | This screen renders the "Approved on {date}" marker following a successful approval | supports success-metrics.md: "Milestone Approval Turnaround" |
| milestone_approval_stale_refresh_shown | trigger reason (re-priced / deliverable removed / already approved) | The stale-refresh dialog is shown to Owen | N/A -- no Stage 2 metric measures refused-attempt friction directly; retained so a pattern of stale refreshes (which would slow real-world approval turnaround) is visible rather than indistinguishable from ordinary slow review |

## Acceptance Criteria

**FEAT-08.SPEC-001-AC-01:** Given Owen navigates from FEAT-07.SPEC-001 to a milestone whose deliverable and comments have not yet finished loading, when the screen opens, then it shows the Loading state and the Approve control is disabled.

**FEAT-08.SPEC-001-AC-02:** Given Owen is on this screen for a milestone with status "Deliverable Uploaded," when loading completes, then the deliverable panel, comment thread, and an enabled Approve control with its consent line are all shown.

**FEAT-08.SPEC-001-AC-03:** Given Priya is on this screen for the same milestone, when loading completes, then she sees the identical deliverable and comment content, but no Approve control anywhere in the layout.

**FEAT-08.SPEC-001-AC-04:** Given Owen taps the enabled Approve control, when the tap registers, then the control disables, shows an in-progress state, and triggers FEAT-08.SPEC-003.

**FEAT-08.SPEC-001-AC-05:** Given FEAT-08.SPEC-003 returns a successful outcome, when the response arrives, then the Decision area is replaced by "Approved on {date}" with Owen's name, announced to assistive technology as a live update.

**FEAT-08.SPEC-001-AC-06:** Given FEAT-08.SPEC-003 returns a stale-attempt outcome because Nadia changed the milestone since load, when the response arrives, then the stale-refresh dialog appears and, once acknowledged, the screen reloads to the current deliverable, comments, and status.

**FEAT-08.SPEC-001-AC-07:** Given FEAT-08.SPEC-003 returns a write-failure outcome, when the response arrives, then an error banner with a Retry action appears above the Decision area, and the milestone's displayed status is unchanged.

**FEAT-08.SPEC-001-AC-08:** Given Owen loses connectivity while this screen is open, when the loss is detected, then the offline banner "You're offline. Reconnect to approve this milestone." appears, the Approve control disables, and no approval is queued.

**FEAT-08.SPEC-001-AC-09:** Given Owen regains connectivity after the offline banner appeared, when the screen confirms the milestone's current state is fresh, then the Approve control re-enables (if the milestone is still eligible).

**FEAT-08.SPEC-001-AC-10:** Given Owen taps Approve twice in rapid succession, when the first tap disables the control, then the second tap has no effect and only one request is sent.

**FEAT-08.SPEC-001-AC-11:** Given the milestone's status is already "Approved" when this screen first loads, when loading completes, then the "Approved on {date}" marker is shown immediately with no Decision area, for every role that can view the screen.

**FEAT-08.SPEC-001-AC-12:** Given Dana is in a logged, read-only support session viewing this milestone, when the screen loads, then she sees the full deliverable and comment content with every control shown disabled and labeled "Support access is read-only."

**FEAT-08.SPEC-001-AC-13:** Given an unauthenticated visitor follows a milestone link, when the link resolves, then they are redirected to FEAT-05's sign-in request page.

**FEAT-08.SPEC-001-AC-14:** Given a client contact's session has expired, when they attempt to open this screen, then they see FEAT-05's expired-link page with a fresh-link request option.

**FEAT-08.SPEC-001-AC-15:** Given the deliverable panel fails to load, when the failure occurs, then a load-error banner with Retry appears in its place, and the Approve control stays disabled until it succeeds.

**FEAT-08.SPEC-001-AC-16:** Given Owen taps "Reply in thread," when the tap registers, then he navigates to FEAT-07.SPEC-002 and this screen closes.

**FEAT-08.SPEC-001-AC-17:** Given Nadia reopens a milestone that Owen previously approved, when Owen reloads this screen, then he sees the Approve control again instead of the "Approved on {date}" marker.

**FEAT-08.SPEC-001-AC-18:** Given Priya follows a link to a milestone belonging to a different client company than her own, when the link resolves, then she sees a plain explanation and a fresh-link option, never that company's data.

**FEAT-08.SPEC-001-AC-19:** Given Owen approves a milestone whose live state matches what he was shown, when the approval succeeds, then the milestone_approval_confirmed_on_screen event fires with the elapsed time from tap to confirmation.

**FEAT-08.SPEC-001-AC-20:** Given two of Owen's own sessions both have this screen open for the same milestone and one approves it first, when the second session's Approve tap is processed, then FEAT-08.SPEC-003 refuses it as stale and this screen shows the stale-refresh dialog, resolving to the current Approved state -- the concurrent-edit conflict on the shared Milestone entity.

**FEAT-08.SPEC-001-AC-21:** Given Owen or Priya taps the Back control, when the tap registers, then they navigate to FEAT-05's portal home and this screen closes, with no data change of any kind.

**FEAT-08.SPEC-001-AC-22:** Given Owen or Priya taps the Deliverable panel's "Open" link, when the tap registers, then FEAT-06's viewing surface for the current deliverable opens (or the linked external asset opens in a new context), and this screen remains open behind it.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 10 (empty, loading, ready-owen, ready-readonly, approving, approved, stale refresh, error, load error, offline) | 10 |
| Business Rules | 4 | 4 |
| Edge Cases | 8 | 8 |

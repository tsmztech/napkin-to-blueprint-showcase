---
document_type: spec
spec_type: screen
spec_id: FEAT-08.SPEC-002
spec_name: Milestone Reopen Screen
spec_slug: milestone-reopen-screen
parent_feature: FEAT-08
parent_feature_name: Milestone Approval
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 17
---

# Screen Spec: Milestone Reopen Screen

## Overview

**Name:** Milestone Reopen Screen
**ID:** FEAT-08.SPEC-002
**Type:** Screen
**Purpose:** Nadia reopens an approved milestone through a deliberate, explained, logged confirmation -- never a silent status edit -- when genuinely necessary.
**Parent Feature:** FEAT-08 -- Milestone Approval

## Scope and Non-Goals

**In Scope:**
- The reopen confirmation flow: what reopening does, and Nadia's deliberate confirmation of it
- Showing the milestone's current approval details (who approved it, when) before Nadia confirms
- The loading, error, offline, and already-changed states around the reopen action

**Non-Goals:**
- Approving a milestone -- the reverse action belongs entirely to Owen on FEAT-08.SPEC-001; Nadia has no approve capability anywhere in the product.
- Editing the milestone's name, price, or schedule once reopened -- handled by FEAT-04.SPEC-001 (Milestone & Payment Schedule Editor), which becomes reachable again only after this screen's reopen action succeeds.
- Reversing or cancelling the invoice the original approval generated -- excluded per the Brief's Non-Goals: correcting that invoice is owned entirely by FEAT-09 via a credit note or new invoice (XBR-04); this screen has no invoice-editing surface.
- Defining the exact authorization and eligibility rule behind reopen -- owned by FEAT-08.SPEC-006 (Approval Authorization & Eligibility Rules), which this screen enforces and reflects rather than restates.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01 (project view, milestones area) | Nadia opens an Approved milestone from her own project view and chooses Reopen | The milestone reference |
| FEAT-04.SPEC-002 (Milestone Timeline & Client View, freelancer's own equivalent view) | Nadia sees a milestone marked Approved and chooses Reopen from its detail | The milestone reference |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen, for any of her own milestones currently Approved | Confirm reopen, once the milestone's live status is confirmed as Approved and connectivity is present | -- |
| Owen (Client Primary Contact) | None -- this screen has no client-facing route | None | This screen is never reachable from the client portal; no navigation path or deep link into it exists for a client contact |
| Priya (Client Reviewer Contact) | None | None | Same as Owen -- unreachable from the client portal |
| Dana (Support Operator) | None -- reopening is a write action, and Dana's support access is read-only everywhere (ASMP-18); she has no reason to reach this specific confirmation screen | None | This screen is not part of the read-only support session surface; Dana views milestone status through FEAT-08.SPEC-001's read-only rendering instead |
| Unauthenticated | No | No | Redirected to Nadia's own sign-in; this is a freelancer-only screen, not a client-portal one |
| Expired session | No | No | Redirected to sign-in with a session-expired notice; no in-progress state exists to preserve |

## Layout and Content

**Header:** "Reopen this milestone?" with the milestone's name and project, and a back control that returns to the milestone's detail view without reopening anything.

**Body, in order from top to bottom:**
- **Current approval summary:** "Approved on {date} by {approving contact's name}." Read-only.
- **Explanation block:** Fixed copy explaining the consequence: "Reopening lets you make changes to this milestone again. It will show as Reopened to {client company name}, and this action is recorded. The original approval stays on record." (the client company name is the milestone's owning Client's `client_name`).
- **Confirm area:** A "Reopen Milestone" confirmation control and a "Cancel" control, side by side.

**Footer:** None -- the confirm/cancel pair sits in the body, since this screen has only one decision to make.

### Responsive Behavior

- **Compact size class:** Single column, full width, in the order described above; the Confirm and Cancel controls stack full-width, Reopen above Cancel.
- **Medium size class and above:** Uniform scaling, no structural change -- the content is capped at a consistent platform-wide narrow-form width (the design layer's decision) and centered; Confirm and Cancel sit side by side rather than stacked.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back control | Tap | Navigate away without reopening | Screen closes | Standard navigation transition |
| Cancel control | Tap | Discard the reopen attempt, no write occurs | Screen closes, returning to the milestone's detail view | Standard navigation transition |
| "Reopen Milestone" control | Tap, while enabled | 1. Disable both controls and show an in-progress state. 2. Trigger FEAT-08.SPEC-005 (Reopen Recording). 3. Await the outcome. | Confirm control shows an in-progress state during the request | Success: navigate to the milestone's detail view, now showing "Reopened." Already-changed: refresh message described in States. Connectivity/failure: the relevant error/offline state described in States. |
| "Reopen Milestone" control | Tap, while disabled (loading or offline) | No action -- inert until its enabling condition is met | None | The control's disabled appearance communicates why |

### Accessibility Notes

- **Focus order:** Back control -> Current approval summary (read, not focusable) -> Explanation block (read, not focusable) -> Reopen Milestone control -> Cancel control.
- **Confirmation announcement:** A successful reopen's navigation away from this screen is preceded by an announced confirmation so Nadia is not left uncertain whether the action completed before the screen changes.
- **Dynamic announcements:** The already-changed message and any error/offline banner are announced immediately when they appear.
- **Keyboard alternatives:** Every action on this screen (Back, Cancel, Reopen) is reachable and actionable by keyboard alone; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Empty | N/A -- this screen exists only for a milestone that is currently Approved, reached exclusively from an Approved milestone's own detail view; there is no list or browsing surface within this screen for an empty condition to apply to | Never entered -- excluded by this screen's own entry points, which only route in from an already-Approved milestone | N/A |
| Loading | Approval summary shows a loading placeholder; Reopen and Cancel controls disabled | Screen first opens | The milestone's current approval details finish loading |
| Ready to confirm (default) | Approval summary, explanation, and both controls fully rendered and enabled | Loading completes for a milestone still "Approved" | Nadia taps Reopen or Cancel, or the milestone's status changes underneath her |
| Confirming (in progress) | Reopen control shows an in-progress state; Cancel is also disabled during the request | Nadia taps Reopen Milestone | FEAT-08.SPEC-005 returns an outcome |
| Already changed | A message in place of the confirm area: "This milestone's status has changed. Here's the current state." with an acknowledgment action that navigates to the milestone's current detail view | FEAT-08.SPEC-005 returns the already-changed outcome | Nadia acknowledges the message and navigates away |
| Error (reopen failed) | An error banner above the confirm area: "Something went wrong reopening this milestone. Try again." with a Retry action | FEAT-08.SPEC-005 returns the write-failure outcome | Nadia taps Retry or navigates away |
| Load error | An error banner in place of the approval summary, with a Retry action; the Reopen control stays disabled until it succeeds | The current approval details fail to load | Nadia taps Retry and loading succeeds |
| Offline/Degraded | A persistent banner: "You're offline. Reconnect to reopen this milestone." The Reopen control is disabled for the duration; no reopen is queued for later -- reopening never appears to succeed without a live connection | Connectivity is lost while the screen is open, or the screen loads without connectivity | Connectivity is restored and the milestone's current state is confirmed fresh |

## Validation Rules

Validation and eligibility governed by FEAT-08.SPEC-006 (Approval Authorization & Eligibility Rules). See that spec for the milestone-state precondition and the rule that this action is Nadia-only; this screen renders the resulting Access and Visibility and States behavior above.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back control tap | The milestone's detail view (FEAT-01 project view or FEAT-04.SPEC-002) | -- |
| Cancel tap | Same as Back | -- |
| Successful reopen | The milestone's detail view, now showing "Reopened" | -- |
| Already-changed acknowledgment | The milestone's current detail view | -- |

## Data Model

**Creates:** None directly -- the reopen write itself is performed by FEAT-08.SPEC-005, which this screen triggers.
**Reads:** Milestone -- `name`, `status`, `approved_at`, `approved_by`. Client -- `client_name`, for the explanation copy.
**Updates:** None directly -- Milestone `status` is updated only by FEAT-08.SPEC-005, triggered from this screen.
**Deletes:** None.

## Business Rules

- Reopen is Nadia-only and requires the milestone to currently be Approved -- governed entirely by FEAT-08.SPEC-006 and enforced by FEAT-08.SPEC-005; this screen only reflects their outcomes.
- Reopening is a deliberate, logged action, never a silent edit (Key Capabilities) -- this screen exists specifically so reopening always passes through an explicit confirmation with a stated consequence, rather than a status field a freelancer could change incidentally from a list view.
- The original approval record is never altered by a reopen (XBR-04) -- the explanation copy states this plainly so Nadia understands the reopen adds a new event rather than erasing the prior one.

## Edge Cases

- **Nadia taps Reopen Milestone twice in rapid succession** -- The controls disable on the first tap; the second tap has no effect while disabled, so exactly one request is ever sent from this screen.
- **The milestone was already reopened (or re-approved and reopened again) by an action from another of Nadia's sessions since this screen loaded** -- FEAT-08.SPEC-005 returns the already-changed outcome; this screen shows the acknowledgment message and, once acknowledged, sends Nadia to the milestone's current state rather than claiming a reopen that did not happen as she expected.
- **Nadia navigates away mid-confirmation (Confirming state) and returns** -- The screen reloads fresh on return and reflects whatever outcome the in-flight request ultimately produced -- there is no separate draft state to restore, since Reopen has no intermediate input to lose.
- **Nadia opens this screen for a milestone that is not currently Approved (reached through a stale link or a shared bookmark)** -- Loading resolves with the live status already not "Approved"; this screen shows the already-changed message immediately rather than a confirm area for an action that cannot succeed.
- **Nadia cancels after reading the explanation** -- No write occurs; she returns to the milestone's detail view exactly as it was before she opened this screen.
- **The client company (Owen or Priya) opens the milestone at the exact moment Nadia's reopen commits** -- Not a conflict for this screen: Nadia's Confirm/Cancel are the only writes this screen ever attempts, and the client side has no write path into this same action; the client contact's next load of FEAT-08.SPEC-001 simply reflects the reopened state.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01 (Client & Project Management) | Navigation (inbound) | Nadia arrives here from an Approved milestone in her project view |
| FEAT-04.SPEC-002 (Milestone Timeline & Client View) | Navigation (inbound) | Nadia arrives here from the milestone's freelancer-side detail |
| FEAT-08.SPEC-005 (Reopen Recording) | Triggers (outbound) | The Reopen Milestone confirmation fires this automation |
| FEAT-08.SPEC-006 (Approval Authorization & Eligibility Rules) | References (inbound) | Governs whether reopen is actionable and who may reach it |
| FEAT-08.SPEC-001 (Milestone Review & Approval Screen) | References (outbound) | A successful reopen here returns that screen to its unapproved, Approve-enabled state |
| FEAT-04.SPEC-001 (Milestone & Payment Schedule Editor) | Navigation (outbound) | Becomes reachable for edits on this milestone again only after a successful reopen |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| milestone_reopen_screen_viewed | time since the original approval | Screen finishes loading in the Ready-to-confirm state | N/A -- no Stage 2 metric measures reopen-screen views; retained as the funnel step preceding milestone_reopened_by_freelancer (FEAT-08.SPEC-005), which is itself N/A for the same reason -- reopen frequency has no connected success metric, but the event pair keeps the reopen path observable |
| milestone_reopen_confirmed | time since the original approval | Nadia's reopen succeeds | N/A -- see above; the reopen path has no connected Stage 2 metric, and this event is retained purely for observability of the Key Capability's usage, not for a specific target |
| milestone_reopen_cancelled | -- | Nadia taps Cancel | N/A -- no Stage 2 metric measures abandoned reopens; retained so a pattern of hesitation before reopening (which could indicate the explanation copy is unclear) remains visible |

## Acceptance Criteria

**FEAT-08.SPEC-002-AC-01:** Given Nadia opens this screen for a milestone whose approval details have not yet finished loading, when the screen opens, then it shows the Loading state and both controls are disabled.

**FEAT-08.SPEC-002-AC-02:** Given Nadia is on this screen for a milestone that is currently Approved, when loading completes, then she sees "Approved on {date} by {approving contact's name}," the explanation block, and enabled Reopen Milestone and Cancel controls.

**FEAT-08.SPEC-002-AC-03:** Given Nadia taps the enabled Reopen Milestone control, when the tap registers, then both controls disable, an in-progress state shows, and FEAT-08.SPEC-005 is triggered.

**FEAT-08.SPEC-002-AC-04:** Given FEAT-08.SPEC-005 returns a successful outcome, when the response arrives, then Nadia is navigated to the milestone's detail view, now showing "Reopened."

**FEAT-08.SPEC-002-AC-05:** Given FEAT-08.SPEC-005 returns an already-changed outcome, when the response arrives, then the acknowledgment message "This milestone's status has changed. Here's the current state." appears, and acknowledging it navigates to the milestone's current detail view.

**FEAT-08.SPEC-002-AC-06:** Given FEAT-08.SPEC-005 returns a write-failure outcome, when the response arrives, then an error banner with a Retry action appears, and the milestone's approval details remain unchanged.

**FEAT-08.SPEC-002-AC-07:** Given Nadia loses connectivity while this screen is open, when the loss is detected, then the offline banner "You're offline. Reconnect to reopen this milestone." appears and the Reopen control disables.

**FEAT-08.SPEC-002-AC-08:** Given Nadia regains connectivity after the offline banner appeared, when the screen confirms the milestone is still Approved, then the Reopen control re-enables.

**FEAT-08.SPEC-002-AC-09:** Given Nadia taps Cancel, when the tap registers, then no write occurs and she returns to the milestone's detail view unchanged.

**FEAT-08.SPEC-002-AC-10:** Given Nadia taps Reopen Milestone twice in rapid succession, when the first tap disables the controls, then the second tap has no effect and only one request is sent.

**FEAT-08.SPEC-002-AC-11:** Given Owen or Priya attempts to reach this screen through any path, when the attempt is made, then no such route exists in the client portal.

**FEAT-08.SPEC-002-AC-12:** Given Dana is in a support session, when she views this milestone, then she sees its status through FEAT-08.SPEC-001's read-only rendering, never through this screen.

**FEAT-08.SPEC-002-AC-13:** Given an unauthenticated visitor attempts to reach this screen, when the attempt is made, then they are redirected to Nadia's own sign-in.

**FEAT-08.SPEC-002-AC-14:** Given Nadia's session has expired, when she attempts to open this screen, then she is redirected to sign-in with a session-expired notice.

**FEAT-08.SPEC-002-AC-15:** Given the current approval details fail to load, when the failure occurs, then a load-error banner with Retry appears in place of the approval summary, and the Reopen control stays disabled.

**FEAT-08.SPEC-002-AC-16:** Given Nadia reaches this screen through a stale link for a milestone that is no longer Approved, when loading completes, then she sees the already-changed message immediately, with no confirm area shown.

**FEAT-08.SPEC-002-AC-17:** Given Nadia taps the Back control, when the tap registers, then no write occurs and she navigates to the milestone's detail view unchanged, identically to Cancel.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 4 | 4 |
| States | 8 (empty, loading, ready, confirming, already changed, error, load error, offline) | 8 |
| Business Rules | 3 | 3 |
| Edge Cases | 6 | 6 |

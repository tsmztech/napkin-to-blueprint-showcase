---
document_type: spec
spec_type: screen
spec_id: FEAT-30.SPEC-001
spec_name: Contextual Help Tooltip
spec_slug: contextual-help-tooltip
parent_feature: FEAT-30
parent_feature_name: Contextual Help & Guidance
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 15
---

# Screen Spec: Contextual Help Tooltip

## Overview

**Name:** Contextual Help Tooltip
**ID:** FEAT-30.SPEC-001
**Type:** Screen
**Purpose:** An inline, on-demand explanation for an unfamiliar control, overlaid on a host screen starting at the user's first encounter with the control and offered on every later encounter until permanently dismissed, with a permanent-dismiss action.
**Parent Feature:** FEAT-30 -- Contextual Help & Guidance

## Scope and Non-Goals

**In Scope:**
- The help-affordance icon that appears next to an unfamiliar control, starting at the first encounter and on every later encounter until permanent dismissal, on any freelancer-dashboard or client-portal host screen
- The on-demand explanation popover triggered by that affordance
- The permanent-dismiss action, which triggers FEAT-30.SPEC-004
- A temporary close that leaves the tip eligible to reappear on a later encounter
- Content and role-scoping decisions delegated to FEAT-30.SPEC-005

**Non-Goals:**
- A guided, multi-step product tour or sequenced walkthrough engine -- excluded by adjacency analysis: the feature's Rationale states both sides' flows are "designed to be self-explanatory in the moment (one-click accept, one-click approve)"; a forced or sequenced tour would contradict that design premise.
- Reviewing or restoring a previously dismissed tip -- excluded per the Entity-Lifecycle Coverage Matrix's Read (list) row: "No screen lists a user's dismissal history"; the Key Capabilities name only forward, permanent dismissal, never a review-or-restore list.
- Operator-visible or operator-actionable guidance -- excluded per scope-boundaries.md SC-04: Dana's support sessions are read-only and she never acts as, or on behalf of, a freelancer or client contact, so this overlay never renders in her support-session view.
- A general-purpose chat, messaging, or live-support channel reached from the tip -- excluded per scope-boundaries.md SC-15: comments (FEAT-07) and request-changes notes (FEAT-03) already cover in-product communication.

## Entry Points

This is an overlay, not a destination screen -- it has no default entry of its own. It is instantiated on host screens at every encounter of a given control while the tip is eligible (FEAT-30.SPEC-005 Eligibility to render); the rows below name the first-encounter moment for each host.

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-20 (Onboarding / First-Run Setup) -- any first-run setup screen | Nadia's first open of that screen reaches an unfamiliar setup control | The control's tip_id and the current host screen's identity |
| FEAT-08 (Milestone Approval) -- the milestone approval screen | Owen sees the Approve control for the first time | The control's tip_id (Approve control) |
| FEAT-05 (Client Portal Access) -- first magic-link sign-in and first portal screens | A contact's first sign-in, or first view of portal content | The control's tip_id and the current host screen's identity |
| Any other host screen on the freelancer dashboard or client portal | The viewing user's first encounter with a control this feature defines a tip for | The control's tip_id |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full -- every tip scoped to a control her role can use | Open, temporarily close, and permanently dismiss any tip she is shown | -- |
| Owen (Client Primary Contact) | Own-only -- tips scoped to the controls his Primary role has access to on his own client's portal | Open, temporarily close, and permanently dismiss any tip he is shown | -- |
| Priya (Client Reviewer Contact) | Own-only -- tips scoped to the controls her Reviewer role has access to; never a tip explaining a Primary-only control (e.g., Approve), because FEAT-30.SPEC-005 excludes it and the host screen itself never renders that control to her (XBR-08) | Open, temporarily close, and permanently dismiss any tip she is shown | -- |
| Dana (Support Operator) | No | No | This overlay never renders in a support session; per the Access Matrix, Dana's Notifications & Help entitlement is limited to delivery warnings, and per scope-boundaries.md SC-04 she never acts as, or on behalf of, a freelancer or client contact. She sees the underlying host screen's data read-only, without the guidance layer. |
| Unauthenticated | No | No | The host screen itself is unreachable while unauthenticated (per that screen's own Access rules); since this overlay only ever renders on top of an already-loaded host screen, an unauthenticated visitor never encounters it. |
| Expired session | No | No | The host screen redirects to its own re-authentication path (e.g., FEAT-05's expired-link page for portal screens) before any overlay could render; no tip state is lost because none was in progress. |

## Layout and Content

**Help affordance:** A small, consistent icon (e.g., an info glyph) positioned immediately adjacent to the unfamiliar control it explains, on the host screen, at every encounter of the control while the tip is eligible (from the first encounter until permanent dismissal). The icon is part of the host screen's own layout region for that control -- it does not add a new layout region of its own.

**Popover (on activation):** An inline callout anchored to the affordance, positioned so it does not obscure the control it explains. Contains, top to bottom:
- A brief explanation (one to three sentences) of what the adjacent control does, in plain language scoped to the viewing role's own entitlements (FEAT-30.SPEC-005)
- Two actions, left-aligned in the popover's footer: "Got it" (temporary close) and "Don't show this again" (permanent dismiss)
- A close (X) control in the popover's top-right corner, equivalent to "Got it"

Only one popover is open at a time per screen; opening a second affordance's popover closes any popover already open.

### Responsive Behavior

- **Compact size class:** The popover anchors below the affordance and spans the available width up to a consistent platform-wide maximum, so it never runs off-screen; it never covers the control it explains.
- **Medium size class and above:** The popover anchors directly beside or below the affordance depending on available space, at a fixed, consistent platform-wide width; no structural change beyond positioning.
- **Affordance icon:** Uniform scaling, no structural change across breakpoints.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Help affordance icon | Tap/click | Opens the popover for this control's tip, per FEAT-30.SPEC-005's content and eligibility rules | Popover appears anchored to the affordance | Explanation text visible in the popover |
| Help affordance icon (popover already open for this tip) | Tap/click | Closes the popover (toggle) | Popover disappears | Host screen returns to its unmodified view |
| "Got it" / close (X) | Tap/click | Closes the popover without recording a dismissal | Popover disappears; nothing is recorded, and the affordance is offered again at the next encounter and every later one until permanent dismissal | Host screen returns to its unmodified view |
| Tap/click outside the popover | Tap/click | Closes the popover, same as "Got it" | Popover disappears | Host screen returns to its unmodified view |
| "Don't show this again" | Tap/click | Closes the popover and triggers FEAT-30.SPEC-004 to permanently record the dismissal for this tip_id and this user | Popover disappears; affordance is removed for this tip on every future render for this user | Host screen returns to its unmodified view; no confirmation message beyond the popover closing (advisory, non-blocking, per FEAT-30.SPEC-005) |

### Accessibility Notes

- **Focus order:** Help affordance icon receives focus in the host screen's existing tab order (immediately after the control it explains); once the popover opens, focus moves into the popover ("Got it" action first, then "Don't show this again," then close); closing the popover by any method returns focus to the affordance icon.
- **Announcements:** Opening the popover announces its explanation text to assistive technology; closing it (by any of the three close paths) announces that the popover has closed and guidance for this control remains available via the affordance.
- **Keyboard alternatives:** The affordance is reachable and activatable by keyboard; the popover's actions are reachable by keyboard; "Escape" closes the popover the same as "Got it." There are no pointer-only gestures on this overlay.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Available (default) | Affordance icon visible next to its control; popover closed | Any encounter (first or later) of the tip's control by a user for whom the tip is eligible per FEAT-30.SPEC-005 (entitled role, catalog entry, not dismissed); this includes every encounter after a temporary close | User taps the affordance, or permanently dismisses the tip |
| Open | Popover visible with explanation text and both close actions | User taps the affordance | User taps "Got it," the close (X), "Don't show this again," or outside the popover |
| Dismissed | Affordance icon is not rendered at all for this tip_id, for this user, on any host screen | FEAT-30.SPEC-004 records a permanent dismissal for this tip_id and user | Never -- dismissal is one-directional (Entity-Lifecycle Coverage Matrix, State Transition row) |
| Loading | N/A -- static contextual content already part of the host screen's own render; there is no separate load step for the tip's explanation text | -- | -- |
| Offline/Degraded | Already-loaded tip content (affordance and, if open, its popover) remains available and interactive; a tip not yet loaded when connectivity was lost is simply not offered on this render -- no error, no retry prompt. A "Don't show this again" chosen while offline closes the popover, suppresses the tip on this device, and is held and retried automatically on reconnection (FEAT-30.SPEC-004) | Connectivity lost while the host screen is open | Connectivity restored -- any held dismissal is retried and the next render re-evaluates tip eligibility normally |

## Validation Rules

Content selection, role scoping, and dismissal-suppression rules are governed by FEAT-30.SPEC-005 (Contextual Help Content & Behavior Rules). See that spec for the complete rule set. This screen has no user-entered field input to validate.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Any close action ("Got it," close X, outside tap, or "Don't show this again") | No navigation -- the popover closes and the user remains on the host screen already in view | -- |

## Data Model

**Creates:** None directly on this screen. A Help-Tip Dismissal State record defaulted to "not dismissed" is created as a side effect of a tip's first eligible render -- see FEAT-30.SPEC-004.
**Reads:** Help-Tip Dismissal State (feature-local state on Freelancer Account or Client Contact) -- checked via FEAT-30.SPEC-005 immediately before deciding whether to render a given tip's affordance. Also reads the viewing role from Freelancer Account (Nadia) or Client Contact (Owen's or Priya's Primary/Reviewer role) for content scoping.
**Updates:** None directly -- the "Don't show this again" action triggers FEAT-30.SPEC-004, which owns the write.
**Deletes:** None.

## Business Rules

- Guidance is always advisory, never blocking -- the control this tip explains remains fully usable whether or not its tip has been opened or dismissed (FEAT-30.SPEC-005).
- Content and eligibility for every tip are scoped by role per FEAT-30.SPEC-005 and XBR-08 -- a role never sees a tip for a control it has no entitlement to use.
- Permanent dismissal is one-directional: once "Don't show this again" is chosen, that tip never renders again for that user, on any host screen, on either the freelancer dashboard or the client portal (Entity-Lifecycle Coverage Matrix).
- The affordance is offered at every encounter of its control, from the user's first encounter until permanent dismissal, whenever the tip is eligible to render as defined by FEAT-30.SPEC-005 (entitled role, catalog entry, not dismissed). A temporary close ("Got it," close (X), outside tap, Escape) never ends eligibility; only "Don't show this again" does.

## Edge Cases

- **User taps the affordance twice in rapid succession** -- The second tap toggles the popover closed (per the Interactions table); no duplicate popover opens.
- **Connectivity lost while the popover is open** -- The already-rendered explanation text remains visible and interactive (per the Offline/Degraded state); no error is shown.
- **The "Don't show this again" write (FEAT-30.SPEC-004) fails for a reason other than lost connectivity** -- The popover has already closed optimistically; the write is not retried, so the tip's affordance may reappear on a later encounter because the dismissal was never recorded. No error is shown to the user -- this failure is silent and non-blocking, consistent with the advisory-only rule (FEAT-30.SPEC-005).
- **"Don't show this again" is chosen while the device has no connectivity** -- The popover closes and the tip stays suppressed on this device; FEAT-30.SPEC-004 holds the dismissal and retries it automatically on reconnection, the same retry path as FEAT-30.SPEC-002 and FEAT-30.SPEC-003. Only if the held request is lost before reconnection (device storage cleared or session ended) does the tip reappear at a later encounter; no error is shown in either case.
- **The same user dismisses the same tip from two open sessions at effectively the same time** -- Both writes set the record to the same end state (dismissed = true); there is no meaningful conflict to resolve, and no second confirmation or error is shown.
- **A user's role changes mid-session (e.g., a Client Contact is promoted from Reviewer to Primary by FEAT-18)** -- Tips newly eligible under the new role are offered on the next render that includes the now-accessible controls; dismissal states are tracked per tip_id, so the role change does not retroactively dismiss or restore anything.
- **A control this tip would explain is not shown to the viewing role at all** -- No affordance is offered for it; the host screen never renders the control, so there is nothing to attach a tip to (XBR-08).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-30.SPEC-004 (Help Tip Dismissal Recording) | Triggers (outbound) | "Don't show this again" triggers the permanent dismissal write |
| FEAT-30.SPEC-005 (Contextual Help Content & Behavior Rules) | References (inbound) | Supplies tip content, role scoping, the advisory-only rule, and the dismissal-suppression check |
| FEAT-20 (Onboarding / First-Run Setup) | References (inbound) | Host feature whose first-run setup screens embed this overlay |
| FEAT-08 (Milestone Approval) | References (inbound) | Host feature whose Approve control this overlay explains for Owen |
| FEAT-05 (Client Portal Access) | References (inbound) | Host feature whose first sign-in and first portal screens embed this overlay |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| help_tip_shown | tip_id, host_feature (FEAT-20 \| FEAT-08 \| FEAT-05 \| other), role | Popover opens after an affordance tap | When host_feature = FEAT-20: supports success-metrics.md: "First-Session Activation". When host_feature = FEAT-08: supports success-metrics.md: "Milestone Approval Turnaround". When host_feature = FEAT-05: supports success-metrics.md: "Client Portal Login Success". These three are the overlaid features feature-overview.md's Non-Goals name as consuming this signal as a raw measurement input; FEAT-30 defines no metric or dashboard of its own for tip views on any other host_feature (N/A -- no success-metrics.md metric names a general help-tip-viewing behavior). |
| help_tip_closed_temporarily | tip_id, host_feature | User taps "Got it," the close (X), or outside the popover | N/A -- a temporary close is not itself cited by any success-metrics.md metric; only the shown event (above) and the permanent-dismissal event (FEAT-30.SPEC-004) feed the overlaid features' metrics. |

## Acceptance Criteria

**FEAT-30.SPEC-001-AC-01:** Given Nadia is on a first-run setup screen (FEAT-20) and encounters an unfamiliar control for the first time, when she taps the help affordance beside it, then a popover opens showing a brief explanation of that control.

**FEAT-30.SPEC-001-AC-02:** Given Owen is viewing the milestone approval screen (FEAT-08) for the first time, when he taps the help affordance beside the Approve control, then a popover opens explaining what approving does, and this emits a help_tip_shown event with host_feature FEAT-08.

**FEAT-30.SPEC-001-AC-03:** Given Priya (Reviewer) is on the same milestone view Owen sees, when she looks for a help affordance beside an Approve control, then none is shown, because her role never has an Approve control rendered to it in the first place (XBR-08).

**FEAT-30.SPEC-001-AC-04:** Given Nadia has an open help popover, when she taps "Got it," then the popover closes, no dismissal is recorded, and the affordance is offered again at her next encounter of the control and at every later encounter until she chooses "Don't show this again."

**FEAT-30.SPEC-001-AC-05:** Given Owen has an open help popover, when he taps "Don't show this again," then the popover closes and FEAT-30.SPEC-004 records a permanent dismissal for that tip and Owen.

**FEAT-30.SPEC-001-AC-06:** Given Nadia previously dismissed a tip permanently, when she encounters the same control again on any screen, then no affordance is shown for that tip.

**FEAT-30.SPEC-001-AC-07:** Given Dana is in a read-only support session viewing Nadia's dashboard, when the underlying screen renders, then no help affordances or popovers appear anywhere on it.

**FEAT-30.SPEC-001-AC-08:** Given a visitor is not signed in, when they attempt to reach any host screen this overlay would appear on, then they cannot reach that screen at all, and this overlay never renders.

**FEAT-30.SPEC-001-AC-09:** Given Priya has an open help popover, when she taps outside the popover, then it closes the same as tapping "Got it," and the tip remains available for a later encounter.

**FEAT-30.SPEC-001-AC-10:** Given Owen has one help popover open, when he taps a different control's help affordance, then the first popover closes and the second one opens.

**FEAT-30.SPEC-001-AC-11:** Given Nadia loses connectivity while a help popover is open, when she continues reading it, then the already-rendered explanation stays visible and no error appears.

**FEAT-30.SPEC-001-AC-12:** Given Nadia taps "Don't show this again" and the dismissal write fails for a reason other than lost connectivity, so it is not retried, when she encounters the same control again later, then the affordance may still appear, and no error was ever shown to her for the earlier failed attempt.

**FEAT-30.SPEC-001-AC-13:** Given Priya is promoted from Reviewer to Primary contact by FEAT-18 mid-session, when she next encounters the Approve control, then a help affordance for it is offered to her for the first time, since it is now eligible under her new role.

**FEAT-30.SPEC-001-AC-14:** Given Nadia dismisses the same tip from two open sessions at effectively the same time, when both dismissal writes complete, then the tip is recorded as dismissed exactly once in effect, with no error or conflict shown in either session.

**FEAT-30.SPEC-001-AC-15:** Given Owen taps "Don't show this again" while his device has no connectivity, when the popover closes, then the tip is not offered again on that device, no error is shown, and once connectivity returns FEAT-30.SPEC-004 completes the dismissal automatically so the tip stays suppressed on every future render.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 4 (available, open, dismissed, offline/degraded) | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |

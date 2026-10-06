---
document_type: spec
spec_type: screen
spec_id: FEAT-30.SPEC-003
spec_name: Client Portal Help Reference
spec_slug: client-portal-help-reference
parent_feature: FEAT-30
parent_feature_name: Contextual Help & Guidance
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Screen Spec: Client Portal Help Reference

## Overview

**Name:** Client Portal Help Reference
**ID:** FEAT-30.SPEC-003
**Type:** Screen
**Purpose:** A short, browsable help reference covering the client-facing portal, scoped to the viewing contact's role, reachable from any portal screen, so a client contact needs no external documentation.
**Parent Feature:** FEAT-30 -- Contextual Help & Guidance

## Scope and Non-Goals

**In Scope:**
- A browsable list of short reference topics covering the client-portal product areas, scoped to what the viewing contact's role (Primary or Reviewer) can actually do (XBR-08)
- Expand/collapse of each topic to reveal its explanation
- A per-topic "Don't show this again" action for topics that correspond to a contextual tip also shown inline (FEAT-30.SPEC-001)
- A single, consistent entry point placed on the client portal home, reachable from every portal screen

**Non-Goals:**
- Showing Owen and Priya the same topic set -- excluded by design: their entitlements differ (XBR-08), so this reference's content set differs by role, distinct from FEAT-30.SPEC-002's freelancer-only topic set (feature-overview.md Discovery Rationale for this spec).
- A native-app-only or native-gesture-dependent guidance experience -- excluded per scope-boundaries.md SC-06: the product ships no native apps, so this screen must work as ordinary content in a mobile browser.
- Configurable, freelancer-authored, or multi-language help content -- excluded per scope-boundaries.md SC-11 and SC-20: content is fixed, English-only product content.
- A general-purpose chat, messaging, or live-support channel reached from this screen -- excluded per scope-boundaries.md SC-15: comments (FEAT-07) and request-changes notes (FEAT-03) already cover in-product communication.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-05 (Client Portal Access) -- portal home, and every other portal screen | The viewing contact taps the Help entry point present on every portal screen | None -- the reference opens to its default topic list, scoped to the viewer's role |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | No | No | Nadia does not sign into a client portal; this screen is not part of her dashboard experience (see FEAT-30.SPEC-002 for her equivalent reference). |
| Owen (Client Primary Contact) | Own-only -- the Primary-scoped topic set for his own client company's portal | Expand/collapse topics; dismiss any topic that has a corresponding contextual tip | -- |
| Priya (Client Reviewer Contact) | Own-only -- the Reviewer-scoped topic set for her own client company's portal, which never includes topics explaining Primary-only actions (accepting, approving, paying, downloading invoices) | Expand/collapse topics; dismiss any topic that has a corresponding contextual tip | -- |
| Dana (Support Operator) | No | No | Per the Access Matrix, Dana's Notifications & Help entitlement is limited to delivery warnings; per scope-boundaries.md SC-04 she never acts as, or on behalf of, a client contact, and she has no client-portal access at all (Client Portal Access row: None). |
| Unauthenticated | No | No | Redirected to the magic-link sign-in request page (FEAT-05); no portal content, including this reference, is reachable before authenticating. |
| Expired session | No | No | The expired/invalid link page (FEAT-05) offers a fresh sign-in link; no in-progress reference state exists to preserve, since this is a static, non-editable screen. |

## Layout and Content

**Header:** Screen title "Help" with a close/back control that returns to the portal screen the contact opened it from.

**Body:** A single-column list of topic entries, grouped under the client-facing product areas relevant to the viewing role. For Owen (Primary), groups include: Proposals and Acceptance, Deliverables and Feedback, Milestone Approval, Invoices and Payments. For Priya (Reviewer), groups include only: Deliverables and Feedback -- her role has no Proposals & Acceptance, Milestone Approval (approve), or Invoicing & Payments entitlement (Access Matrix), so those groups and their topics do not appear in her list at all. Each topic entry, collapsed by default, shows only its title; tapping it expands to show its short explanation inline.

Topics that correspond to a contextual tip also shown inline via FEAT-30.SPEC-001 additionally show, once expanded, a "Don't show this again" action identical in effect to the one on the inline tip. Topics with no corresponding inline tip show no dismiss action.

### Responsive Behavior

- **Compact size class (the default for this mostly-mobile audience):** Single-column list, full width; group headings remain sticky at the top of their group while scrolling.
- **Medium size class and above:** List remains single-column, capped at a consistent platform-wide reading width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Help entry point (on host portal screens) | Tap | Opens this screen, scoped to the viewer's role | Screen transitions to the reference list | Reference list appears, showing only the viewer's entitled groups |
| Close/back control | Tap | Returns to the portal screen this was opened from | Screen closes | Animated transition back to the portal screen |
| Topic entry (collapsed) | Tap | Expands the topic to show its explanation | Topic entry shows its explanation inline | Explanation text visible below the topic title |
| Topic entry (expanded) | Tap | Collapses the topic | Explanation hides | Only the title remains visible |
| "Don't show this again" (topics with a corresponding contextual tip only) | Tap | Triggers FEAT-30.SPEC-004 to permanently record the dismissal for that tip_id and the viewing contact | The corresponding inline tip (FEAT-30.SPEC-001) is suppressed on every future render for this contact; the topic stays in the list with its explanation, and its "Don't show this again" action is replaced by the static note "Inline tips for this topic are turned off" (FEAT-30.SPEC-005 Dismissal suppression; no restore control) | No confirmation message beyond the action completing, consistent with the advisory-only rule (FEAT-30.SPEC-005) |

### Accessibility Notes

- **Focus order:** Close/back control -> group headings and topic entries in their displayed order (role-scoped) -> (within an expanded topic) "Don't show this again" when present.
- **Announcements:** Expanding or collapsing a topic announces its new state to assistive technology; a successful dismissal announces that guidance for that topic has been turned off.
- **Keyboard alternatives:** Every topic entry and action on this screen is reachable and activatable by keyboard; there are no pointer-only gestures, consistent with this screen having no native-app-only affordance (SC-06).

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Default | Role-scoped topic list, all topics collapsed | Screen opens | User expands a topic |
| Topic expanded | The expanded topic shows its explanation (and dismiss action, if applicable) | User taps a collapsed topic | User taps it again, or taps another topic |
| Topic already dismissed | The topic and its full explanation remain in the role-scoped list and expand/collapse normally; instead of "Don't show this again," the expanded topic shows the static, non-interactive note "Inline tips for this topic are turned off" and no restore control | The topic's tip_id has dismissed = true for the viewing contact (dismissed earlier here, inline via FEAT-30.SPEC-001, or held offline on this device) | Never -- dismissal is one-directional |
| Loading | N/A -- static contextual content bundled with the screen; there is no separate load step | -- | -- |
| Offline/Degraded | The role-scoped topic list and every topic's explanation remain available and interactive, since this is static content already delivered with the screen; only a dismiss action's write may be deferred -- it is held and retried automatically on reconnection per FEAT-30.SPEC-004, and the topic shows as already dismissed on this device meanwhile | Connectivity lost while the screen is open | Connectivity restored -- any deferred dismiss write is retried automatically |

## Validation Rules

This screen has no user-entered field input. The dismiss-eligibility and role-scoping rules governing which topics are shown and which carry a dismiss action are defined by FEAT-30.SPEC-005.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Close/back control tap | The portal screen this was opened from | -- |

## Data Model

**Creates:** None directly. A Help-Tip Dismissal State record defaulted to "not dismissed" is created, when needed, as a side effect of a tip's first eligible evaluation -- see FEAT-30.SPEC-004.
**Reads:** Help-Tip Dismissal State (feature-local state on Client Contact) -- checked via FEAT-30.SPEC-005 for each topic that corresponds to a contextual tip, to decide whether to show its "Don't show this again" action (dismissed = false or no record) or the static "Inline tips for this topic are turned off" note (dismissed = true). Reads the Client Contact's role (Primary or Reviewer) to scope which topic groups appear (XBR-08). Reads the fixed help-content catalog for topic titles and explanations.
**Updates:** None directly -- "Don't show this again" triggers FEAT-30.SPEC-004, which owns the write.
**Deletes:** None.

## Business Rules

- Guidance is always advisory, never blocking -- browsing or ignoring this reference never restricts any portal capability (FEAT-30.SPEC-005).
- Topic groups and topics are scoped to the viewing contact's Primary/Reviewer role exactly as the rest of the portal is scoped (XBR-08, FEAT-30.SPEC-005) -- a role never sees a topic explaining an action it cannot take.
- Dismissing a topic here has the identical, one-directional effect as dismissing the same tip inline via FEAT-30.SPEC-001 -- there is exactly one dismissal state per tip_id per contact.
- Content is fixed, English-only product content (scope-boundaries.md SC-11, SC-20) -- neither Owen nor Priya can author, edit, or reorder topics.
- This screen carries no native-app-only affordance; it works as ordinary content in a mobile browser (scope-boundaries.md SC-06).

## Edge Cases

- **Owen opens the reference with no connectivity** -- The full Primary-scoped topic list renders normally, since it is static content already delivered with the screen; a dismiss action attempted while offline is held on the device and retried automatically on reconnection (FEAT-30.SPEC-004), with no error shown; if the held request is lost before reconnection, the topic shows its dismiss action again and the inline tip may reappear.
- **Priya dismisses the same topic from this reference and from its inline tip (FEAT-30.SPEC-001) in quick succession** -- Both writes set the same tip_id to dismissed for Priya; the second write is a no-op, with no error.
- **A Reviewer contact (Priya) is promoted to Primary by Nadia, who assigns and changes contact roles via FEAT-18 (Owen can only invite Reviewer colleagues, not change roles)** -- On her next open of this screen, the additional Primary-scoped groups (Proposals and Acceptance, Milestone Approval, Invoices and Payments) appear for the first time; her prior dismissals of Reviewer-scoped topics are unaffected.
- **A Primary contact (Owen) invites a new Reviewer colleague (FEAT-18)** -- The new Reviewer's first open of this screen shows only the Reviewer-scoped groups; no dismissal state exists for them yet, so every topic renders as not-yet-dismissed by default.
- **The fixed help-content catalog is updated between one release and the next (new topic added to a role's group)** -- The new topic appears in that role's list on the next screen load, rendering as not-yet-dismissed by default.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-30.SPEC-001 (Contextual Help Tooltip) | References (inbound) | Shares the same underlying tip content and dismissal state for topics that correspond to an inline tip |
| FEAT-30.SPEC-004 (Help Tip Dismissal Recording) | Triggers (outbound) | "Don't show this again" triggers the permanent dismissal write |
| FEAT-30.SPEC-005 (Contextual Help Content & Behavior Rules) | References (inbound) | Supplies topic content, role scoping, dismissal-suppression checks, and the advisory-only rule |
| FEAT-05 (Client Portal Access) | Navigation (inbound) | Hosts this screen's entry point on the portal home and every other portal screen |
| FEAT-18 (Client Contact Management & Roles) | References (inbound) | Source of the Primary/Reviewer role this screen's content scoping depends on |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| help_reference_opened | role (Primary \| Reviewer) | The viewing contact opens this screen | N/A -- no success-metrics.md metric names browsing the client portal help reference; unlike FEAT-30.SPEC-001's inline tips (which overlay specific measured moments such as first sign-in), this is a general lookup with no corresponding Stage 2 metric (feature-overview.md Non-Goals: no help-engagement analytics dashboard is in scope for FEAT-30). |
| help_reference_topic_expanded | topic_id, role | The viewing contact expands a topic | N/A -- same reason as help_reference_opened. |

## Acceptance Criteria

**FEAT-30.SPEC-003-AC-01:** Given Owen is on the portal home, when he taps the Help entry point, then the Client Portal Help Reference opens showing his Primary-scoped topic groups, collapsed.

**FEAT-30.SPEC-003-AC-02:** Given Priya is on a portal screen, when she taps the Help entry point, then the reference opens showing only her Reviewer-scoped groups, with no Proposals & Acceptance, Milestone Approval, or Invoicing & Payments group present.

**FEAT-30.SPEC-003-AC-03:** Given Owen has expanded the topic explaining the Approve control, when he taps "Don't show this again," then FEAT-30.SPEC-004 records the dismissal and the corresponding inline tip no longer appears to him on future encounters.

**FEAT-30.SPEC-003-AC-04:** Given Priya expands a topic with no corresponding inline tip, when she looks for a dismiss action, then none is shown, and the topic always remains in her list.

**FEAT-30.SPEC-003-AC-05:** Given Nadia is signed into her freelancer dashboard, when she looks for any way to reach the Client Portal Help Reference, then no path exists -- she uses FEAT-30.SPEC-002 instead.

**FEAT-30.SPEC-003-AC-06:** Given a visitor is not signed in, when they attempt to reach any client-portal screen, then they are redirected to the magic-link sign-in request page, and this reference is unreachable until they sign in.

**FEAT-30.SPEC-003-AC-07:** Given a contact's magic link has expired, when they try to open the portal, then they see the expired-link page with a fresh-link option, and this reference is unreachable until a new sign-in succeeds.

**FEAT-30.SPEC-003-AC-08:** Given Nadia promotes Priya to Primary contact via FEAT-18, when she next opens this reference, then the Primary-scoped groups appear for her for the first time.

**FEAT-30.SPEC-003-AC-09:** Given Owen loses connectivity while this reference is open, when he continues browsing topics, then every topic's explanation remains visible and expandable with no error.

**FEAT-30.SPEC-003-AC-10:** Given Owen taps "Don't show this again" while offline, when connectivity is lost at that moment, then the dismissal is held on his device, the topic shows the "Inline tips for this topic are turned off" note meanwhile, and FEAT-30.SPEC-004 retries the write automatically once connectivity returns, with no error shown.

**FEAT-30.SPEC-003-AC-11:** Given Priya taps the close/back control, when the screen closes, then she returns to the exact portal screen she opened this reference from.

**FEAT-30.SPEC-003-AC-12:** Given Owen has already permanently dismissed the tip for a topic (here or inline), when he opens the Client Portal Help Reference and expands that topic, then the topic and its full explanation are shown, the "Don't show this again" action is not rendered, and the static note "Inline tips for this topic are turned off" appears in its place with no restore control.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 4 (default, topic expanded, topic already dismissed, offline/degraded) | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |

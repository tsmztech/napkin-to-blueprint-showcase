---
document_type: spec
spec_type: screen
spec_id: FEAT-30.SPEC-002
spec_name: Freelancer Help Reference
spec_slug: freelancer-help-reference
parent_feature: FEAT-30
parent_feature_name: Contextual Help & Guidance
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Screen Spec: Freelancer Help Reference

## Overview

**Name:** Freelancer Help Reference
**ID:** FEAT-30.SPEC-002
**Type:** Screen
**Purpose:** A short, browsable help reference covering the freelancer dashboard, reachable from any dashboard screen, so Nadia needs no external documentation.
**Parent Feature:** FEAT-30 -- Contextual Help & Guidance

## Scope and Non-Goals

**In Scope:**
- A browsable list of short reference topics covering the freelancer-side product areas (client and project setup, proposals, milestones and payments, deliverables, invoicing, branding, settings)
- Expand/collapse of each topic to reveal its explanation
- A per-topic "Don't show this again" action for topics that correspond to a contextual tip also shown inline (FEAT-30.SPEC-001), so dismissing it here has the same effect as dismissing it there
- A single, consistent entry point placed on the freelancer dashboard, reachable from every dashboard screen

**Non-Goals:**
- Configurable, freelancer-authored, or multi-language help content -- excluded per scope-boundaries.md SC-11 (help content is fixed product content, not freelancer-authored) and SC-20 (English only at launch).
- A general-purpose chat, messaging, or live-support channel reached from this screen -- excluded per scope-boundaries.md SC-15: comments (FEAT-07) and request-changes notes (FEAT-03) already cover in-product communication.
- A guided, multi-step product tour -- excluded by adjacency analysis: the feature's Rationale states the freelancer flow is "designed to be self-explanatory in the moment," and this reference is a passive lookup, not a forced walkthrough.
- Search across topics -- the reference is deliberately short (product-features.md's Description calls it "a short help reference"); a small, fixed topic set does not warrant a search capability.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-12 (Freelancer Financial Dashboard) and every other freelancer dashboard screen | Nadia taps the Help entry point present on every dashboard screen | None -- the reference opens to its default topic list |
| FEAT-12 (Freelancer Financial Dashboard) -- empty dashboard | Zero-state prompt directs a brand-new Nadia toward this reference alongside FEAT-02's proposal prompt | None |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Expand/collapse topics; dismiss any topic that has a corresponding contextual tip | -- |
| Owen (Client Primary Contact) | No | No | Owen operates only within his client portal (FEAT-05) and has no navigation path to the freelancer dashboard; this screen does not exist in his experience. |
| Priya (Client Reviewer Contact) | No | No | Priya operates only within her client portal (FEAT-05) and has no navigation path to the freelancer dashboard; this screen does not exist in her experience. |
| Dana (Support Operator) | No | No | Per the Access Matrix, Dana's Notifications & Help entitlement is limited to delivery warnings; per scope-boundaries.md SC-04 she never acts as, or on behalf of, the freelancer, so this reference is not part of her read-only support-session view. |
| Unauthenticated | No | No | Redirected to Nadia's sign-in screen; after signing in, the user lands on the dashboard (FEAT-12), not this screen directly. |
| Expired session | No | No | Session-expired dialog directs Nadia to sign in again; no in-progress reference state exists to preserve, since this is a static, non-editable screen. |

## Layout and Content

**Header:** Screen title "Help" with a close/back control that returns to the dashboard screen Nadia opened it from.

**Body:** A single-column list of topic entries, grouped under the freelancer-side product areas: Clients and Projects, Proposals, Milestones and Payment Schedules, Deliverables, Invoicing and Payments, Branding, and Account Settings. Each group heading is followed by its topic entries in a fixed, deploy-time order. Each topic entry, collapsed by default, shows only its title; tapping it expands to show its short explanation inline, below the title.

Topics that correspond to a contextual tip also shown inline via FEAT-30.SPEC-001 additionally show, once expanded, a "Don't show this again" action identical in effect to the one on the inline tip. Topics with no corresponding inline tip (broader reference-only content, such as an overview of the invoicing area) show no dismiss action -- they are pure reference content with no dismissal state.

### Responsive Behavior

- **Compact size class:** Single-column list, full width; group headings remain sticky at the top of their group while scrolling.
- **Medium size class and above:** List remains single-column, capped at a consistent platform-wide reading width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Help entry point (on host dashboard screens) | Tap | Opens this screen | Screen transitions to the reference list | Reference list appears |
| Close/back control | Tap | Returns to the dashboard screen this was opened from | Screen closes | Animated transition back to the dashboard |
| Topic entry (collapsed) | Tap | Expands the topic to show its explanation | Topic entry shows its explanation inline | Explanation text visible below the topic title |
| Topic entry (expanded) | Tap | Collapses the topic | Explanation hides | Only the title remains visible |
| "Don't show this again" (topics with a corresponding contextual tip only) | Tap | Triggers FEAT-30.SPEC-004 to permanently record the dismissal for that tip_id and Nadia | The corresponding inline tip (FEAT-30.SPEC-001) is suppressed on every future render; the topic stays in the list with its explanation, and its "Don't show this again" action is replaced by the static note "Inline tips for this topic are turned off" (FEAT-30.SPEC-005 Dismissal suppression; no restore control) | No confirmation message beyond the action completing, consistent with the advisory-only rule (FEAT-30.SPEC-005) |

### Accessibility Notes

- **Focus order:** Close/back control -> group headings and topic entries in their displayed order -> (within an expanded topic) "Don't show this again" when present.
- **Announcements:** Expanding or collapsing a topic announces its new state (expanded/collapsed) to assistive technology; a successful dismissal announces that guidance for that topic has been turned off.
- **Keyboard alternatives:** Every topic entry and action on this screen is reachable and activatable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Default | Full topic list, all topics collapsed | Screen opens | User expands a topic |
| Topic expanded | The expanded topic shows its explanation (and dismiss action, if applicable); other topics remain collapsed | User taps a collapsed topic | User taps it again, or taps another topic (only one topic is expanded at a time is not required -- multiple topics may be expanded simultaneously) |
| Topic already dismissed | The topic and its full explanation remain in the list and expand/collapse normally; instead of "Don't show this again," the expanded topic shows the static, non-interactive note "Inline tips for this topic are turned off" and no restore control | The topic's tip_id has dismissed = true for Nadia (dismissed earlier here, inline via FEAT-30.SPEC-001, or held offline on this device) | Never -- dismissal is one-directional |
| Loading | N/A -- static contextual content bundled with the screen; there is no separate load step | -- | -- |
| Offline/Degraded | The full topic list and every topic's explanation remain available and interactive, since this is static content already delivered with the screen; only the dismiss action's write may be deferred -- it is held and retried automatically on reconnection per FEAT-30.SPEC-004, and the topic shows as already dismissed on this device meanwhile | Connectivity lost while the screen is open | Connectivity restored -- any deferred dismiss write is retried automatically |

## Validation Rules

This screen has no user-entered field input. The dismiss-eligibility and content-scoping rules governing which topics carry a dismiss action are defined by FEAT-30.SPEC-005.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Close/back control tap | The dashboard screen this was opened from | -- |

## Data Model

**Creates:** None directly. A Help-Tip Dismissal State record defaulted to "not dismissed" is created, when needed, as a side effect of a tip's first eligible evaluation -- see FEAT-30.SPEC-004.
**Reads:** Help-Tip Dismissal State (feature-local state on Freelancer Account) -- checked via FEAT-30.SPEC-005 for each topic that corresponds to a contextual tip, to decide whether to show its "Don't show this again" action (dismissed = false or no record) or the static "Inline tips for this topic are turned off" note (dismissed = true), per FEAT-30.SPEC-005 Dismissal suppression. Reads the fixed help-content catalog for topic titles and explanations.
**Updates:** None directly -- "Don't show this again" triggers FEAT-30.SPEC-004, which owns the write.
**Deletes:** None.

## Business Rules

- Guidance is always advisory, never blocking -- browsing or ignoring this reference never restricts any dashboard capability (FEAT-30.SPEC-005).
- Dismissing a topic here has the identical, one-directional effect as dismissing the same tip inline via FEAT-30.SPEC-001 -- there is exactly one dismissal state per tip_id per user, checked by both surfaces (FEAT-30.SPEC-005).
- Topics with no corresponding inline tip are pure reference content and carry no dismissal state at all -- they always remain in the list.
- Content is fixed, English-only product content (scope-boundaries.md SC-11, SC-20) -- Nadia cannot author, edit, or reorder topics.

## Edge Cases

- **Nadia opens the reference with no connectivity** -- The full topic list and explanations render normally, since this is static content already delivered with the screen; only a dismiss action attempted while offline is held on the device and retried automatically on reconnection (FEAT-30.SPEC-004), with no error shown; if the held request is lost before reconnection, the topic shows its dismiss action again and the inline tip may reappear.
- **Nadia dismisses the same topic from this reference and from its inline tip (FEAT-30.SPEC-001) in quick succession** -- Both writes set the same tip_id to dismissed; the second write is a no-op, and both surfaces show the topic as dismissed with no error.
- **Nadia expands every topic and then collapses them all** -- The list returns to its fully collapsed default with no data loss, since nothing here is user-entered content to preserve.
- **The fixed help-content catalog is updated between one release and the next (new topic added)** -- The new topic appears in the list on the next screen load; no dismissal state exists for it yet, so it renders as not-yet-dismissed by default.
- **Nadia opens this reference before ever encountering a corresponding inline tip** -- The reference entry still shows its dismiss action, since dismissing from the reference is equally valid and suppresses the tip's future inline appearance too.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-30.SPEC-001 (Contextual Help Tooltip) | References (inbound) | Shares the same underlying tip content and dismissal state for topics that correspond to an inline tip |
| FEAT-30.SPEC-004 (Help Tip Dismissal Recording) | Triggers (outbound) | "Don't show this again" triggers the permanent dismissal write |
| FEAT-30.SPEC-005 (Contextual Help Content & Behavior Rules) | References (inbound) | Supplies topic content, dismissal-suppression checks, and the advisory-only rule |
| FEAT-12 (Freelancer Financial Dashboard) | Navigation (inbound) | Hosts this screen's entry point, including the empty-dashboard zero-state prompt |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| help_reference_opened | entry_context (dashboard screen name) | Nadia opens this screen | N/A -- no success-metrics.md metric names browsing the freelancer help reference; unlike FEAT-30.SPEC-001's inline tips (which overlay specific measured moments), this is a general lookup with no corresponding Stage 2 metric (feature-overview.md Non-Goals: no help-engagement analytics dashboard is in scope for FEAT-30). |
| help_reference_topic_expanded | topic_id | Nadia expands a topic | N/A -- same reason as help_reference_opened. |

## Acceptance Criteria

**FEAT-30.SPEC-002-AC-01:** Given Nadia is on any dashboard screen, when she taps the Help entry point, then the Freelancer Help Reference opens showing the full topic list, collapsed.

**FEAT-30.SPEC-002-AC-02:** Given Nadia is on the Freelancer Help Reference, when she taps a collapsed topic, then it expands to show its explanation inline.

**FEAT-30.SPEC-002-AC-03:** Given Nadia has expanded a topic that corresponds to an inline tip, when she taps "Don't show this again," then FEAT-30.SPEC-004 records the dismissal and the corresponding inline tip on the dashboard no longer appears on future encounters.

**FEAT-30.SPEC-002-AC-04:** Given Nadia expands a topic with no corresponding inline tip, when she looks for a dismiss action, then none is shown, and the topic always remains in the list.

**FEAT-30.SPEC-002-AC-05:** Given Nadia is on the empty (zero-state) dashboard, when the dashboard renders, then a zero-state prompt directs her toward this reference alongside the prompt to draft a first proposal (FEAT-02).

**FEAT-30.SPEC-002-AC-06:** Given Owen or Priya is signed into their client portal, when they look for any way to reach the Freelancer Help Reference, then no path exists -- this screen is not part of the client portal.

**FEAT-30.SPEC-002-AC-07:** Given Dana is in a read-only support session viewing Nadia's account, when she looks for a help entry point, then none is shown to her.

**FEAT-30.SPEC-002-AC-08:** Given Nadia loses connectivity while this reference is open, when she continues browsing topics, then every topic's explanation remains visible and expandable with no error.

**FEAT-30.SPEC-002-AC-09:** Given Nadia taps "Don't show this again" while offline, when connectivity is lost at that moment, then the dismissal is held on her device, the topic shows the "Inline tips for this topic are turned off" note meanwhile, and FEAT-30.SPEC-004 retries the write automatically once connectivity returns, with no error shown to her.

**FEAT-30.SPEC-002-AC-11:** Given Nadia has already permanently dismissed the tip for a topic (here or inline), when she opens the Freelancer Help Reference and expands that topic, then the topic and its full explanation are shown, the "Don't show this again" action is not rendered, and the static note "Inline tips for this topic are turned off" appears in its place with no restore control.

**FEAT-30.SPEC-002-AC-10:** Given Nadia taps the close/back control, when the screen closes, then she returns to the exact dashboard screen she opened this reference from.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 4 (default, topic expanded, topic already dismissed, offline/degraded) | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |

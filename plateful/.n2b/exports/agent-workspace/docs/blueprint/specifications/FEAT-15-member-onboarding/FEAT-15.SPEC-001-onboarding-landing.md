---
document_type: spec
spec_type: screen
spec_id: FEAT-15.SPEC-001
spec_name: Onboarding Landing
spec_slug: onboarding-landing
parent_feature: FEAT-15
parent_feature_name: Member Onboarding
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Screen Spec: Onboarding Landing

## Overview

**Name:** Onboarding Landing
**ID:** FEAT-15.SPEC-001
**Type:** Screen
**Purpose:** A newly accepted Other Adult Member lands directly on the household's current Weekly Plan and Grocery List, with a short role-explanation banner, the first and only time onboarding fires for their accepted invitation.
**Parent Feature:** FEAT-15 -- Member Onboarding

## Scope and Non-Goals

**In Scope:**
- Displaying the current Weekly Plan and Grocery List together on first landing, embedding the same views the household already relies on
- A role-explanation banner naming what the new Other Adult Member can do versus what the organiser manages
- The explained empty state when the household has no plan yet
- Degrading to the standard offline plan/list view when there is no connectivity at the moment of acceptance
- Guarding against direct or repeat access to this screen, in coordination with FEAT-15.SPEC-003

**Non-Goals:**
- Re-rendering the plan or grocery list's own layout, interactions, or states -- owned by FEAT-03.SPEC-001 (Weekly Plan View) / FEAT-23.SPEC-001 (Weekly Plan (Manual Week Builder)) and FEAT-06.SPEC-001 (Grocery List); this spec adds only the banner and the first-use empty-state variant (Feature Breakdown Brief, Shared UI Patterns)
- Determining who is eligible to see this screen or whether it fires more than once -- owned by FEAT-15.SPEC-003 (Onboarding Eligibility & Once-Only Rule); this screen only enforces that spec's outcome, it does not restate the rule
- A distinct onboarding-specific offline experience -- excluded per the feature's own States field: the landing degrades to the standard offline plan/list view rather than building a parallel offline treatment (feature-overview.md, Non-Goals)
- Sending a notification or email for this first-use moment -- excluded per the feature's Communications field: onboarding is an in-app experience only; the invitation itself, sent by FEAT-09, is the communication (feature-overview.md, Non-Goals)

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-15.SPEC-002 (Onboarding Trigger & Completion) | Routes here immediately after invitation acceptance succeeds, once FEAT-15.SPEC-003 confirms eligibility and once-only status | The new member's Member Profile reference; a flag marking this as the onboarding-routed landing, which drives the role-explanation banner and the first-use empty-state variant |

This screen has no other entry point: onboarding is never navigated to directly, only routed to on acceptance (Feature Breakdown Brief, Internal Dependency Map, Default Entry).

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Sam (Other Adult Member) | Yes -- only when routed here by FEAT-15.SPEC-002 for his own just-accepted invitation, and only once | Everything an ordinary Weekly Plan View and Grocery List visit allows him (view plan, open list), plus dismissing the role-explanation banner | -- |
| Maya (Organiser) | No | No | Never routed here -- the organiser creates the household and is never shown this flow (feature-overview.md, Access field); a direct request renders the ordinary Weekly Plan View instead |
| Jordan (young kid profile, no login -- MVP) | No | No | N/A -- young kid profiles never accept invitations and hold no login through which this screen could ever be requested (scope-boundaries.md SC-02) |
| Jordan (older kid, limited login -- Later) | No | No | Never routed here -- an older kid's first introduction is FEAT-17's own, not this flow (feature-overview.md, Non-Goals); a direct request renders the ordinary Weekly Plan View instead |
| Riley (Operator, support) | No | No | Never routed here -- Riley's Household Invitations access is None (Access Matrix, user-persona.md); a direct request renders no household data |
| Unauthenticated | No | No | Redirected to the sign-in screen; no household data of any kind is shown |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue."; after re-authenticating, the member lands on the ordinary Weekly Plan View, not Onboarding Landing again, since the once-only gate (FEAT-15.SPEC-003) already recorded this member's onboarding as shown |

## Layout and Content

**Header:** Product header identical to the embedded plan screen's own header (FEAT-03.SPEC-001 or FEAT-23.SPEC-001, whichever the household's tier produces). No distinct onboarding header is introduced.

**Body:**
- A role-explanation banner, positioned directly below the header and above the plan content: "Welcome, {display_name}! You can see the week's plan, suggest swaps, shop from the grocery list, and rate meals. {organiser display name} manages setup, budget, schedule, and plan approval." with a "Got it" dismiss control at its trailing edge.
- Below the banner, the household's current Weekly Plan exactly as FEAT-03.SPEC-001 (paid tier, AI-generated) or FEAT-23.SPEC-001 (free tier, manually built) specifies it -- this spec embeds that view rather than redescribing its fields, cards, or actions.
- Alongside the plan content, the entry to the shared Grocery List exactly as FEAT-03.SPEC-001/FEAT-23.SPEC-001 already provide it (e.g., an "Open grocery list" action), leading to FEAT-06.SPEC-001.
- When the household has no plan yet, the plan content region is replaced by the first-use empty-state block: a plain message, "No plan yet -- once {organiser display name} finishes setting up the household, you'll see the week's plan and list here." No grocery list entry is shown in this state, since a list is derived from a plan that does not yet exist.

**Footer:** None -- identical to the embedded plan screen, which places no persistent footer content of its own.

### Responsive Behavior

- **Compact breakpoint:** Banner stacks full width above the embedded plan content; "Got it" sits at the banner's trailing edge, wrapping to its own line if the welcome text and organiser line together exceed one line's width.
- **Medium size class and above:** Banner spans the same width as the embedded plan content's own capped, centered container; no structural change beyond width capping. The embedded plan and list content itself adapts exactly as FEAT-03.SPEC-001/FEAT-23.SPEC-001 and FEAT-06.SPEC-001 already define.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Role-explanation banner "Got it" | Tap | Collapses the banner for the remainder of this one-time viewing | Banner disappears; plan/list content shifts up to fill the space | Standard collapse transition; no confirmation message needed |
| "Open grocery list" entry (inherited from the embedded plan view) | Tap | Navigate to FEAT-06.SPEC-001 (Grocery List) | Screen transitions | Standard navigation transition |
| Embedded plan content (all of its own elements: swap taps, approval, picks, etc.) | (per FEAT-03.SPEC-001 / FEAT-23.SPEC-001) | Delegated entirely to those specs -- this spec introduces no new interaction on the embedded content itself | (per those specs) | (per those specs) |

Display-only elements: the welcome/role-explanation text itself (once rendered) and the first-use empty-state message are informational only; only the "Got it" dismiss is interactive on the banner.

### Accessibility Notes

- **Focus order:** Role-explanation banner (message, then "Got it") -> embedded plan content, in the focus order FEAT-03.SPEC-001/FEAT-23.SPEC-001 already defines -> "Open grocery list" entry.
- **Dynamic-change announcement:** The banner's collapse on "Got it" is announced to assistive technology so the resulting layout shift is not silently missed.
- **Empty-state announcement:** When the empty-state block renders in place of plan content, its message is announced on screen load, since there is no plan content whose absence would otherwise be self-evident to assistive technology.
- **Keyboard alternatives:** "Got it" and "Open grocery list" are both reachable and activatable by keyboard; there are no pointer-only gestures introduced by this spec.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Populated (default) | Role-explanation banner above the embedded Weekly Plan and an entry to the Grocery List, exactly as described in Layout and Content | The household has an existing Weekly Plan at the moment of routing | Member navigates away (to the grocery list, or elsewhere); the screen does not re-enter this state again for this member (FEAT-15.SPEC-003) |
| Empty (no plan yet) | Role-explanation banner above the first-use empty-state message; no grocery list entry shown | The household has no Weekly Plan at the moment of routing | The organiser completes setup and a plan is created -- the member's next visit is the ordinary Weekly Plan View (FEAT-03.SPEC-001/FEAT-23.SPEC-001), not a second onboarding render |
| Offline/Degraded | Standard offline plan/list treatment as FEAT-03.SPEC-001/FEAT-06.SPEC-001 already define, with the role-explanation banner still rendered from the locally available Member Profile data | Connectivity is lost at or immediately after routing | Connectivity returns -- the underlying plan/list view resumes its normal state per those specs |
| Loading | N/A -- landing in context is immediate on acceptance; there is no separate loading state distinct from the embedded plan view's own load, which those specs already cover (feature-overview.md, States) | -- | -- |
| Error | N/A -- this is a one-time guided landing, not an operation that can fail independently of invitation acceptance itself; any load failure of the embedded plan or list is handled by FEAT-03.SPEC-001/FEAT-23.SPEC-001/FEAT-06.SPEC-001's own error states (feature-overview.md, States) | -- | -- |

## Validation Rules

No user input is collected on this screen beyond the banner's dismiss control, which carries no validation. Eligibility to reach this screen at all -- and whether it may render more than once -- is governed by FEAT-15.SPEC-003 (Onboarding Eligibility & Once-Only Rule); this screen re-confirms that spec's outcome at render time rather than defining its own access rule.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| "Open grocery list" tap | FEAT-06.SPEC-001 (Grocery List) | FEAT-06 (Shared Grocery List) |
| Any further navigation from the embedded plan content (e.g., tap a meal, tap swap) | Per FEAT-03.SPEC-001 / FEAT-23.SPEC-001's own Navigation Out | FEAT-03 or FEAT-23 |
| Subsequent visit after this one-time landing | Ordinary Weekly Plan View | FEAT-03 or FEAT-23 |

## Data Model

**Creates:** None.
**Reads:** Member Profile -- display_name (personalizes the banner), member_type and status (consumed indirectly through FEAT-15.SPEC-003's eligibility confirmation, not re-validated here); Household -- the organiser's display_name (the banner's "manages setup, budget, schedule, and plan approval" line, and the empty-state message); Weekly Plan -- read exactly as FEAT-03.SPEC-001/FEAT-23.SPEC-001 define; Grocery List -- read exactly as FEAT-06.SPEC-001 defines.
**Updates:** None -- this screen performs no writes of its own; recording that onboarding has been shown is derived from the Invitation's own one-time Accepted transition (FEAT-15.SPEC-003), not written by this screen.
**Deletes:** None.

## Business Rules

- XBR-18: an accepted invitation triggers first-use onboarding exactly once; this screen only ever renders through FEAT-15.SPEC-002's routing, and only under the conditions FEAT-15.SPEC-003 confirms.
- Eligibility and once-only status are governed entirely by FEAT-15.SPEC-003 -- this screen does not restate who may reach it or how many times it may render.
- The plan and grocery list shown are exactly the views FEAT-03.SPEC-001 (or FEAT-23.SPEC-001 on the free tier) and FEAT-06.SPEC-001 already specify -- no independent rendering logic exists here (Feature Breakdown Brief, Shared UI Patterns).
- Per XBR-03 and the Grocery List Live-Update Trust success metric's clause that no household member ever sees a week's plan without its matching list, the plan and list appear together on first landing -- never one before the other.

## Edge Cases

- **Household has no plan yet at the moment of landing** -- The first-use empty state is shown (see States) rather than a blank or broken view.
- **Connectivity is lost between invitation acceptance and landing** -- Degrades to the standard offline plan/list view (FEAT-03.SPEC-001/FEAT-06.SPEC-001's own offline treatment); the role-explanation banner still renders from locally available Member Profile data, since it does not depend on connectivity.
- **Member dismisses the banner, then navigates away before viewing the plan** -- The banner does not reappear on any later visit; per FEAT-15.SPEC-003, this screen never renders a second time for the same accepted invitation, so the member's next visit is the ordinary Weekly Plan View directly.
- **Member's status changes (removed) between FEAT-15.SPEC-002's routing decision and this screen's render** -- Render-time re-confirmation (FEAT-15.SPEC-003) denies rendering; the member sees whatever a removed member sees elsewhere in the product (owned by FEAT-09/FEAT-18), never partial plan or list data.
- **No concurrent-edit conflict applies to this screen** -- This screen performs no writes to the Weekly Plan or Grocery List entities; it only embeds their read views. Contention for those entities (per the dependency map's Contention notes) is fully owned by FEAT-03.SPEC-001/FEAT-23.SPEC-001/FEAT-06.SPEC-001's own specs, which this screen does not duplicate.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-15.SPEC-002 (Onboarding Trigger & Completion) | Navigation (inbound) | Routes here once eligibility and once-only status are confirmed |
| FEAT-15.SPEC-003 (Onboarding Eligibility & Once-Only Rule) | References (inbound) | Governs who may reach this screen and whether it renders more than once |
| FEAT-03.SPEC-001 (Weekly Plan View) | References (inbound) | Embeds the paid-tier, AI-generated plan display |
| FEAT-23.SPEC-001 (Weekly Plan (Manual Week Builder)) | References (inbound) | Embeds the free-tier, manually built plan display |
| FEAT-06.SPEC-001 (Grocery List) | Navigation (outbound) | Opens the shared grocery list from the landing |
| FEAT-03.SPEC-011 (Real-Time Plan Sync) | References (inbound) | Owns live synchronization of the embedded plan content |
| FEAT-06.SPEC-005 (Live Grocery List Sync) | References (inbound) | Owns live synchronization and offline reconciliation of the shared grocery list |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| member_onboarding_shown_empty_household | member reference, household reference | The Empty (no plan yet) state renders instead of a populated plan and list | N/A -- no success-metrics.md metric carries `Connected Feature: Member Onboarding`; this event is retained per product-features.md's Signals field as the feature's own diagnostic count of how often a new member lands before household setup is finished |

No further event is defined for this screen beyond the once-only landing itself: the populated landing's own view is not separately counted here, since the routing and completion signals that mark a successful onboarding are owned by FEAT-15.SPEC-002 (member_onboarding_started, member_onboarding_completed), not restated on this screen.

## Acceptance Criteria

**FEAT-15.SPEC-001-AC-01:** Given Sam is routed to Onboarding Landing after accepting his invitation, when the screen renders, then he sees the household's current Weekly Plan and Grocery List entry together with the role-explanation banner.

**FEAT-15.SPEC-001-AC-02:** Given Sam is on a free-tier household's Onboarding Landing, when the screen renders, then it embeds the Weekly Plan (Manual Week Builder) view (FEAT-23.SPEC-001) rather than the AI-generated view.

**FEAT-15.SPEC-001-AC-03:** Given Sam sees the role-explanation banner, when he taps "Got it", then the banner collapses and the plan/list content shifts up, remaining visible for the rest of this one-time viewing.

**FEAT-15.SPEC-001-AC-04:** Given Sam is routed to Onboarding Landing and the household has no plan yet, when the screen renders, then he sees the explained empty state naming that the organiser has not finished setup, instead of a blank or broken view.

**FEAT-15.SPEC-001-AC-05:** Given Sam loses connectivity between accepting his invitation and landing, when the screen renders, then it shows the standard offline plan/list treatment with the role-explanation banner still present.

**FEAT-15.SPEC-001-AC-06:** Given Sam is on Onboarding Landing, when he taps "Open grocery list", then he navigates to FEAT-06.SPEC-001 (Grocery List).

**FEAT-15.SPEC-001-AC-07:** Given Maya (Organiser) would otherwise reach this screen, when FEAT-15.SPEC-003 is evaluated for her, then the screen never renders and she is routed to the ordinary Weekly Plan View instead.

**FEAT-15.SPEC-001-AC-08:** Given Jordan (older kid, limited login) requests this screen, when FEAT-15.SPEC-003 is evaluated, then the screen never renders for that role.

**FEAT-15.SPEC-001-AC-09:** Given Sam has already viewed Onboarding Landing once, when he opens the app again later, then he sees the ordinary Weekly Plan View, not Onboarding Landing a second time.

**FEAT-15.SPEC-001-AC-10:** Given an unauthenticated visitor requests this screen's URL directly, when the request is evaluated, then they are redirected to the sign-in screen and no household data is shown.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 2 (banner dismiss, open grocery list) | 2 |
| States | 3 (populated, empty, offline) | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |

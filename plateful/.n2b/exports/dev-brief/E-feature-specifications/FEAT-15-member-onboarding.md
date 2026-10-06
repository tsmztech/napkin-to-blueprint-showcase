# FEAT-15 — Member Onboarding

This chapter covers FEAT-15, Member Onboarding, a Important-tier feature. It contains 3 specifications carrying 29 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-15.SPEC-001 | Onboarding Landing | screen | 10 |
| FEAT-15.SPEC-002 | Onboarding Trigger & Completion | automation | 8 |
| FEAT-15.SPEC-003 | Onboarding Eligibility & Once-Only Rule | logic-rule | 11 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Member Onboarding

## Summary

**Feature:** Member Onboarding
**ID:** FEAT-15
**Description:** An adult who accepts a household invitation is guided from acceptance to seeing the current plan and grocery list for the first time.
**Priority:** Important
**Phase:** MVP
**Type:** Lifecycle
**Rationale:** The brief's growth model depends on invited members having a smooth first experience — "growth is expected to come mostly from households inviting other households" (BRIEF.md, Business Context) is a household-to-household version of the same principle, and a confusing first join would undermine it. Included at MVP alongside Household Invitations (FEAT-09), since an invitation without a guided first-use is only half the feature.

**Key Capabilities:**
- Land in context — Newly joined member is taken directly to the current plan and grocery list, not a generic empty home screen
- Understand their role — Newly joined member sees a brief explanation of what they can do (view the plan, suggest swaps, shop, rate) versus what the organiser manages

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-15.SPEC-001 | Onboarding Landing | Screen | Other Adult Member | Newly accepted member's first-use view: the current plan and grocery list plus a short role-explanation banner, with an explained empty state and an offline-degraded fallback |
| FEAT-15.SPEC-002 | Onboarding Trigger & Completion | Automation | Other Adult Member | Fires on FEAT-09's invitation-acceptance event, routes the new member to the Onboarding Landing exactly once, and emits the feature's signals |
| FEAT-15.SPEC-003 | Onboarding Eligibility & Once-Only Rule | Logic/Rule | Other Adult Member | Governs who is ever routed through onboarding (Other Adult Member only) and that a re-invited former member always onboards fresh rather than being restored to prior data |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Land in context | FEAT-15.SPEC-001 | Primary purpose of the Onboarding Landing screen — displays the current Weekly Plan and Grocery List directly, or the explained empty state if no plan exists yet | Phase 2 (Explicit) |
| Understand their role | FEAT-15.SPEC-001 | Role-explanation banner on the same screen, naming Sam's entitlements (view plan, suggest swaps, shop, rate) against the organiser's (setup, budget, schedule, plan approval, billing) per the Access Matrix | Phase 2 (Explicit) |

**Analyst-Discovered Specs** — Specs not directly tied to a Key Capability, surfaced by Phases 3–6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-15.SPEC-002 | Onboarding Trigger & Completion | Phase 4 (Trigger-Response Analysis) | Neither Key Capability describes how onboarding actually starts; the feature's own Interactions field ("Depends on Household Invitations & Membership (FEAT-09) for the acceptance event it follows") and XBR-18 require a standalone trigger-response spec that crosses a feature boundary — landing does not happen on its own |
| FEAT-15.SPEC-003 | Onboarding Eligibility & Once-Only Rule | Phase 5 (Rule-Constraint Discovery), confirmed by Phase 3 (re-join is a lifecycle question) | The Validation & Limits field ("shown exactly once per newly accepted invitation"), the Access field (only Sam's role, never Maya, either kid row, Riley, or an unauthorized visitor), and XBR-18's re-join clause are three conditional rules shared by both SPEC-001 (guarding direct access) and SPEC-002 (deciding whether to fire) — they cross the standalone-spec threshold as rules shared across multiple specs within the feature |

## Entity-Lifecycle Coverage Matrix

This feature manages no entity of its own. Its two Connected Entities are declared read-only in product-features.md ("Member Profile (read — the newly created one), Invitation (read — the accepted one)"), and the dependency map assigns their full lifecycle to other features: Member Profile is created by FEAT-01/FEAT-09 and Invitation is created and updated by FEAT-09. A full CRUD matrix would therefore be entirely N/A by definition; the Referenced Entities table below is the correct and complete coverage for a feature of this shape. The once-only tracking required by SPEC-003 rides on the Invitation's own one-time acceptance event (an Invitation transitions to Accepted exactly once, per FEAT-09) rather than requiring a new persisted flag on Member Profile — this keeps the feature within its declared read-only access to both entities.

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Invitation | FEAT-15.SPEC-002, FEAT-15.SPEC-003 | The accepted invitation whose acceptance event (owned by FEAT-09) is the sole trigger for onboarding, and whose one-time Accepted transition is what makes onboarding naturally fire once |
| Member Profile | FEAT-15.SPEC-001, FEAT-15.SPEC-002, FEAT-15.SPEC-003 | The newly created Other Adult Member profile the Landing screen personalizes for, and whose member_type/status the Automation and Eligibility Rule check to confirm the role and to distinguish a fresh join from a re-join |
| Weekly Plan | FEAT-15.SPEC-001 | Read directly on the Landing screen; owned and written by FEAT-03 (generated) or FEAT-23 (manual) |
| Grocery List | FEAT-15.SPEC-001 | Read directly on the Landing screen; owned and written by FEAT-06 |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Invited adult accepts a household invitation (FEAT-09 event) | Route the new member to the Onboarding Landing, applying the eligibility and once-only rule first | Standalone Automation | FEAT-15.SPEC-002 |
| Onboarding Trigger fires for a former member who was previously removed and is now re-invited | Route through the same onboarding flow again rather than restoring prior household data | Standalone Logic/Rule | FEAT-15.SPEC-003 |
| Onboarding Landing loads and the household has no plan yet | Show the explained empty state instead of a blank or broken view | Inline in triggering screen | FEAT-15.SPEC-001 |
| Onboarding Landing loads with no connectivity | Degrade to the standard offline plan/list view rather than a distinct onboarding-specific offline treatment | Inline in triggering screen | FEAT-15.SPEC-001 |
| Onboarding Landing displays the live current Weekly Plan and Grocery List | Real-time synchronization of plan and list content across devices | Cross-feature — owned by the real-time synchronization boundary already specified elsewhere | FEAT-03.SPEC-011 / FEAT-06.SPEC-005 |
| A role other than Other Adult Member (organiser, either kid row, Riley, or an unauthorized visitor) would otherwise reach the Onboarding Landing | Refuse to route or render; the flow is never shown | Standalone Logic/Rule | FEAT-15.SPEC-003 |

No Notification spec is produced: the feature's Communications field is explicit — "N/A — this is an in-app first-use experience, not a separate notification (the invitation itself, sent by FEAT-09, is the communication)." No Integration spec is produced: the External Touchpoints slice names none for this feature and explicitly instructs "Do not invent an external capability"; the one external-shaped dependency the Landing screen relies on (real-time sync) is already owned by FEAT-03 and FEAT-06's Integration/Automation specs, referenced above rather than duplicated.

## Shared Context

**Shared Entities:**
- Invitation — read by SPEC-002 (as the trigger source) and SPEC-003 (to determine the acceptance event and detect a re-join). Fields used: status (must be Accepted), and implicitly the invited contact detail's history for re-join detection via Member Profile status.
- Member Profile — read by all three specs. Fields used: member_type (must be Other Adult Member for eligibility), status (Invited/Active/Left/Removed — Removed-then-reinvited is the re-join case), display_name (personalizes the role-explanation banner).
- Weekly Plan and Grocery List — read only by SPEC-001, exactly as rendered by FEAT-03/FEAT-23 and FEAT-06; SPEC-001 does not reimplement their display logic, it embeds it.

**Shared UI Patterns:**
- Existing plan/list surfaces — SPEC-001 is not a new rendering of the plan and grocery list; it presents the same views FEAT-03/FEAT-23 and FEAT-06 already specify, adding only the role-explanation banner and the first-use empty-state variant unique to onboarding. The Spec Writer for SPEC-001 should describe the banner and empty-state addition, not redescribe the underlying plan/list layouts.

**Shared Validation:**
- FEAT-15.SPEC-003 is the single source for "who is eligible" and "fires once, and a re-join is always fresh." SPEC-001 references it to refuse rendering for an ineligible role or a second showing; SPEC-002 references it to decide whether to fire at all. Neither spec should restate the eligibility or once-only logic independently.

## Internal Dependency Map

```
[FEAT-09 invitation-acceptance event] -> [invited adult accepts] -> SPEC-002 (Onboarding Trigger & Completion)
SPEC-002 (Onboarding Trigger & Completion) -> [checks eligibility and once-only status via] -> SPEC-003 (Onboarding Eligibility & Once-Only Rule)
SPEC-002 (Onboarding Trigger & Completion) -> [eligible and not yet shown] -> SPEC-001 (Onboarding Landing)
SPEC-001 (Onboarding Landing) -> [guards direct or repeat access via] -> SPEC-003 (Onboarding Eligibility & Once-Only Rule)
SPEC-001 (Onboarding Landing) -> [displays] -> FEAT-03 / FEAT-23 (current Weekly Plan)
SPEC-001 (Onboarding Landing) -> [displays] -> FEAT-06 (current Grocery List)
```

**Default Entry:** SPEC-001 (Onboarding Landing) — but only ever reached by way of SPEC-002's trigger; the feature has no independent navigation entry point since onboarding is never navigated to directly, only routed to on acceptance.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-15.SPEC-002 | Inbound | FEAT-09 (Household Invitations & Membership) | Onboarding starts only when an invitation is accepted and a Member Profile with Other Adult Member access is created | Invited adult accepts their invitation (XBR-18) |
| FEAT-15.SPEC-003 | Inbound | FEAT-09 (Household Invitations & Membership) | Re-join detection reads Member Profile status set by FEAT-09 when a member leaves or is removed and later re-invited | A previously removed member accepts a new invitation |
| FEAT-15.SPEC-001 | Outbound | FEAT-03 (AI Weekly Dinner Plan Generation) | Landing shows the current week's AI-generated plan | New member lands in context on a paid-tier household with a generated plan |
| FEAT-15.SPEC-001 | Outbound | FEAT-23 (Manual Weekly Planning) | Landing shows the current manually built week | New member lands in context on a free-tier household with a manually built plan |
| FEAT-15.SPEC-001 | Outbound | FEAT-06 (Shared Grocery List) | Landing shows and opens the shared grocery list alongside the plan | New member views or opens the list from the landing view |

## Non-Functional Notes

**Data volumes / growth:** N/A — onboarding is a single, per-member, per-acceptance event rather than a growing data set; it produces no stored records of its own, only signal emissions, so it carries no independent volume or growth profile (product-features.md, Data Notes: "Derived: none. Source: existing household data").

**Responsiveness:** The Onboarding Landing reuses the same plan and grocery list surfaces the household already relies on, so it inherits their responsiveness bar rather than defining a new one: the list must feel instant (ASMP-22), and — per the Grocery List Live-Update Trust success metric's clause that "no household member ever sees a week's plan without its matching list" — the first-time landing must show the plan and list together, not the plan before the list or vice versa.

**Data sensitivity / privacy:** The feature reads existing household personal data (Weekly Plan, Grocery List — private to the household, never sold, ASMP-14/ASMP-26) and an adult Member Profile (personal sign-in and display data protected under account-protection expectations); it never touches kid-profile data, since neither kid row ever goes through this flow (Access field). No new sensitive data is created — only read and briefly displayed.

**Compliance flags:** N/A — no health, financial, or children's-privacy regime applies to this feature specifically; it is scoped to the Other Adult Member role only and never surfaces kid-profile data (ASMP-26 governs Household Setup & Member Profiles, not this feature's read-only display).

**Signals:** member_onboarding_started and member_onboarding_completed are emitted by FEAT-15.SPEC-002 at the start and successful completion of a routed onboarding; member_onboarding_shown_empty_household is emitted by FEAT-15.SPEC-001 when the empty-state variant renders instead of a populated plan and list. All three trace directly to the feature's Signals field in product-features.md.

## Non-Goals

- **Restoring a re-joined member's prior household data automatically** — Excluded per XBR-18 and the feature's Re-join flow line: "a previously removed member who is re-invited goes through the same onboarding again rather than being silently restored to old data." FEAT-15.SPEC-003 makes this an explicit rule rather than a silent omission.
- **A dedicated onboarding notification or email** — Excluded per the feature's Communications field ("N/A — this is an in-app first-use experience, not a separate notification"). The invitation message itself remains FEAT-09's responsibility; onboarding never sends its own message through any channel.
- **Onboarding for the organiser, either kid row, or Riley (Operator, support)** — Excluded per the Access field: the organiser creates the household and is never shown this flow, young kid profiles never accept invitations, a Later-phase older-kid login gets its own introduction under FEAT-17 (not this feature), and Riley's Household Invitations access is None. Grounded in the Access Matrix (user-persona.md).
- **A distinct onboarding-specific offline experience** — Excluded by the feature's own States field: "the landing view degrades to the standard offline plan/list view if there is no connectivity at the moment of acceptance." Onboarding intentionally reuses FEAT-03/FEAT-06's existing offline behavior rather than building a parallel one.
- **Multiple households or a household-selection step during onboarding** — Excluded per scope-boundaries.md (SC-03): "there is one household per account in v1"; a newly joined member never has more than one household to be routed into, so no selection step is needed.



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



# Automation Spec: Onboarding Trigger & Completion

## Overview

**Name:** Onboarding Trigger & Completion
**ID:** FEAT-15.SPEC-002
**Type:** Automation
**Purpose:** Fires when an invited adult's invitation acceptance succeeds, confirms eligibility and once-only status through FEAT-15.SPEC-003, routes the new member to the Onboarding Landing exactly once, and emits the feature's start and completion signals.
**Parent Feature:** FEAT-15 -- Member Onboarding

## Scope and Non-Goals

**In Scope:**
- Receiving the acceptance outcome from FEAT-09.SPEC-007 (Invitation Acceptance Processing)
- Checking eligibility and once-only status via FEAT-15.SPEC-003 before routing
- Routing the new member to FEAT-15.SPEC-001 (Onboarding Landing) when eligible and not yet shown
- Falling back to the ordinary Weekly Plan View when onboarding does not fire
- Emitting member_onboarding_started and member_onboarding_completed

**Non-Goals:**
- Creating the Member Profile or transitioning the Invitation to Accepted -- owned by FEAT-09.SPEC-007; this automation only reacts to that outcome, it does not perform or duplicate it
- Determining who is eligible or whether onboarding has already fired for a given invitation -- owned by FEAT-15.SPEC-003; this automation calls that rule rather than restating its logic
- Rendering the plan and grocery list -- owned by FEAT-15.SPEC-001, FEAT-03.SPEC-001/FEAT-23.SPEC-001, and FEAT-06.SPEC-001
- Sending any notification or email for this moment -- excluded per the feature's Communications field: onboarding has no message of its own; the invitation itself (FEAT-09) is the communication (feature-overview.md, Non-Goals)

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Invitation acceptance succeeds | FEAT-09.SPEC-007 (Invitation Acceptance Processing) | Fires on that automation's "Acceptance succeeds" outcome (new Member Profile created, Invitation.status set to Accepted) | The new Member Profile reference (member_type, status, display_name) and the just-accepted Invitation reference (status, and whether its contact detail maps to a previously Removed or Left Member Profile of this household) |

This is the feature's only trigger path -- an event-driven trigger sourced from a specific outcome of a spec in another feature (FEAT-09), consistent with the Internal Dependency Map's statement that landing "does not happen on its own."

## Processing Logic

1. Receive the newly created Member Profile reference and the just-accepted Invitation reference from FEAT-09.SPEC-007's "Acceptance succeeds" outcome.
2. Emit member_onboarding_started, recording the trigger moment.
3. Evaluate eligibility per FEAT-15.SPEC-003: confirm the Member Profile's member_type is Other Adult Member.
4. Evaluate once-only status per FEAT-15.SPEC-003: confirm onboarding has not already been shown for this accepted Invitation's transition.
5. Evaluate re-join status per FEAT-15.SPEC-003: determine whether the Member Profile's status immediately before this acceptance was Removed or Left.
6. If eligible and not yet shown, route the member to FEAT-15.SPEC-001 (Onboarding Landing), passing the Member Profile reference. This applies identically whether or not step 5 flagged a re-join -- a re-join always onboards fresh, with no prior data read or applied.
7. If ineligible, or if onboarding has already been shown for this transition, do not route to Onboarding Landing; instead route the member to the ordinary Weekly Plan View (FEAT-03.SPEC-001 or FEAT-23.SPEC-001, per the household's tier) as their first screen.
8. When FEAT-15.SPEC-001 successfully completes its first render for this member (whether the populated or the explained empty-state variant), emit member_onboarding_completed.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Onboarding fires -- eligible, first time | FEAT-15.SPEC-003 confirms Other Adult Member and no prior onboarding shown for this Invitation's transition | None persisted by this automation itself (FEAT-15 owns no entity of its own) | Member is routed to and lands on FEAT-15.SPEC-001 | FEAT-15.SPEC-001 |
| Onboarding fires -- eligible re-join | Same as above, and FEAT-15.SPEC-003 additionally flags this as a re-join (prior status Removed or Left) | None; no prior data restored | Member is routed to FEAT-15.SPEC-001, landing exactly as a first-time member would | FEAT-15.SPEC-001, FEAT-15.SPEC-003 |
| No onboarding -- ineligible role | FEAT-15.SPEC-003 determines the accepted role is not Other Adult Member (evaluated defensively; FEAT-09.SPEC-007 always creates Other Adult Member profiles on acceptance, so this path is not expected to occur in practice) | None | Member is routed to the ordinary Weekly Plan View instead | FEAT-03.SPEC-001, FEAT-23.SPEC-001 |
| No onboarding -- already shown | FEAT-15.SPEC-003 determines onboarding was already shown for this accepted Invitation's transition (e.g., a duplicate event delivery, or a reload reaching this automation again) | None | Member is routed to the ordinary Weekly Plan View | FEAT-03.SPEC-001, FEAT-23.SPEC-001 |
| Automation failure | A processing error occurs between receiving the acceptance outcome and completing the eligibility check | None -- the completed Member Profile and Invitation Accepted state from FEAT-09.SPEC-007 are unaffected | Member is routed to the ordinary Weekly Plan View as a safe, non-blocking default; onboarding is simply not shown this one time | FEAT-03.SPEC-001, FEAT-23.SPEC-001 |

## Data Model

**Reads:** Member Profile -- member_type, status, display_name (FEAT-15.SPEC-003's eligibility and re-join inputs). Invitation -- status, and whether the invited contact detail maps to a previously Removed or Left Member Profile of this household (re-join detection).
**Creates:** None.
**Updates:** None -- once-only tracking rides on the Invitation's own one-time Accepted transition (owned by FEAT-09), not on a new field this automation writes, keeping the feature within its declared read-only access to both entities (feature-overview.md, Entity-Lifecycle Coverage Matrix).
**Deletes:** None.

## Business Rules

- XBR-18: an accepted invitation triggers first-use onboarding exactly once per accepted invitation; this automation fires at most once per Invitation's Accepted transition, never on a reload or re-entry into an already-processed transition.
- Eligibility and once-only status are governed entirely by FEAT-15.SPEC-003 -- this automation calls that rule rather than re-implementing it.
- A re-invited former member is always routed through onboarding again rather than being silently restored to old data (XBR-18, feature-overview.md Non-Goals).
- This automation runs synchronously as part of the acceptance flow -- the member is routed before FEAT-09.SPEC-002's Accept & Join screen is considered fully resolved.

## Edge Cases

- **Member reloads or re-enters mid-viewing of Onboarding Landing** -- Per FEAT-15.SPEC-003, onboarding was already shown for this transition; this automation is not re-triggered by a reload (its only trigger is FEAT-09.SPEC-007's acceptance outcome, which fires once), and FEAT-15.SPEC-001 itself re-confirms once-only status at render time.
- **Household has no plan yet at routing time** -- Routing still proceeds to FEAT-15.SPEC-001, which renders its own explained empty state; this automation does not branch on plan existence.
- **FEAT-09.SPEC-007's acceptance outcome is a race-lost rejection** (invitation already Accepted, Revoked, or Expired) -- This automation never fires, since its only trigger is specifically FEAT-09.SPEC-007's "Acceptance succeeds" outcome.
- **Concurrent trigger firing** (two different invited adults' acceptances resolve at effectively the same time) -- Each acceptance is a distinct Invitation and Member Profile; this automation runs once per Invitation independently, and neither run reads or affects the other's Member Profile or routing decision.
- **Trigger fires while a previous run is still in flight** -- Cannot occur under ordinary use: FEAT-09.SPEC-010's race resolution ensures only the first acceptance for a given Invitation ever reaches FEAT-09.SPEC-007's "Acceptance succeeds" outcome, so this automation is never started twice for the same transition; a second acceptance attempt for the same Invitation is rejected upstream and never reaches this automation.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-007 (Invitation Acceptance Processing) | Triggered by (inbound) | Fires on that automation's "Acceptance succeeds" outcome |
| FEAT-15.SPEC-003 (Onboarding Eligibility & Once-Only Rule) | References (outbound) | Eligibility, once-only, and re-join status checked before routing |
| FEAT-15.SPEC-001 (Onboarding Landing) | Affects (outbound) | Routes the eligible, not-yet-shown member here |
| FEAT-03.SPEC-001 (Weekly Plan View) | Affects (outbound) | Fallback destination when onboarding does not fire, paid tier |
| FEAT-23.SPEC-001 (Weekly Plan (Manual Week Builder)) | Affects (outbound) | Fallback destination when onboarding does not fire, free tier |

## Analytics and Success Signals

- **member_onboarding_started** (trigger source: invitation acceptance) -- N/A -- no success-metrics.md metric carries `Connected Feature: Member Onboarding`; retained per product-features.md's Signals field as the feature's own funnel-start count.
- **member_onboarding_completed** (elapsed time from started to completed) -- N/A -- same reason; retained as the feature's own funnel-completion count.

## Acceptance Criteria

**FEAT-15.SPEC-002-AC-01:** Given Sam's invitation acceptance succeeds via FEAT-09.SPEC-007, when this automation receives the outcome, then it emits member_onboarding_started and evaluates his eligibility via FEAT-15.SPEC-003.

**FEAT-15.SPEC-002-AC-02:** Given FEAT-15.SPEC-003 confirms Sam is an eligible Other Adult Member with no prior onboarding shown for this transition, when evaluation completes, then Sam is routed to FEAT-15.SPEC-001 (Onboarding Landing).

**FEAT-15.SPEC-002-AC-03:** Given Sam is a previously removed member who has just been re-invited and accepted, when this automation processes his acceptance, then FEAT-15.SPEC-003 flags the re-join and Sam is still routed to FEAT-15.SPEC-001 with no prior data restored.

**FEAT-15.SPEC-002-AC-04:** Given FEAT-15.SPEC-001 successfully renders to Sam for the first time, when the landing's first paint completes, then this automation emits member_onboarding_completed.

**FEAT-15.SPEC-002-AC-05:** Given Sam reloads or re-enters the app after onboarding has already been shown once for his acceptance, when this is evaluated, then FEAT-15.SPEC-003 reports already-shown and Sam is routed to the ordinary Weekly Plan View instead of Onboarding Landing again.

**FEAT-15.SPEC-002-AC-06:** Given the household has no plan yet at the moment Sam is routed, when routing completes, then Sam still lands on FEAT-15.SPEC-001, which renders its own explained empty state.

**FEAT-15.SPEC-002-AC-07:** Given a processing error occurs between receiving the acceptance outcome and completing the eligibility check, when the failure occurs, then Sam is routed to the ordinary Weekly Plan View as a safe default and onboarding is not shown this one time.

**FEAT-15.SPEC-002-AC-08:** Given two different invited adults' acceptances resolve at effectively the same time, when both reach this automation, then each is processed independently against its own Invitation and Member Profile with no interference between the two runs.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 5 (eligible first-time, eligible re-join, ineligible role, already shown, automation failure) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Logic/Rule Spec: Onboarding Eligibility & Once-Only Rule

## Overview

**Name:** Onboarding Eligibility & Once-Only Rule
**ID:** FEAT-15.SPEC-003
**Type:** Logic/Rule
**Purpose:** Governs which role is ever routed through Member Onboarding, and enforces that onboarding fires exactly once per accepted invitation -- including the case where a re-invited former member onboards fresh rather than being restored to prior data.
**Parent Feature:** FEAT-15 -- Member Onboarding
**Governed Entity:** Member Profile (the onboarding-relevant fields consulted for routing), read alongside the paired Invitation's status.

## Scope and Non-Goals

**In Scope:**
- The eligibility rule: only a Member Profile with member_type Other Adult Member is ever routed through onboarding
- The once-only rule: onboarding is shown exactly once per accepted Invitation's transition
- The re-join rule: a previously Removed or Left member who is re-invited and accepts again is eligible and onboards fresh, with no data restoration
- Authorization Rules for reaching (being routed to) and rendering (viewing) the Onboarding Landing, across every role in the Access Matrix

**Non-Goals:**
- Creating or updating the Member Profile or Invitation records themselves -- owned by FEAT-01 and FEAT-09; this spec only reads their status and member_type to decide routing, per the feature's declared read-only access to both entities (feature-overview.md, Entity-Lifecycle Coverage Matrix)
- Restoring a re-joined member's dietary rules, ratings, or prior household history -- excluded per XBR-18 and the feature's Non-Goals: a re-join always onboards fresh rather than being silently restored to old data
- Field-level validation of Member Profile or Invitation data unrelated to onboarding routing (e.g., display_name length, sign_in format, contact_detail format) -- owned by FEAT-01.SPEC-014 and FEAT-09.SPEC-010 respectively; this spec addresses only the fields it actually reads for routing decisions

## Governed Entity

**Entity:** Member Profile
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| display_name | text | First name or nickname; read only to personalize the role-explanation banner on FEAT-15.SPEC-001, not a routing input |
| member_type | enum | Organiser, Other Adult Member, young kid profile (no login), or older kid limited login (Later); the sole role-eligibility gate for onboarding |
| sign_in | text | Email and protected sign-in for adults only; outside this spec's read scope |
| age_band | enum | Kid profiles only; never populated for an Other Adult Member and never read by onboarding |
| parental_consent_confirmation | boolean | Kid profiles only; never populated for an Other Adult Member and never read by onboarding |
| notification_preferences | derived | Per-member plan-ready/nudge toggles; outside this spec's read scope |
| status | enum | Invited, Active, Left, or Removed; paired with the Invitation's Accepted transition to determine both eligibility-to-fire-now and re-join status |

A second entity is consulted alongside Member Profile but is not independently governed here: **Invitation.status** (Sent, Accepted, Revoked, Expired), whose own field validation, race resolution, and lifecycle are owned by FEAT-09.SPEC-010. This spec reads only whether the specific Invitation this evaluation concerns has just transitioned to Accepted -- it does not restate Invitation's own rules.

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-15.SPEC-002 | Onboarding Trigger & Completion | On the invitation-acceptance trigger, before any routing decision is made |
| FEAT-15.SPEC-001 | Onboarding Landing | On screen entry -- re-confirms authorization at render time, catching any change between routing and render |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| member_type | Must equal Other Adult Member for this Member Profile to ever be eligible for onboarding | Always | On automation trigger (FEAT-15.SPEC-002) and on screen entry (FEAT-15.SPEC-001) | N/A -- this is a routing rule, not a form field; an ineligible member_type produces no error message, only a silent redirect away from onboarding (see Authorization Rules) | Yes (blocking for onboarding routing) |
| status | Must be Active, and that Active status must be the direct result of the specific Invitation transition currently being evaluated (not a stale Active status from long-standing membership), for onboarding to fire now | Always | On automation trigger and on screen entry | N/A -- routing rule, not a form field | Yes |
| display_name | No validation beyond data type -- already validated at profile creation by FEAT-01.SPEC-014/FEAT-09.SPEC-007; onboarding only reads it to personalize the role-explanation banner | Always | -- | -- | No |
| sign_in | No validation beyond data type -- outside this spec's read scope | Always | -- | -- | No |
| age_band | No validation beyond data type -- never populated for an Other Adult Member; onboarding never reads it | Always | -- | -- | No |
| parental_consent_confirmation | No validation beyond data type -- never populated for an Other Adult Member; onboarding never reads it | Always | -- | -- | No |
| notification_preferences | No validation beyond data type -- outside this spec's read scope | Always | -- | -- | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Onboarding eligibility | member_type, status | member_type must equal Other Adult Member AND status must be Active as a direct result of the specific Invitation transition this onboarding run is evaluating | N/A -- no user-facing error message; an ineligible combination routes silently to the ordinary Weekly Plan View (see Authorization Rules) |
| Once-only gate | status, Invitation.status (paired) | Onboarding may fire only on the transition where the paired Invitation's status becomes Accepted for the first time; once FEAT-15.SPEC-001 has rendered for that transition, no later evaluation of the same transition may fire onboarding again | N/A -- silent routing away, not a blocking error |
| Re-join freshness | status (prior value: Removed or Left; current value: Active via a new Invitation) | A Member Profile whose status was Removed or Left immediately before this acceptance, and is now Active via a newly accepted Invitation, is eligible exactly as a first-time member -- no lookup against the prior profile's dietary rules, ratings, or history is performed or permitted | N/A -- data-isolation rule, not a validation failure |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Be routed to Onboarding Landing (fire onboarding) | Sam (Other Adult Member) | Only on the specific Invitation-acceptance transition FEAT-15.SPEC-002 is currently evaluating, and only once per that transition | -- |
| Be routed to Onboarding Landing (fire onboarding) | Maya (Organiser) | Never -- the organiser creates the household and is never shown this flow | Structurally unreachable (organiser accounts are never produced by an invitation acceptance); if ever evaluated, routing goes to the ordinary Weekly Plan View instead, with no error message |
| Be routed to Onboarding Landing (fire onboarding) | Jordan (young kid profile, no login -- MVP) | Never -- young kid profiles never accept invitations | Structurally unreachable; no login exists through which this role could ever receive this routing |
| Be routed to Onboarding Landing (fire onboarding) | Jordan (older kid, limited login -- Later) | Never -- gets its own short introduction under FEAT-17, not this flow | Routing to FEAT-15.SPEC-001 is never evaluated for this role; FEAT-17 owns its own first-use path |
| Be routed to Onboarding Landing (fire onboarding) | Riley (Operator, support) | Never -- Riley's Household Invitations access is None | Structurally unreachable; Riley has no acceptance event that could trigger this rule |
| Render Onboarding Landing (direct or repeat request) | Sam (Other Adult Member) | Only once per accepted Invitation's transition, and only immediately following FEAT-15.SPEC-002's routing for that transition | Any later or direct request (reload, re-entry, deep link) renders the ordinary Weekly Plan View instead -- never a second onboarding render, and never an error message |
| Render Onboarding Landing (direct or repeat request) | Maya (Organiser) | Never | Any request renders the ordinary Weekly Plan View instead; Onboarding Landing itself never renders |
| Render Onboarding Landing (direct or repeat request) | Jordan (young kid profile, no login -- MVP) | Never | No request is possible without a login; Onboarding Landing never renders for this role |
| Render Onboarding Landing (direct or repeat request) | Jordan (older kid, limited login -- Later) | Never | Any request renders that role's own standard landing instead; Onboarding Landing itself never renders |
| Render Onboarding Landing (direct or repeat request) | Riley (Operator, support) | Never | Any request shows no household data; Onboarding Landing itself never renders |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| "onboarding-shown" evaluation | Derived at each check from whether FEAT-15.SPEC-001 has already rendered for the current Invitation's Accepted transition -- read from the Invitation's own one-time Accepted state, not a separate stored flag (feature-overview.md, Entity-Lifecycle Coverage Matrix) | On every routing decision (FEAT-15.SPEC-002) and every render-time re-confirmation (FEAT-15.SPEC-001) | No -- system-derived gate, not user-overridable |
| "is-re-join" flag | Derived by checking whether the Member Profile's status immediately before this Accepted transition was Removed or Left | On every routing decision | No |

## Business Rules

- XBR-18 (owned jointly with FEAT-09): an accepted invitation creates a Member Profile with Other Adult Member access and triggers first-use onboarding exactly once per accepted invitation; a re-invited former member onboards again rather than being restored to old data.
- The once-only gate rides on the Invitation's own one-time Accepted transition (FEAT-09), not on a new persisted flag on Member Profile -- this keeps the feature within its declared read-only access to both entities (feature-overview.md, Entity-Lifecycle Coverage Matrix).
- Eligibility, once-only status, and re-join freshness are the single source of truth for FEAT-15.SPEC-001 and FEAT-15.SPEC-002 -- neither spec restates this logic independently (Feature Breakdown Brief, Shared Validation).

## Edge Cases

- **Member Profile's member_type is somehow not Other Adult Member at evaluation time** (e.g., a hypothetical future role change mid-flow) -- Routing is denied for that transition; the member sees the ordinary Weekly Plan View, never an error message, since this is a routing gate, not a form validation.
- **Invitation's Accepted transition is evaluated twice in rapid succession** (e.g., a duplicate event delivery) -- The once-only gate treats the second evaluation as already-shown, since FEAT-15.SPEC-001 has already rendered (or is already rendering) for that same transition; no second onboarding fires.
- **A member is Removed and re-invited multiple times across the product's lifetime** -- Each new accepted Invitation is evaluated independently; every one is a fresh eligible transition, and each onboards exactly once, with no memory carried between onboarding runs beyond the current transition's own eligibility check.
- **Member Profile status reads Active but the specific Invitation this evaluation is tied to is not the one that produced the current Active status** (e.g., a long-since-onboarded member) -- Ineligible for firing again; only the Invitation transition currently being evaluated can ever satisfy the once-only gate, never a historical Active status alone.
- **Authorization boundary crossed mid-flow** -- a member's status changes (e.g., removed) between FEAT-15.SPEC-002's routing decision and FEAT-15.SPEC-001's render -- Render-time re-confirmation (Enforced By) catches this: if status is no longer Active, the screen does not render and the member instead sees whatever a removed member sees elsewhere in the product (owned by FEAT-09/FEAT-18, not restated here).

## Acceptance Criteria

**FEAT-15.SPEC-003-AC-01:** Given Sam's Member Profile has member_type Other Adult Member and status Active from the Invitation acceptance currently being evaluated, when eligibility is checked, then Sam is confirmed eligible and onboarding is permitted to fire.

**FEAT-15.SPEC-003-AC-02:** Given a hypothetical evaluation where the accepted role's member_type is not Other Adult Member, when eligibility is checked, then onboarding is denied and the member is routed to the ordinary Weekly Plan View instead, with no error message shown.

**FEAT-15.SPEC-003-AC-03:** Given Sam is a previously Removed member whose new Invitation has just been accepted, when re-join status is checked, then he is flagged as a re-join and still confirmed eligible, with no lookup performed against his prior profile's data.

**FEAT-15.SPEC-003-AC-04:** Given onboarding has already rendered once for Sam's current accepted Invitation, when the once-only gate is evaluated again (e.g., a duplicate event), then it reports already-shown and onboarding does not fire a second time.

**FEAT-15.SPEC-003-AC-05:** Given Sam has already viewed Onboarding Landing once, when he reloads or re-enters the app later, then render-time re-confirmation denies a second render and he sees the ordinary Weekly Plan View instead.

**FEAT-15.SPEC-003-AC-06:** Given Maya (Organiser) is evaluated against "Be routed to Onboarding Landing," when the check runs, then the action is always denied and she is never routed to this flow.

**FEAT-15.SPEC-003-AC-07:** Given Jordan (young kid profile, no login) is evaluated against "Be routed to Onboarding Landing," when the check runs, then the action is always denied, since young kid profiles never accept invitations and hold no login to be routed through.

**FEAT-15.SPEC-003-AC-08:** Given Jordan (older kid, limited login) is evaluated against "Render Onboarding Landing," when the check runs, then the action is always denied and no path to this screen is ever produced for that role.

**FEAT-15.SPEC-003-AC-09:** Given Riley (Operator, support) attempts any request that would render Onboarding Landing, when the check runs, then the action is denied and Riley sees no household data through this path.

**FEAT-15.SPEC-003-AC-10:** Given an unauthenticated visitor requests the Onboarding Landing screen directly, when the request is evaluated, then rendering is denied and the visitor is redirected to sign-in.

**FEAT-15.SPEC-003-AC-11:** Given a member's status changes to removed between FEAT-15.SPEC-002's routing decision and FEAT-15.SPEC-001's render, when the screen re-confirms authorization at render time, then rendering is denied and the member does not see any plan or list data.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 7 | 7 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 10 | 10 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |

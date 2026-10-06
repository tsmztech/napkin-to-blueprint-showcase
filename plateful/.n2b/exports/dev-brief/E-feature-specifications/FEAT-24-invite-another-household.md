# FEAT-24 — Invite Another Household

This chapter covers FEAT-24, Invite Another Household, a Important-tier feature. It contains 7 specifications carrying 69 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-24.SPEC-001 | Invite Another Household Screen | screen | 10 |
| FEAT-24.SPEC-002 | Referral Welcome Screen | screen | 10 |
| FEAT-24.SPEC-003 | Personal Referral Link Provisioning | automation | 8 |
| FEAT-24.SPEC-004 | Household Referral Recording | automation | 10 |
| FEAT-24.SPEC-005 | Referral Upgrade Tracking | automation | 7 |
| FEAT-24.SPEC-006 | Household Referral Rules | logic-rule | 15 |
| FEAT-24.SPEC-007 | Referral Joined Notification | notification | 9 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Invite Another Household

## Summary

**Feature:** Invite Another Household
**ID:** FEAT-24
**Description:** Any adult in a household can share Plateful with another family through a personal invite link. When that family sets up its own household from the link, Plateful records which household invited them.
**Priority:** Important
**Phase:** MVP
**Type:** User-Facing
**Rationale:** BRIEF.md, Business Context expects growth "mostly from households inviting other households," and BRIEF.md, Success Criteria states "most paying households were invited by another household." Household Invitations & Membership (FEAT-09) adds people to one household; nothing in the draft let one household bring in another or recorded where a new household came from, so the brief's growth criterion could be neither supported nor measured. Important rather than Core because the plan-and-list loop works without it; MVP because the first users are parents from the founder's kids' school and online parenting groups, where word of mouth starts on day one. No rewards or credits are attached — the brief names none.

**Key Capabilities:**
- Share an invite link — An adult member shares a personal link through any messaging they already use
- Start a household from a link — A new family following the link begins its own household setup, with the referral recorded
- See who joined — The inviting member sees how many families have set up a household from their link

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-24.SPEC-001 | Invite Another Household Screen | Screen | Maya, Sam | Adult member sees their share link, the number of families who joined, and shares the link through their own messaging |
| FEAT-24.SPEC-002 | Referral Welcome Screen | Screen | Unauthorized Visitor | A visitor following the link sees a welcome page naming the inviter and starts their own household setup, or is told they already have one |
| FEAT-24.SPEC-003 | Personal Referral Link Provisioning | Automation | Maya, Sam | Creates and persists an adult member's one reusable personal link the first time it is needed |
| FEAT-24.SPEC-004 | Household Referral Recording | Automation | Maya, Sam | Attributes a completed new-household setup to the inviting household's link, enforcing the one-referral and no-self-referral rules |
| FEAT-24.SPEC-005 | Referral Upgrade Tracking | Automation | Maya, Sam | Updates a recorded referral's upgraded flag when the referred household's subscription becomes paid |
| FEAT-24.SPEC-006 | Household Referral Rules | Logic/Rule | All | Governs the one-link-per-member limit, single-attribution and no-self-referral rules, the 30-day counting window, the already-has-a-household disposition, and who may see or act on referrals |
| FEAT-24.SPEC-007 | Referral Joined Notification | Notification | Maya, Sam | Tells the inviting member when a family they invited finishes setting up its household |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Share an invite link | FEAT-24.SPEC-001, FEAT-24.SPEC-003 | Screen shows and shares the link; provisioning automation creates the one reusable link per adult member | Phase 2 (Explicit) |
| Start a household from a link | FEAT-24.SPEC-002, FEAT-24.SPEC-004, FEAT-24.SPEC-006 | Welcome screen hands the visitor off into FEAT-01 setup; recording automation attributes the referral once setup completes; rules govern the 30-day window, self-referral block, and already-has-household disposition | Phase 2 (Explicit) |
| See who joined | FEAT-24.SPEC-001, FEAT-24.SPEC-005, FEAT-24.SPEC-007 | Screen displays the count of joined families; upgrade tracking derives whether they went on to pay; notification tells the inviter the moment one finishes setup | Phase 2 (Explicit) |

**Analyst-Discovered Specs** — Specs not directly tied to a single Key Capability, surfaced by Phases 3–6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-24.SPEC-003 | Personal Referral Link Provisioning | Phase 4 (Trigger-Response — default data generation) | Validation & Limits states "one reusable personal link per adult member"; nothing in the explicit capabilities names the process that creates and persists that link the first time a member needs it |
| FEAT-24.SPEC-004 | Household Referral Recording | Phase 4 (Trigger-Response, cross-entity and cross-feature effect) | A new household's setup completing in FEAT-01 must attribute a referral against the inviting household (XBR-20) — a cross-feature effect with enough processing logic (attribution, self-referral and multi-attribution checks) to need its own Automation rather than living inline on either screen |
| FEAT-24.SPEC-005 | Referral Upgrade Tracking | Phase 3 (Entity-Lifecycle Analysis — Update) | The Household Referral CRUD matrix's Update operation was empty until the `upgraded` field's source — the referred household's Subscription, per Data Notes and the dependency map's Household Referral lifecycle — was traced to a spec |
| FEAT-24.SPEC-006 | Household Referral Rules | Phase 5 (Rule-Constraint Discovery) | The Validation & Limits field's three interacting conditions (one link per member, single referring household, no self-referral) plus the 30-day counting window (XBR-20) and the Access field's role split (Maya/Sam Full, both Jordan rows and Riley None, unauthorized visitor link-only) cross the standalone-spec threshold |
| FEAT-24.SPEC-007 | Referral Joined Notification | Phase 4 (Notification surfacing) | The Communications field states "the inviting member gets an in-app note when a family they invited finishes setting up" — a message with a defined audience, content, and trigger, not a same-screen toast |

## Entity-Lifecycle Coverage Matrix

**Entity: Household Referral**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-24.SPEC-004 | System writes the record once a new household's setup completes within 30 days of opening the link | Governed by FEAT-24.SPEC-006's attribution rules |
| Read (single) | N/A | No screen surfaces an individual referral record; only the aggregate count and derived upgrade share are shown, per the Data Notes field — an explicit non-goal, not an omission | -- |
| Read (list) | FEAT-24.SPEC-001, FEAT-24.SPEC-005 | Invite screen reads the count of referrals for display; upgrade tracking reads records to evaluate against Subscription changes | -- |
| Update | FEAT-24.SPEC-005 | Sets the `upgraded` flag when the referred household's Subscription becomes paid | The only field this feature ever updates on the record |
| Delete/Archive | N/A | The record is written once and never edited by any role beyond the `upgraded` flag (dependency map, Household Referral Contention) and is kept as the product's only record of household-to-household growth for success measurement (dependency map, Household Referral Data Sensitivity) — no deletion or purge mechanism is defined, recorded as an explicit non-goal. Cascade behavior when a referenced household is deleted is owned by FEAT-18 (Account & Data Management), not this feature — flagged here, not resolved, per this feature's decision authority | -- |
| State Transition | N/A | The record carries no lifecycle states beyond its one field update (`upgraded` false → true); it is never itself transitioned through statuses | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Household | FEAT-24.SPEC-001, FEAT-24.SPEC-002, FEAT-24.SPEC-004, FEAT-24.SPEC-006 | Screens and rules read the inviting household's identity and member/link ownership, and check whether a visitor already belongs to a household |
| Subscription | FEAT-24.SPEC-005 | Upgrade tracking reads the referred household's subscription tier to determine whether it went on to pay |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Adult member opens the Invite Another Household screen and no personal link exists yet | System provisions and persists the member's one reusable personal link | Standalone Automation | FEAT-24.SPEC-003 |
| Member taps share or copy on an existing link | Link is shared through the member's own messaging, outside the product | Inline in triggering screen | FEAT-24.SPEC-001 |
| Link cannot be created | Retry is offered | Inline in triggering screen | FEAT-24.SPEC-001 |
| Visitor opens the link | Welcome page shown with the inviter's first name and a start-setup option | Inline in triggering screen | FEAT-24.SPEC-002 |
| Visitor already belongs to a household | Visitor is told they already have one; nothing is recorded | Standalone Logic/Rule | FEAT-24.SPEC-006 |
| A new household completes setup from a link within 30 days | Referral is recorded against the inviting household; self-referral and multi-attribution are blocked | Standalone Automation | FEAT-24.SPEC-004 |
| A new household completes setup from a link after 30 days | No referral is recorded | Standalone Logic/Rule | FEAT-24.SPEC-006 |
| A referral is recorded | Inviting member is sent an in-app note once the referred family finishes setting up | Standalone Notification | FEAT-24.SPEC-007 |
| Referred household's subscription changes to paid | Household Referral's `upgraded` flag is updated | Standalone Automation | FEAT-24.SPEC-005 |
| Referring or referred household is deleted (FEAT-18) | Cascade behavior for its Household Referral records | Cross-feature — logged in touchpoints | FEAT-18 responsibility |

## Shared Context

**Shared Entities:**
- Household Referral — created by SPEC-004 under SPEC-006's attribution rules, updated by SPEC-005, read in aggregate by SPEC-001 and SPEC-005. Fields: referring_household, referring_member_link, new_household, created_date, upgraded.

**Shared UI Patterns:**
- Public link-landing pattern — SPEC-002's welcome page follows the same "no household data exposed to an unauthorized visitor" pattern the Access Matrix notes apply across the product (e.g., FEAT-09's invitation-acceptance screen); Spec Writers should describe the visitor experience consistently with that constraint.

**Shared Validation:**
- SPEC-006 defines the one-link-per-member limit, single-attribution and no-self-referral rules, the 30-day window, and role access. SPEC-001, SPEC-002, and SPEC-004 all reference SPEC-006 rather than duplicating these rules.

## Internal Dependency Map

```
SPEC-001 (Invite Another Household Screen) -> [member opens screen, no link yet] -> SPEC-003 (Personal Referral Link Provisioning)
SPEC-001 (Invite Another Household Screen) -> [member taps share] -> [member's own messaging, outside the product]
SPEC-001 (Invite Another Household Screen) -> [validated by] -> SPEC-006 (Household Referral Rules)
SPEC-002 (Referral Welcome Screen) -> [visitor taps "start setup"] -> FEAT-01 (Household Setup & Member Profiles)
SPEC-002 (Referral Welcome Screen) -> [governed by] -> SPEC-006 (Household Referral Rules)
FEAT-01 (Household Setup & Member Profiles) -> [new household's setup completes] -> SPEC-004 (Household Referral Recording)
SPEC-004 (Household Referral Recording) -> [validated by] -> SPEC-006 (Household Referral Rules)
SPEC-004 (Household Referral Recording) -> [record created] -> SPEC-001 (Invite Another Household Screen) [count updates]
SPEC-004 (Household Referral Recording) -> [record created] -> SPEC-007 (Referral Joined Notification)
FEAT-14 (Subscription & Billing Management) -> [referred household's subscription becomes paid] -> SPEC-005 (Referral Upgrade Tracking)
SPEC-005 (Referral Upgrade Tracking) -> [updates] -> Household Referral record (read by SPEC-001, and by success-metrics.md's growth measurement)
```

**Default Entry:** SPEC-001 (Invite Another Household Screen) for a signed-in adult household member; SPEC-002 (Referral Welcome Screen) for an unauthorized visitor arriving from a shared link.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-24.SPEC-001 | Inbound | FEAT-25 (Weekly Waste & Spend Check-In) | Entry into the Invite Another Household screen from the check-in trend card | User taps "invite another family" on the check-in card (End-of-Week Check-In & Inviting Another Family, step 3) |
| FEAT-24.SPEC-002 | Outbound | FEAT-01 (Household Setup & Member Profiles) | Visitor begins their own new household's setup from the welcome page | Visitor taps the start-setup option on the Referral Welcome Screen |
| FEAT-24.SPEC-004 | Inbound | FEAT-01 (Household Setup & Member Profiles) | A new household's completed setup is the trigger that creates the Household Referral record | New household's setup completes within 30 days of the link being opened |
| FEAT-24.SPEC-005 | Inbound | FEAT-14 (Subscription & Billing Management) | A referred household's subscription tier change is read to update the referral's upgraded flag | Referred household's Subscription transitions to a paid tier |
| Household Referral (Delete/Archive) | Inbound | FEAT-18 (Account & Data Management) | Cascade behavior for Household Referral records when a referencing household is deleted is owned by FEAT-18, not this feature | Household deletion completes |

## Non-Functional Notes

**Data volumes / growth:** Household Referral records grow at roughly the same order of magnitude as new households — several thousand households in the first year (assumptions-constraints.md ASMP-24, scope-boundaries.md SC-15) — with each household producing at most one inbound referral record and one outbound personal link per adult member; volumes stay small relative to the product's other entities.

**Responsiveness:** The personal link is ready instantly once created (feature's States field); the Invite Another Household screen's count and referral-joined note need no faster response, since neither is a real-time collaborative surface like the grocery list (assumptions-constraints.md ASMP-22 does not apply here).

**Data sensitivity / privacy:** Low — the Household Referral record links two households by identity and date only; an unauthorized visitor sees only the inviting member's first name, never any other household data (feature's Access field; dependency map, Household Referral Data Sensitivity). The record is used only for the product's own growth measurement and is never sold or used for advertising (assumptions-constraints.md ASMP-14, ASMP-26).

**Compliance flags:** N/A — the Household Referral record holds no children's data and no payment detail (that sits with the Subscription entity FEAT-14 owns); general personal-data rights (export, deletion) apply to the household records it links, not to this feature's own record content, per assumptions-constraints.md ASMP-27.

## Non-Goals

- **Rewards, credits, or discounts for inviting other households** — Excluded per scope-boundaries.md (SC-10): the brief expects households to invite households but names no incentive; this feature records referrals without paying for them, keeping money flows limited to the household subscription.
- **Multiple households per account / re-attribution** — Excluded per scope-boundaries.md (SC-03) and XBR-20: there is one household per account in v1, so a visitor who already has a household is told so and nothing is recorded; a new household can never be attributed to more than one referring household.
- **Cross-household social or community features (public profiles, recipe-sharing feeds)** — Excluded per scope-boundaries.md (SC-09): growth runs through direct household-to-household invite links, not a social layer; the Referral Welcome Screen shows only the inviter's first name and a start-setup option, never a profile or feed.
- **Individual referral record detail view** — Intentional design decision surfaced by the CRUD matrix: the feature's Data Notes name only the aggregate count and the derived paying share as displayed; no capability or journey step asks the inviting member to inspect a single referral's detail, so no Read (single) spec exists.
- **Deletion or purge of Household Referral records** — Intentional lifecycle decision surfaced by the CRUD matrix: the record is written once and never edited beyond its upgrade flag (dependency map, Household Referral Contention) and is retained as the product's only record of referral-driven growth (dependency map, Household Referral Data Sensitivity); no retention window applies. Cascade behavior on deletion of a household it references belongs to FEAT-18, not this feature.



# Screen Spec: Invite Another Household Screen

## Overview

**Name:** Invite Another Household Screen
**ID:** FEAT-24.SPEC-001
**Type:** Screen
**Purpose:** An adult member sees their personal share link, how many families have set up a household from it, and shares the link through their own messaging.
**Parent Feature:** FEAT-24 -- Invite Another Household

## Scope and Non-Goals

**In Scope:**
- Displaying the signed-in adult member's own personal referral link (provisioning it on first need via FEAT-24.SPEC-003)
- Showing the count of families who have set up a household from that link
- Sharing or copying the link through the member's own messaging, outside the product
- The empty, loading, error, and offline states for this screen

**Non-Goals:**
- Creating and persisting the personal link itself -- owned by FEAT-24.SPEC-003 (Personal Referral Link Provisioning); this screen only displays the result and triggers provisioning when none exists yet
- Recording that a new household was referred -- owned by FEAT-24.SPEC-004 (Household Referral Recording); this screen only reads the resulting count
- Showing an individual referral record's detail -- excluded per the Feature Breakdown Brief's Entity-Lifecycle Coverage Matrix: the product's Data Notes name only the aggregate count and the derived paying share as displayed, and no capability asks the inviting member to inspect a single referral
- Rewards, credits, or discounts for a successful referral -- excluded per scope-boundaries.md SC-10: the brief expects households to invite households but names no incentive, and this feature records referrals without paying for them

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-25 (Weekly Waste & Spend Check-In), check-in trend card | Household member taps "invite another family" on the check-in card | None -- screen loads the signed-in member's own link state |
| Household navigation (direct entry) | Signed-in adult member navigates to the screen directly | None -- screen loads the signed-in member's own link state |
| FEAT-24.SPEC-007 (Referral Joined Notification) | Member taps "See who's joined" | None -- screen loads the signed-in member's own link state, including the updated joined-families count |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen -- her own link and the household's joined-family count | Share, copy, and retry link creation | -- |
| Sam (Other Adult Member) | Full screen -- his own link and the household's joined-family count | Share, copy, and retry link creation | -- |
| Jordan (young kid profile, no login -- MVP) | No | No | N/A -- no login exists for this profile type; the Household Referrals column of the Access Matrix gives this row None |
| Jordan (older kid, limited login -- Later) | No | No | N/A -- the Household Referrals column of the Access Matrix gives this row None; the older-kid login has no household-referral entitlement |
| Riley (Operator, support) | No | No | N/A -- the Household Referrals column of the Access Matrix gives Riley None; operator support access has no path through this screen (FEAT-22, XBR-14) |
| Unauthenticated | No | No | Redirected to FEAT-01.SPEC-001 (Account Sign-Up & Sign-In); this screen is never reachable without an active household session |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." No in-progress data exists on this screen to preserve -- the link and count reload fresh after re-authentication |

## Layout and Content

**Header:** Screen title "Invite another household."

**Body:**
- An explanatory line: "Share Plateful with another family. When they set up their own household from your link, you'll see it here."
- The member's personal link, shown as read-only text in a bordered field, with a "Copy" action beside it
- A "Share" button, positioned below the link field, that hands the link to the member's own messaging
- Below the share controls, a joined-families summary: "{joined_count} families have joined from your link" (or the Empty state's explanation when the count is zero)

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Link field, Copy action, Share button, and the joined-families summary stack full width, in the order listed above.
- **Medium size class and above:** Content area caps at the platform-wide narrow content width and is horizontally centered; the link field and Copy action sit on one row; no further structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Screen (on open) | Load | If the signed-in member has no personal link yet, trigger FEAT-24.SPEC-003 (Personal Referral Link Provisioning) | Screen shows Loading while the link is provisioned | Link field and Share button populate once provisioning completes |
| "Copy" action | Tap | Copies the link text to the member's clipboard | None (link field unchanged) | Confirmation text "Copied" briefly replaces the action label |
| "Share" button | Tap | Hands the link to the member's own messaging (outside the product) | None | The member's own messaging surface opens; no in-product state changes as a result |
| "Retry" button (Error state only) | Tap | Re-triggers FEAT-24.SPEC-003 (Personal Referral Link Provisioning) | Screen returns to Loading | Success: link field and Share button populate. Failure: Error state re-appears |

### Accessibility Notes

- **Focus order:** Link field -> Copy action -> Share button -> joined-families summary (read-only, not a focus stop unless it is the Empty state's explanation, which follows the Share button).
- **Copy announcement:** The "Copied" confirmation is announced to assistive technology when it appears.
- **Loading announcement:** The transition from Loading to a populated link, or to the Error state, is announced so the change is not silently missed.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Link field and Share button shown as a skeleton; joined-families summary hidden | Screen first opens, or Retry is tapped | Link provisioning (FEAT-24.SPEC-003) succeeds or fails |
| Populated, no joins yet | Link field and Share/Copy actions active; joined-families summary reads "No families have joined yet -- share your link to get started." | Link exists and the joined-family count is zero | The count becomes one or more (screen re-renders on next load or refresh) |
| Populated, with joins | Link field and Share/Copy actions active; joined-families summary reads "{joined_count} families have joined from your link" | Link exists and the joined-family count is one or more | N/A -- this is the screen's steady state once the household has referrals |
| Error | Link field and Share button hidden; message "We couldn't create your invite link." with a Retry button | Link provisioning (FEAT-24.SPEC-003) fails | Member taps Retry and provisioning succeeds |
| Offline/Degraded | Banner: "You're offline. An already-created link can still be copied and shared; creating a new one needs a connection." An existing link's Copy and Share actions remain active; Retry (if in Error state) is disabled | Connectivity is lost while this screen is open | Connectivity returns -- banner clears and Retry re-enables if applicable |

## Validation Rules

Validation governed by FEAT-24.SPEC-006 (Household Referral Rules) for the one-reusable-link-per-adult-member limit enforced during provisioning. This screen itself collects no user input requiring field-level validation.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| "Share" button tap | Member's own messaging (external, outside the product) | -- |
| Back navigation | Screen the member arrived from (FEAT-25 check-in card, or household navigation) | FEAT-25 (Weekly Waste & Spend Check-In), if arrived from there |

## Data Model

**Creates:** None directly -- the personal link itself is created by FEAT-24.SPEC-003 on this screen's trigger.
**Reads:** Personal Referral Link -- the signed-in member's own link, provisioned by FEAT-24.SPEC-003. Household Referral -- the count of records where referring_household is the signed-in member's household (aggregate count only, no individual record fields shown).
**Updates:** None.
**Deletes:** None.

## Business Rules

- One reusable personal link per adult member (FEAT-24.SPEC-006) -- this screen always shows the same link for a given member across every visit, never generating a second one.
- The joined-families count reflects every Household Referral record attributed to the signed-in member's household, regardless of which adult member's link was used, since referring_household (not referring_member_link alone) is the household-level attribution the count reads (Feature Breakdown Brief, Shared Context).
- The link is ready instantly once provisioned; no plan-generation-style wait is expected (Feature Breakdown Brief, Non-Functional Notes, Responsiveness).

## Edge Cases

- **Member taps Share or Copy while offline** -- Both actions still work on an already-created link, since the link text is already available on the screen; no connection is required to copy or hand off already-loaded text.
- **Link provisioning fails twice in a row** -- The Error state and Retry button persist; no limit is placed on retry attempts, since link creation carries no cost or side effect to repeat.
- **Member double-taps Share rapidly** -- Each tap independently hands the link to the member's own messaging; since sharing has no in-product state to corrupt, repeated taps are harmless and not specifically debounced.
- **Joined-families count changes while the screen is open (a referral completes elsewhere in real time)** -- The count reflects its value as of when the screen loaded; a live-updating count is not required, per the Feature Breakdown Brief's Non-Functional Notes (this screen is not a real-time collaborative surface like the grocery list). The member sees the current count on next screen open or refresh.
- **No concurrent-edit conflict applies to this screen** -- This screen never writes to the Household Referral record; the dependency map's Contention note for Household Referral states the record is written once by the system and never edited by any role, so no conflicting-write scenario exists here.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-24.SPEC-003 (Personal Referral Link Provisioning) | Triggers (outbound) | Provisions the member's link on first need or on Retry |
| FEAT-24.SPEC-004 (Household Referral Recording) | References (inbound) | Supplies the Household Referral records this screen's count is derived from |
| FEAT-24.SPEC-005 (Referral Upgrade Tracking) | References (inbound) | Derives the paying share shown when the household reviews its referral results (Data Notes) |
| FEAT-24.SPEC-006 (Household Referral Rules) | References (inbound) | Governs the one-link-per-member limit this screen's link display relies on |
| FEAT-25 (Weekly Waste & Spend Check-In) | Navigation (inbound) | Check-in trend card's "invite another family" tap arrives here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| referral_link_shared | entry source (check_in_card / direct) | Member taps "Share" | N/A -- no Stage 2 metric measures share attempts directly; retained as a funnel-diagnostic signal ahead of the metric-bearing household-creation event (FEAT-24.SPEC-004) |
| referral_link_copied | -- | Member taps "Copy" | N/A -- diagnostic signal only |
| referral_link_provisioning_failed | -- | Link provisioning fails and the Error state appears | N/A -- diagnostic signal only |
| referral_screen_viewed | joined_count at view time | Screen finishes loading (Populated state, either variant) | supports success-metrics.md: "Household-to-Household Invitation Growth" |

## Acceptance Criteria

**FEAT-24.SPEC-001-AC-01:** Given Sam is on the Invite Another Household Screen and already has a personal link, when the screen loads, then it shows his link, a Share button, a Copy action, and the current joined-families count.

**FEAT-24.SPEC-001-AC-02:** Given Maya opens this screen for the first time with no personal link yet, when the screen loads, then FEAT-24.SPEC-003 provisions her link automatically and it appears once ready.

**FEAT-24.SPEC-001-AC-03:** Given Sam's household has zero joined families, when the screen loads, then the summary reads "No families have joined yet -- share your link to get started."

**FEAT-24.SPEC-001-AC-04:** Given Maya's household has three joined families, when the screen loads, then the summary reads "3 families have joined from your link."

**FEAT-24.SPEC-001-AC-05:** Given Sam taps "Copy" on his link, when the copy completes, then the confirmation text "Copied" briefly replaces the action label.

**FEAT-24.SPEC-001-AC-06:** Given Maya taps "Share" on her link, when the action fires, then her own messaging surface opens with the link ready to send.

**FEAT-24.SPEC-001-AC-07:** Given link provisioning fails for Sam, when the Error state appears, then he sees "We couldn't create your invite link." with a Retry button.

**FEAT-24.SPEC-001-AC-08:** Given Sam taps Retry after a provisioning failure, when provisioning succeeds this time, then his link and Share button appear normally.

**FEAT-24.SPEC-001-AC-09:** Given Maya loses connectivity after her link has already loaded, when she taps Copy or Share, then both actions complete normally using the already-loaded link text.

**FEAT-24.SPEC-001-AC-10:** Given Maya loses connectivity before her link has loaded, when the screen is in the Loading or Error state, then the banner "You're offline. An already-created link can still be copied and shared; creating a new one needs a connection." appears and Retry is disabled.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 4 | 4 |
| States | 5 (loading, populated-no-joins, populated-with-joins, error, offline) | 5 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



# Screen Spec: Referral Welcome Screen

## Overview

**Name:** Referral Welcome Screen
**ID:** FEAT-24.SPEC-002
**Type:** Screen
**Purpose:** A visitor who followed a household's personal referral link sees a welcome page naming the inviter and starts their own household setup, or is told they already have one.
**Parent Feature:** FEAT-24 -- Invite Another Household

## Scope and Non-Goals

**In Scope:**
- Resolving the personal referral link the visitor opened and showing the inviting member's first name
- Carrying the referral context (which household's link, and the moment it was opened) forward into account creation, for later attribution by FEAT-24.SPEC-004
- Handing the visitor off to FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) to start their own household setup
- Detecting a visitor who is already signed in as a member of a household and showing the "already have one" disposition directly, without proceeding to account creation

**Non-Goals:**
- Creating the new account or the new Household record -- owned by FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) and FEAT-01.SPEC-003 (Household Naming & Guided Setup Start); this screen only hands off
- Recording the Household Referral once setup completes -- owned by FEAT-24.SPEC-004 (Household Referral Recording); this screen only carries the context that recording depends on
- Evaluating the 30-day counting window or the single-attribution and no-self-referral rules -- owned by FEAT-24.SPEC-006 (Household Referral Rules); this screen shows a signed-in visitor's "already have a household" disposition, which is a distinct, screen-visible case from those rules' create-time checks
- Cross-household social or community content (public profiles, recipe feeds) -- excluded per scope-boundaries.md SC-09: growth runs through direct household-to-household invite links, not a social layer; this screen shows only the inviter's first name and a start-setup option

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| Personal referral link (external) | Visitor opens the shareable link provisioned by FEAT-24.SPEC-003 | The specific personal link's owning member and household; the moment the link is opened, recorded for the 30-day counting window (FEAT-24.SPEC-006) |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Unauthorized visitor (not signed in, no household) | Yes -- the welcome message and start-setup option | Start setup | -- |
| Maya (Organiser, signed in, already has a household) | Partial -- only the "already have a household" disposition message | No | Shown "You already have a household on Plateful." with a link into her own household's current plan, instead of the welcome message |
| Sam (Other Adult Member, signed in, already has a household) | Partial -- only the "already have a household" disposition message | No | Same disposition as Maya: "You already have a household on Plateful." with a link into his own household's current plan |
| Jordan (young kid profile, no login -- MVP) | No | No | N/A -- no login exists for this profile type; a young kid profile can never open this link as itself |
| Jordan (older kid, limited login -- Later) | Partial, if signed in -- only the "already have a household" disposition message | No | Same disposition as Maya and Sam: an older-kid login belongs to a household already, so no start-setup option is offered |
| Riley (Operator, support) | No | No | N/A -- the Household Referrals column of the Access Matrix gives Riley None; operator support access has no path through this screen (FEAT-22, XBR-14) |
| Expired session | Yes -- treated as an unauthorized visitor for this screen's purposes | Start setup | The visitor's prior session state is irrelevant here; this screen never requires an active session to show its content |

## Layout and Content

**Header:** Product name/logo, centered. No back navigation (this is a direct-link entry screen).

**Body -- unauthorized visitor:**
- A welcome line naming the inviter: "{inviter_first_name} shared Plateful with you."
- A short explanation of the product: a family meal planner that plans the week's dinners around each person's allergies and diets, with one shared grocery list
- "Start your household" button

**Body -- visitor already belongs to a household:**
- Plain message: "You already have a household on Plateful." -- no other household data of any kind is shown, per the Feature Breakdown Brief's Non-Goals and dependency map's Household Referral Data Sensitivity note
- A link back into the visitor's own current plan

**Footer:** Legal text line linking to terms and privacy information (static content).

### Responsive Behavior

- **Compact breakpoint:** Welcome message, explanation, and button stack full width.
- **Medium size class and above:** Content area caps at the platform-wide narrow width and is horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| "Start your household" button | Tap | Navigate to FEAT-01.SPEC-001 (Account Sign-Up & Sign-In), carrying the inviter's first name for display and the underlying referral link context for later attribution | Screen changes | Standard navigation transition; FEAT-01.SPEC-001 shows "{inviter_first_name} invited you to try Plateful" above its form |
| "Go to your plan" link (already-has-a-household body only) | Tap | Navigate into the visitor's own household's current plan | Screen changes | Standard navigation transition |
| "Retry" button (Error state only) | Tap | Re-attempts resolving the personal referral link | Screen returns to Loading | Success: welcome body or already-has-a-household body appears, whichever the link and session resolve to. Failure: Error state re-appears |

### Accessibility Notes

- **Focus order:** Welcome message, then "Start your household" button (unauthorized-visitor body); or the disposition message, then "Go to your plan" link (already-has-a-household body).
- **Body-switch announcement:** If the screen resolves the visitor's signed-in status after an initial render, the switch from the welcome body to the already-has-a-household body is announced so the change is not silently missed.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Skeleton in place of the body while the link and any active session are resolved | Screen first opens, or Retry is tapped from the Error state | Link resolves to a valid welcome body, an already-has-a-household body, an invalid-link body, or the Error state |
| Valid link, unauthorized visitor | Welcome body shown as described in Layout and Content | The link resolves to an active personal referral link and no household session is active | Visitor taps "Start your household" |
| Already has a household | "You already have a household on Plateful." message shown | The link resolves to an active personal referral link, and the visitor is signed in as a member of any household (including the referring household itself) | Visitor taps "Go to your plan," or navigates away |
| Invalid or unresolvable link | Generic message: "This invite link is no longer active." with no further detail, and a link to FEAT-01.SPEC-001 for a visitor who wants to start their own household without invitation context | The link does not resolve to any personal referral link (malformed, tampered, or its owning household no longer exists) | Visitor navigates to FEAT-01.SPEC-001 or away |
| Error | Inline error banner in place of the body: "Something went wrong loading this page." with a Retry button; the link is not declared invalid | Link resolution is attempted (connectivity is present and the link itself may still be valid) but the resolution call fails for a reason other than the link being invalid or the visitor being offline -- e.g., a transient backend or processing failure | Visitor taps Retry and resolution succeeds, landing on the welcome body or the already-has-a-household body, whichever the link and session resolve to |
| Offline/Degraded | Banner: "You're offline. This page needs a connection to load." with the body left blank until connectivity returns | Connectivity is lost or absent while this screen attempts to load | Connectivity returns -- the screen resolves and renders normally |

## Validation Rules

Validation governed by FEAT-24.SPEC-006 (Household Referral Rules) for the "already has a household" disposition and the eligibility check that determines whether a subsequent household completion (via FEAT-01.SPEC-003) can be attributed. This screen applies the signed-in-visitor check on load; the 30-day counting window itself is evaluated later, at household completion (FEAT-24.SPEC-004), not here.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| "Start your household" tap | FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) | FEAT-01 |
| "Go to your plan" tap | The visitor's own current plan (FEAT-03.SPEC-001 or FEAT-23's manual week, per the visitor's household tier) | FEAT-03 or FEAT-23 |
| Invalid-link body, "start your own household" link | FEAT-01.SPEC-001 (Account Sign-Up & Sign-In), with no referral context | FEAT-01 |

## Data Model

**Creates:** None -- account and household creation are owned by FEAT-01.
**Reads:** Personal Referral Link -- resolves to the owning Member Profile's first name and owning Household, to render the welcome message and confirm the link is still tied to an existing household. Any active session -- to determine whether the visitor already belongs to a household.
**Updates:** None.
**Deletes:** None.

## Business Rules

- Only the inviter's first name is shown -- no other data about the referring household is ever exposed to an unauthorized visitor, per the dependency map's Household Referral Data Sensitivity note and the Feature Breakdown Brief's Shared UI Patterns (Public link-landing pattern).
- A visitor who already belongs to a household (any household, including the referring one) is shown the disposition message and never proceeds to account creation from this screen, consistent with scope-boundaries.md SC-03 (one household per account in v1) and XBR-20.
- The moment this screen resolves a valid link is the moment recorded as the link being "opened," which starts the 30-day counting window that FEAT-24.SPEC-006 and FEAT-24.SPEC-004 evaluate at household completion.
- This screen never itself creates or updates a Household Referral record -- attribution happens once household setup completes, at FEAT-01.SPEC-003, per FEAT-24.SPEC-004.

## Edge Cases

- **Visitor opens the link, starts setup, but does not sign in as the same account by the time they complete household creation** -- Each device or session that opens the link carries its own referral context independently; whichever session completes FEAT-01.SPEC-003's household creation is the one FEAT-24.SPEC-004 evaluates. A visitor who opens the link on one device and creates the account on another without carrying the link (e.g., typing the product's address manually instead) completes as an unattributed household -- no referral is recorded, since no link context was carried.
- **The referring household is deleted after the link is opened but before the visitor completes setup** -- FEAT-24.SPEC-004 finds no live referring household to attribute against at completion time and records no referral; this screen itself is unaffected, since it already resolved the link before the deletion. Cascade behavior for existing Household Referral records against a deleted household is FEAT-18's responsibility, flagged in the Feature Breakdown Brief's Cross-Feature Touchpoints, not resolved by this screen.
- **Link does not resolve to any personal referral link (malformed or tampered)** -- The invalid-link body shows the generic "This invite link is no longer active." message with no further detail, so no information about whether any link ever existed is leaked, consistent with the pattern FEAT-09.SPEC-002 (Invitation Acceptance) uses for its own invalid-link case.
- **Visitor opens the link while already signed in as a member of the referring household itself** -- Shown the same "You already have a household on Plateful." disposition as any other already-a-member visitor; no self-referral can be recorded because no account creation occurs from this state (XBR-20's no-self-referral rule is also enforced again at FEAT-24.SPEC-004 as a second line of defense).
- **Visitor navigates away and reopens the same link later, still within the 30-day window** -- The link resolves fresh each time; the counting window is anchored to this visit's open moment, which resets on every fresh open of a still-valid link, since only the household-completion moment at FEAT-24.SPEC-004 checks elapsed time against the most recent open.
- **Link resolution fails for a reason other than the link being invalid or the visitor being offline (a transient backend or processing failure)** -- The visitor is connected and the link may still be valid, so the screen shows the Error state ("Something went wrong loading this page." with Retry) rather than the "This invite link is no longer active." message, since that message would misrepresent a still-valid link as permanently dead. Retry re-attempts resolution; a visitor who never retries simply never proceeds past the Error state, and no referral context is lost since nothing has been carried forward yet at this point.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-24.SPEC-003 (Personal Referral Link Provisioning) | References (inbound) | Supplies the personal referral link this screen resolves |
| FEAT-24.SPEC-006 (Household Referral Rules) | References (inbound) | Governs the already-has-a-household disposition and the link-open moment that starts the counting window |
| FEAT-24.SPEC-004 (Household Referral Recording) | References (outbound) | Evaluates the referral context this screen carries forward, once household setup completes |
| FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) | Navigation (outbound) | "Start your household" hands off here, carrying the inviter's first name and the referral context |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| referral_link_opened | link status at open time (valid / already_has_household / invalid) | The screen resolves the referral link | N/A -- no Stage 2 metric measures link opens directly; retained as a funnel-diagnostic signal ahead of the metric-bearing household-creation event (FEAT-24.SPEC-004) |
| referral_start_setup_tapped | -- | Visitor taps "Start your household" | N/A -- diagnostic signal only |

## Acceptance Criteria

**FEAT-24.SPEC-002-AC-01:** Given an unauthorized visitor opens Sam's personal referral link, when the screen loads, then it shows "Sam shared Plateful with you." with the "Start your household" button.

**FEAT-24.SPEC-002-AC-02:** Given the visitor taps "Start your household", when the action fires, then they are taken to FEAT-01.SPEC-001, which shows "Sam invited you to try Plateful" above its form.

**FEAT-24.SPEC-002-AC-03:** Given the visitor is already signed in as Maya, a member of her own household, when she opens any personal referral link, then she sees "You already have a household on Plateful." with a link to her own current plan, and no start-setup option.

**FEAT-24.SPEC-002-AC-04:** Given the visitor opens a link that does not resolve to any personal referral link, when the screen loads, then it shows "This invite link is no longer active." with no further detail.

**FEAT-24.SPEC-002-AC-05:** Given the referring household is deleted after the link is opened but before the visitor signs up, when the visitor later completes FEAT-01.SPEC-003, then no Household Referral record is created, per FEAT-24.SPEC-004.

**FEAT-24.SPEC-002-AC-06:** Given the visitor is signed in as a member of the referring household itself and opens that same household's own link, when the screen loads, then they see "You already have a household on Plateful." rather than any start-setup option.

**FEAT-24.SPEC-002-AC-07:** Given the visitor loses connectivity while this screen attempts to load, when the load is attempted, then the banner "You're offline. This page needs a connection to load." appears and the body remains blank.

**FEAT-24.SPEC-002-AC-08:** Given the visitor opens a valid link and reopens it two days later, still unauthorized, when the screen loads the second time, then the welcome message and start-setup option appear exactly as on first open, and the counting window anchors to this most recent open.

**FEAT-24.SPEC-002-AC-09:** Given a visitor viewing the "already have a household" disposition, when they tap "Go to your plan", then they are taken to their own household's current plan (FEAT-03 or FEAT-23, depending on their household's tier).

**FEAT-24.SPEC-002-AC-10:** Given the link resolution call fails for a transient reason while the visitor is connected and the link itself may still be valid, when the screen attempts to load, then it shows the Error state ("Something went wrong loading this page." with a Retry button) rather than the invalid-link message, and tapping Retry re-attempts resolution.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 3 | 3 |
| States | 6 (loading, valid-unauthorized, already-has-household, invalid-link, error, offline) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Automation Spec: Personal Referral Link Provisioning

## Overview

**Name:** Personal Referral Link Provisioning
**ID:** FEAT-24.SPEC-003
**Type:** Automation
**Purpose:** Creates and persists an adult member's one reusable personal referral link the first time it is needed, and returns the existing one on every later request.
**Parent Feature:** FEAT-24 -- Invite Another Household

## Scope and Non-Goals

**In Scope:**
- Checking whether the requesting adult member already owns a personal referral link
- Creating exactly one durable, reusable personal referral link per adult member the first time one is needed
- Returning the existing link on any subsequent request from the same member
- Reporting creation failure back to the triggering screen for its retry offer

**Non-Goals:**
- Displaying the link, the Share and Copy actions, or the joined-families count -- owned by FEAT-24.SPEC-001 (Invite Another Household Screen); this automation only creates and returns the link value
- Recording a Household Referral when the link is used -- owned by FEAT-24.SPEC-004 (Household Referral Recording); provisioning a link and recording a referral are separate moments, per the Feature Breakdown Brief's Side-Effect Inventory
- Issuing a link to a kid profile or to Riley (Operator) -- excluded per the Access Matrix in user-persona.md: the Household Referrals column gives both kid rows and Riley None, so no request from those roles can reach this automation
- Expiring or rotating a member's link -- product-features.md's Validation & Limits states "one reusable personal link per adult member" with no expiry or rotation named; the link is durable for the life of the member's account

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Invite Another Household Screen opens with no link yet | FEAT-24.SPEC-001 (Invite Another Household Screen) | Fires when the signed-in adult member (Maya or Sam) has no personal referral link on record | Requesting Member Profile reference, its owning Household |
| Member taps Retry after a provisioning failure | FEAT-24.SPEC-001 (Invite Another Household Screen) | Fires when the member retries after the Error state | Same as above |

## Processing Logic

1. Receive the requesting Member Profile's reference and confirm it is an adult member (Organiser or Other Adult Member) of an active Household -- FEAT-24.SPEC-006 governs this eligibility check.
2. Check whether this Member Profile already owns a personal referral link.
3. If a link already exists, return it unchanged -- no second link is ever created for the same member.
4. If no link exists, generate one new, durable, reusable personal referral link tied to this Member Profile and its Household.
5. Persist the new link so every future request from this member (this screen, this device, or any other) returns the same value.
6. Return the link to the triggering screen for display.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Existing link returned | The requesting member already owns a link | None | FEAT-24.SPEC-001 displays the link immediately | FEAT-24.SPEC-001 |
| New link created | The requesting member owns no link yet, and creation succeeds | New personal referral link persisted, tied to the requesting member and household | FEAT-24.SPEC-001 displays the newly created link | FEAT-24.SPEC-001 |
| Creation failure | The requesting member owns no link yet, and creation cannot be completed | No link persisted | FEAT-24.SPEC-001 shows its Error state: "We couldn't create your invite link." with Retry | FEAT-24.SPEC-001 |

## Data Model

**Reads:** Member Profile -- member_type (to confirm the requester is an adult member) and status (to confirm Active); Household -- status (to confirm the household is Active, not Closed/Deleted).
**Creates:** Personal Referral Link -- a durable identifier owned by the requesting adult Member Profile, created once per member. This is the identifier that a Household Referral record's referring_member_link field references once the link is used to attribute a new household (FEAT-24.SPEC-004).
**Updates:** None.
**Deletes:** None.

## Business Rules

- One reusable personal link per adult member (product-features.md, Validation & Limits) -- this automation never creates a second link for a member who already has one; the check-then-create step (Processing Logic, Step 2-4) is what enforces this.
- Only Maya and Sam (adult members, per the Household Referrals column of the Access Matrix) can trigger this automation; a kid profile or Riley never reaches FEAT-24.SPEC-001's trigger condition in the first place.
- The link is ready instantly once created -- no queued or delayed provisioning step exists (Feature Breakdown Brief, Non-Functional Notes, Responsiveness).
- Link creation requires connectivity; an already-created link remains usable offline (FEAT-24.SPEC-001's Offline/Degraded state), but this automation itself cannot run without a connection.

## Edge Cases

- **Member has no connectivity when this automation is triggered** -- Creation fails immediately with the same Creation failure outcome; FEAT-24.SPEC-001's Offline/Degraded state (rather than its Error state) is what the member sees in this specific case, since the screen can distinguish "no connection" from "connection present but creation failed."
- **The requesting household is Closed/Deleted at the moment of the request** -- Creation is refused; no link is created for a member of a household that no longer exists. This is a theoretical edge case in current product flow, since a member of a deleted household has no path back to FEAT-24.SPEC-001 in the first place.
- **Concurrent trigger firing (Maya opens the Invite Another Household Screen on her phone and her laptop at effectively the same time, neither having a link yet)** -- Each request runs its own check-then-create independently; the check-then-create step is idempotent, so only one link is ever persisted for Maya regardless of which request's create step lands first. The second request to complete finds the first request's link already persisted and returns it rather than creating a duplicate.
- **Trigger fires while a previous run is in flight (Retry tapped again before the first attempt has returned)** -- FEAT-24.SPEC-001 disables its Retry button while a provisioning attempt is in progress, so a second run for the same member cannot start until the first completes. Runs for different members proceed independently and never queue behind one another.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-24.SPEC-001 (Invite Another Household Screen) | Triggered by (inbound) | Fires on first screen open with no link, and on Retry after a failure |
| FEAT-24.SPEC-001 (Invite Another Household Screen) | Affects (outbound) | Returns the link (or the failure outcome) for display |
| FEAT-24.SPEC-004 (Household Referral Recording) | References (outbound) | The link this automation creates is the identifier a later Household Referral record's referring_member_link field references |
| FEAT-24.SPEC-006 (Household Referral Rules) | References (inbound) | Governs the one-link-per-member limit this automation enforces |

## Analytics and Success Signals

- **referral_link_provisioned** (outcome: existing_returned / newly_created) -- N/A -- no Stage 2 metric measures link provisioning itself; retained as a funnel-diagnostic signal ahead of the metric-bearing household-creation event (FEAT-24.SPEC-004, which supports success-metrics.md: "Household-to-Household Invitation Growth")
- **referral_link_provisioning_failed** (reason: offline / processing_error) -- N/A -- diagnostic signal only; a failure here must never silently block the member from later retrying, so this event measures how often that retry path is exercised

## Acceptance Criteria

**FEAT-24.SPEC-003-AC-01:** Given Maya opens the Invite Another Household Screen for the first time and has no personal link, when this automation fires, then a new link is created and persisted for her, tied to her Member Profile and household.

**FEAT-24.SPEC-003-AC-02:** Given Sam already has a personal link from a prior visit, when he opens the Invite Another Household Screen again, then this automation returns his existing link unchanged and creates no second one.

**FEAT-24.SPEC-003-AC-03:** Given Maya is signed in on both her phone and her laptop with no link yet on either, when she opens the screen on both at effectively the same time, then only one link is ever persisted for her, and both devices end up displaying that same link.

**FEAT-24.SPEC-003-AC-04:** Given Sam has no connectivity when the automation is triggered, when creation is attempted, then it fails and FEAT-24.SPEC-001 shows its Offline/Degraded state rather than its Error state.

**FEAT-24.SPEC-003-AC-05:** Given Maya has connectivity but link creation fails for a processing reason, when the automation completes, then FEAT-24.SPEC-001 shows "We couldn't create your invite link." with a Retry button.

**FEAT-24.SPEC-003-AC-06:** Given Sam taps Retry after a provisioning failure, when this automation fires again and succeeds, then his link is created and displayed, and no earlier failed attempt leaves behind a partial or duplicate link.

**FEAT-24.SPEC-003-AC-07:** Given Sam taps Retry a second time while the first retry attempt is still in flight, when the Retry button is inspected, then it is disabled and the second tap has no effect until the first attempt completes.

**FEAT-24.SPEC-003-AC-08:** Given a request somehow arrives from a kid profile or Riley (Operator) context, when this automation is invoked, then no link is created, since the Household Referrals column of the Access Matrix grants neither role any access to FEAT-24.SPEC-001's trigger in the first place.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (first open with no link, Retry after failure) | 2 |
| Outcome Paths | 3 (existing returned, newly created, creation failure) | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 4 | 4 |



# Automation Spec: Household Referral Recording

## Overview

**Name:** Household Referral Recording
**ID:** FEAT-24.SPEC-004
**Type:** Automation
**Purpose:** Attributes a new household's completed setup to the referring household's link when eligible, enforcing the single-attribution, no-self-referral, and 30-day counting-window rules.
**Parent Feature:** FEAT-24 -- Invite Another Household

## Scope and Non-Goals

**In Scope:**
- Evaluating, at the moment a new household is created, whether it arrived via a personal referral link and is still eligible for attribution
- Creating exactly one Household Referral record when eligible
- Enforcing single-attribution (a new household is attributed to at most one referring household, ever) and no-self-referral
- Enforcing the 30-day counting window between the link's most recent open and the new household's creation
- Triggering FEAT-24.SPEC-007 (Referral Joined Notification) when a record is created

**Non-Goals:**
- Creating the new Household or Member Profile records themselves -- owned by FEAT-01.SPEC-003 (Household Naming & Guided Setup Start); this automation fires once that creation succeeds and only writes the Household Referral record
- Displaying the referring member's link, the joined-families count, or the referral welcome message -- owned by FEAT-24.SPEC-001 and FEAT-24.SPEC-002; this automation is invisible to the new household's own setup experience
- Updating the `upgraded` flag once the new household later becomes a paying one -- owned by FEAT-24.SPEC-005 (Referral Upgrade Tracking); this automation only ever sets the initial record with `upgraded` false
- Any reward, credit, or discount tied to a successful referral -- excluded per scope-boundaries.md SC-10: this automation records the referral without paying for it

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| New household completes creation | FEAT-01.SPEC-003 (Household Naming & Guided Setup Start) | Fires every time a Household record is successfully created, regardless of entry source | New Household reference, its first Member Profile (the account that created it), and -- when present -- the referral context carried from FEAT-24.SPEC-002: the personal referral link opened, its owning Member Profile and Household, and the moment the link was most recently opened |

## Processing Logic

1. Receive the newly created Household from FEAT-01.SPEC-003.
2. Determine whether a referral link context was carried into this account's creation (per FEAT-24.SPEC-002). If none was carried, stop -- this is an unattributed household and no further processing occurs.
3. If a referral link context exists, identify the referring Household and the specific personal referral link (and its owning Member Profile) that was opened.
4. Check self-referral: if the referring Household is the same as the new Household, stop and record no referral (FEAT-24.SPEC-006, no-self-referral rule).
5. Check single attribution: confirm the new Household has never before been the `new_household` on any Household Referral record. If it has, stop and record no referral (FEAT-24.SPEC-006, single-attribution rule).
6. Check the 30-day counting window: confirm the new Household's creation date falls within 30 days of the referral link's most recent open moment (FEAT-24.SPEC-002). If the window has elapsed, stop and record no referral.
7. If all checks pass, create one Household Referral record: referring_household set to the referring Household, referring_member_link set to the personal referral link identifier, new_household set to the new Household, created_date set to the current date, upgraded set to false.
8. Trigger FEAT-24.SPEC-007 (Referral Joined Notification) for the Member Profile that owns the personal referral link used.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Referral recorded | No referral context carried is missing, no self-referral, no prior attribution for this new household, and the household was created within 30 days of the link's most recent open | New Household Referral record created (upgraded: false) | None on the new household's own setup screens; the referring member's joined-families count (FEAT-24.SPEC-001) reflects the new record on next view; FEAT-24.SPEC-007 delivers the referring member's in-app note | FEAT-24.SPEC-001, FEAT-24.SPEC-007 |
| No referral context | The new household's creation carried no referral link context | None | None -- this is the ordinary, unattributed household-creation path and is never surfaced as an error | FEAT-01.SPEC-003 |
| Self-referral blocked | Referral context exists, but the referring household equals the new household | None | None to the new household's own setup; this is a silent, no-action outcome per FEAT-24.SPEC-006 | -- |
| Already attributed | Referral context exists, but the new household already carries a Household Referral record from a prior link open | None -- the earlier record stands unchanged | None to the new household's own setup | -- |
| Window elapsed | Referral context exists and passes self-referral and single-attribution checks, but more than 30 days have passed since the link's most recent open | None | None to the new household's own setup | -- |
| Automation failure | An internal processing error prevents the write after all checks pass | No partial write -- no Household Referral record is left in an inconsistent state | The new household's own setup (FEAT-01.SPEC-003) completes normally regardless, since referral recording is a side effect of household creation, never a precondition for it | FEAT-01.SPEC-003 |

## Data Model

**Reads:** Household -- the new household's creation date and identity; Household Referral -- existing records, to check whether the new household already carries one (single-attribution check).
**Creates:** Household Referral -- referring_household, referring_member_link, new_household, created_date, upgraded (set to false).
**Updates:** None -- this automation never modifies an existing Household Referral record; that is FEAT-24.SPEC-005's exclusive role.
**Deletes:** None.

## Business Rules

- XBR-20: a new household set up from an invite link within 30 days is attributed to at most one referring household, never itself; someone who already has a household is told so and nothing is recorded (enforced upstream at FEAT-24.SPEC-002, and re-enforced here as this automation's own self-referral check).
- Attribution is a one-time, system-only write -- no role, including Maya or Sam, can create, edit, or back-date a Household Referral record directly (FEAT-24.SPEC-006, Authorization Rules).
- Household creation never waits on this automation -- FEAT-01.SPEC-003's own success and navigation are unaffected by whether a referral is recorded, per this feature's Non-Goals (recording is a side effect, not a precondition).
- The referring_member_link on the created record identifies whichever adult member's personal link was opened, even if a different adult member of the same household also has a link -- attribution is per-link, but the joined-families count shown on FEAT-24.SPEC-001 aggregates by referring_household, per the Feature Breakdown Brief's Shared Context.

## Edge Cases

- **Visitor opens two different households' referral links in the same session before completing setup (e.g., a link from Sam's household, then later a link from an unrelated household's member)** -- Last-touch attribution: whichever link's context was most recently carried at the moment FEAT-01.SPEC-003's household creation succeeds is the one this automation evaluates; the earlier link's context is discarded once superseded.
- **The referring household is deleted between the link being opened and the new household completing setup** -- The referring Household no longer resolves to a live record at evaluation time, so no referral is recorded; this is treated the same as "no referral context" rather than as a failure. Cascade behavior for any Household Referral records that already reference a since-deleted household is FEAT-18's responsibility, flagged in the Feature Breakdown Brief's Cross-Feature Touchpoints, not resolved by this automation.
- **New household's creation date lands exactly 30 days after the link's most recent open** -- The window is inclusive of day 30; a household created on day 31 or later falls outside the window and no referral is recorded.
- **Concurrent trigger firing (two different visitors complete FEAT-01.SPEC-003 at effectively the same time, both having opened the same household's same personal link)** -- Each new household's creation is evaluated independently against its own referral context; both can be recorded as separate Household Referral records against the same referring household and the same referring_member_link, since single-attribution is scoped to the new_household side of the relationship, not the referring side. Two different families can legitimately be referred by the same link.
- **Trigger fires while a previous run is in flight for the same new household (a duplicate household-creation signal from FEAT-01.SPEC-003, e.g., a retried request after a network hiccup)** -- The single-attribution check (Processing Logic, Step 5) makes a second attempt for the same new household a no-op: the second run finds the first run's record already present and stops without creating a duplicate.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-003 (Household Naming & Guided Setup Start) | Triggered by (inbound) | New household creation success fires this automation |
| FEAT-24.SPEC-002 (Referral Welcome Screen) | References (inbound) | Supplies the referral link context (referring household, member, and open moment) this automation evaluates |
| FEAT-24.SPEC-006 (Household Referral Rules) | References (inbound) | Governs single-attribution, no-self-referral, and the 30-day counting window enforced here |
| FEAT-24.SPEC-001 (Invite Another Household Screen) | Affects (outbound) | The referring household's joined-families count reflects a newly recorded referral |
| FEAT-24.SPEC-007 (Referral Joined Notification) | Triggers (outbound) | A recorded referral fires the referring member's in-app note |
| FEAT-24.SPEC-005 (Referral Upgrade Tracking) | References (outbound) | The record this automation creates is later read and updated by upgrade tracking |

## Analytics and Success Signals

- **referred_household_created** (window_check: within_window; attribution: recorded) -- supports success-metrics.md: "Household-to-Household Invitation Growth"
- **referral_recording_skipped** (reason: no_context / self_referral / already_attributed / window_elapsed) -- N/A -- no Stage 2 metric measures skipped attributions directly; retained so the growth funnel's drop-off points remain observable rather than silently absorbed into "no referral"

## Acceptance Criteria

**FEAT-24.SPEC-004-AC-01:** Given a visitor followed Sam's personal referral link 3 days ago and completes FEAT-01.SPEC-003 today, when this automation fires, then a Household Referral record is created with referring_household set to Sam's household, referring_member_link set to Sam's link, new_household set to the new household, and upgraded set to false.

**FEAT-24.SPEC-004-AC-02:** Given a new household's creation carried no referral link context at all, when this automation fires, then no Household Referral record is created and FEAT-01.SPEC-003 completes exactly as it would for any other new household.

**FEAT-24.SPEC-004-AC-03:** Given a visitor opened their own household's own personal link before creating a second household under a different account, when this automation fires, then no referral is recorded, since the referring household and the new household are the same.

**FEAT-24.SPEC-004-AC-04:** Given a new household already carries a Household Referral record from an earlier link open, when a second referral context somehow reaches this automation for the same new household, then no second record is created and the original record is left unchanged.

**FEAT-24.SPEC-004-AC-05:** Given a visitor opened a link 31 days before completing FEAT-01.SPEC-003, when this automation fires, then no referral is recorded, since the 30-day counting window has elapsed.

**FEAT-24.SPEC-004-AC-06:** Given a visitor opened a link exactly 30 days before completing FEAT-01.SPEC-003, when this automation fires, then the referral is recorded, since the window is inclusive of day 30.

**FEAT-24.SPEC-004-AC-07:** Given the referring household was deleted after the link was opened but before the new household completed setup, when this automation fires, then no referral is recorded and the outcome is treated as having no live referral context.

**FEAT-24.SPEC-004-AC-08:** Given two different visitors each complete FEAT-01.SPEC-003 having opened the same personal link, when this automation fires for each, then two separate Household Referral records are created, both attributing to the same referring household and link.

**FEAT-24.SPEC-004-AC-09:** Given a Household Referral record is created, when the write completes, then FEAT-24.SPEC-007 (Referral Joined Notification) fires for the Member Profile that owns the referring_member_link.

**FEAT-24.SPEC-004-AC-10:** Given a duplicate household-creation signal arrives for a new household that already has a Household Referral record, when this automation runs a second time, then it makes no change and creates no duplicate record.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 6 (recorded, no context, self-referral blocked, already attributed, window elapsed, automation failure) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Automation Spec: Referral Upgrade Tracking

## Overview

**Name:** Referral Upgrade Tracking
**ID:** FEAT-24.SPEC-005
**Type:** Automation
**Purpose:** Sets a recorded Household Referral's `upgraded` flag when the referred household's own Subscription becomes paid.
**Parent Feature:** FEAT-24 -- Invite Another Household

## Scope and Non-Goals

**In Scope:**
- Detecting when a referred household's Subscription transitions to paid
- Updating the matching Household Referral record's `upgraded` field to true
- Leaving `upgraded` at false for a referred household that never upgrades, or that upgrades and later reverts to free

**Non-Goals:**
- Deciding whether a household's Subscription becomes paid, or processing the payment itself -- owned entirely by FEAT-14 (Subscription & Billing Management); this automation only reads the outcome
- Creating the Household Referral record itself -- owned by FEAT-24.SPEC-004 (Household Referral Recording); this automation only updates an already-existing record
- Reverting `upgraded` back to false if the household later downgrades or cancels -- excluded per the dependency map's Household Referral entity definition, which lists `upgraded` as "whether the new household went on to pay" without a stated downgrade-reversal behavior; once true, the flag reflects that the household did go on to pay at least once, which is the fact the growth metric depends on
- Any reward, credit, or discount tied to a referred household's upgrade -- excluded per scope-boundaries.md SC-10: no money passes between households in this feature

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Household's subscription tier changes to paid | FEAT-14.SPEC-008 (Apply Subscription Change), "Upgrade applied" outcome | Fires whenever a household's Subscription tier is set to paid, for any household in the product | Household reference whose Subscription changed, new tier (paid) |

## Processing Logic

1. Receive the Household reference whose Subscription just changed to paid.
2. Check whether this Household is the `new_household` on any existing Household Referral record.
3. If no matching record exists, stop -- this household was never referred, or its referral fell outside the eligibility rules at creation time (FEAT-24.SPEC-006), so there is nothing to update.
4. If a matching record exists and its `upgraded` field is already true, stop -- no change is needed.
5. If a matching record exists and its `upgraded` field is false, set it to true.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Upgrade recorded | The subscribing household has a matching Household Referral record with `upgraded` currently false | Household Referral -- `upgraded` set to true | None directly visible on any screen; the referring household's derived paying share (FEAT-24.SPEC-001, Data Notes) reflects the change on next view | FEAT-24.SPEC-001 |
| Already recorded | A matching record exists and `upgraded` is already true | None | None | -- |
| No matching referral | The subscribing household was never a referred household | None | None -- this is the ordinary path for the large majority of subscription upgrades, which have no referral to update | -- |
| Automation failure | An internal processing error prevents the update | Household Referral record's `upgraded` field retains its prior value | None user-visible -- FEAT-14's own upgrade confirmation (FEAT-14.SPEC-010) is unaffected, since this automation's failure never blocks or alters the subscription change itself | -- |

## Data Model

**Reads:** Household Referral -- existing records, matched by new_household, to find the one (if any) belonging to the subscribing household; Subscription -- read indirectly through FEAT-14.SPEC-008's trigger, which supplies the tier change itself rather than requiring a separate read here.
**Creates:** None.
**Updates:** Household Referral -- `upgraded` field only, set from false to true. This is the only field this feature ever updates on the record (Feature Breakdown Brief, Entity-Lifecycle Coverage Matrix).
**Deletes:** None.

## Business Rules

- This automation is the exclusive writer of the Household Referral's `upgraded` field -- no screen or role sets it directly (FEAT-24.SPEC-006, Authorization Rules).
- A household's Subscription change never waits on this automation -- FEAT-14's own upgrade flow and confirmation are entirely unaffected by whether a matching referral exists or by this automation's outcome, since referral tracking is a side effect, not a precondition, of any subscription change (mirrors FEAT-24.SPEC-004's equivalent rule for household creation).
- Once `upgraded` is set to true, it is never reverted, per this feature's decision authority: the dependency map's Household Referral entity defines no downgrade-reversal behavior, and the record functions as the product's own history of referral-driven growth (dependency map, Household Referral Data Sensitivity), not a live subscription-status mirror.

## Edge Cases

- **Household upgrades, downgrades, and upgrades again** -- The `upgraded` flag is already true from the first upgrade and needing no further change; the second upgrade's trigger reaches Step 4 of Processing Logic and stops as "already recorded," leaving the flag as-is.
- **Household referral record does not yet exist when the subscription upgrade fires (a rare ordering case where FEAT-24.SPEC-004 has not yet completed for a household that upgrades within moments of completing its own setup)** -- No matching record is found at this moment, so no update occurs and none is queued; since Household Referral records are created once, synchronously, at household completion (FEAT-24.SPEC-004), and a paid-tier upgrade cannot occur before a household exists, this ordering is not reachable in practice, but the "no matching referral" outcome handles it safely if it ever occurred.
- **Concurrent trigger firing (two different referred households upgrade to paid at effectively the same time)** -- Each triggers its own independent evaluation against its own Household Referral record; the two updates target different records and do not interact.
- **Trigger fires while a previous run is in flight for the same household (e.g., a duplicate upgrade signal from FEAT-14.SPEC-008 after a retried payment confirmation)** -- Step 4's already-true check makes a second run for the same household a no-op once the first run has completed; if both runs somehow evaluate before either writes, the update itself (setting `upgraded` to true) is idempotent, so no inconsistent state results.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-14.SPEC-008 (Apply Subscription Change) | Triggered by (inbound) | The "Upgrade applied" outcome fires this automation |
| FEAT-24.SPEC-004 (Household Referral Recording) | References (inbound) | Supplies the Household Referral record this automation updates |
| FEAT-24.SPEC-006 (Household Referral Rules) | References (inbound) | Confirms `upgraded` is the only field this feature updates on the record, and that this automation is its exclusive writer |
| FEAT-24.SPEC-001 (Invite Another Household Screen) | Affects (outbound) | The referring household's derived paying share reflects an updated `upgraded` flag |

## Analytics and Success Signals

- **referred_household_upgraded** (days_since_referral_created: derived) -- supports success-metrics.md: "Household-to-Household Invitation Growth"

## Acceptance Criteria

**FEAT-24.SPEC-005-AC-01:** Given a referred household's Subscription changes to paid, when this automation fires, then the matching Household Referral record's `upgraded` field is set to true.

**FEAT-24.SPEC-005-AC-02:** Given a household's Subscription changes to paid but the household was never referred, when this automation fires, then no Household Referral record is found or updated.

**FEAT-24.SPEC-005-AC-03:** Given a referred household's `upgraded` flag is already true, when its Subscription changes to paid again (e.g., after a re-subscription), then this automation makes no change, since the flag is already true.

**FEAT-24.SPEC-005-AC-04:** Given a referred household upgrades, later downgrades to free, when this automation is inspected after the downgrade, then the `upgraded` flag remains true, since it is never reverted.

**FEAT-24.SPEC-005-AC-05:** Given two different referred households upgrade to paid at effectively the same time, when this automation fires for each, then each household's own Household Referral record is updated independently and correctly.

**FEAT-24.SPEC-005-AC-06:** Given a duplicate upgrade signal arrives for a household whose referral record was just updated, when this automation runs a second time, then no error occurs and the flag remains true.

**FEAT-24.SPEC-005-AC-07:** Given an internal processing error prevents this automation's update, when the failure occurs, then FEAT-14's own upgrade confirmation to the referred household proceeds unaffected.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 4 (recorded, already recorded, no matching referral, automation failure) | 4 |
| Business Rules | 3 | 3 |
| Edge Cases | 4 | 4 |



# Logic/Rule Spec: Household Referral Rules

## Overview

**Name:** Household Referral Rules
**ID:** FEAT-24.SPEC-006
**Type:** Logic/Rule
**Purpose:** Governs the one-link-per-member limit, single-attribution and no-self-referral rules, the 30-day counting window, the already-has-a-household disposition, and who may see or act on Household Referral data.
**Parent Feature:** FEAT-24 -- Invite Another Household
**Governed Entity:** Household Referral

## Scope and Non-Goals

**In Scope:**
- Field-level rules for every field on the Household Referral record
- The one reusable personal link per adult member limit
- Single-attribution (a new household attributed to at most one referring household, ever) and no-self-referral
- The 30-day counting window between a link's most recent open and the new household's creation
- The already-has-a-household disposition for a visitor who opens a link while already a member of any household
- Authorization for every action the product defines on the Household Referral record and the personal referral link, per role

**Non-Goals:**
- The step-by-step processing that creates the Household Referral record -- owned by FEAT-24.SPEC-004 (Household Referral Recording), which enforces the rules defined here
- The step-by-step processing that updates the `upgraded` field -- owned by FEAT-24.SPEC-005 (Referral Upgrade Tracking), which enforces the rule defined here
- Rewards, credits, or discounts tied to a referral -- excluded per scope-boundaries.md SC-10: this feature records referrals without paying for them
- General household-membership rules (invitations, organiser hand-over, leaving) -- owned by FEAT-09 (Household Invitations & Membership); this spec governs only household-to-household referral, a distinct mechanism per the Feature Breakdown Brief's Shared Context

## Governed Entity

**Entity:** Household Referral
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| referring_household | reference (Household) | The household whose member's link was used |
| referring_member_link | reference (Personal Referral Link) | The specific adult member's personal link that was opened |
| new_household | reference (Household) | The household created from the link |
| created_date | date | The date the new household completed setup |
| upgraded | boolean | Whether the new household's Subscription has gone on to become paid |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-24.SPEC-001 | Invite Another Household Screen | Authorization on screen entry (Household Referrals column); one-link-per-member limit relied on for display |
| FEAT-24.SPEC-002 | Referral Welcome Screen | Already-has-a-household disposition on load; link-open moment recorded for the counting window |
| FEAT-24.SPEC-003 | Personal Referral Link Provisioning | One-link-per-member limit enforced at creation time; authorization on who may trigger provisioning |
| FEAT-24.SPEC-004 | Household Referral Recording | Single-attribution, no-self-referral, and 30-day counting window enforced at record-creation time |
| FEAT-24.SPEC-005 | Referral Upgrade Tracking | `upgraded` field derivation enforced as this automation's exclusive write |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| referring_household | Required; must reference an existing Household | Always | On record creation (FEAT-24.SPEC-004) | N/A -- system-only field, no user-facing form exists for this record; creation is refused silently and no referral is recorded (see Cross-Field Rules) | Yes |
| referring_member_link | Required; must reference an existing personal referral link owned by an adult member of referring_household | Always | On record creation | N/A -- system-only field; see above | Yes |
| new_household | Required; must reference an existing Household; no validation beyond that reference and the cross-field rules below | Always | On record creation | N/A -- system-only field; see above | Yes |
| created_date | Required; set automatically to the date the new household completed setup; no user input | Always | On record creation | N/A -- system-set field, never entered by any role | Yes |
| upgraded | Required; boolean; defaults to false | Always | On record creation, and on update by FEAT-24.SPEC-005 | N/A -- system-set field, never entered by any role | Yes |

This entity is created and updated entirely by automations (FEAT-24.SPEC-004, FEAT-24.SPEC-005); it exposes no user-facing entry form, so every field's "error message" is N/A in the sense of user-visible text -- an invalid combination instead results in no record being created, per the Cross-Field Rules and Business Rules below.

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| No self-referral | referring_household, new_household | new_household must not equal referring_household | N/A -- enforced silently by FEAT-24.SPEC-004 stopping before record creation; the visitor sees the already-has-a-household disposition (FEAT-24.SPEC-002) rather than an error, since they never reach a state where self-referral could otherwise occur |
| Single attribution | new_household | A given new_household value may appear on at most one Household Referral record, ever, across the product's lifetime | N/A -- enforced silently by FEAT-24.SPEC-004; a second candidate referral for an already-attributed household is simply not recorded |
| 30-day counting window | created_date, referring_member_link (via its most-recent-open moment) | created_date must fall within 30 days (inclusive) of the referring_member_link's most recent open moment | N/A -- enforced silently by FEAT-24.SPEC-004; a household created outside the window is simply not recorded, with no error surfaced to the new household's own setup |
| One link per member | referring_member_link (via the owning Member Profile) | A given adult Member Profile owns at most one personal referral link, ever | N/A -- enforced at link-creation time (FEAT-24.SPEC-003), which returns the existing link rather than creating a second one; no error state exists because a second creation attempt is never a failure, only a no-op |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Provision (create) own personal referral link | Maya (Organiser), Sam (Other Adult Member) | Always, for the requesting adult member's own link only | Control (the Invite Another Household Screen) is not shown at all to Jordan (young kid profile, no login), Jordan (older kid, limited login), or Riley (Operator) -- the Household Referrals column of the Access Matrix gives each of these rows None |
| View own personal referral link and joined-families count | Maya (Organiser), Sam (Other Adult Member) | Always, for their own household's link and count | Same as above -- screen not shown to Jordan (either row) or Riley |
| View an individual Household Referral record's detail | No role | Never -- no such capability exists in the product | No screen or control anywhere exposes a single referral record's detail; only the aggregate count and the derived paying share are shown (Feature Breakdown Brief, Entity-Lifecycle Coverage Matrix, Read (single): N/A) |
| Create a Household Referral record directly | No role | Never -- creation happens only through FEAT-24.SPEC-004's automated attribution at household completion | No user-facing action exists to create a referral record; the record is written once by the system when a new household's setup completes (dependency map, Household Referral Contention) |
| Edit any field on an existing Household Referral record | No role | Never -- the record is written once and never edited by any role beyond the system's own `upgraded` update (FEAT-24.SPEC-005) | No screen or control exposes any edit path; the dependency map's Household Referral Contention note states the record is never edited by any role |
| Delete or archive a Household Referral record | No role | Never -- no deletion or purge mechanism is defined for this record (Feature Breakdown Brief, Non-Goals) | No screen or control exposes a delete path; retention is indefinite, as the product's only record of referral-driven growth |
| View the referral welcome page (personal referral link, unauthenticated) | Unauthorized visitor | Only for a link addressed to an existing personal referral link; no household session active | -- (this is the intended, unrestricted entry point for the feature) |
| Start household setup from a referral welcome page | Unauthorized visitor | Only when the visitor is not already signed in as a member of any household | A visitor already signed in as a member of any household sees "You already have a household on Plateful." instead of a start-setup option, and the setup path is not offered |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| created_date | The date the new household's setup completes (FEAT-01.SPEC-003 success) | On record creation only | No |
| upgraded | false | On record creation | No (only ever changed by FEAT-24.SPEC-005's system write, never by any role) |
| referring_member_link's most-recent-open moment | The moment the referral welcome page (FEAT-24.SPEC-002) most recently resolved that link for a given visit | Recalculated on every fresh open of a still-valid link | No (system-derived; not user-editable) |

## Business Rules

- XBR-20: a new household set up from an invite link within 30 days is attributed to at most one referring household, never itself; someone who already has a household is told so and nothing is recorded.
- The counting window is 30 days, inclusive of day 30, measured from the referring link's most recent open to the new household's creation date -- a household created on day 31 or later falls outside the window.
- One reusable personal link exists per adult member for the life of their account; the link is never expired or rotated by this feature (product-features.md, Validation & Limits).
- The joined-families count and the derived paying share (shown on FEAT-24.SPEC-001) are computed from Household Referral records where referring_household matches the viewing member's household -- not filtered to the viewing member's own referring_member_link alone, since the Access Matrix grants both adult roles Full visibility of the household's referral results as a whole.
- No money, reward, or credit is exchanged for a referral (scope-boundaries.md SC-10); this spec governs attribution and visibility only.

## Edge Cases

- **A household's only adult member leaves and a different adult later becomes the sole adult** -- The departing member's personal referral link is unaffected by this spec; it belongs to that Member Profile, not to the household, and Household Referral records already attributed against that member's link remain unchanged (they attribute to referring_household, which persists independent of individual membership changes).
- **Two adult members of the same household each have their own personal referral link, and both are used to refer different new households in the same week** -- Each new household's Household Referral record independently records its own referring_member_link (Sam's or Maya's); the household's joined-families count on FEAT-24.SPEC-001 sums both, since the count aggregates by referring_household.
- **A visitor's session carries stale referral context after navigating away from the welcome page and back through browser history** -- The 30-day window is measured from the most recent resolved open of the link (FEAT-24.SPEC-002's own re-open behavior), so returning through history re-triggers a fresh resolution rather than reusing a stale timestamp.
- **The referring household is deleted after a referral has already been recorded (upgraded or not)** -- The existing Household Referral record is unaffected by this spec; cascade behavior for a referenced household's deletion is FEAT-18's responsibility, flagged in the Feature Breakdown Brief's Cross-Feature Touchpoints and not resolved here.
- **A household is created exactly at the 30-day boundary while the referring link was opened at a different time of day** -- The window compares calendar dates (created_date vs. the link's open date), inclusive of day 30, so time-of-day differences within the same calendar dates do not affect eligibility.
- **An adult member requests link creation twice in immediate succession before the first request has completed** -- The one-link-per-member limit's check-then-create step (enforced in FEAT-24.SPEC-003) ensures the second request finds the first request's link already persisted, rather than creating a duplicate.

## Acceptance Criteria

**FEAT-24.SPEC-006-AC-01:** Given a new household completes setup exactly 30 days after its referral link was opened, when FEAT-24.SPEC-004 evaluates it, then the referral is recorded, since the window is inclusive of day 30.

**FEAT-24.SPEC-006-AC-02:** Given a new household completes setup 31 days after its referral link was opened, when FEAT-24.SPEC-004 evaluates it, then no referral is recorded.

**FEAT-24.SPEC-006-AC-03:** Given a visitor opens their own household's own personal link, when they view FEAT-24.SPEC-002, then they see "You already have a household on Plateful." and no start-setup option, and no self-referral is ever recorded.

**FEAT-24.SPEC-006-AC-04:** Given a new household already carries a Household Referral record, when a second referral context reaches FEAT-24.SPEC-004 for the same new household, then no second record is created.

**FEAT-24.SPEC-006-AC-05:** Given Sam already has a personal referral link, when FEAT-24.SPEC-003 is triggered again for him, then his existing link is returned and no second link is created.

**FEAT-24.SPEC-006-AC-06:** Given Maya (Organiser) is signed in, when she looks for the Invite Another Household Screen, then it is available to her, per the Household Referrals column of the Access Matrix.

**FEAT-24.SPEC-006-AC-07:** Given Jordan (young kid profile, no login) has no path to sign in, when any attempt is made to reach the Invite Another Household Screen, then no such attempt is possible, since this role has no login at all.

**FEAT-24.SPEC-006-AC-08:** Given Riley (Operator, support) is signed in for a support session, when Riley looks for the Invite Another Household Screen or any referral data, then none is shown, per the Household Referrals column of the Access Matrix giving Riley None.

**FEAT-24.SPEC-006-AC-09:** Given Maya wants to see a single referral record's detail, when she looks at the Invite Another Household Screen, then no such detail view exists anywhere in the product -- only the aggregate count and derived paying share are shown.

**FEAT-24.SPEC-006-AC-10:** Given a Household Referral record exists, when any role attempts to edit or delete it directly, then no control for doing so exists anywhere in the product.

**FEAT-24.SPEC-006-AC-11:** Given a referred household's Subscription becomes paid, when FEAT-24.SPEC-005 updates the matching record, then `upgraded` is set to true, and no role can set this field directly themselves.

**FEAT-24.SPEC-006-AC-12:** Given Sam's household has referrals from both Maya's link and Sam's own link, when the joined-families count is computed for FEAT-24.SPEC-001, then it sums referrals from both links, since the count aggregates by referring_household.

**FEAT-24.SPEC-006-AC-13:** Given a household is deleted after having referred another household, when the existing Household Referral record is inspected, then it remains unchanged by this spec, since cascade behavior on deletion is FEAT-18's responsibility.

**FEAT-24.SPEC-006-AC-14:** Given two adult members of the same household each hold their own personal referral link, when both links are used to refer different new households, then two separate Household Referral records are created, one per new household, both attributing to the same referring household.

**FEAT-24.SPEC-006-AC-15:** Given an unauthorized visitor (not signed in, no household) opens a valid personal referral link, when the welcome page loads, then they are able to proceed to start-setup, since this role and state combination is the intended, unrestricted entry point for the feature.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 4 | 4 |
| Authorization Rules | 8 | 8 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Notification Spec: Referral Joined Notification

## Overview

**Name:** Referral Joined Notification
**ID:** FEAT-24.SPEC-007
**Type:** Notification
**Purpose:** Tells the inviting member, in-app, when a family they invited finishes setting up its household.
**Parent Feature:** FEAT-24 -- Invite Another Household

## Scope and Non-Goals

**In Scope:**
- The in-app note delivered when a Household Referral record is created for the inviting member's link
- The batched variant when more than one referral completes for the same member on the same day
- Deduplication and delivery behavior for this notification (this notification defines no preference or quiet-hours surface -- see Non-Goals)

**Non-Goals:**
- Deciding whether a referral is eligible to be recorded -- owned by FEAT-24.SPEC-004 (Household Referral Recording) and governed by FEAT-24.SPEC-006 (Household Referral Rules); this notification begins only once a record has already been created
- Email or push delivery of this note -- product-features.md's Communications field for this feature names only an "in-app note," and Member Profile's notification_preferences field (dependency map) enumerates no email or push channel for referral events; adding one would exceed what the product definition establishes
- A dedicated on/off preference toggle for this notification -- Member Profile's notification_preferences field (dependency map) defines only plan-ready and nightly-nudge toggles; product-features.md names no separate control for this note, so none is invented here

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always, when a Household Referral record is created | Product-features.md's Communications field states this is delivered as "an in-app note"; the recipient is inside the product whenever they next open it, and the moment carries no urgency that would justify interrupting them through email or push (Feature Breakdown Brief, Non-Functional Notes, Responsiveness: this is not a real-time-critical surface) |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Household Referral record created | FEAT-24.SPEC-004 (Household Referral Recording) | Fires every time this automation creates a new Household Referral record | The referring_member_link's owning Member Profile (the recipient), the referring Household, and the new household's creation date |

## Audience and Preferences

**Recipients:** The one adult Member Profile that owns the referring_member_link used for this specific referral -- traced to the Household Referrals column of the Access Matrix in user-persona.md, which gives Maya and Sam Full access. Only the member whose own link produced the referral is notified, not every adult member of the household; a household with two members each holding their own link receives this note only for the member whose link was actually used for a given referral.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| N/A -- no dedicated preference toggle exists for this notification | -- | Always on | -- (product-features.md and the dependency map's Member Profile field list define no such control; see Non-Goals) |

**Quiet Hours:** N/A -- this notification is in-app only, delivered passively the next time the recipient opens the product rather than interrupting them, so no quiet-hours window applies; product-features.md defines quiet hours nowhere for this feature.

## Content Definition

**In-app:**
- **Title:** A family you invited has joined Plateful!
- **Body:** They've set up their own household using your invite link.
- **CTA:** See who's joined -- deep-links to FEAT-24.SPEC-001 (Invite Another Household Screen) for the recipient's own updated joined-families count

**Batched variant (2+ referrals complete for the same recipient on the same day):**
- **In-app title:** {count} families you invited have joined Plateful!
- **In-app body:** They've set up their own households using your invite link.
- **CTA:** See who's joined -- deep-links to FEAT-24.SPEC-001 (Invite Another Household Screen)

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {count} | Derived -- number of Household Referral records created for this recipient's referring_member_link on the same calendar day | 3 | Never empty -- the batched variant only renders with 2 or more same-day referrals |

No other placeholders are used: the content names no new household by name, consistent with the dependency map's Household Referral Data Sensitivity note that the record links two households by identity and date only, and that an unauthorized visitor sees only the inviter's first name, never the reverse.

## Delivery Rules

**Batching:** All Household Referral records created for the same recipient's referring_member_link on the same calendar day are delivered as one notification, using the batched variant when 2 or more complete that day. A referral completing on a later day produces its own, separate notification.
**Deduplication:** At most one notification per Household Referral record. FEAT-24.SPEC-004's single-attribution rule (FEAT-24.SPEC-006) already guarantees a given new household produces at most one record, so no record can ever trigger this notification twice.
**Retry on failure:** In-app delivery has no retry: the note is delivered the next time the recipient opens the product, so there is no failure mode analogous to a channel outage to retry against.
**Expiry:** This notification does not expire -- it remains available as an unread in-app note until the recipient views it, since it reports a completed fact (a referral was recorded) rather than a time-sensitive action the recipient must take.

## Edge Cases

- **The Household Referral record's referring_member_link owner leaves the household or is removed before this notification is delivered** -- The notification is still delivered to that same Member Profile if their account still exists and they can still sign in (e.g., they left this household but retain their own account); if the Member Profile itself is removed entirely (FEAT-18, household deletion or member removal), the notification is cancelled silently, since there is no longer a recipient to deliver it to.
- **Two referrals for the same recipient complete on the same day, seconds apart** -- The first referral's notification is held briefly for same-day batching per the Batching rule, and the second referral's completion is folded into the same batched notification rather than producing two separate in-app notes.
- **A referral is recorded, then the referring household is deleted before the recipient opens the app to see the note** -- The notification is still delivered as scheduled; it reports a fact about a completed referral, not a live state of the referring household, so the referring household's own subsequent deletion does not retract it. (This is a rare ordering case, since the recipient is themselves a member of the referring household in every case this feature defines.)
- **The recipient never opens the app** -- The notification has no expiry and remains queued as an unread in-app note indefinitely; since this is the sole channel and there is no fallback channel, delivery is guaranteed only in the sense that it is waiting whenever the recipient next opens the product.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-24.SPEC-004 (Household Referral Recording) | Triggered by (inbound) | A newly created Household Referral record fires this notification |
| FEAT-24.SPEC-001 (Invite Another Household Screen) | Navigation (outbound) | The CTA deep-links here, to the recipient's own updated joined-families count |
| FEAT-24.SPEC-006 (Household Referral Rules) | References (inbound) | Single-attribution guarantees this notification is never delivered twice for the same referral |

## Analytics and Success Signals

- **referral_joined_notification_delivered** (batched: yes / no; count) -- supports success-metrics.md: "Household-to-Household Invitation Growth"
- **referral_joined_notification_opened** (batched: yes / no) -- N/A -- no Stage 2 metric measures notification engagement directly for this feature; retained so this notification's own effectiveness at driving members back to FEAT-24.SPEC-001 remains observable

## Acceptance Criteria

**FEAT-24.SPEC-007-AC-01:** Given a Household Referral record is created for Sam's link, when this notification fires, then Sam receives an in-app note titled "A family you invited has joined Plateful!" with the body "They've set up their own household using your invite link."

**FEAT-24.SPEC-007-AC-02:** Given Sam receives this notification, when he taps "See who's joined", then he is taken to FEAT-24.SPEC-001 (Invite Another Household Screen), showing his updated joined-families count.

**FEAT-24.SPEC-007-AC-03:** Given Maya's household has two adult members each with their own link, and only Sam's link produced this referral, when the notification fires, then only Sam receives it -- Maya does not.

**FEAT-24.SPEC-007-AC-04:** Given three referrals complete for Sam's link on the same calendar day, when the notifications are delivered, then Sam receives one batched note titled "3 families you invited have joined Plateful!" rather than three separate notes.

**FEAT-24.SPEC-007-AC-05:** Given a referral completes for Sam today and another completes for him tomorrow, when the notifications are delivered, then today's produces one note and tomorrow's produces its own separate note -- they are never merged across days.

**FEAT-24.SPEC-007-AC-06:** Given Sam has not opened the product since a referral was recorded, when he eventually opens it, then the notification is still there, since it has no expiry.

**FEAT-24.SPEC-007-AC-07:** Given a Household Referral record has already triggered this notification once, when any process re-evaluates that same record, then no second notification is ever produced, since single-attribution guarantees the record itself is created only once.

**FEAT-24.SPEC-007-AC-08:** Given the referring household is deleted after this notification has already been recorded as fired but before Sam opens the app, when Sam eventually opens it, then the notification is still delivered as originally scheduled.

**FEAT-24.SPEC-007-AC-09:** Given Sam's Member Profile is removed entirely before this notification is delivered, when the removal completes, then the notification is cancelled silently, since no recipient remains to deliver it to.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (in-app) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on -- no toggle exists) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 4 | 4 |

---
document_type: spec
spec_type: screen
spec_id: FEAT-24.SPEC-001
spec_name: Invite Another Household Screen
spec_slug: invite-another-household-screen
parent_feature: FEAT-24
parent_feature_name: Invite Another Household
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

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

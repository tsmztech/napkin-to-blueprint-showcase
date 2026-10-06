---
document_type: spec
spec_type: screen
spec_id: FEAT-24.SPEC-002
spec_name: Referral Welcome Screen
spec_slug: referral-welcome-screen
parent_feature: FEAT-24
parent_feature_name: Invite Another Household
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

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

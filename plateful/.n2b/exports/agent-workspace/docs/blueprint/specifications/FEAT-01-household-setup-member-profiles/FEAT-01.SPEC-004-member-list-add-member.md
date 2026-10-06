---
document_type: spec
spec_type: screen
spec_id: FEAT-01.SPEC-004
spec_name: Member List & Add Member
spec_slug: member-list-add-member
parent_feature: FEAT-01
parent_feature_name: Household Setup & Member Profiles
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Screen Spec: Member List & Add Member

## Overview

**Name:** Member List & Add Member
**ID:** FEAT-01.SPEC-004
**Type:** Screen
**Purpose:** The organiser sees every household member and starts adding an adult or kid profile; other adult members view the list.
**Parent Feature:** FEAT-01 -- Household Setup & Member Profiles

## Scope and Non-Goals

**In Scope:**
- Listing every current Member Profile in the household
- Starting the add-adult or add-kid-profile flow (routing into FEAT-01.SPEC-005, with a detour through FEAT-01.SPEC-007 for kids)
- An entry point into inviting another adult (hands off to FEAT-09)
- Read-only viewing of the list for Sam (Other Adult Member)

**Non-Goals:**
- Editing an existing member's details -- handled by FEAT-01.SPEC-005 (Member Profile Detail), reached by tapping a member card
- Removing a member -- owned by Account & Data Management (FEAT-18) per the dependency map and XBR-16; this screen creates and lists members but never removes one
- Sending or managing the invitation itself -- owned by FEAT-09 (Household Invitations & Membership); this screen only offers the entry point

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-003 (Household Naming & Guided Setup Start) | Organiser continues from naming the household | Guided-setup wizard context (step indicator continues) |
| FEAT-01.SPEC-010 (Household Settings Hub) | Organiser or Sam taps "Members" | None -- list opens in its standalone (non-wizard) chrome |
| FEAT-09 (Household Invitations & Membership) | An accepted invitation completes | The list refreshes to include the new Member Profile |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen, every member's card | Add an adult or kid profile, invite another adult, tap into any member's detail | -- |
| Sam (Other Adult Member) | Full screen, every member's card | View only -- Add and Invite controls are not shown; tapping a card opens the member's detail in read-only mode (kid profiles) or Sam's own detail (edit access to his own notification preferences only, per FEAT-01.SPEC-016) | Attempting to reach Add/Invite by direct navigation shows "Only the organiser can add members" and returns to this list |
| Jordan (young kid profile, no login -- MVP) | No | No | N/A -- no login exists for this profile type |
| Jordan (older kid, limited login -- Later) | No | No | N/A -- household setup is not part of the older-kid login's entitlements |
| Riley (Operator, support) | View, only through FEAT-22 (from v1) | No | Riley never reaches this screen directly; the member list is visible only inside the separate read-only support view |
| Unauthenticated | No | No | Redirected to FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." No in-progress add-member entry exists on this list screen itself to preserve |

## Layout and Content

**Header (guided setup):** Wizard shell, "Step 2 of 8," title "Who's eating with you?" **Header (standalone, from Settings Hub):** Title "Members," back arrow to FEAT-01.SPEC-010.

**Body:** A vertical list of member cards, one per current Member Profile, each showing: display name, member type (Organiser / Other Adult Member / Kid), and up to three dietary-rule badges (e.g., "Peanut allergy," "Vegetarian") summarizing that member's rules. Below the list, two actions for the organiser only: "Add an adult," "Add a kid profile." A third action, "Invite a partner" (visible to the organiser only), hands off to FEAT-09.

For Sam, the same list renders without the Add/Invite actions beneath it.

An empty state (only the organiser's own card exists) shows a short prompt above the actions: "Add the people eating with you."

**Footer (guided setup only):** "Continue" button, enabled once at least the organiser's own profile exists (always true by this point).

### Responsive Behavior

- **Compact breakpoint:** Member cards stack full width, one per row; action buttons stack full width below the list.
- **Medium size class and above:** Member cards render in a two-column grid; action buttons remain full width in a row beneath the grid.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Member card | Tap | Navigate to FEAT-01.SPEC-005 (Member Profile Detail) for that member | Screen changes | Standard transition; Sam's tap opens the same screen in his role's view/edit scope |
| "Add an adult" (organiser only) | Tap | Navigate to FEAT-01.SPEC-005 in create mode, member_type pre-set to Other Adult Member | Screen changes | Standard transition |
| "Add a kid profile" (organiser only) | Tap | Navigate to FEAT-01.SPEC-007 (Parental Consent Confirmation) first | Screen changes | Standard transition; kid creation always detours through consent first |
| "Invite a partner" (organiser only) | Tap | Navigate to FEAT-09 (Household Invitations & Membership) | Screen changes | Standard transition, leaving this feature |
| "Continue" button (guided setup) | Tap | Proceeds to the next guided-setup step | Screen changes | Standard transition to FEAT-01.SPEC-008 |

### Accessibility Notes

- **Focus order:** Member cards in list order -> "Add an adult" -> "Add a kid profile" -> "Invite a partner" (organiser only) -> Continue (guided setup only).
- **Dynamic update announcement:** When a member is added and the list refreshes (returning from FEAT-01.SPEC-005 or an accepted invitation), the new card's addition is announced ("{display name} added").
- **Keyboard alternatives:** All actions are keyboard-reachable; no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Minimal (organiser only) | One card (the organiser), prompt "Add the people eating with you." | First arrival in guided setup | A member is added |
| Populated | Two or more member cards | At least one member beyond the organiser exists | Always the state once populated |
| Loading | Cards render with a brief inline placeholder | List first loads (standalone entry from Settings Hub) | Data loads (typically under a second) |
| Error | Banner: "Couldn't load your household's members. Try again." with retry | Member data fails to load | Retry succeeds |
| Offline/Degraded | Previously loaded list remains fully viewable; Add actions remain reachable, entering FEAT-01.SPEC-005/007 which queue their own saves per FEAT-01.SPEC-013 | Connectivity lost while this screen is open | Connectivity restored |

## Validation Rules

Validation governed by FEAT-01.SPEC-014 (Household & Member Field Validation Rules), specifically the 12-member household cap enforced when "Add an adult" or "Add a kid profile" is attempted at the limit.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Member card tap | FEAT-01.SPEC-005 (Member Profile Detail) | -- |
| "Add an adult" tap | FEAT-01.SPEC-005 (Member Profile Detail, create mode) | -- |
| "Add a kid profile" tap | FEAT-01.SPEC-007 (Parental Consent Confirmation) | -- |
| "Invite a partner" tap | FEAT-09 (Household Invitations & Membership) | FEAT-09 |
| "Continue" tap (guided setup) | FEAT-01.SPEC-008 (Weekly Budget & Schedule Setup) | -- |
| Back arrow (standalone) | FEAT-01.SPEC-010 (Household Settings Hub) | -- |

## Data Model

**Creates:** None directly -- member creation happens on FEAT-01.SPEC-005/007.
**Reads:** Member Profile -- display_name, member_type, status (Active members only), plus a summary of each member's Dietary Rule badges, for every member of the household.
**Updates:** None.
**Deletes:** None.

## Business Rules

- Attempting to add a 13th member profile is blocked per FEAT-01.SPEC-014 with a message citing the 12-member cap; the Add actions remain visible but the resulting create screen shows the block.
- Only the organiser can add members or send invitations (FEAT-01.SPEC-016).
- A member added via accepted invitation (FEAT-09) appears in this list automatically without the organiser taking any action here.

## Edge Cases

- **Household is at the 12-member cap** -- "Add an adult" and "Add a kid profile" remain visible (they are not hidden); tapping either surfaces the cap message from FEAT-01.SPEC-014 rather than opening the create screen.
- **Sam attempts to reach the add-member screen directly (e.g., a stale link)** -- He is shown "Only the organiser can add members" and returned to this list, per FEAT-01.SPEC-016.
- **An invitation is accepted while Maya has this screen open** -- The list updates to include the new member without requiring a manual refresh (live-updating, not a snapshot), consistent with the dependency map's low-contention note for Member Profile creation.
- **Network failure while loading the list** -- Error banner with retry; no member cards are shown until the retry succeeds.
- **Organiser navigates away and back mid-guided-setup** -- The list re-fetches current members; nothing added so far is lost.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-003 (Household Naming & Guided Setup Start) | Navigation (inbound) | Guided setup continues here |
| FEAT-01.SPEC-005 (Member Profile Detail) | Navigation (outbound) | Add-adult and member-detail entry |
| FEAT-01.SPEC-007 (Parental Consent Confirmation) | Navigation (outbound) | Add-kid-profile entry point |
| FEAT-01.SPEC-008 (Weekly Budget & Schedule Setup) | Navigation (outbound) | Next guided-setup step |
| FEAT-01.SPEC-010 (Household Settings Hub) | Navigation (inbound/outbound) | Standalone entry and return |
| FEAT-01.SPEC-013 (Setup Draft Persistence & Offline Queuing) | References (inbound) | Offline behavior for downstream add flows |
| FEAT-01.SPEC-014 (Household & Member Field Validation Rules) | References (inbound) | 12-member cap enforcement |
| FEAT-01.SPEC-016 (Household Setup Authorization Rules) | References (inbound) | Role-gated Add/Invite visibility |
| FEAT-09 (Household Invitations & Membership) | Navigation (outbound); Navigation (inbound) | Invite entry point; accepted invitations refresh this list |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| member_list_viewed | member count, role of viewer | Screen opens | N/A -- no Stage 2 metric measures list views directly; retained as a funnel step for First-Session Onboarding Completion analysis |
| member_added | member type (adult / kid) | A member is successfully created (event actually recorded by FEAT-01.SPEC-005/FEAT-01.SPEC-007, surfaced here as the list's refresh trigger) | supports success-metrics.md: "First-Session Onboarding Completion" |

## Acceptance Criteria

**FEAT-01.SPEC-004-AC-01:** Given Maya has just named her household, when this screen loads, then she sees her own member card and the "Add an adult," "Add a kid profile," and "Invite a partner" actions.

**FEAT-01.SPEC-004-AC-02:** Given Maya taps "Add an adult", then she is taken to FEAT-01.SPEC-005 in create mode with member_type pre-set to Other Adult Member.

**FEAT-01.SPEC-004-AC-03:** Given Maya taps "Add a kid profile", then she is taken to FEAT-01.SPEC-007 (Parental Consent Confirmation) before any kid profile detail screen.

**FEAT-01.SPEC-004-AC-04:** Given Sam opens this screen, when it loads, then he sees every member card but no "Add" or "Invite" actions.

**FEAT-01.SPEC-004-AC-05:** Given Maya's household already has 12 members, when she taps "Add an adult", then the resulting screen shows the 12-member-cap message from FEAT-01.SPEC-014 and no 13th member is created.

**FEAT-01.SPEC-004-AC-06:** Given Sam attempts to navigate directly to the add-member screen, then he is shown "Only the organiser can add members" and returned to this list.

**FEAT-01.SPEC-004-AC-07:** Given Maya has this screen open, when Sam's invited partner accepts their invitation via FEAT-09, then the new member's card appears on Maya's list without her needing to refresh manually.

**FEAT-01.SPEC-004-AC-08:** Given the member list fails to load, when the screen opens, then the banner "Couldn't load your household's members. Try again." appears with a retry option.

**FEAT-01.SPEC-004-AC-09:** Given Maya is offline, when she taps "Add an adult", then she is still taken to FEAT-01.SPEC-005, which queues the save per FEAT-01.SPEC-013.

**FEAT-01.SPEC-004-AC-10:** Given Maya has added all household members in guided setup, when she taps "Continue", then she is taken to FEAT-01.SPEC-008 (Weekly Budget & Schedule Setup).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 5 (minimal, populated, loading, error, offline) | 5 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |

---
document_type: spec
spec_type: screen
spec_id: FEAT-01.SPEC-010
spec_name: Household Settings Hub
spec_slug: household-settings-hub
parent_feature: FEAT-01
parent_feature_name: Household Setup & Member Profiles
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 18
---

# Screen Spec: Household Settings Hub

## Overview

**Name:** Household Settings Hub
**ID:** FEAT-01.SPEC-010
**Type:** Screen
**Purpose:** The organiser revisits and edits any setup area later; other adult members view household facts and link out to their own preferences.
**Parent Feature:** FEAT-01 -- Household Setup & Member Profiles

## Scope and Non-Goals

**In Scope:**
- Displaying the household's current facts: name, member count, budget, schedule, plan-arrival day/time
- Re-opening every setup screen in edit mode (household name, members, budget & schedule)
- Editing the plan-arrival day and time inline on this hub (organiser only), governed by FEAT-07.SPEC-004
- Linking out to units/currency/locale (FEAT-16), notification preferences (FEAT-07, FEAT-13), support access record (FEAT-22), and calendar connection (FEAT-21, Later)
- Linking out to invitation management (FEAT-09.SPEC-001), organiser hand-over initiation (FEAT-09.SPEC-003), a pending hand-over request addressed to the signed-in adult (FEAT-09.SPEC-004), and leaving the household (FEAT-09.SPEC-005)
- Read-only display of household facts for Sam, plus his own link to his own preferences

**Non-Goals:**
- Editing units, currency, or aisle names directly -- owned by FEAT-16; this hub only links out to it
- Performing invitation sending/revoking, the organiser hand-over transfer itself, or member removal -- owned by FEAT-09 and FEAT-18; this hub only links to those screens, which own the actions and their own authorization checks
- Billing management -- owned by FEAT-14; not exposed from this hub at all, since Billing access is None for everyone but the organiser through FEAT-14's own screens

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) | Existing organiser or adult member signs in to an account with a household | None |
| FEAT-01.SPEC-002 (Password Recovery) | Reset completes for an account with a household | None |
| FEAT-01.SPEC-009 (Setup Complete & Next Steps) | Organiser returns to the app after completing guided setup without choosing a next step | None |
| Any screen | Organiser or Sam navigates to "Household settings" from the product's persistent navigation | None |
| FEAT-09.SPEC-012 (Invitation Accepted Confirmation) | Organiser taps "View household" | None |
| FEAT-09.SPEC-013 (Member Left Household Notification) | Organiser taps "View household" | None |
| FEAT-09.SPEC-004 (Organiser Hand-Over Acceptance) | New organiser taps "Go to Household Settings" on the Success state, or the recipient confirms a decline, or taps "Back to Household" on the withdrawn state | None -- the hub loads with the viewer's current role (organiser access after a completed hand-over) |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen, all facts and edit entry points | Edit household name, members, budget & schedule, plan-arrival day/time; open all linked-out settings; manage invitations (FEAT-09.SPEC-001); hand over the organiser role (FEAT-09.SPEC-003) | -- |
| Sam (Other Adult Member) | Full screen, all household facts (read-only) | View only for household-owned facts including plan-arrival day/time; can open his own notification preferences (FEAT-07, FEAT-13) to edit them; can act on a hand-over request addressed to him (FEAT-09.SPEC-004) when one is pending; can leave the household (FEAT-09.SPEC-005) | Edit entry points for household name, members, budget, schedule, and plan-arrival are not shown to Sam; a direct navigation attempt to one shows "Only the organiser can change this" and returns here. "Household Invitations" and "Hand over organiser role" are not shown to Sam; a direct navigation attempt shows the destination screen's own denial message and returns here. |
| Jordan (young kid profile, no login -- MVP) | No | No | N/A -- no login exists for this profile type |
| Jordan (older kid, limited login -- Later) | No | No | N/A -- household setup is not part of the older-kid login's entitlements |
| Riley (Operator, support) | View, only through FEAT-22 (from v1) | No | Riley never reaches this screen directly; equivalent household facts are visible only inside the separate read-only support view |
| Unauthenticated | No | No | Redirected to FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." No in-progress edit exists on this hub screen itself to preserve |

## Layout and Content

**Header:** Title "Household settings," with the household name as a subtitle.

**Body:** A vertical list of setting rows, each showing its current summary value and (organiser only) a chevron indicating it opens an edit screen:
- "Household name" -- current name; opens FEAT-01.SPEC-003 in edit mode (organiser only)
- "Members" -- member count; opens FEAT-01.SPEC-004 (organiser: full access; Sam: read-only)
- "Budget & schedule" -- current budget and count of time-constrained nights; opens FEAT-01.SPEC-008 in edit mode (organiser only)
- "Plan arrival" -- current day and time-slot summary (e.g., "Sunday" with the chosen Evening slot from platform parameter: `plan-arrival-time-slots`); tapping the row (organiser only) expands an inline day/time picker in place, governed by FEAT-07.SPEC-004; Sam sees the same summary as a plain read-only fact, no chevron
- "Units, currency & aisles" -- links to FEAT-16 (organiser only, per that feature's own access rules)
- "Notification preferences" -- opens the signed-in adult's own FEAT-01.SPEC-005 (Member Profile Detail) at its plan-ready and nightly-nudge toggles, whose rules are owned by FEAT-07 and FEAT-13 (every adult, own preferences only)
- "Household Invitations" -- opens FEAT-09.SPEC-001 (organiser only); absent from Sam's list entirely
- "Hand over organiser role" -- opens FEAT-09.SPEC-003 (organiser only); absent from Sam's list entirely
- Pending hand-over entry -- shown only to an Other Adult Member who currently has a hand-over request addressed to them: "{organiser display name} wants to make you the organiser" opens FEAT-09.SPEC-004; absent for everyone else, including the organiser herself, since only one such request can exist at a time (XBR-15) and it is addressed to exactly one recipient
- "Leave household" -- opens FEAT-09.SPEC-005 (Other Adult Member only); absent from the organiser's list entirely, per FEAT-09.SPEC-005's XBR-15 gate
- "Support access record" -- links to FEAT-22's record of when and why support viewed the household (organiser: full view; not shown to Sam, per the Access Matrix's Support View column)
- "Connect calendar" (Later phase) -- links to FEAT-21 (organiser only)

For Sam, rows without an organiser-only edit path render as plain informational rows (no chevron); "Notification preferences," his pending hand-over entry (when one is addressed to him), and "Leave household" remain live links to their own destinations.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Rows stack full width, one per row.
- **Medium size class and above:** Rows remain single-column but cap at a consistent platform-wide content width, horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| "Household name" row (organiser only) | Tap | Navigate to FEAT-01.SPEC-003 in edit mode | Screen changes | Standard transition |
| "Members" row | Tap | Navigate to FEAT-01.SPEC-004 | Screen changes | Standard transition; organiser sees full access, Sam sees read-only |
| "Budget & schedule" row (organiser only) | Tap | Navigate to FEAT-01.SPEC-008 in edit mode | Screen changes | Standard transition |
| "Plan arrival" row (organiser only) | Tap | Expands an inline day/time picker (day selector + time-slot selector, values from platform parameter: `plan-arrival-time-slots`) | Row expands in place | Picker appears inline, focus moves to the day selector |
| Day selector (plan-arrival picker, organiser only) | Select | Sets the pending day component of plan_arrival_day_time | Selector shows chosen day | Immediate |
| Time-slot selector (plan-arrival picker, organiser only) | Select | Sets the pending time-slot component of plan_arrival_day_time | Selector shows chosen slot | Immediate |
| "Save" button (plan-arrival picker, organiser only) | Tap | Validate and save plan_arrival_day_time per FEAT-07.SPEC-004 (day and time-slot must both be present) | Inline saving confirmation; row collapses back to summary on success | Success: toast "Plan arrival updated." Failure: inline error ("Choose both a day and a time for your plan to arrive." when the pair is incomplete), picker stays open with the pending selection retained |
| "Units, currency & aisles" row (organiser only) | Tap | Navigate to FEAT-16 | Screen changes | Standard transition, leaving this feature |
| "Notification preferences" row | Tap | Navigate to the signed-in member's own FEAT-01.SPEC-005 (Member Profile Detail), Notification preferences toggles (rules owned by FEAT-07/FEAT-13) | Screen changes | Standard transition |
| "Household Invitations" row (organiser only) | Tap | Navigate to FEAT-09.SPEC-001 | Screen changes | Standard transition, leaving this feature |
| "Hand over organiser role" row (organiser only) | Tap | Navigate to FEAT-09.SPEC-003 | Screen changes | Standard transition, leaving this feature |
| Pending hand-over entry (Other Adult Member with an addressed request) | Tap | Navigate to FEAT-09.SPEC-004 | Screen changes | Standard transition, leaving this feature |
| "Leave household" row (Other Adult Member only) | Tap | Navigate to FEAT-09.SPEC-005 | Screen changes | Standard transition, leaving this feature |
| "Support access record" row (organiser only) | Tap | Navigate to FEAT-22's support-visit record | Screen changes | Standard transition, leaving this feature |
| "Connect calendar" row (organiser only, Later) | Tap | Navigate to FEAT-21 | Screen changes | Standard transition, leaving this feature |

### Accessibility Notes

- **Focus order:** Rows in the order listed above, top to bottom; when the plan-arrival picker is expanded, focus order continues into the day selector, then the time-slot selector, then Save, before resuming the remaining rows.
- **Role-based visibility announcement:** Rows not shown to Sam are simply absent from the page structure (not present-but-hidden), so no announcement of a missing control is needed. The pending hand-over entry appears and disappears the same way -- absent from the structure, not hidden -- so it needs no separate "new item" announcement beyond the normal screen-load read-out.
- **Picker expand/collapse announcement:** The plan-arrival picker expanding or collapsing, and the "Plan arrival updated" confirmation, are announced.
- **Keyboard alternatives:** All actions are keyboard-reachable; no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loaded (default) | All applicable rows shown with current summary values | Screen opens and household data loads | Always the state once loaded |
| Loading | Rows render with a brief inline placeholder | Screen first opens | Data loads (typically under a second) |
| Error | Banner: "Couldn't load your household settings. Try again." with retry | Household data fails to load | Retry succeeds |
| Offline/Degraded | Previously loaded summary values remain fully viewable; edit entry points remain reachable, leading to screens that queue their own saves per FEAT-01.SPEC-013 | Connectivity lost while this screen is open | Connectivity restored |

## Validation Rules

**Option B -- Inline (the plan-arrival picker is this screen's only direct input; every other row displays summaries and links only):**

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| plan_arrival_day_time (day + time-slot pair) | Both a day and a time-slot must be present together; allowed values governed by FEAT-07.SPEC-004 (platform parameter: `plan-arrival-time-slots`) | On Save, in the plan-arrival picker | "Choose both a day and a time for your plan to arrive." |
| All other rows | No direct input; all editing happens on the destination screens | -- | N/A -- no validation applies here |

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| "Household name" row tap | FEAT-01.SPEC-003 (Household Naming & Guided Setup Start, edit mode) | -- |
| "Members" row tap | FEAT-01.SPEC-004 (Member List & Add Member) | -- |
| "Budget & schedule" row tap | FEAT-01.SPEC-008 (Weekly Budget & Schedule Setup, edit mode) | -- |
| "Plan arrival" row tap, Save | -- (inline picker on this screen; no navigation) | -- |
| "Units, currency & aisles" row tap | FEAT-16.SPEC-001 (Units & Currency Settings) | FEAT-16 (Units, Currency & Locale Configuration) |
| "Notification preferences" row tap | FEAT-01.SPEC-005 (Member Profile Detail, own profile, Notification preferences toggles) | -- (rules owned by FEAT-07 / FEAT-13) |
| "Household Invitations" row tap | FEAT-09.SPEC-001 (Household Invitations Manager) | FEAT-09 (Household Invitations & Membership) |
| "Hand over organiser role" row tap | FEAT-09.SPEC-003 (Organiser Hand-Over Initiation) | FEAT-09 (Household Invitations & Membership) |
| Pending hand-over entry tap | FEAT-09.SPEC-004 (Organiser Hand-Over Acceptance) | FEAT-09 (Household Invitations & Membership) |
| "Leave household" row tap | FEAT-09.SPEC-005 (Leave Household) | FEAT-09 (Household Invitations & Membership) |
| "Support access record" row tap | FEAT-22.SPEC-003 (Household Support Access Record) | FEAT-22 (Operator Read-Only Support Access) |
| "Connect calendar" row tap (Later) | FEAT-21.SPEC-001 (Calendar Connection Settings) | FEAT-21 (Family Calendar Sync) |

## Data Model

**Creates:** None.
**Reads:** Household -- household_name, weekly_budget, weekly_schedule (summarized), plan_arrival_day_time; Household.pending_organiser_handover (the workflow marker defined in FEAT-09.SPEC-003, read to determine whether a pending hand-over entry addressed to the signed-in Other Adult Member should render); Member Profile -- count of Active members; Support Request -- access_record summary, for the support-access row.
**Updates:** Household -- plan_arrival_day_time (organiser only, per FEAT-07.SPEC-004). All other edits happen on the destination screens this hub links to.
**Deletes:** None.

## Business Rules

- Only the organiser sees edit entry points for household name, members (add/invite), budget & schedule, and plan-arrival day/time, per FEAT-01.SPEC-016.
- Every adult sees and controls only their own notification preferences from this hub, per XBR-13.
- The support-access record shown here is the organiser's View-level visibility into FEAT-22's read-only support access, per XBR-14 -- Maya can see when and why Riley viewed the household, but this hub never grants her any control over that access itself.
- The allowed values, default, and save validation for plan_arrival_day_time are governed by FEAT-07.SPEC-004; this hub enforces them inline but does not define them, per that spec's Business Rules ("FEAT-01.SPEC-010's edit screen ... defer[s] to it rather than duplicating the value rules").
- "Household Invitations," "Hand over organiser role," and "Leave household" link to FEAT-09's own screens, which own the underlying actions and their own authorization checks (FEAT-09.SPEC-011); this hub's row-visibility rules mirror those checks so a control is never shown that the destination screen would itself reject.
- A pending hand-over entry appears only for the specific Other Adult Member the outstanding request names (Household.pending_organiser_handover); no other member, and never the organiser herself, sees it, per XBR-15's single-outstanding-request rule.

## Edge Cases

- **Sam attempts to reach an organiser-only edit screen via direct navigation** -- Shown "Only the organiser can change this" and returned to this hub.
- **Household data fails to load** -- Error banner with retry; no stale or partial summary values are shown in place of failed data.
- **Organiser edits budget from another device while this hub is open** -- The summary value updates to reflect the change without requiring a manual refresh, consistent with this entity's low-contention profile; a failed save on the other device leaves this hub's displayed value unchanged.
- **Calendar connection (Later phase) is not yet available for a v1 household** -- The "Connect calendar" row is simply absent until FEAT-21 ships; this is a phase gate, not a permission gate.
- **Organiser is offline and taps an edit row** -- Navigation still proceeds to the destination screen, which itself surfaces the offline state per its own spec.
- **Organiser saves an incomplete plan-arrival pair (a day selected but no time-slot, or vice versa)** -- Save is blocked with "Choose both a day and a time for your plan to arrive."; the hub's displayed summary remains the previous complete value.
- **Organiser changes plan-arrival day/time from another device while this hub is open** -- The summary value updates to reflect the change without requiring a manual refresh, consistent with the Household entity's low-contention profile (the same pattern as the budget summary); a failed save on the other device leaves this hub's displayed value unchanged.
- **A pending hand-over request addressed to Sam is cancelled, declined, or accepted while he has this hub open** -- The pending hand-over entry is removed from the hub without further action from him, since the underlying marker it reads has been cleared.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) | Navigation (inbound) | Returning organiser/adult lands here |
| FEAT-01.SPEC-003 (Household Naming & Guided Setup Start) | Navigation (outbound) | Household name edit entry point |
| FEAT-01.SPEC-004 (Member List & Add Member) | Navigation (outbound) | Members entry point |
| FEAT-01.SPEC-008 (Weekly Budget & Schedule Setup) | Navigation (outbound) | Budget & schedule edit entry point |
| FEAT-01.SPEC-009 (Setup Complete & Next Steps) | Navigation (inbound) | Organiser returning without choosing a next step lands here |
| FEAT-01.SPEC-016 (Household Setup Authorization Rules) | References (inbound) | Role-gated row visibility |
| FEAT-16 (Units, Currency & Locale Configuration) | Navigation (outbound) | Locale settings entry point |
| FEAT-07 (Weekly Plan Ready Notification) | Navigation (outbound) | Plan-ready preference entry point |
| FEAT-07.SPEC-004 (Plan-Arrival Day & Time Setting Rule) | References (inbound) | Governs allowed values, default, and save validation for the inline plan-arrival picker |
| FEAT-13 (Tonight's Dinner Reminder) | Navigation (outbound) | Nightly nudge preference entry point |
| FEAT-09.SPEC-001 (Household Invitations Manager) | Navigation (outbound) | Household Invitations entry point |
| FEAT-09.SPEC-003 (Organiser Hand-Over Initiation) | Navigation (outbound) | Hand-over-role entry point |
| FEAT-09.SPEC-004 (Organiser Hand-Over Acceptance) | Navigation (outbound) | Pending hand-over entry point for the addressed recipient |
| FEAT-09.SPEC-005 (Leave Household) | Navigation (outbound) | Leave-household entry point |
| FEAT-22 (Operator Read-Only Support Access) | Navigation (outbound) | Support access record entry point |
| FEAT-21 (Family Calendar Sync) | Navigation (outbound) | Calendar connection entry point (Later) |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| household_settings_opened | role of viewer | Screen opens | N/A -- no Stage 2 metric tracks settings visits; retained as an operational signal, not a first-session-onboarding step this metric measures |
| household_settings_edit_entry_selected | which row selected | Organiser taps an edit entry point | N/A -- the resulting edit itself is measured by the destination screen's own events (e.g., household_name_edited); this row-selection event is a UI-navigation signal only |
| plan_arrival_updated | new day/time-slot pair | Organiser saves a change in the plan-arrival picker | N/A -- no Stage 2 metric tracks this setting change directly; FEAT-07's own delivery-eligibility metrics measure the resulting plan-ready message, not this edit |

## Acceptance Criteria

**FEAT-01.SPEC-010-AC-01:** Given Maya signs in to an account with an existing household, when the sign-in succeeds, then she lands on this hub showing the household's name, budget, and schedule summaries.

**FEAT-01.SPEC-010-AC-02:** Given Maya taps "Household name", then she is taken to FEAT-01.SPEC-003 in edit mode, pre-filled with the current name.

**FEAT-01.SPEC-010-AC-03:** Given Sam opens this hub, when it loads, then he sees all household facts but no edit chevrons on household name, members, budget & schedule, or plan arrival, and no "Household Invitations" or "Hand over organiser role" rows at all.

**FEAT-01.SPEC-010-AC-04:** Given Sam attempts to navigate directly to the budget edit screen, then he sees "Only the organiser can change this" and is returned to this hub.

**FEAT-01.SPEC-010-AC-05:** Given Sam opens "Notification preferences" from this hub, then he reaches his own preference screen and can edit only his own settings.

**FEAT-01.SPEC-010-AC-06:** Given Maya opens "Support access record", then she is taken to FEAT-22's record showing when and why Riley viewed the household.

**FEAT-01.SPEC-010-AC-07:** Given the household's data fails to load, when this screen opens, then the banner "Couldn't load your household settings. Try again." appears with a retry option.

**FEAT-01.SPEC-010-AC-08:** Given Maya has this hub open on her phone and updates the budget from her laptop, when the laptop save succeeds, then the phone's displayed budget summary updates without a manual refresh.

**FEAT-01.SPEC-010-AC-09:** Given Maya is offline, when she taps "Household name", then she is still taken to FEAT-01.SPEC-003, which itself shows the offline state for the household-name edit.

**FEAT-01.SPEC-010-AC-10:** Given the product is running before FEAT-21 (Family Calendar Sync) has shipped, when this hub loads, then no "Connect calendar" row appears.

**FEAT-01.SPEC-010-AC-11:** Given Maya taps "Plan arrival", selects Wednesday and one of the available Morning slots, and taps "Save", then the household's plan_arrival_day_time updates to Wednesday paired with that slot, the toast "Plan arrival updated" appears, and the row collapses back to the new summary.

**FEAT-01.SPEC-010-AC-12:** Given Maya selects a day but no time-slot in the plan-arrival picker and taps "Save", then the error "Choose both a day and a time for your plan to arrive." appears, the picker stays open, and the household's previous plan_arrival_day_time value remains active.

**FEAT-01.SPEC-010-AC-13:** Given Sam opens this hub, when it loads, then he sees the plan-arrival day/time as a read-only summary with no chevron and no picker control.

**FEAT-01.SPEC-010-AC-14:** Given Maya taps "Household Invitations", then she is taken to FEAT-09.SPEC-001 (Household Invitations Manager).

**FEAT-01.SPEC-010-AC-15:** Given Maya taps "Hand over organiser role", then she is taken to FEAT-09.SPEC-003 (Organiser Hand-Over Initiation).

**FEAT-01.SPEC-010-AC-16:** Given Sam has a hand-over request currently addressed to him, when he opens this hub, then he sees the pending hand-over entry "Maya wants to make you the organiser" and tapping it takes him to FEAT-09.SPEC-004 (Organiser Hand-Over Acceptance).

**FEAT-01.SPEC-010-AC-17:** Given Maya (Organiser) opens this hub, then no "Leave household" row and no pending hand-over entry appear for her, since both actions are exclusive to an Other Adult Member per FEAT-09.SPEC-005 and XBR-15.

**FEAT-01.SPEC-010-AC-18:** Given Sam taps "Leave household", then he is taken to FEAT-09.SPEC-005 (Leave Household).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 15 | 15 |
| States | 4 (loaded, loading, error, offline) | 4 |
| Business Rules | 6 | 6 |
| Edge Cases | 8 | 8 |

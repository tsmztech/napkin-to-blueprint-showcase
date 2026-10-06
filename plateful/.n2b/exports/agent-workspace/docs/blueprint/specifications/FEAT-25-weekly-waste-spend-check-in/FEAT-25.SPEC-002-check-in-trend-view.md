---
document_type: spec
spec_type: screen
spec_id: FEAT-25.SPEC-002
spec_name: Check-In Trend View
spec_slug: check-in-trend-view
parent_feature: FEAT-25
parent_feature_name: Weekly Waste & Spend Check-In
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

# Screen Spec: Check-In Trend View

## Overview

**Name:** Check-In Trend View
**ID:** FEAT-25.SPEC-002
**Type:** Screen
**Purpose:** Household sees its recent check-in answers alongside its starting point and weekly budget, with a path to invite another family.
**Parent Feature:** FEAT-25 -- Weekly Waste & Spend Check-In

## Scope and Non-Goals

**In Scope:**
- Displaying the household's recent Waste & Spend Check-In history (answered and skipped weeks), most recent first
- Displaying the household's change against its one-time starting point and against its weekly budget, using values computed by FEAT-25.SPEC-004
- Showing a skipped week as a plain gap in the history, with no data and no explanation beyond that
- Surfacing the "invite another family" link out to Invite Another Household (FEAT-24)
- Appearing as an extension of the same card surface FEAT-25.SPEC-001 renders, per the Feature Breakdown Brief's Shared Context: "Single evolving card"

**Non-Goals:**
- Answering, correcting, or skipping the current week's question -- owned by FEAT-25.SPEC-001 (Weekly Check-In Card); this view is read-only
- Computing the change-against-starting-point or change-against-budget values themselves -- owned by FEAT-25.SPEC-004 (Check-In Trend Calculation); this view only displays what that spec derives
- Comparing the household's trend to any other household -- excluded per feature-overview.md's Non-Goals: "the trend view shows a household only its own answers against its own starting point and budget, never a comparison to other households"
- Reminding the household about a gap in its own history -- excluded per feature-overview.md's Non-Goals, consistent with the feature's no-follow-up-reminder policy

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-25.SPEC-001 (Weekly Check-In Card) | The card has at least one answered or skipped week, or a starting point, once the household's history exists | Household id; the same current-week record context the card already holds, plus the household's check-in history |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full trend content: history, starting-point comparison, budget comparison, invite link | Tap "invite another family" | -- |
| Sam (Other Adult Member) | Full trend content: same content Maya sees -- the trend is the household's own shared history, not per-member | Tap "invite another family" | -- |
| Jordan (young kid profile, no login -- MVP) | No | No | Trend content is not shown; a no-login profile has no access to any screen in the product |
| Jordan (older kid, limited login -- Later) | No | No | Trend content is not shown in this role's view of the week's plan; the Access Matrix's Waste Check-In column is None for this role |
| Riley (Operator, support -- from v1) | No | No | Trend content is not shown, even while Riley holds an open Support Request's read-only view of the household's plan (FEAT-22); the Access Matrix's Waste Check-In column is None for Riley |
| Unauthenticated | No | No | Redirected to the sign-in screen; no trend content is exposed |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- the trend view carries no unsaved state to preserve, since it is read-only |

## Layout and Content

Trend content appears on the same card surface as FEAT-25.SPEC-001, directly below the current week's question, once the household has a starting point or at least one answered/skipped week. It does not open as a separate screen; it is a continuation region of the same card.

**"Since you started" section:** A short line comparing the most recent answered week against the starting point, in plain terms derived by FEAT-25.SPEC-004 -- e.g., "You started at 'A lot' -- this week you're at 'A little'." for waste, and, when both the current week's spend and the starting point's typical spend exist, a second line such as "You're spending {amount} less than when you started." (or "more," or "about the same," per FEAT-25.SPEC-004's derivation). When the household has a starting point but no answered week yet (only Skipped weeks so far), this section shows: "Answer this week's check-in to see how you're doing against your starting point."

**"This week vs. your budget" section:** When the most recent answered week provided a spend value and the household's weekly_budget is set, a line stating the difference, e.g., "{amount} under your weekly budget" or "{amount} over your weekly budget" or "Right at your weekly budget," per FEAT-25.SPEC-004. When either value is missing, this section shows: "Add a spend amount to compare against your weekly budget." (spend missing) or is omitted entirely (weekly_budget never set at FEAT-01).

**Recent weeks list:** Below the two summary sections, a scrollable list of the household's recent weeks, most recent at the top. Each row shows the week's date range, its waste answer ("None" / "A little" / "A lot"), and its spend if provided. A Skipped week's row shows the week's date range only, with a plain "No answer" marker and no other data -- the feature's own gap-in-the-trend behavior.

**"Invite another family" link:** Sits below the recent weeks list, a single tappable text link: "Know another family who'd want this? Invite them."

### Responsive Behavior

- **Compact breakpoint:** Both summary sections and the recent-weeks list stack full width in the order described above.
- **Medium size class and above:** Layout is unchanged in structure; the trend content is capped at the same platform-wide reading width as the surrounding card and plan content, horizontally centered.
- **Recent weeks list:** Scrolls independently within its own region at every size; it does not cause the surrounding card or plan to scroll horizontally.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Recent weeks list | Scroll | Reveals older weeks in the household's history | List scroll position changes | Standard scroll feedback; no additional data loads beyond what is already synced to the device |
| "Invite another family" link | Tap | Navigate to FEAT-24.SPEC-001 (Invite Another Household Screen) | Screen transitions to the invite flow | Standard navigation transition |

Display-only elements: the "Since you started" section, the "This week vs. your budget" section, and each recent-week row are non-interactive; they present derived and stored values with no action attached.

### Accessibility Notes

- **Focus order:** "Since you started" section -> "This week vs. your budget" section -> recent weeks list (in displayed order, most recent first) -> "Invite another family" link.
- **Content announcements:** When the trend content first becomes available (a starting point or first answer is saved), its appearance is announced to assistive technology as new content, consistent with FEAT-25.SPEC-001's own save-confirmation announcement.
- **Skipped-week rows:** Each Skipped row's "No answer" marker is announced explicitly, distinguishing it from a row with data.
- **Keyboard alternatives:** Scrolling the recent weeks list and activating the invite link are both reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Hidden (Not Yet Available) | Trend content is entirely absent from the card; only FEAT-25.SPEC-001's First-Use variant shows | Household has no starting point and no answered or skipped week yet | A starting point is set or the first week is answered/skipped (FEAT-25.SPEC-001 or FEAT-25.SPEC-003) |
| Showing (Data Available) | Both summary sections (or their "not yet available" placeholder text) and the recent weeks list render as described above | A starting point exists, or at least one week has been answered or skipped | Household navigates away, or a new week's data changes what is shown |
| Loading | A brief inline indicator over the recent weeks list region only; summary sections retain their last-known values if already loaded | Trend content is becoming available for the first time this session, or the household's history is being fetched | Load completes (Showing) or fails (Error) |
| Error | Inline message within the trend region: "Couldn't load your check-in history. Try again." with a retry control | The household's history fails to load | Household taps retry and the load succeeds |
| Offline/Degraded | Previously synced weeks and their already-computed comparison values remain fully viewable; a week not yet synced to this device does not appear until connectivity returns | Connectivity is lost while viewing, or the view is opened while already offline | Connectivity returns and any not-yet-synced weeks appear |

## Validation Rules

Not applicable -- this screen accepts no user input beyond navigation and scrolling; every value shown is read-only, sourced from FEAT-25.SPEC-001's stored answers and FEAT-25.SPEC-004's derivations.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| "Invite another family" tap | FEAT-24.SPEC-001 (Invite Another Household Screen) | FEAT-24 (Invite Another Household) |

## Data Model

**Creates:** None.
**Reads:** Waste & Spend Check-In -- the household's recent records (week, waste_amount, spend, status) and its starting point (starting_point_waste, starting_point_spend), most recent first. Household -- weekly_budget (FEAT-01.SPEC-008) and currency (FEAT-16.SPEC-001), for the budget comparison and spend display. The two derived values (change against starting point, change against weekly budget) computed by FEAT-25.SPEC-004.
**Updates:** None.
**Deletes:** None.

## Business Rules

- The trend view shows a household only its own answers against its own starting point and its own budget -- never a comparison to any other household (feature-overview.md's Non-Goals).
- A Skipped week is shown as a gap: its row carries no waste or spend data, and no change value is computed for it (FEAT-25.SPEC-004).
- Spend figures are displayed in the household's currently configured currency; a later currency change converts existing figures for display, per XBR-11 (owned by FEAT-16).
- Every past check-in record remains visible here for the life of the household account and after a downgrade to the free tier, per scope-boundaries.md SC-18; this view applies no retention window or truncation of history beyond ordinary scrolling.
- If Household's weekly_budget has never been set, the "This week vs. your budget" section is omitted entirely rather than shown with a placeholder, since no comparison is possible without a budget value.

## Edge Cases

- **Household has a starting point but has skipped every week since** -- The "Since you started" section shows its no-answered-week placeholder text; the recent weeks list shows only "No answer" rows.
- **Household's weekly_budget is set after several weeks of check-ins already exist** -- The budget comparison begins appearing for weeks answered from that point forward; weeks answered before the budget was set show no budget-comparison line, since none can be computed retroactively for a value that did not exist at the time.
- **Currency changes between when a week's spend was recorded and when the trend is viewed** -- The displayed spend converts to the newly configured currency before any comparison runs, per XBR-11; the household never sees mismatched-currency figures side by side.
- **Very long check-in history (the household has used Plateful for a long time)** -- The recent weeks list continues to scroll to show older weeks; no week is dropped from the household's own view (scope-boundaries.md SC-18).
- **Household's weekly_budget changes while the trend view is open** -- The budget-comparison line reflects the new figure the next time the view renders (feature-overview.md's Side-Effect Inventory), not live mid-view; no conflict or error is shown, since this view performs no writes of its own.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-25.SPEC-001 (Weekly Check-In Card) | Navigation (inbound) | Trend content appears within the same card surface once a starting point or an answer exists |
| FEAT-25.SPEC-004 (Check-In Trend Calculation) | References (inbound) | Supplies the change-against-starting-point and change-against-budget values this view displays |
| FEAT-25.SPEC-003 (Weekly Check-In Cycle) | References (inbound) | Supplies the finalized (Skipped or locked-Answered) historical records this view reads |
| FEAT-01.SPEC-008 (Weekly Budget & Schedule Setup) | References (inbound) | Supplies the household's weekly_budget for the budget comparison |
| FEAT-16.SPEC-001 (Units, Currency Settings) | References (inbound) | Supplies the household's configured currency for spend display |
| FEAT-24.SPEC-001 (Invite Another Household Screen) | Navigation (outbound) | "Invite another family" link |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| checkin_trend_viewed | weeks_shown (count), has_baseline (bool), has_budget_comparison (bool) | Trend content renders in the Showing state | supports success-metrics.md: "Reported Food Waste and Spend Reduction" |
| checkin_invite_link_tapped | -- | Household taps "Invite another family" | N/A -- this event's referral-growth measure belongs to FEAT-24 (Invite Another Household), not to a metric connected to this feature; recorded here for completeness of this screen's own interaction coverage |

## Acceptance Criteria

**FEAT-25.SPEC-002-AC-01:** Given Maya's household has just set its starting point for the first time, when the card finishes saving, then the trend content appears on the same surface showing the "Since you started" section.

**FEAT-25.SPEC-002-AC-02:** Given Maya's household started at "A lot" and this week's answer is "A little," when she views the trend content, then the "Since you started" section states the waste change from "A lot" to "A little."

**FEAT-25.SPEC-002-AC-03:** Given Sam's household provided a starting typical spend and this week's spend is lower, when he views the trend content, then the spend comparison line states the household is spending less than when it started.

**FEAT-25.SPEC-002-AC-04:** Given Maya's household's most recent answered week provided a spend value and the household's weekly_budget is set, when she views the trend content, then the "This week vs. your budget" section shows the exact difference and whether it is under, over, or at budget.

**FEAT-25.SPEC-002-AC-05:** Given the household's most recent answered week provided no spend value, when Sam views the trend content, then the "This week vs. your budget" section shows its no-spend-provided placeholder text instead of a comparison.

**FEAT-25.SPEC-002-AC-06:** Given the household's weekly_budget has never been set, when Maya views the trend content, then the "This week vs. your budget" section is omitted entirely.

**FEAT-25.SPEC-002-AC-07:** Given a prior week was marked Skipped by FEAT-25.SPEC-003, when Maya scrolls the recent weeks list to that week, then its row shows only the date range and a "No answer" marker, with no waste or spend data.

**FEAT-25.SPEC-002-AC-08:** Given Maya's household has no starting point and no answered or skipped week yet, when she opens the plan, then no trend content appears anywhere on the card.

**FEAT-25.SPEC-002-AC-09:** Given Sam is viewing the trend content, when he taps "Invite another family," then he is navigated to FEAT-24.SPEC-001 (Invite Another Household Screen).

**FEAT-25.SPEC-002-AC-10:** Given Jordan is a young kid profile with no login, then no device or account of Jordan's can reach this trend content.

**FEAT-25.SPEC-002-AC-11:** Given Riley is viewing a household's plan through an open Support Request (FEAT-22), when the plan renders, then no trend content appears anywhere in Riley's view.

**FEAT-25.SPEC-002-AC-12:** Given Maya loses connectivity while viewing the trend content, when she scrolls to a week that already synced to her device before she went offline, then that week's data and comparison values remain fully visible.

**FEAT-25.SPEC-002-AC-13:** Given Maya's household's history fails to load, when the trend region attempts to render, then the inline message "Couldn't load your check-in history. Try again." appears with a retry control.

**FEAT-25.SPEC-002-AC-14:** Given Maya's household's weekly_budget changes in Household Setup, when she next opens the trend content, then the budget-comparison line reflects the new figure.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 2 | 2 |
| States | 5 (hidden, showing, loading, error, offline) | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |

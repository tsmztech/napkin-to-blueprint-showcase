---
document_type: spec
spec_type: screen
spec_id: FEAT-03.SPEC-001
spec_name: Weekly Plan View
spec_slug: weekly-plan-view
parent_feature: FEAT-03
parent_feature_name: AI Weekly Dinner Plan Generation
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 20
---

# Screen Spec: Weekly Plan View

## Overview

**Name:** Weekly Plan View
**ID:** FEAT-03.SPEC-001
**Type:** Screen
**Purpose:** Household views the current week's seven dinners with cost, time, safety badges, and pantry callouts, and the organiser approves the week.
**Parent Feature:** FEAT-03 -- AI Weekly Dinner Plan Generation

## Scope and Non-Goals

**In Scope:**
- Displaying the current week's seven Planned Meals with cook time, rough cost, safety badge, pantry callout, and swap-suggestion review
- The weekly total banner (estimated cost against budget, over-budget note)
- The organiser's one-tap approval action
- Entry points arriving from the plan-ready notification, the nightly nudge, and first-use onboarding
- Cross-feature entry points into swap, recipe detail, safety reporting, the grocery list, leftover lunches, ratings, the weekly check-in, and (Later) dinner voting

**Non-Goals:**
- Building or editing the plan by hand -- owned by Manual Weekly Planning (FEAT-23); this screen is a paid-tier, AI-generated plan's display and approval surface
- Executing a swap, reviewing a suggestion in detail, or picking a safe alternative -- owned by One-Tap Meal Swap (FEAT-04.SPEC-001 Meal Swap (Direct), FEAT-04.SPEC-002 Suggest a Swap, FEAT-04.SPEC-003 Review Swap Suggestions); this screen only opens those flows and reflects their outcome
- Browsing or reusing a past week's plan -- owned by Weekly Plan History (FEAT-19) per the Entity-Lifecycle Coverage Matrix's Read (list) disposition; this screen shows only the current week
- Free-tier display -- free-tier households land on FEAT-03.SPEC-002 (Free-Tier Plan Placeholder & Upgrade Prompt) instead of this screen, per XBR-05

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-07.SPEC-002 (Plan-Ready Notification Message) | Household member taps the "next week's plan is ready" notification | The newly generated week is opened directly |
| FEAT-13.SPEC-002 (Tonight's Dinner Nudge Message) | Household member taps the "Tonight: ..." nudge | Scrolled/highlighted to tonight's dinner within the current week |
| FEAT-15.SPEC-001 (Onboarding Landing) | A newly joined Other Adult Member completes first-use join | Lands in context on the current week's plan |
| Default entry (in-app navigation) | Household member opens the app's plan section | Current week's plan, no special context |
| FEAT-03.SPEC-004 (First-Plan Generation on Upgrade) | Generation completes for a household's first AI plan | The newly generated first plan is opened directly with the "your first plan is ready" framing |
| FEAT-24.SPEC-002 (Referral Welcome Screen) | A visitor whose own household is on the paid tier taps "Go to your plan" | Current week's plan, no special context |
| FEAT-13.SPEC-004 (Same-Day Swap Correction Message) | Household member taps the same-day correction | Scrolled/highlighted to tonight's dinner, showing the new dinner |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen: all seven dinners, weekly total, pending swap suggestions, (Later) vote splits | Approve the week (FEAT-03.SPEC-008); tap swap on any dinner (opens FEAT-04.SPEC-001); open pending suggestions for review (opens FEAT-04.SPEC-003); report a safety concern; open the grocery list; respond to leftover-lunch and check-in cards; rate meals | -- |
| Sam (Other Adult Member) | Full screen: all seven dinners, weekly total, his own pending suggestions | Suggest a swap (opens FEAT-04.SPEC-002, Own-only); report a safety concern (Own-only); mark a leftover lunch eaten or skipped (a status update, not a plan change); respond to check-in and rating prompts; open the grocery list. No Approve control. | Approve control is not shown to Sam; a direct swap (rather than a suggestion) is not offered -- Sam only sees "Suggest a swap," per the Access Matrix's Meal Swap Own-only entitlement |
| Jordan (young kid profile, no login -- MVP) | None -- no login exists for this row | None | No sign-in path exists for this profile; an adult acts on Jordan's behalf (e.g., recording a rating) |
| Jordan (older kid, limited login -- Later) | Full screen View: all seven dinners, weekly total; (Later) vote split for a night when voting is enabled | Respond to (Later) voting only, via FEAT-17; no swap, approval, or safety-report actions | Approve, swap, and "report a safety concern" controls are not shown to this row, per the Access Matrix's None entries for Meal Swap and Safety Reports |
| Riley (Operator, support -- from v1) | Full screen View, read-only, and only while a Support Request for this household is open (FEAT-22, XBR-14) | None -- no actions available | Every control (Approve, Suggest a swap, report a safety concern, rating, check-in, leftover response) is disabled; attempting one shows "Support access is read-only." |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in as a household member, the user lands on this screen (or FEAT-03.SPEC-002 if free tier) |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." No plan data is shown until re-authentication succeeds; no in-progress action (e.g., a swap suggestion draft) survives -- the user re-opens the relevant flow after signing back in |

## Layout and Content

**Header:** Screen title showing the plan's week range (e.g., "This Week"). A "grocery list" shortcut icon (right-aligned) navigates to FEAT-06.SPEC-001 (Grocery List).

**Top banner area:** The weekly check-in card (FEAT-25.SPEC-001, Weekly Check-In Card) appears first when due, above the plan content. Below it, the **weekly total banner**: the week's estimated cost against the household's weekly_budget, in the household's configured currency (per FEAT-16.SPEC-004, Cross-Feature Value Conversion Rule), plus the over-budget note when FEAT-03.SPEC-006 has attached one. If pending swap suggestions exist from Sam, a "Sam suggested N swaps -- review" summary row appears here, above the dinner list, visible to Maya only.

**Body:** A vertically stacked list of seven dinner cards, one per night (Sunday through Saturday, ordered by the household's week start), plus any leftover-lunch cards interleaved on the day they apply. Each **dinner card** (the shared UI pattern used consistently across this screen's states and referenced by FEAT-03.SPEC-002) shows:
- The night's label (e.g., "Monday") and, when tonight falls within the current week, a "Tonight" emphasis marker
- Recipe name
- Cook time and rough cost
- The "checked against allergies" safety badge with the "always check labels" disclaimer, per XBR-01
- A vegetarian-option indicator when the dinner carries one
- A pantry callout line when the dinner uses a logged Pantry Item (e.g., "uses the spinach and feta you already have")
- A "Swap" action
- A vote-split indicator (Later, older-kid voting enabled) below the recipe name
- A pending-suggestion indicator when Sam has an open suggestion for that night, visible to Maya, tapping through to FEAT-04.SPEC-003 (Review Swap Suggestions) to accept or decline it

**Footer:** The Approve action, visible to Maya only, fixed at the bottom of the screen while any dinner remains unapproved for the week; replaced by an "Approved" status indicator once approval is recorded, or an "Active (adopted)" indicator after auto-adoption (FEAT-03.SPEC-005).

### Responsive Behavior

- **Compact breakpoint:** Dinner cards stack full-width, one per row, in the vertical list described above; the Approve footer remains pinned to the bottom of the viewport.
- **Medium size class and above:** The dinner list renders as a fixed-width column capped at a consistent platform-wide reading width and horizontally centered; the weekly total banner and check-in card remain full-width above it. No structural change beyond width capping.
- **Dinner card:** Cook time, cost, and badge remain on one line at every breakpoint; the pantry callout and vote-split line wrap to their own line rather than truncating.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Dinner card | Tap (recipe name/body) | Navigate to FEAT-08.SPEC-002 (Recipe Detail View) | Screen transitions to recipe detail | Standard navigation transition |
| Swap action on a dinner card | Tap (Maya) | Navigate to FEAT-04.SPEC-001 (Meal Swap (Direct)) for that slot | Screen transitions to swap flow | Standard navigation transition |
| Swap action on a dinner card | Tap (Sam) | Navigate to FEAT-04.SPEC-002 (Suggest a Swap) for that slot | Screen transitions to suggest flow | Standard navigation transition |
| Pending-suggestion indicator | Tap (Maya) | Navigate to FEAT-04.SPEC-003 (Review Swap Suggestions) for that night | Screen transitions to the review flow | Standard navigation transition |
| "Report a safety concern" control on a dinner card | Tap (Maya, Sam) | Hand off to FEAT-02.SPEC-001 (Report a Safety Concern) for that meal | Screen transitions to the safety-report flow | Standard navigation transition |
| Leftover-lunch card | Tap "Confirm" or "Skip" | Records the leftover-lunch response via FEAT-11.SPEC-001 (Leftover Lunch Card) | Card shows the recorded response | Toast: "Marked as [eaten/skipped]." |
| Rating prompt (on a cooked meal) | Tap thumbs up/down | Records the rating via FEAT-12.SPEC-001 (Post-Dinner Rating Prompt) | Prompt clears from that dinner card | Brief confirmation animation |
| Weekly check-in card | Answer the prompt | Records the answer via FEAT-25.SPEC-001 (Weekly Check-In Card) | Card clears or shows the recorded trend | Toast: "Thanks -- your answer is saved." |
| Vote-split indicator (Later) | Tap | Navigate to FEAT-17.SPEC-003 (Vote Outcome & Resolution) | Screen transitions to vote detail | Standard navigation transition |
| Grocery list shortcut icon | Tap | Navigate to FEAT-06.SPEC-001 (Grocery List) | Screen transitions to the grocery list | Standard navigation transition |
| Approve action (footer, Maya only) | Tap | Trigger approval via FEAT-03.SPEC-008 | Footer changes to "Approved" indicator; loading state shown briefly during approval | Toast: "This week's plan is approved." Success sets Weekly Plan status to Approved. |
| Approve action | Tap, while already approved this week | No action -- control is replaced by the Approved indicator, so a second approval cannot be initiated | None | -- |
| Retry control (Error state) | Tap | Re-request the current week's plan data | Screen re-attempts load | Loading indicator shown during retry |

### Accessibility Notes

- **Focus order:** Header title -> grocery list shortcut -> check-in card (when present) -> weekly total banner -> pending-suggestions summary (Maya, when present) -> dinner cards in night order (each card: recipe name -> badge -> swap/suggestion controls -> report-concern control) -> Approve footer action.
- **Dynamic announcements:** A successful approval, rating, leftover response, or check-in answer is announced to assistive technology via the toast text at the moment it appears. An over-budget note appearing after generation is announced when the banner first renders. A live update to a dinner card from an accepted swap (decided on FEAT-04.SPEC-003) is announced when it arrives via FEAT-03.SPEC-011.
- **Safety badge:** The "checked against allergies" badge and "always check labels" disclaimer are conveyed as text, never by color or icon alone, per ASMP-29.
- **Keyboard alternatives:** Every action on this screen (swap, accept/decline, report, rate, respond to check-in/leftover, approve) is reachable by keyboard focus and activation; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (first plan pending) | "Your first plan is on its way" message in place of the dinner list, with an explanation of the typical wait | A paid household has not yet had a first plan generated (FEAT-03.SPEC-004 has not yet completed) | First-plan generation completes and the screen re-loads with the plan |
| Loading (generation in progress) | A short, explained wait message (e.g., "Building this week's plan...") replaces the dinner list; no indefinite spinner | Scheduled or first-plan generation (SPEC-003, SPEC-004) is in progress for this household | Generation completes (success or failure) |
| Populated | Full dinner list, weekly total banner, and Approve footer as described in Layout and Content | Plan data has loaded successfully | User navigates away |
| Error (generation failed) | The previous week's plan remains visible in full, with a banner: "This week's plan couldn't be generated. Retry?" and a Retry control | Generation fails (SPEC-003 outcome: Automation failure) | User taps Retry and generation succeeds, or the next scheduled attempt succeeds |
| Offline/Degraded | The most recently loaded plan remains fully viewable, including its badges, callouts, and total; Approve, Swap, and suggestion Accept/Decline controls are disabled with the inline note "Reconnect to approve or change the plan." Rating and leftover responses queue locally and sync on reconnect. | Connectivity is lost while this screen is open or opened | Connectivity restored -- controls re-enable and any queued responses sync automatically |

## Validation Rules

Validation of who may approve the week, when, and how often is governed by FEAT-03.SPEC-008 (Plan Approval Authorization Rule). See that spec for the full authorization logic; this screen only surfaces the resulting Approve control or Approved/Active indicator.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Dinner card tap | FEAT-08.SPEC-002 (Recipe Detail View) | FEAT-08 (Recipe Library) |
| Swap action (Maya) | FEAT-04.SPEC-001 (Meal Swap (Direct)) | FEAT-04 (One-Tap Meal Swap) |
| Swap action (Sam) | FEAT-04.SPEC-002 (Suggest a Swap) | FEAT-04 (One-Tap Meal Swap) |
| Pending-suggestion indicator tap (Maya) | FEAT-04.SPEC-003 (Review Swap Suggestions) | FEAT-04 (One-Tap Meal Swap) |
| "Report a safety concern" | FEAT-02.SPEC-001 (Report a Safety Concern) | FEAT-02 (Dietary Rules & Allergy Safety Engine) |
| Grocery list shortcut | FEAT-06.SPEC-001 (Grocery List) | FEAT-06 (Shared Grocery List) |
| Vote-split indicator (Later) | FEAT-17.SPEC-003 (Vote Outcome & Resolution) | FEAT-17 (Older-Kid Dinner Voting) |
| Check-in card "invite another family" follow-up | Invite Another Household | FEAT-24 (Invite Another Household) |

## Data Model

**Creates:** None -- this screen displays and approves an existing Weekly Plan; it does not create one.
**Reads:** Weekly Plan -- week, origin, status, approval, estimated_total, over_budget_note. Planned Meal (seven per plan, plus leftover-lunch entries) -- night, meal_kind, recipe, safety_badge, vegetarian_option, cook_time, rough_cost, pantry_callout, status, swap_history. Household -- weekly_budget, currency (for the total banner), member count context. Swap Suggestion -- suggesting_member, night, proposed_recipe, outcome (for the pending-suggestion indicator).
**Updates:** Weekly Plan -- approval and status, set by the Approve action via FEAT-03.SPEC-008. Planned Meal status may update indirectly through actions handed off to FEAT-04.SPEC-004 (Apply Meal Swap), FEAT-02.SPEC-004 (Safety Concern Intake & Removal), and FEAT-11.SPEC-001/FEAT-12.SPEC-001 -- this screen surfaces those outcomes rather than writing the fields itself.
**Deletes:** None.

## Business Rules

- XBR-01: Every meal shown on this screen has already passed the app-enforced allergy and religious-rule check; the "checked against allergies" badge and "always check labels" disclaimer are shown on every dinner card without exception.
- XBR-03: The grocery list shortcut always reflects the current plan -- no meal on this screen is ever shown without its ingredients already present in the linked Grocery List.
- XBR-06: Sam's swap action opens a suggestion flow (FEAT-04.SPEC-002), never a direct swap; only Maya's accept/decline decision on FEAT-04.SPEC-003 (Review Swap Suggestions) changes the plan from a suggestion.
- XBR-07: Only Maya may tap Approve, once per week; if the week reaches its start with no approval recorded, FEAT-03.SPEC-005 adopts the plan automatically and this screen shows "Active (adopted)" instead of "Approved."
- The weekly total banner and any over-budget note are computed by FEAT-03.SPEC-006 (Budget Fit & Estimated Total Rule) and displayed here verbatim -- this screen performs no independent cost calculation.
- Ingredient quantities behind the displayed cook time/cost figures are sized to the household by FEAT-03.SPEC-007 (Household-Scaled Quantity Rule).
- Approval authorization (who, how often, what happens on a second attempt) is governed by FEAT-03.SPEC-008.

## Edge Cases

- **User navigates here before any plan has ever been generated** -- Empty state shows "Your first plan is on its way" rather than a blank week or an error.
- **User taps Swap and Approve in rapid succession** -- Approve is disabled while any swap or suggestion action for this week is in flight, to avoid approving a plan mid-change; it re-enables once the action resolves.
- **Another household member changes the plan while this screen is open (accepted suggestion, safety removal, auto-adoption)** -- The screen live-updates via FEAT-03.SPEC-011 (Real-Time Plan Sync Integration); the affected dinner card updates in place with a brief highlight, and no reload is required. Resolution follows the dependency map's Contention note for Weekly Plan: a safety removal always wins over any concurrent change, and slot changes are reject-with-refresh with at most one active swap per slot -- a second, conflicting swap attempt on the same slot is rejected with "This dinner just changed -- here's the latest." and the card refreshes to the current recipe.
- **Maya taps Approve while a safety removal is mid-flight on some dinner** -- Approve is rejected with "One of this week's dinners just changed for safety reasons -- review it before approving." and the affected card is highlighted; Maya retries once the removal has resolved.
- **A dinner card has no pantry callout and no vote split** -- Those lines are simply omitted from the card; no placeholder text appears.
- **Household loses connectivity mid-approval** -- The approval attempt fails gracefully with the Offline/Degraded messaging; approval is not recorded until connectivity returns and the action is retried.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-03.SPEC-003 (Scheduled Weekly Plan Generation) | Triggered by (inbound) | Generation completion populates this screen with the new week |
| FEAT-03.SPEC-004 (First-Plan Generation on Upgrade) | Triggered by (inbound) | First-plan completion populates this screen from the Empty state |
| FEAT-03.SPEC-005 (Auto-Adoption at Week Start) | References (inbound) | Adoption sets the "Active (adopted)" status shown here in place of Approved |
| FEAT-03.SPEC-006 (Budget Fit & Estimated Total Rule) | References (inbound) | Supplies the weekly total banner and any over-budget note |
| FEAT-03.SPEC-007 (Household-Scaled Quantity Rule) | References (inbound) | Supplies the household-sized cook time/cost figures behind each dinner card |
| FEAT-03.SPEC-008 (Plan Approval Authorization Rule) | Triggers (outbound) | Approve action invokes this rule |
| FEAT-03.SPEC-011 (Real-Time Plan Sync Integration) | References (inbound) | Live-updates this screen when the plan changes from any source |
| FEAT-07.SPEC-002 (Plan-Ready Notification Message) | Navigation (inbound) | Notification tap opens this screen |
| FEAT-07.SPEC-001 (Plan-Ready Notification Trigger) | Triggered by (inbound) | Fires when this screen's generation source (FEAT-03.SPEC-003/004) completes |
| FEAT-13.SPEC-002 (Tonight's Dinner Nudge Message) | Navigation (inbound) | Nudge tap opens this screen scrolled to tonight |
| FEAT-15.SPEC-001 (Onboarding Landing) | Navigation (inbound) | First-use landing for a newly joined Other Adult Member opens this screen |
| FEAT-04.SPEC-001 (Meal Swap (Direct)) | Navigation (outbound) | Maya's swap action opens the direct-swap flow |
| FEAT-04.SPEC-002 (Suggest a Swap) | Navigation (outbound) | Sam's swap action opens the suggest-a-swap flow |
| FEAT-04.SPEC-003 (Review Swap Suggestions) | Navigation (outbound) | Maya's pending-suggestion indicator opens the review flow |
| FEAT-04.SPEC-004 (Apply Meal Swap) | References (inbound) | Underlies both the direct swap and an accepted suggestion; its outcome is what FEAT-03.SPEC-011 propagates back to this screen |
| FEAT-02.SPEC-001 (Report a Safety Concern) | Navigation (outbound) | "Report a safety concern" opens this flow |
| FEAT-06.SPEC-001 (Grocery List) | Navigation (outbound) | Grocery list shortcut opens this screen |
| FEAT-08.SPEC-002 (Recipe Detail View) | Navigation (outbound) | Dinner card tap opens recipe detail |
| FEAT-11.SPEC-001 (Leftover Lunch Card) | References (outbound) | Leftover-lunch card responses recorded here; the card pattern is embedded within this screen |
| FEAT-12.SPEC-001 (Post-Dinner Rating Prompt) | References (outbound) | Rating prompts recorded here |
| FEAT-17.SPEC-003 (Vote Outcome & Resolution, Later) | Navigation (outbound) | Vote-split indicator opens vote review |
| FEAT-25.SPEC-001 (Weekly Check-In Card) | References (outbound) | Check-in card recorded here |
| FEAT-16.SPEC-004 (Cross-Feature Value Conversion Rule) | References (inbound) | Cost figures display in the household's configured unit system and currency |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| plan_viewed | entry source (notification / nudge / onboarding / default nav), origin (AI-generated) | Screen loads with a populated plan | supports success-metrics.md: "Weekly Planning Time" |
| plan_approved | time from open to approve, suggestion count reviewed | Maya taps Approve and it succeeds | supports success-metrics.md: "Weekly Planning Time" |
| weeknight_dinner_time_fit_viewed | night, whether the household marked that night time-constrained, dinner's cook time | A dinner card for a schedule-constrained night renders | supports success-metrics.md: "Weeknight Time-Fit Accuracy" |
| plan_over_budget_shown | estimated overrun amount | The over-budget note renders in the weekly total banner | N/A -- no Stage 2 metric measures over-budget display frequency; retained to observe how often the budget-fit rule's fallback surfaces to users |

## Acceptance Criteria

**FEAT-03.SPEC-001-AC-01:** Given Maya opens the app from the "next week's plan is ready" notification, when the screen loads, then she sees all seven dinners with cook time, rough cost, safety badge, and the weekly total banner.

**FEAT-03.SPEC-001-AC-02:** Given Maya is on the Weekly Plan View with an unapproved week, when she taps Approve, then the plan's status changes to Approved, the footer shows an "Approved" indicator, and a toast confirms "This week's plan is approved."

**FEAT-03.SPEC-001-AC-03:** Given Sam is on the Weekly Plan View, when he looks for an Approve control, then none is shown to him.

**FEAT-03.SPEC-001-AC-04:** Given Maya taps a dinner card's recipe name, when the tap registers, then the screen navigates to that recipe's detail view (FEAT-08.SPEC-002).

**FEAT-03.SPEC-001-AC-05:** Given Sam taps Swap on Friday's dinner, when he picks a safe alternative on FEAT-04.SPEC-002 (Suggest a Swap), then Maya sees a pending-suggestion indicator on Friday's card on this screen.

**FEAT-03.SPEC-001-AC-06:** Given Maya sees Sam's pending suggestion on Friday, when she taps the indicator, then the screen navigates to FEAT-04.SPEC-003 (Review Swap Suggestions), and once she accepts it there, Friday's card on this screen updates to the suggested recipe via FEAT-03.SPEC-011.

**FEAT-03.SPEC-001-AC-07:** Given Maya declines Sam's pending suggestion on FEAT-04.SPEC-003, when the decline is recorded, then this screen's pending-suggestion indicator for that night clears and the original recipe remains.

**FEAT-03.SPEC-001-AC-08:** Given Maya or Sam taps "Report a safety concern" on a dinner card, when the tap registers, then the screen hands off to FEAT-02's safety-report flow for that meal.

**FEAT-03.SPEC-001-AC-09:** Given a paid household has never had a plan generated, when a member opens this screen, then it shows "Your first plan is on its way" instead of a blank week.

**FEAT-03.SPEC-001-AC-10:** Given generation is in progress for the household, when a member opens this screen, then a short explained-wait message appears in place of the dinner list.

**FEAT-03.SPEC-001-AC-11:** Given generation fails for the week, when a member opens this screen, then the previous week's plan remains fully visible with a banner offering Retry.

**FEAT-03.SPEC-001-AC-12:** Given Maya loses connectivity while viewing an already-loaded plan, when she attempts to tap Approve, then the control is disabled with the note "Reconnect to approve or change the plan," and the plan itself remains fully viewable.

**FEAT-03.SPEC-001-AC-13:** Given a dinner uses a logged pantry item, when the plan renders, then that dinner's card shows the pantry callout naming the item.

**FEAT-03.SPEC-001-AC-14:** Given the household's estimated total exceeds its weekly_budget for the closest-fitting safe plan, when the screen renders, then the weekly total banner shows the over-budget note computed by FEAT-03.SPEC-006.

**FEAT-03.SPEC-001-AC-15:** Given the week has not been approved by the start of the week, when the week begins, then FEAT-03.SPEC-005 auto-adopts the plan and this screen shows "Active (adopted)" in place of the Approve control.

**FEAT-03.SPEC-001-AC-16:** Given another household member accepts a swap suggestion while Maya has this screen open, when the change is saved, then Maya's screen updates the affected dinner card in place via FEAT-03.SPEC-011, with no manual reload.

**FEAT-03.SPEC-001-AC-17:** Given a safety concern removes a dinner from the plan while Maya is attempting to approve, when she taps Approve, then the approval is rejected with "One of this week's dinners just changed for safety reasons -- review it before approving," and the affected card is highlighted.

**FEAT-03.SPEC-001-AC-18:** Given Riley (Operator) opens this screen against an open Support Request for the household, when the screen loads, then it is fully read-only and every action control (Approve, Swap, report, rate, respond) is disabled with "Support access is read-only."

**FEAT-03.SPEC-001-AC-19:** Given the older-kid login (Later) opens this screen with dinner voting enabled for a night, when the screen renders, then that night's card shows the vote-split indicator and no swap or approval control.

**FEAT-03.SPEC-001-AC-20:** Given Maya's session expires while this screen is open with a pending suggestion showing, when she attempts to tap the pending-suggestion indicator, then a dialog reads "Your session has expired. Sign in to continue." and the pending suggestion is still shown unresolved after she signs back in.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 13 | 13 |
| States | 5 (empty, loading, populated, error, offline) | 5 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |

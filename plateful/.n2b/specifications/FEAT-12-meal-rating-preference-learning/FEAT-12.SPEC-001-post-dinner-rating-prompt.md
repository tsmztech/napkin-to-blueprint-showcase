---
document_type: spec
spec_type: screen
spec_id: FEAT-12.SPEC-001
spec_name: Post-Dinner Rating Prompt
spec_slug: post-dinner-rating-prompt
parent_feature: FEAT-12
parent_feature_name: Meal Rating & Preference Learning
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Screen Spec: Post-Dinner Rating Prompt

## Overview

**Name:** Post-Dinner Rating Prompt
**ID:** FEAT-12.SPEC-001
**Type:** Screen
**Purpose:** A household member gives a thumbs up or thumbs down on a cooked dinner, and can pass their phone to record each young kid profile's rating on that kid's behalf, in one repeating control.
**Parent Feature:** FEAT-12 -- Meal Rating & Preference Learning

## Scope and Non-Goals

**In Scope:**
- Capturing a household member's own thumbs up/down rating for a cooked Planned Meal
- The phone-passed proxy sequence: after the signed-in adult rates, they can step through each young kid profile in the household and record that kid's thumbs up/down on the same control
- Showing the screen's own current rating state (already rated, and what value) before allowing a change
- Letting the signed-in adult change their own already-submitted rating, or a proxy rating they recorded, within this same screen

**Non-Goals:**
- An independent login for a kid to rate directly -- excluded per scope-boundaries.md (SC-02): v1's default is parent-managed profiles with no login for young kids, so every young-kid rating in v1 is recorded by an adult on the kid's behalf; a distinct kid-rating login is the Later-phase FEAT-17.
- Displaying another household member's individual rating, broken out, to anyone (including Maya) -- excluded per the feature's own Data Notes and Access fields: only the aggregate effect on future plans is ever visible, never a per-member rating list.
- Deriving or showing how ratings influence future plan weighting -- governed by FEAT-12.SPEC-004 (Preference Weighting & Tier-Gating Rule); this screen only captures the rating.
- Deleting a rating outright, leaving the meal unrated again -- excluded per the feature's own Validation & Limits field: only a change to the opposite value is supported, not removal.
- Reminding or nudging the household to go back and rate an unrated meal -- excluded per the feature's own States field: rating is opt-in and self-initiated, and the household is never pressed to rate.
- The uniqueness, change-window, and proxy-counting rules themselves -- governed by FEAT-12.SPEC-002; this screen enforces them but does not restate the logic.
- Who may rate for whom -- governed by FEAT-12.SPEC-003; this screen enforces it but does not restate the logic.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-03 (AI Weekly Dinner Plan Generation), week plan | Household member taps "Rate after dinner" on a cooked meal | The Planned Meal reference (which dinner, which night) and the signed-in member's identity |
| FEAT-23 (Manual Weekly Planning), week plan | Household member taps "Rate after dinner" on a cooked meal from a manually planned week | Same as above -- a rating on a manually planned meal is captured identically |

There is no default entry and no top-level navigation destination for this screen (feature-overview.md, Internal Dependency Map): it is reached only from a cooked meal in either week-plan context.

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | Rate herself; record a proxy rating for any young kid profile in the household | -- |
| Sam (Other Adult Member) | Full screen, for his own rating and any young kid profile's proxy rating | Rate himself (own rating only, per FEAT-12.SPEC-003 Own-only); record a proxy rating for any young kid profile | Attempting to view or change another adult's individual rating is not possible from this screen -- there is no control for it (privacy rule, FEAT-12.SPEC-003) |
| Jordan (young kid profile, no login -- MVP) | Not applicable -- no login exists, so this profile never opens this screen directly; its rating is entered by an adult through the proxy sequence | Not applicable | Not applicable -- there is no sign-in path for this profile |
| Jordan (older kid, limited login -- Later) | Full screen, for his own rating only | Rate himself (Own-only, per the Access Matrix) | Attempting to record a rating for anyone else is not possible -- the proxy sequence is not offered to this role |
| Riley (Operator, support) | Not applicable to this screen | Not applicable | Riley's Ratings: View access (user-persona.md Access Matrix) is exercised through FEAT-22 (Operator Read-Only Support Access), which surfaces rating activity in aggregate for diagnosis; Riley never opens this per-meal household screen |
| Unauthenticated | No | No | Redirected to the household sign-in screen; after signing in, the user lands on the current week's plan (FEAT-03 or FEAT-23), not directly back on this prompt -- they must tap "Rate after dinner" again |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- any rating already captured before expiry (including proxy ratings recorded earlier in the sequence) was already submitted and is not lost; only the not-yet-submitted step in progress is discarded |

## Layout and Content

**Header:** The Planned Meal's recipe name and night (e.g., "Tuesday: Lemon Chicken Traybake") with a close control (top-left) that returns to the week plan. No "Save" action -- every tap on a thumbs control submits immediately.

**Body, own-rating step:** A single large thumbs-up control and a single large thumbs-down control, side by side, centered. Below them, when this member has already rated this meal, the previously chosen control is shown in its selected visual state, and the text "You rated this: {up/down}. Tap to change." appears beneath the pair. When no rating exists yet, no such text appears.

**Body, proxy step (each young kid profile in turn):** The same thumbs-up/thumbs-down pair, with the header line above it replaced by "Rating for {kid display name}" and, on first entry to this step, a one-line instruction "Hand the phone to {kid display name}." When a young kid profile already has a rating recorded for this meal, the previously chosen control is shown selected with "Rated: {up/down}. Tap to change." beneath it, matching the own-rating step's pattern.

**Footer:** A row of small step indicators (one dot per household member being rated: the signed-in adult, then each young kid profile) showing progress through the sequence. A "Next" control advances to the next profile once the current step has a rating (or is skipped); "Done" replaces "Next" on the final step and returns to the week plan (FEAT-03 or FEAT-23, whichever supplied the meal).

This is one repeating control across profiles, not separate screens per profile, per the Feature Breakdown Brief's Shared UI Patterns.

### Responsive Behavior

- **Compact breakpoint:** Thumbs-up/thumbs-down pair centered, full width, large tap targets stacked with generous spacing for one-handed use at the table. Step indicators and Next/Done sit in the footer as described above.
- **Medium size class and above:** Same structure, centered content capped at a consistent platform-wide content width (exact value is the design layer's decision); no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Close control | Tap | Navigate back to the source week plan (FEAT-03 or FEAT-23) without requiring a rating | Screen closes | Any ratings already submitted in this session are kept; unrated steps remain unrated |
| Thumbs-up control (own step) | Tap | Submits a thumbs-up rating for the signed-in member and this meal, per FEAT-12.SPEC-002 (submission/change rules) and FEAT-12.SPEC-003 (authorization) | Control shows selected state instantly | Instant confirmation; no loading spinner (Stage 2 Loading: submitting a rating confirms instantly) |
| Thumbs-down control (own step) | Tap | Submits a thumbs-down rating for the signed-in member and this meal, per FEAT-12.SPEC-002 and FEAT-12.SPEC-003 | Control shows selected state instantly | Instant confirmation |
| Thumbs-up or thumbs-down control (own step, already rated) | Tap the opposite control | Changes the existing rating to the new value, per FEAT-12.SPEC-002's changeable-until-archived window | Previously selected control deselects; newly tapped control selects | Instant confirmation; "Tap to change" text updates to reflect the new value |
| Thumbs-up or thumbs-down control (proxy step) | Tap | Records a proxy rating for the current young kid profile, tagged with the recording adult, per FEAT-12.SPEC-002 (proxy-counts-as-the-kid's-rating) and FEAT-12.SPEC-003 (who may proxy) | Control shows selected state for that profile | Instant confirmation |
| Next control | Tap | Advances to the next profile in the sequence (or to Done on the final profile) | Body content switches to the next profile's step | Step indicator advances; instruction line updates to the next profile's name |
| Next control (a profile's step has no rating yet) | Tap | Skips this profile without recording a rating -- the meal remains neutral for that profile (feature-overview.md, unrated-is-neutral) | Body content switches to the next profile's step | Step indicator advances; no confirmation needed since nothing was recorded |
| Done control (final step) | Tap | Returns to the source week plan (FEAT-03 or FEAT-23) | Screen closes | No further confirmation -- ratings were already confirmed individually as each was tapped |

### Accessibility Notes

- **Focus order:** Close control -> current step's instruction/heading text -> thumbs-up control -> thumbs-down control -> Next/Done control.
- **Selection announcements:** When a thumbs control is tapped, the resulting selected/confirmed state is announced to assistive technology (e.g., "Rated thumbs up" or, on the proxy step, "Rated thumbs up for {kid name}").
- **Step change announcements:** Advancing to a new profile's step announces the new heading ("Rating for {kid name}") so a screen-reader user knows whose rating they are now recording.
- **Keyboard alternatives:** Every control on this screen (thumbs-up, thumbs-down, Next, Done, Close) is reachable and operable by keyboard; there are no pointer-only gestures (no swipe-to-advance).

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Not Yet Rated (default, own step) | Both thumbs controls unselected, no "Tap to change" text | Screen opens for a meal/member pair with no existing rating | Member taps a thumbs control |
| Already Rated | The previously chosen control shown selected, "Tap to change" text visible | Screen opens (or a proxy step loads) for a meal/member pair that already has a rating | Member taps the opposite control to change it, or advances/closes without changing it |
| Submitting | Selected control briefly shows a confirmed visual state | A thumbs control is tapped | Confirmation completes (near-instant; no visible loading state per Stage 2 Loading) |
| Error | A small inline notice near the tapped control: "Couldn't save your rating -- we'll keep trying." The selected state is shown optimistically while retry continues in the background | A rating submission fails | Background retry succeeds (notice clears silently) or the member navigates away (retry continues; see Edge Cases) |
| Offline/Degraded | The tapped control shows its selected state immediately; a small banner reads "You're offline -- this rating will save when you reconnect." | Connectivity is lost while this screen is open, or a rating is submitted while already offline | Connectivity restored -- the queued rating syncs automatically using the real-time synchronization boundary this feature relies on (FEAT-03.SPEC-011 / FEAT-06.SPEC-005) |
| Proxy Sequence Mid-Flow | Step indicator shows progress (e.g., dot 2 of 4 highlighted); instruction line names the current profile | Adult taps Next past the own-rating step | Adult reaches the final profile and taps Done, or closes the screen early |

## Validation Rules

Validation governed by FEAT-12.SPEC-002 (Rating Submission, Change & Proxy Rules) -- one rating per member per meal, the changeable-until-archived window, and the proxy-counts-as-the-kid's-rating rule. Authorization -- who may submit a rating for themselves or as a proxy -- governed by FEAT-12.SPEC-003 (Rating Access & Authorization Rules). This screen applies both on every tap of a thumbs control; there is no separate submit step to defer validation to.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Close control tap (any step) | Week plan screen the meal was opened from | FEAT-03 (AI Weekly Dinner Plan Generation) or FEAT-23 (Manual Weekly Planning), matching the entry context |
| Done control tap (final step) | Week plan screen the meal was opened from | FEAT-03 or FEAT-23, matching the entry context |

## Data Model

**Creates:** Rating -- one record per member (or young kid profile) per Planned Meal on first submission. Fields set: member (the signed-in adult, or the young kid profile being proxy-rated), planned_meal (the meal this screen was opened for), value (up or down), recorded_by (set only for a proxy rating -- the adult who tapped the control on the kid's behalf; absent for a member's own rating).
**Reads:** Rating -- this screen loads any existing rating for the current meal/member (or meal/kid-profile) pair to show the Already Rated state before allowing a change. Planned Meal -- recipe name and night, for the header. Member Profile -- the household's young kid profiles, to build the proxy step sequence, and the signed-in adult's identity.
**Updates:** Rating -- the value field, when the member (or the recording adult, for a proxy rating) changes a previously submitted rating, per FEAT-12.SPEC-002's changeable-until-archived window.
**Deletes:** None -- this screen never removes a Rating; deletion or anonymization on membership change is owned by FEAT-18 and FEAT-09 (feature-overview.md, Entity-Lifecycle Coverage Matrix).

## Business Rules

- One rating per household member (including each young kid profile, by proxy) per Planned Meal -- governed by FEAT-12.SPEC-002.
- A rating can be changed up until the plan holding that meal is archived -- governed by FEAT-12.SPEC-002; after archival this screen's thumbs controls no longer accept a change (see Edge Cases).
- Who may submit an own rating or a proxy rating on this screen -- governed by FEAT-12.SPEC-003.
- A rating given while offline is held locally and synced once connectivity returns, using the real-time synchronization boundary FEAT-03.SPEC-011 / FEAT-06.SPEC-005 provides; this feature invents no separate synchronization mechanism of its own (feature-overview.md, Side-Effect Inventory).
- A meal with no rating for a given member is treated as neutral, never assumed liked or disliked, and this screen never prompts or presses the household to return and rate it.
- A repeated down-rating pattern for the same member and meal is evaluated by FEAT-12.SPEC-005 (Repeated-Dislike Learned Update) once a qualifying rating is submitted here; this screen does not itself evaluate or display that pattern.

## Edge Cases

- **Household member taps a thumbs control twice rapidly (double-submit)** -- The second tap on the same control while the first is still confirming is ignored; a tap on the opposite control while the first is still confirming is queued and applied once the first submission settles, resulting in the second value being the one saved.
- **Two adults record the same young kid profile's rating for the same meal at effectively the same time (e.g., Maya on her phone and Sam on his, both proxy-rating Jordan)** -- Concurrent-edit conflict: per the dependency map's Contention note for Rating, resolution is last-write-wins -- since one rating per member per meal is kept and ratings stay changeable, the later-arriving submission is the value that persists; neither adult sees an error, and either can change it again afterward.
- **Adult attempts to change a rating after the plan holding that meal has been archived** -- The thumbs controls for that meal become read-only, showing the last recorded value with no "Tap to change" text; per FEAT-12.SPEC-002, the change window has closed.
- **Adult closes the screen mid-proxy-sequence** -- Every rating already tapped (own and any completed proxy steps) was already submitted individually and is kept; unrated remaining profiles stay neutral (unrated) and are not revisited automatically.
- **Household has no young kid profiles** -- The proxy sequence is skipped entirely; the own-rating step's Next control reads "Done" directly, since there is nothing to step through.
- **A rating submission fails and the household member navigates away before it retries successfully** -- The background retry continues independent of this screen; if it exhausts its retries, the rating is left unsubmitted and the meal remains neutral for that member until they reopen this screen and rate again (Stage 2 Error: no error is surfaced unless retries exhaust, and no retroactive prompt is created per this feature's no-reminders rule).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-12.SPEC-002 (Rating Submission, Change & Proxy Rules) | References (outbound) | This screen enforces the one-rating-per-member-per-meal, change-window, and proxy-counting rules on every submission |
| FEAT-12.SPEC-003 (Rating Access & Authorization Rules) | References (outbound) | This screen enforces who may rate for themselves or as a proxy, and the never-shown-individually privacy rule |
| FEAT-12.SPEC-004 (Preference Weighting & Tier-Gating Rule) | Affects (outbound) | Ratings this screen captures are the input the weighting rule reads; this screen carries no weighting logic or tier-gating display of its own |
| FEAT-03 (AI Weekly Dinner Plan Generation) | Navigation (inbound/outbound) | Entry from the week plan's "Rate after dinner" trigger on a cooked meal; Close/Done return there |
| FEAT-23 (Manual Weekly Planning) | Navigation (inbound/outbound) | Same entry and return pattern for a manually planned week's cooked meal |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| meal_rated_up | member type (adult / young-kid-by-proxy), planned_meal reference, whether this was a first-time rating or a change | A thumbs-up control is tapped and the rating is submitted (new or changed) | N/A -- no success-metrics.md metric names Meal Rating & Preference Learning as its Connected Feature; this event is defined per product-features.md's Signals list for FEAT-12 and is available for a future metric to draw on |
| meal_rated_down | member type (adult / young-kid-by-proxy), planned_meal reference, whether this was a first-time rating or a change | A thumbs-down control is tapped and the rating is submitted (new or changed) | N/A -- same reason as meal_rated_up |
| rating_changed | member type, planned_meal reference, previous value, new value | A member changes a previously submitted rating to the opposite value before archival | N/A -- same reason as meal_rated_up |
| kid_rating_recorded_by_adult | recording adult, young kid profile rated, planned_meal reference, value | An adult submits a rating during the proxy sequence for a young kid profile | N/A -- same reason as meal_rated_up |

## Acceptance Criteria

**FEAT-12.SPEC-001-AC-01:** Given Maya is on the Post-Dinner Rating Prompt for tonight's cooked meal with no existing rating, when she taps the thumbs-up control, then the control shows its selected state instantly and the rating is recorded against Maya and that meal.

**FEAT-12.SPEC-001-AC-02:** Given Sam has already rated tonight's meal thumbs-down, when he reopens the prompt for that meal, then the thumbs-down control shows selected with the text "You rated this: down. Tap to change."

**FEAT-12.SPEC-001-AC-03:** Given Sam already rated tonight's meal thumbs-down, when he taps the thumbs-up control, then his rating changes to thumbs-up and the text updates to "You rated this: up. Tap to change."

**FEAT-12.SPEC-001-AC-04:** Given Maya is on the own-rating step and the household has two young kid profiles, when she taps Next after rating herself, then the screen advances to the proxy step showing "Rating for {first kid's name}" with the instruction "Hand the phone to {first kid's name}."

**FEAT-12.SPEC-001-AC-05:** Given Maya is on a proxy step for Jordan (young kid profile), when she taps the thumbs-up control, then a rating is recorded against Jordan's profile and tonight's meal, tagged with Maya as the recording adult, and this counts as Jordan's one rating for the meal.

**FEAT-12.SPEC-001-AC-06:** Given Maya is on a proxy step for a young kid profile with no rating yet, when she taps Next without tapping a thumbs control, then the screen advances to the next step and that profile's rating for this meal remains neutral (unrated).

**FEAT-12.SPEC-001-AC-07:** Given Sam is on the final proxy step, when he taps Done, then the screen closes and returns to the week plan he opened the meal from.

**FEAT-12.SPEC-001-AC-08:** Given Maya has no young kid profiles in her household, when she completes her own rating, then the Next control reads "Done" directly and no proxy steps are shown.

**FEAT-12.SPEC-001-AC-09:** Given Sam loses connectivity while on the Post-Dinner Rating Prompt, when he taps a thumbs control, then the banner "You're offline -- this rating will save when you reconnect." appears and the rating is submitted automatically once connectivity returns.

**FEAT-12.SPEC-001-AC-10:** Given a rating submission for Maya fails, when the failure occurs, then no error is shown to Maya and the submission retries automatically in the background.

**FEAT-12.SPEC-001-AC-11:** Given Maya and Sam each attempt to record Jordan's (young kid profile) rating for the same meal within moments of each other, when both submissions are processed, then the later-arriving submission is the value that persists for Jordan's rating, per the Contention resolution for the Rating entity.

**FEAT-12.SPEC-001-AC-12:** Given the plan holding tonight's meal has been archived, when Sam opens the Post-Dinner Rating Prompt for that meal, then the thumbs controls show his last recorded value read-only, with no "Tap to change" text.

**FEAT-12.SPEC-001-AC-13:** Given Maya closes the Post-Dinner Rating Prompt after rating herself but before completing the proxy sequence, when she reopens the week plan later, then her own rating is kept and the household is never prompted to finish rating the remaining profiles.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 6 (not yet rated, already rated, submitting, error, offline/degraded, proxy sequence mid-flow) | 6 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |

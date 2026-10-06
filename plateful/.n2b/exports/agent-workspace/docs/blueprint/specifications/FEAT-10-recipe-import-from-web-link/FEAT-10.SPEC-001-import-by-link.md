---
document_type: spec
spec_type: screen
spec_id: FEAT-10.SPEC-001
spec_name: Import by Link
spec_slug: import-by-link
parent_feature: FEAT-10
parent_feature_name: Recipe Import from Web Link
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Screen Spec: Import by Link

## Overview

**Name:** Import by Link
**ID:** FEAT-10.SPEC-001
**Type:** Screen
**Purpose:** Household member pastes a web link to start a recipe import, with an explained progress indicator while extraction runs.
**Parent Feature:** FEAT-10 -- Recipe Import from Web Link

## Scope and Non-Goals

**In Scope:**
- Capturing a pasted web link and submitting it to start an import
- Well-formedness and weekly-limit checks before extraction is requested (via FEAT-10.SPEC-007)
- Showing an explained progress indicator while extraction runs (FEAT-10.SPEC-005)
- Holding a submitted link as a draft when offline and processing it automatically once connectivity returns

**Non-Goals:**
- Reviewing or editing the extracted recipe details -- handled by FEAT-10.SPEC-002 (Review Extracted Recipe); this screen only collects the link and shows extraction progress.
- Manual recipe entry -- handled by FEAT-10.SPEC-003 (Manual Recipe Entry), reached only when extraction fails and the member chooses to proceed manually.
- Bulk import of multiple links in one submission -- excluded per scope-boundaries.md SC-12: the brief names only per-link recipe import; households bring recipes in one link at a time.
- Determining whether the pasted page's content may legally be imported -- BRIEF.md's Open Questions leaves the legality of importing third-party recipe content as a product/legal decision to resolve before v1 ships; this screen assumes that resolution and only implements the functional import behavior approved for build.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-08.SPEC-001 (Recipe Library Browse & Search) | Household member taps "Import from link" | None -- link input starts empty |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | Paste a link and submit it | -- |
| Sam (Other Adult Member) | Full screen | Paste a link and submit it | -- |
| Jordan (young kid profile, no login -- MVP) | No | No | No login exists for this profile -- there is no path to this screen; Maya manages all content on this profile's behalf |
| Jordan (older kid, limited login -- Later) | No | No | "Import from link" is not offered on FEAT-08.SPEC-001 for this role (Recipe Library access is View, not Full); if reached directly, the screen shows "Importing recipes isn't available on this profile." with a link back to the recipe library |
| Riley (Operator, support) | No | No | This screen is not reachable from the read-only support view; Riley's Recipe Library access is View only and carries no import entry point |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in, the user lands on FEAT-08.SPEC-001 (Recipe Library), not this screen |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- a partially typed link is preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Import from Link" with a back arrow (returns to FEAT-08.SPEC-001, Recipe Library Browse & Search).

**Body:** A single-column form with:
- Link input (text input, required) -- placeholder "Paste a recipe link", accepts pasted or typed text
- "Import" action button, below the link input, disabled until the input is non-empty

Below the Import button, a helper line states: "We'll pull the ingredients, steps, and cook time from the page -- you'll be able to review and edit before it's saved."

While extraction runs (Extracting state), the link input and Import button are replaced by a progress indicator with the label "Reading the recipe... this usually takes a few seconds."

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Single-column form as described above, full width; Import button spans the input's width.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-08.SPEC-001 (Recipe Library) | Screen closes | Animated transition back to library |
| Link input | Type or paste | Captures link text | Import button enables once non-empty | Standard input focus state |
| Import button | Tap | 1. Validate the link and check the weekly import allowance via FEAT-10.SPEC-007. 2. If valid, request extraction via FEAT-10.SPEC-005 (Web Page Recipe Extraction). | Screen enters Validating then Extracting state | Progress indicator "Reading the recipe... this usually takes a few seconds." |
| Import button (while extracting) | Tap | No action -- debounced | None | Button/indicator area unchanged; a second submission cannot start while one is in flight |

### Accessibility Notes

- **Focus order:** Back arrow -> Link input -> Import button.
- **Validation announcements:** When the link fails well-formedness or the weekly limit is reached, the resulting message is announced to assistive technology and programmatically associated with the input.
- **Progress announcements:** Entry into the Extracting state announces "Reading the recipe" to assistive technology; completion (success or failure) announces the resulting state.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (default) | Link input empty, Import button disabled | Screen first opens | Member types or pastes into the link input |
| Filling | Link input contains text, Import button enabled | Member types or pastes a non-empty value | Member taps Import or navigates away |
| Validating | Import button shows a brief loading state | Member taps Import | FEAT-10.SPEC-007's well-formedness and weekly-limit checks complete |
| Validation Error | Link input shows an error state with the message from FEAT-10.SPEC-007 | A validation rule fails | Member edits the link and re-submits |
| Extracting | Link input and Import button replaced by the progress indicator "Reading the recipe... this usually takes a few seconds." | Validation passes and extraction is requested (FEAT-10.SPEC-005) | Extraction succeeds or fails |
| Error (extraction failed) | Message "We couldn't read that page." with a "Enter details manually" action and a "Try a different link" action | FEAT-10.SPEC-005 reports extraction failure | Member chooses manual entry (navigates to FEAT-10.SPEC-003) or re-submits a different link |
| Offline/Degraded | Banner "You're offline -- this link will be imported when you reconnect." at top; link input remains editable and submittable; submitting queues the link as a draft locally | Connectivity lost while the screen is open, or the member submits while already offline | Connectivity restored -- the queued link is validated and extraction is requested automatically, and the screen proceeds through Validating/Extracting as normal |

## Validation Rules

Validation governed by FEAT-10.SPEC-007 (Recipe Import Validation & Rate Limit Rules). See that spec for link well-formedness and the weekly import allowance. This screen checks on Import button tap.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-08.SPEC-001 (Recipe Library Browse & Search) | FEAT-08 |
| Extraction succeeds | FEAT-10.SPEC-002 (Review Extracted Recipe) | -- |
| Extraction fails, member chooses "Enter details manually" | FEAT-10.SPEC-003 (Manual Recipe Entry) | -- |

## Data Model

**Creates:** None -- this screen does not create a Recipe record; it only initiates extraction.
**Reads:** Household's current-week import count (used by FEAT-10.SPEC-007's weekly-limit check) -- read-only, no fields displayed on this screen.
**Updates:** None.
**Deletes:** None.

## Business Rules

- Link well-formedness and the 30-import-per-week allowance are enforced by FEAT-10.SPEC-007 -- the member cannot submit a link that fails either check.
- A link submitted while offline is held as a draft and processed automatically once connectivity returns, per the feature's own States field.
- Only one extraction request runs at a time from this screen -- a second Import tap while one is in flight is ignored (debounced).

## Edge Cases

- **Member submits an empty link input** -- Import button remains disabled; no submission is possible.
- **Member taps Import twice rapidly** -- Second tap is ignored while the first request is in flight (button/indicator area unchanged).
- **Member navigates away while extraction is running** -- The extraction request continues; if it completes after the member has left, no notification interrupts them -- the result (populated review draft, or the failure state) is present the next time they return to this flow or reopen the same link.
- **Member pastes a link that was already imported by this household** -- The screen does not detect duplicates itself; duplicate handling occurs at save time on FEAT-10.SPEC-002/FEAT-10.SPEC-003 via FEAT-10.SPEC-006 (Duplicate Import Detection), so extraction still runs normally here.
- **Connectivity is lost mid-extraction** -- The Extracting state does not silently stall; if the request cannot complete, the screen shows the Offline/Degraded banner and holds the link as a draft, retrying automatically once connectivity returns.
- **No concurrent-edit conflict applies to this screen** -- This screen does not create, read, or update any existing shared Recipe record; it only initiates extraction, so no stale-write scenario exists here (the dependency map's Contention notes for Recipe apply starting at save time, covered by FEAT-10.SPEC-002/003/004).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-08.SPEC-001 (Recipe Library Browse & Search) | Navigation (inbound) | Member arrives here from the library's "Import from link" entry point |
| FEAT-10.SPEC-007 (Recipe Import Validation & Rate Limit Rules) | References (inbound) | Link well-formedness and weekly-limit rules applied on Import tap |
| FEAT-10.SPEC-005 (Web Page Recipe Extraction) | Triggers (outbound) | Import tap, after validation passes, requests extraction |
| FEAT-10.SPEC-002 (Review Extracted Recipe) | Navigation (outbound) | Extraction success navigates here with the extracted draft |
| FEAT-10.SPEC-003 (Manual Recipe Entry) | Navigation (outbound) | Extraction failure, if the member proceeds manually, navigates here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| recipe_import_started | entry source (library entry point), offline-at-submission (yes/no) | Link passes FEAT-10.SPEC-007's well-formedness check and extraction is requested | N/A -- no success-metrics.md metric is connected to Recipe Import from Web Link; retained for operational visibility into the import funnel |
| recipe_import_weekly_limit_blocked | current week's import count | Member's submission is blocked by FEAT-10.SPEC-007's 30-import-per-week guard | N/A -- no success-metrics.md metric is connected to Recipe Import from Web Link; retained so the misuse guard's real-world trigger rate is observable |

## Acceptance Criteria

**FEAT-10.SPEC-001-AC-01:** Given Maya is on the Import by Link screen, when she pastes a well-formed link and taps Import, then the screen shows "Reading the recipe... this usually takes a few seconds." and requests extraction via FEAT-10.SPEC-005.

**FEAT-10.SPEC-001-AC-02:** Given Sam is on the Import by Link screen with the link input empty, when he looks at the Import button, then it is disabled and cannot be tapped.

**FEAT-10.SPEC-001-AC-03:** Given Maya pastes a malformed link, when she taps Import, then the link input shows the error message defined by FEAT-10.SPEC-007 and no extraction is requested.

**FEAT-10.SPEC-001-AC-04:** Given Sam has already imported 30 recipes this week, when he attempts to submit another link, then he sees the weekly-limit block message defined by FEAT-10.SPEC-007 and no extraction is requested.

**FEAT-10.SPEC-001-AC-05:** Given extraction succeeds for Maya's submitted link, when the result returns, then she is navigated to FEAT-10.SPEC-002 (Review Extracted Recipe) with the extracted draft populated.

**FEAT-10.SPEC-001-AC-06:** Given extraction fails for Sam's submitted link, when the failure is reported, then the screen shows "We couldn't read that page." with "Enter details manually" and "Try a different link" actions.

**FEAT-10.SPEC-001-AC-07:** Given Sam sees the extraction-failed message, when he taps "Enter details manually", then he is navigated to FEAT-10.SPEC-003 (Manual Recipe Entry).

**FEAT-10.SPEC-001-AC-08:** Given Maya loses connectivity while the link input is filled, when she taps Import, then the banner "You're offline -- this link will be imported when you reconnect." appears and the link is held as a draft.

**FEAT-10.SPEC-001-AC-09:** Given Maya's link was held as a draft while offline, when connectivity returns, then validation and extraction proceed automatically without her needing to re-submit.

**FEAT-10.SPEC-001-AC-10:** Given Maya taps Import while extraction from a prior tap is still running, when she taps a second time, then the second tap has no effect and the progress indicator remains unchanged.

**FEAT-10.SPEC-001-AC-11:** Given the older-kid limited-login role (Later) does not see "Import from link" on FEAT-08.SPEC-001, when they nonetheless reach this screen directly, then it shows "Importing recipes isn't available on this profile." with a link back to the recipe library.

**FEAT-10.SPEC-001-AC-12:** Given Maya taps the back arrow with the link input filled but not yet submitted, when she confirms leaving, then she returns to FEAT-08.SPEC-001 and no import is started.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 4 | 4 |
| States | 7 (empty, filling, validating, validation error, extracting, error, offline) | 7 |
| Business Rules | 3 | 3 |
| Edge Cases | 6 | 6 |

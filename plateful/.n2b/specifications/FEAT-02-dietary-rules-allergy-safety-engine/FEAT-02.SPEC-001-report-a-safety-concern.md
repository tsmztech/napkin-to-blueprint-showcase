---
document_type: spec
spec_type: screen
spec_id: FEAT-02.SPEC-001
spec_name: Report a Safety Concern
spec_slug: report-a-safety-concern
parent_feature: FEAT-02
parent_feature_name: Dietary Rules & Allergy Safety Engine
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Screen Spec: Report a Safety Concern

## Overview

**Name:** Report a Safety Concern
**ID:** FEAT-02.SPEC-001
**Type:** Screen
**Purpose:** An adult flags a meal they believe is unsafe, with an optional short note, from wherever the meal is currently shown.
**Parent Feature:** FEAT-02 -- Dietary Rules & Allergy Safety Engine

## Scope and Non-Goals

**In Scope:**
- The confirmation dialog an adult reaches by tapping "report a safety concern" on a shown Planned Meal
- Capturing the optional short note (up to 500 characters)
- Submitting the report and showing the immediate in-screen result of that submission

**Non-Goals:**
- Removing the meal from the plan, excluding the recipe, and creating the Support Request -- performed by FEAT-02.SPEC-004 (Safety Concern Intake & Removal), which this screen's submission triggers
- Reviewing or resolving the report -- owned by the operator's Support View (FEAT-22), outside this feature
- Reporting anything other than a currently shown Planned Meal -- excluded per the feature's own Access field (only adults reporting on a meal already displayed by another feature can reach this dialog); there is no standalone "browse past concerns" screen in this feature
- Kid-initiated reports -- excluded per scope-boundaries.md (SC-02): young kid profiles have no login in v1, so only Maya and Sam can reach this screen

## Entry Points

{This screen has no feature of its own that opens it directly -- it is reached only from a "report a safety concern" control that other features' meal-display specs place next to a shown Planned Meal.}

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-03 (AI Weekly Dinner Plan Generation), planned meal card | Adult taps "report a safety concern" on a dinner | The Planned Meal reference and its Recipe |
| FEAT-23 (Manual Weekly Planning), planned meal card | Adult taps "report a safety concern" on a picked dinner | The Planned Meal reference and its Recipe |
| FEAT-19 (Weekly Plan History), a re-used past week's meal card | Adult taps "report a safety concern" on a meal shown from history | The Planned Meal reference and its Recipe |
| FEAT-08 / FEAT-10 (Recipe Library / Recipe Import), recipe detail when opened from a plan context | Adult taps "report a safety concern" on the recipe as placed in the current plan | The Planned Meal reference and its Recipe |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full dialog | Submit a report on any household meal | -- |
| Sam (Other Adult Member) | Full dialog | Submit a report on any household meal (Safety Reports: Own-only means Sam acts only through his own submissions, not that he can report only his own meals -- any adult may flag any meal) | -- |
| Jordan (young kid profile, no login -- MVP) | No | No | The "report a safety concern" control is never shown on any screen a young kid profile could reach, since young kid profiles have no login at all |
| Jordan (older kid, limited login -- Later) | No | No | The "report a safety concern" control does not appear on any screen the older-kid login can reach; Dinner Voting and Grocery List are the only surfaces available to this role, and neither shows the control |
| Riley (Operator, support) | No | No | Riley has no access to any household screen outside the read-only Support View (FEAT-22); this dialog never renders for Riley |
| Unauthenticated | No | No | Redirected to the sign-in screen; no in-progress report context survives, since none can exist before sign-in |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- any typed note is preserved locally and restored in the dialog after re-authentication succeeds |

## Layout and Content

**Header:** Dialog title "Report a safety concern" with a close control (top-right, dismisses without submitting) and, below the title, the name of the meal being reported (recipe name and the night it is planned for).

**Body:** A short explanatory line: "This meal will be removed from your plan right away, and we'll offer safe alternatives. Your household's operator will review the ingredients." Below it, a single optional multi-line text field labeled "What's wrong? (optional)" with a visible remaining-character count that starts at 500 and counts down as the adult types.

**Footer:** Two actions, right-aligned: "Cancel" (secondary, dismisses without submitting) and "Report and remove" (primary).

### Responsive Behavior

- **Compact breakpoint:** The dialog occupies the full screen width with the meal name and note field stacked vertically; footer actions stack full-width, "Report and remove" above "Cancel".
- **Medium size class and above:** The dialog renders as a centered modal capped at a consistent platform-wide dialog width (exact value is the design layer's decision); footer actions remain side by side, right-aligned.
- **Note field:** Grows from 3 visible lines (compact) to 4 visible lines (medium and above); no structural change beyond line count.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Close control | Tap | Dismiss the dialog without submitting | Dialog closes, returns to the calling screen | No confirmation needed -- nothing has been submitted |
| Note field | Type | Captures free text up to 500 characters | Remaining-character count updates | Count updates live; typing beyond 500 characters is blocked at the field |
| Cancel button | Tap | Dismiss the dialog without submitting | Dialog closes, returns to the calling screen | No confirmation needed -- nothing has been submitted |
| Report and remove button | Tap | 1. Submit the report (meal reference, reporting member, optional note) to FEAT-02.SPEC-004 (Safety Concern Intake & Removal). 2. Wait for confirmation that the meal was removed. | Button shows a brief loading state during submission | Success: dialog replaces its content with a confirmation message and a "Show me alternatives" action; the calling screen updates to show the meal as removed. Failure: inline error banner in the dialog with a Retry option; the meal and note are unchanged. |
| Report and remove button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |
| "Show me alternatives" (post-submission) | Tap | Navigate to FEAT-04.SPEC-001 (Meal Swap, safe alternatives list) for the now-empty slot | Dialog closes | Alternatives list opens for the affected night |

### Accessibility Notes

- **Focus order:** Close control -> meal name (read-only, announced but not focusable) -> note field -> Cancel -> Report and remove.
- **Submission announcements:** On successful submission, the confirmation message and "Show me alternatives" action are announced to assistive technology as the dialog's content changes. On failure, the error banner is announced and focus moves to it.
- **Character count:** The remaining-character count is associated with the note field so assistive technology announces it as the field's description, not as a separate unlabelled element.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Ready (default) | Meal name shown, note field empty, both actions enabled | Dialog opens | Adult types in the note field or taps an action |
| Filling | Note field contains text, remaining-character count updated | Adult types in the note field | Adult taps Cancel, Close, or Report and remove |
| Submitting | "Report and remove" shows a loading state, both actions disabled | Adult taps "Report and remove" | Submission completes or fails |
| Submitted | Confirmation message and "Show me alternatives" action replace the form | Submission completes successfully | Adult taps "Show me alternatives" or Close |
| Error | Error banner "Couldn't submit your report. Check your connection and try again." with Retry; note text preserved | Submission fails | Adult taps Retry (returns to Submitting) or Close |
| Offline/Degraded | Banner "You're offline -- reporting a safety concern needs a connection, since the meal must be removed right away." "Report and remove" is disabled while offline | Connectivity lost while the dialog is open | Connectivity restored -- banner clears and "Report and remove" re-enables; the adult resubmits manually (nothing is auto-queued, since a safety removal must happen the moment it is confirmed, not later) |

## Validation Rules

**Option B -- Inline (simple validation not warranting a standalone Logic/Rule spec):**

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| Note | Optional; maximum 500 characters | On change (input is blocked past 500) | "Your note can be up to 500 characters." (shown only if pasted text exceeds the limit; typing is capped silently at 500) |

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Close or Cancel | Returns to the calling screen (no navigation) | -- |
| Successful submission, "Show me alternatives" tapped | FEAT-04.SPEC-001 (Meal Swap) | FEAT-04 (One-Tap Meal Swap) |
| Successful submission, Close tapped instead | Returns to the calling screen, now showing the meal as removed | -- |

## Data Model

**Creates:** Support Request -- kind (safety concern), raised_by (the reporting member), planned_meal/recipe (the reported meal and its recipe), note (the optional text entered here, up to 500 characters), status (Raised). Created by FEAT-02.SPEC-004 on submission from this screen.
**Reads:** Planned Meal -- night, recipe, for display in the dialog header.
**Updates:** None directly -- Planned Meal status and swap_history are updated by FEAT-02.SPEC-004, not by this screen.
**Deletes:** None.

## Business Rules

- Submitting from this screen always triggers FEAT-02.SPEC-004 (Safety Concern Intake & Removal), which removes the meal, excludes the recipe for the household, and creates the Support Request -- this screen never partially completes a report.
- FEAT-02.SPEC-009 (Safety Concern Eligibility & Re-offer Policy) governs the excluded recipe's re-offer eligibility once this report is submitted; this screen has no visibility into that eligibility state.
- XBR-08: A safety-concern report removes the meal from the plan at once, excludes the recipe while the report is open, drops its ingredients from the grocery list, offers safe alternatives, reaches the operator for review, and tells the household the outcome.

## Edge Cases

- **Adult opens the dialog on a meal another adult has already reported** -- The dialog still opens (this screen does not check for an existing open report); FEAT-02.SPEC-004 recognizes the meal already has an open Support Request and does not create a duplicate, and the submission confirms with the same "removed" message since the meal is already off the plan.
- **Adult taps "Report and remove" twice rapidly** -- The second tap is ignored while the first submission is in progress (button in loading state).
- **Meal is swapped out by another household member while this dialog is open** -- Submission proceeds against the meal reference captured when the dialog opened; if that slot no longer holds the reported recipe, FEAT-02.SPEC-004 still creates the report against the recipe as reported, and the confirmation message states the report was recorded, since the underlying safety concern about that recipe stands regardless of the slot's current contents. There is no concurrent-edit conflict here because this screen never re-saves the Planned Meal itself -- it only submits a new report, which FEAT-02.SPEC-004 owns.
- **Network failure during submission** -- Error banner: "Couldn't submit your report. Check your connection and try again." with a Retry button. Note text preserved.
- **Adult closes the dialog with unsaved note text** -- No confirmation dialog is shown; a note is not a commitment until submitted, so closing simply discards it.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-004 (Safety Concern Intake & Removal) | Triggers (outbound) | Submitting the report fires this automation |
| FEAT-02.SPEC-009 (Safety Concern Eligibility & Re-offer Policy) | References (outbound) | Governs the excluded recipe's re-offer state after submission |
| FEAT-04.SPEC-001 (Meal Swap alternatives list) | Navigation (outbound) | "Show me alternatives" opens the safe alternatives list for the emptied slot |
| FEAT-03 (AI Weekly Dinner Plan Generation), planned meal card | Navigation (inbound) | Entry point when reporting from the AI-generated plan |
| FEAT-23 (Manual Weekly Planning), planned meal card | Navigation (inbound) | Entry point when reporting from a manually picked plan |
| FEAT-19 (Weekly Plan History), re-used meal card | Navigation (inbound) | Entry point when reporting from a re-used past week |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| safety_concern_report_opened | entry source (plan / manual plan / history) | Dialog opens | supports success-metrics.md: "Zero Allergy Incidents" |
| safety_concern_reported | note provided (yes/no), entry source | Submission succeeds | supports success-metrics.md: "Zero Allergy Incidents" |
| safety_concern_report_failed | reason (network) | Submission fails | supports success-metrics.md: "Zero Allergy Incidents" |
| safety_concern_report_abandoned | note text entered (yes/no) | Adult closes or cancels without submitting | N/A -- no Stage 2 metric measures abandonment of this dialog; retained so the reporting flow's friction is observable |

## Acceptance Criteria

**FEAT-02.SPEC-001-AC-01:** Given Maya is viewing Thursday's dinner on her weekly plan, when she taps "report a safety concern" on it, then the dialog opens showing the recipe name and Thursday as the night.

**FEAT-02.SPEC-001-AC-02:** Given Maya has the dialog open with the note field empty, when she taps "Report and remove", then the meal is removed from the plan, the dialog shows a confirmation message, and a "Show me alternatives" action appears.

**FEAT-02.SPEC-001-AC-03:** Given Sam has the dialog open, when he types a note describing the concern and taps "Report and remove", then the report is submitted with his note attached and he sees the same confirmation Maya would see.

**FEAT-02.SPEC-001-AC-04:** Given Maya is typing in the note field, when her note reaches 500 characters, then further typing is blocked and the remaining-character count shows 0.

**FEAT-02.SPEC-001-AC-05:** Given Maya has the dialog open, when she taps the close control, then the dialog closes without submitting anything and no Support Request is created.

**FEAT-02.SPEC-001-AC-06:** Given Sam taps "Report and remove" and the submission fails due to a network error, then the error banner "Couldn't submit your report. Check your connection and try again." appears with a Retry button, and his note text is preserved.

**FEAT-02.SPEC-001-AC-07:** Given Maya taps "Report and remove" twice in rapid succession, when the first tap is already processing, then the second tap has no effect and only one report is submitted.

**FEAT-02.SPEC-001-AC-08:** Given Maya's session expires while the dialog is open with note text entered, when the session-expired dialog appears and she signs back in, then the safety concern dialog reopens with her note text restored.

**FEAT-02.SPEC-001-AC-09:** Given Sam successfully submits a report and sees the confirmation, when he taps "Show me alternatives", then he is taken to the safe alternatives list (FEAT-04.SPEC-001) for the emptied slot.

**FEAT-02.SPEC-001-AC-10:** Given Jordan is signed in through the Later-phase older-kid limited login, when Jordan views a planned meal on any screen that login can reach, then no "report a safety concern" control is shown.

**FEAT-02.SPEC-001-AC-11:** Given Maya loses connectivity while the dialog is open, then a banner states reporting needs a connection and "Report and remove" is disabled until connectivity returns.

**FEAT-02.SPEC-001-AC-12:** Given a meal already has an open safety report from Sam, when Maya opens the dialog on the same meal and submits her own report, then her submission still confirms as removed, and no duplicate Support Request is created (per FEAT-02.SPEC-004).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 7 | 7 |
| States | 4 (ready/filling, submitting error, offline, submitted) | 4 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |

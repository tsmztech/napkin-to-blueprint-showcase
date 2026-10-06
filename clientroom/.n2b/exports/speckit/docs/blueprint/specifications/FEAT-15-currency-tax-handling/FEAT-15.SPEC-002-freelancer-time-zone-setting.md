---
document_type: spec
spec_type: screen
spec_id: FEAT-15.SPEC-002
spec_name: Freelancer Time Zone Setting
spec_slug: freelancer-time-zone-setting
parent_feature: FEAT-15
parent_feature_name: Currency & Tax Handling
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Screen Spec: Freelancer Time Zone Setting

## Overview

**Name:** Freelancer Time Zone Setting
**ID:** FEAT-15.SPEC-002
**Type:** Screen
**Purpose:** Nadia views and changes her own time zone, which grounds every reminder day-count and local-time display across the product.
**Parent Feature:** FEAT-15 -- Currency & Tax Handling

## Scope and Non-Goals

**In Scope:**
- Displaying Nadia's current time zone
- Letting Nadia change her time zone
- Saving the new value so it takes effect for all future reminder-day-count and local-time-display computation

**Non-Goals:**
- Rendering any individual date, due date, or time in a viewer's own time zone -- owned entirely by FEAT-15.SPEC-006 (Time Zone & Local Date/Time Display Rule); this screen only captures Nadia's own value
- Any other account setting (name, sign-in email, business details, notification preferences) -- owned entirely by Settings & Account Management (FEAT-21); this screen owns only the `time_zone` field
- Setting a time zone for a client contact -- the product defines time zone only for the freelancer's account; a client contact's own device/browser supplies their local time zone for FEAT-15.SPEC-006's rendering, with no setting screen of their own
- Historical tracking of past time zone values -- the entity carries a single current value with no versioned history, consistent with this feature's Entity-Lifecycle Coverage Matrix, which records only Create/Read/Update for this field

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-21 (Settings & Account Management) | Nadia opens her time zone preference from her account settings area | None -- screen loads her current value |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Change and save her own time zone | -- |
| Owen (Client Primary Contact) | None -- this is a freelancer-account setting with no client-facing surface | None | This screen has no route reachable from Owen's portal; a direct attempt resolves the same as any out-of-scope client link, per XBR-09 |
| Priya (Client Reviewer Contact) | None -- this is a freelancer-account setting with no client-facing surface | None | This screen has no route reachable from Priya's portal; a direct attempt resolves the same as any out-of-scope client link, per XBR-09 |
| Dana (Support Operator) | Full screen, read-only (Freelancer Account is View-only for Dana per the Access Matrix) | View only | The time zone selector and Save control are not rendered; the current value is shown as plain text |
| Unauthenticated | No | No | Redirected to sign-in |
| Expired session | No | No | Redirected to sign-in; any unsaved selection is preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Time Zone" with a back control returning to account settings (FEAT-21).

**Body:** A single field:
- Time zone selector (selection input, required, always has a value): a searchable list of recognized time zones, showing the current UTC offset next to each option. Preselected to Nadia's currently saved time zone (or the default captured at sign-up, FEAT-20, if never changed).
- Helper text directly beneath the selector: "This sets the time zone used for your payment reminder countdowns and any dates or times shown to you."

**Footer:** Save button.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout, full width; Save remains directly below the helper text.
- **Medium size class and above:** Uniform scaling, no structural change -- the form is capped at a consistent platform-wide form width and horizontally centered.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back control | Tap | Navigate to FEAT-21 (account settings) | Screen closes | Standard transition |
| Time zone selector | Select | Captures the chosen time zone | Field shows chosen time zone and its UTC offset | Standard selection state |
| Save button | Tap | Saves the freelancer account's `time_zone` field | Button shows loading state during save | Success: toast "Time zone updated" and the selector reflects the saved value. Failure: error banner. |
| Save button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back control -> time zone selector -> Save.
- **Save feedback:** The "Time zone updated" toast is announced on success; on save failure, focus moves to the error banner.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; the time zone selector's search input accepts typed queries with keyboard-navigable results.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loaded (default) | Selector preselected to Nadia's current time zone, Save enabled | Screen first opens | Nadia changes the selection |
| Changed | Selector shows the newly chosen time zone, Save enabled | Nadia selects a different time zone | Nadia taps Save or navigates away |
| Saving | Save button shows loading spinner, selector disabled | Nadia taps Save | Save completes or fails |
| Error | Error banner "Couldn't update your time zone. Check your connection and try again." with Retry | Save operation fails | Nadia taps Retry or navigates away |
| Offline/Degraded | Banner "You're offline -- this change will be saved when you reconnect." at top; selector remains usable, Save queues the change locally | Connectivity lost while the screen is open | Connectivity restored -- queued save submits automatically and the standard success feedback appears |

## Validation Rules

**Option B -- Inline (simple validation not warranting a standalone spec):**

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| Time zone | Required -- must be a recognized time zone from the selector's list; the field is never left blank since a value is always preselected | On submit | "Choose a time zone to continue." |

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back control tap | Account settings | FEAT-21 |
| Successful save | This screen, re-rendered with the saved value | -- |

## Data Model

**Creates:** None -- the Freelancer Account record itself is created at sign-up (FEAT-20), which sets an initial time zone default; this screen only lets Nadia change it afterward.
**Reads:** Freelancer Account -- `time_zone`.
**Updates:** Freelancer Account -- `time_zone`.
**Deletes:** None.

## Business Rules

- Saving a new time zone recomputes the basis used for every future reminder-day-count and local-time display for Nadia's account, governed by FEAT-15.SPEC-006 -- this screen does not itself recompute anything, only writes the new value.
- Time zone is the only Freelancer Account field this feature owns; every other account field is out of scope here and lives in FEAT-21.
- There is no concurrent-edit conflict handling beyond last-write-wins: the dependency map's Contention note for Freelancer Account states two open sessions of Nadia's resolve last-write-wins per field, and `time_zone` is not the sign-in-email exception that requires re-verification.

## Edge Cases

- **Nadia navigates away with an unsaved selection** -- Confirmation dialog: "You have an unsaved time zone change. Discard?" with "Discard" and "Keep Editing" options.
- **Nadia taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in loading state).
- **Nadia changes her time zone from two open sessions in quick succession** -- Last-write-wins, consistent with the dependency map's Contention note for the Freelancer Account entity: the later save's value is what persists, and the earlier session's selector re-renders to the newer value on its next load.
- **Nadia saves the same time zone she already had selected** -- Save proceeds normally and shows the same success feedback; no distinct "no change" state is defined.
- **Network failure during save** -- Error banner: "Couldn't update your time zone. Check your connection and try again." with a Retry button. Selection is preserved.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-15.SPEC-006 (Time Zone & Local Date/Time Display Rule) | Triggers (outbound) | The value saved here is the basis this rule uses for every reminder-day-count and local-time computation grounded in Nadia's time zone |
| FEAT-21 (Settings & Account Management) | Navigation (inbound) | Sole entry point into this screen, from the freelancer's account settings area |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| timezone_set | new time zone value | Nadia's save succeeds | N/A -- no success-metrics.md metric measures time zone configuration directly; retained since product-features.md's Signals field for FEAT-15 names `timezone_set` explicitly as a required signal, and the value this event carries is what makes "Invoice Currency and Tax Correctness"-adjacent local-time correctness verifiable downstream in FEAT-15.SPEC-006 |

## Acceptance Criteria

**FEAT-15.SPEC-002-AC-01:** Given Nadia opens her time zone preference for the first time, when the screen loads, then the selector shows the time zone default captured at her sign-up.

**FEAT-15.SPEC-002-AC-02:** Given Nadia selects a different time zone, when she taps Save, then the change is saved and she sees a "Time zone updated" toast.

**FEAT-15.SPEC-002-AC-03:** Given Nadia has just saved a new time zone, when any future reminder day-count or local-time display is computed for her account, then it uses the newly saved value, per FEAT-15.SPEC-006.

**FEAT-15.SPEC-002-AC-04:** Given Dana opens this screen inside a support session, when the screen loads, then she sees Nadia's current time zone as plain text with no selector or Save control rendered.

**FEAT-15.SPEC-002-AC-05:** Given Nadia has an unsaved time zone selection, when she taps the back control, then a confirmation dialog appears asking "You have an unsaved time zone change. Discard?"

**FEAT-15.SPEC-002-AC-06:** Given Nadia taps Save twice in rapid succession, when the first save is still in progress, then the second tap has no effect and the button remains in its loading state.

**FEAT-15.SPEC-002-AC-07:** Given Nadia changes her time zone in one session while an older session of hers is also open on this screen, when both saves land, then the later save's value persists and the older session's selector reflects it on its next load (last-write-wins).

**FEAT-15.SPEC-002-AC-08:** Given the save operation fails for a connectivity reason, when the failure occurs, then Nadia sees "Couldn't update your time zone. Check your connection and try again." with a Retry action.

**FEAT-15.SPEC-002-AC-09:** Given Nadia loses connectivity after selecting a new time zone, when she taps Save, then the banner "You're offline -- this change will be saved when you reconnect." appears and the change is submitted automatically once connectivity returns.

**FEAT-15.SPEC-002-AC-10:** Given Nadia saves the same time zone she already had, when the save completes, then it succeeds with the same success feedback as any other save.

**FEAT-15.SPEC-002-AC-11:** Given Nadia successfully saves a time zone, when the save completes, then the timezone_set event is emitted with the new value.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 4 | 4 |
| States | 5 (loaded, changed, saving, error, offline) | 5 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |

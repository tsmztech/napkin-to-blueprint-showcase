---
document_type: spec
spec_type: screen
spec_id: FEAT-05.SPEC-001
spec_name: Pantry List & Item Entry
spec_slug: pantry-list-item-entry
parent_feature: FEAT-05
parent_feature_name: Pantry-Aware Suggestions
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 15
---

# Screen Spec: Pantry List & Item Entry

## Overview

**Name:** Pantry List & Item Entry
**ID:** FEAT-05.SPEC-001
**Type:** Screen
**Purpose:** Household member views the household's current pantry list, adds an item they already have on hand in plain language, and clears an item once it is used or gone.
**Parent Feature:** FEAT-05 -- Pantry-Aware Suggestions

## Scope and Non-Goals

**In Scope:**
- Displaying every Active Pantry Item for the household as a list
- A single free-text add-item field for logging a new item
- Manually clearing (hard-deleting) a Pantry Item once it is used or gone
- Displaying the "used it up?" prompt chip on a row when FEAT-05.SPEC-002 has surfaced it, and handling the tap that clears the item from that prompt
- Empty, loading, error, and offline-degraded presentation for this screen

**Non-Goals:**
- Detailed pantry inventory fields such as quantity, unit, or expiry date -- excluded per scope-boundaries.md (SC-11): the product follows the lighter "tell me what I have" model instead of a detailed inventory, so the item carries only a name.
- A separate single-item detail view -- excluded per the Feature Breakdown Brief's Entity-Lifecycle Coverage Matrix: a Pantry Item carries no fields beyond its name and status, so a dedicated detail screen would duplicate the row shown here with no added information.
- Determining when the "used it up?" prompt should appear -- governed by FEAT-05.SPEC-002 (Used-It-Up Prompt Trigger); this screen only renders the prompt chip once that automation has surfaced it and handles the tap.
- Merging a newly typed item into an existing entry with the same name -- governed by FEAT-05.SPEC-003 (Pantry Item Duplicate Merge); this screen submits the typed name and displays whichever row results.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| Main navigation (default entry) | Household member (Maya or Sam) taps "Pantry" in the app's primary navigation | None -- the current Active pantry list loads |
| FEAT-06.SPEC-001 (Grocery List) | Maya or Sam taps "Already have it" on a plan-derived grocery list line, then taps the "View Pantry" action on the confirmation toast (action offered only to roles with Pantry Input access, per FEAT-05.SPEC-007) | The newly logged or merged item, shown highlighted at the top of the list |
| FEAT-05.SPEC-002 (Used-It-Up Prompt Trigger) | Household member opens the pantry list after the automation has surfaced a prompt | The prompt chip is shown on the affected row without further navigation context |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen -- the household's complete Active pantry list | Add an item, clear an item, tap "used it up?" | -- |
| Sam (Other Adult Member) | Full screen -- the household's complete Active pantry list | Add an item, clear an item, tap "used it up?" | -- |
| Jordan (young kid profile, no login -- MVP) | No -- has no login and no product access | No | This role has no account and cannot open any screen; there is nothing to redirect, since the profile never authenticates |
| Jordan (older kid, limited login -- Later) | No -- Pantry Input is None for this role | No | The "Pantry" entry is not shown in this role's main navigation; a direct link is redirected to the plan (FEAT-03) with no error message, since this area does not exist for this role |
| Riley (Operator, support -- from v1) | Full screen, read-only, and only while an open Support Request exists for the household (FEAT-22) | No -- cannot add, clear, or answer the "used it up?" prompt | Outside an open Support Request, this screen is not reachable; the add-item field and clear/prompt actions are not rendered in the read-only support view |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in as a household member, the user lands on the pantry list, not an intermediate screen |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- any item text typed but not yet saved is preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Pantry" with the primary navigation's standard back/menu control; no page-level action button in the header (adding happens inline in the body).

**Body:** A single-column list screen.
- **Add-item field:** A single free-text input pinned above the list, with placeholder text "What do you have?" and an inline "Add" control beside it. This is the Shared Context's Add-item field, reused verbatim by the inbound FEAT-06 "already have it" path (governed by FEAT-05.SPEC-007).
- **Pantry list:** Below the add-item field, every Active Pantry Item for the household as one row each, ordered most-recently-added first. Each row shows:
  - The item's name (free text, as entered)
  - A small "added by" indicator naming the member who logged it
  - A clear action (a tap target that removes the row)
  - A "used it up?" prompt chip, shown only on a row where FEAT-05.SPEC-002 has surfaced the prompt, placed alongside the clear action

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Add-item field and list are full width, single column, as described above; the clear action and "used it up?" chip remain visible on the row without requiring a swipe, consistent with the product's one-handed, large-tap-target accessibility baseline (ASMP-29).
- **Medium size class and above:** The list remains single-column, capped at a consistent platform-wide content width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Add-item field | Type | Captures free-text item name | Field shows entered text | Standard input focus state |
| Add-item field / Add control | Tap "Add" (or submit) | Validates the name via FEAT-05.SPEC-004, then triggers FEAT-05.SPEC-003 (Pantry Item Duplicate Merge) to save or merge it | New or merged row appears at the top of the list; field clears | Row appears instantly (per the feature's own Loading-state expectation); no full-page spinner |
| Add-item field | Submit with invalid name | Validation via FEAT-05.SPEC-004 fails | Field shows error state | Exact error message from FEAT-05.SPEC-004 shown inline below the field; typed text is preserved |
| Pantry row: clear action | Tap | Sets the item's status to Used/Removed and hard-deletes the row (no restore path) | Row is removed from the list with a brief exit animation | -- |
| Pantry row: "used it up?" chip | Tap | Sets the item's status to Used/Removed and hard-deletes the row, same as the manual clear action | Row is removed from the list with a brief exit animation | -- |

### Accessibility Notes

- **Focus order:** Add-item field -> Add control -> each pantry row's clear action (and "used it up?" chip, when present), top to bottom in list order.
- **Add feedback:** The newly added or merged row is announced to assistive technology when it appears at the top of the list.
- **Validation announcements:** When the add-item field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Clear feedback:** A row's removal (manual clear or "used it up?" tap) is announced to assistive technology.
- **Keyboard alternatives:** Every action on this screen -- typing, adding, clearing, and answering the "used it up?" prompt -- is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty | A short prompt to add what's on hand replaces the list body (per the feature's own Empty-state expectation); add-item field remains at the top | Household has zero Active pantry items | An item is successfully added |
| Populated | The pantry list as described in Layout and Content | One or more Active pantry items exist | List becomes empty (last item cleared) |
| Loading | A brief inline indicator on the add-item field while the add request is in flight | User submits a new item | Save succeeds (row appears) or fails (Error state) |
| Error | The typed item remains visible in the add-item field with a retry option, per the feature's own Error-state expectation | The add request fails | User retries and the save succeeds, or navigates away |
| Offline/Degraded | Banner "You're offline -- this item will be saved when you reconnect." at the top; the add-item field and list remain usable; a submitted add is queued locally and shown in the list immediately as a pending row | Connectivity is lost while the screen is open, or the user adds an item while already offline | Connectivity restored -- the queued add syncs automatically (reconciled by FEAT-05.SPEC-003) and the pending marker clears |

## Validation Rules

Validation governed by FEAT-05.SPEC-004 (Pantry Item Field Validation). See that spec for the item name's required, 1-80 character rule. This screen applies validation on submit of the add-item field.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back/menu control | Main navigation destination the household last used | -- |
| (No other outbound navigation -- this screen has no detail view to navigate into per the Entity-Lifecycle Coverage Matrix) | -- | -- |

## Data Model

**Creates:** Pantry Item -- item_name (from the add-item field, validated by FEAT-05.SPEC-004), added_by (the signed-in household member), status set to Active. When the typed name matches an existing Active item, FEAT-05.SPEC-003 merges into that entry instead of creating a new one.
**Reads:** Pantry Item -- item_name, added_by, status (Active only), used_prompt (to render the "used it up?" chip when set by FEAT-05.SPEC-002) for every item belonging to the household.
**Updates:** Pantry Item -- status set to Used/Removed on a manual clear or a tapped "used it up?" prompt.
**Deletes:** Pantry Item -- hard-deleted immediately when status is set to Used/Removed (manual clear or "used it up?" tap); no restore path -- re-adding the same name creates a fresh entry.

## Business Rules

- FEAT-05.SPEC-004 (Pantry Item Field Validation) is the single source of truth for the item_name rule; this screen defers to it rather than re-deriving the character limit or required-field check.
- FEAT-05.SPEC-003 (Pantry Item Duplicate Merge) runs automatically whenever a submitted or synced item name matches an existing Active item; the user cannot bypass it.
- XBR-04: this screen is the surface where FEAT-06's inbound "already have it" tap creates or merges a Pantry Item, governed by FEAT-05.SPEC-007.
- Clearing an item (manually or via the "used it up?" prompt) is a hard delete with no restore path; an item that goes unused for weeks is never auto-removed by the product (see Non-Goals of the Feature Breakdown Brief).

## Edge Cases

- **User submits the add-item field twice rapidly** -- The second submit is ignored while the first save is in progress (the Add control shows its loading state).
- **User taps clear on a row while the "used it up?" chip is also present on it** -- Either control produces the same result: the item is set to Used/Removed and removed from the list. There is no double-clear.
- **Network failure during add** -- Error state per the States table: the typed item stays visible with a retry option, per the feature's own Error state; the item is not saved until retried successfully.
- **Household member clears an item that another household member is simultaneously clearing** -- The clear (hard delete) is idempotent: whichever request lands first removes the row; the second request finds no matching Active item and completes as a no-op, per the dependency map's Contention note for Pantry Item ("clearing is idempotent"). No error is shown to either member.
- **Another household member adds an item while this screen is open** -- The list is live-updating: the new row appears without the viewer needing to refresh.
- **Household member re-adds a name that was previously cleared** -- A fresh Active entry is created (per the Entity-Lifecycle Coverage Matrix: "re-adding the same name creates a fresh entry"), not a restore of the cleared row.
- **Item logged while offline is cleared before it syncs** -- The locally queued add and the local clear reconcile on reconnect so no item ever appears; nothing is sent to the household's shared list.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-05.SPEC-004 (Pantry Item Field Validation) | References (inbound) | Item-name validation rules applied to the add-item field |
| FEAT-05.SPEC-003 (Pantry Item Duplicate Merge) | Triggers (outbound) | Every add and every offline sync passes through the merge check |
| FEAT-05.SPEC-002 (Used-It-Up Prompt Trigger) | Triggers (inbound) | Surfaces the "used it up?" prompt chip shown on a row when the consuming dinner has passed |
| FEAT-05.SPEC-007 (Pantry Item Off-Grocery-List Exclusion Rule) | References (inbound) | Governs the inbound "already have it" create/merge path from the Shared Grocery List |
| FEAT-06 Shared Grocery List (grocery list screen) | Navigation (inbound) | User arrives here after tapping "already have it" on a grocery list line |
| FEAT-03 AI Weekly Dinner Plan Generation | References (outbound) | The weekly plan's pantry callout (FEAT-05.SPEC-006, gated by FEAT-05.SPEC-005) reflects Active items shown on this screen |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| pantry_item_added | added_by role, entry source (direct add / synced offline) | An item is successfully saved as a new Active entry | supports success-metrics.md: "Pantry Items Used" |
| pantry_item_cleared | clear source (manual / used-it-up prompt) | An item's status is set to Used/Removed and the row is hard-deleted | supports success-metrics.md: "Pantry Items Used" |
| pantry_used_prompt_answered | outcome (cleared / dismissed, if a dismiss path exists on this screen) | Household member acts on a "used it up?" prompt chip shown on this screen | supports success-metrics.md: "Pantry Items Used" |
| pantry_list_viewed_empty | N/A | The screen renders its Empty state | N/A -- this event measures reach into the empty-state experience, not a metric in success-metrics.md |

## Acceptance Criteria

**FEAT-05.SPEC-001-AC-01:** Given Maya is on the Pantry List screen with an empty pantry, when she types "spinach, feta" into the add-item field and taps Add, then a new row "spinach, feta" appears at the top of the list, showing Maya as the added-by member.

**FEAT-05.SPEC-001-AC-02:** Given Sam is on the Pantry List screen, when he submits the add-item field with no text entered, then the field shows an error state with the exact message from FEAT-05.SPEC-004 and no row is added.

**FEAT-05.SPEC-001-AC-03:** Given Maya is on the Pantry List screen with a pantry item "milk" already Active, when she taps the clear action on that row, then the row disappears immediately and the item's status is set to Used/Removed with no restore option.

**FEAT-05.SPEC-001-AC-04:** Given Sam is on the Pantry List screen and FEAT-05.SPEC-002 has surfaced a "used it up?" prompt chip on the "eggs" row, when he taps the chip, then the row disappears immediately and the item's status is set to Used/Removed.

**FEAT-05.SPEC-001-AC-05:** Given Maya has zero Active pantry items, when she opens the Pantry List screen, then the Empty state appears with a short prompt to add what's on hand, and the add-item field remains available.

**FEAT-05.SPEC-001-AC-06:** Given Sam is on the Pantry List screen with a slow connection, when he submits a new item and the save fails, then the typed item remains visible in the add-item field with a retry option and no row is added.

**FEAT-05.SPEC-001-AC-07:** Given Maya loses connectivity while on the Pantry List screen, when she adds "yoghurt", then the banner "You're offline -- this item will be saved when you reconnect." appears and "yoghurt" is shown in the list as a pending row.

**FEAT-05.SPEC-001-AC-08:** Given Maya's offline add from AC-07 is still pending, when connectivity returns, then the pending row syncs automatically (through FEAT-05.SPEC-003) and its pending marker clears without user action.

**FEAT-05.SPEC-001-AC-09:** Given Maya submits an item and taps Add twice in rapid succession, when the first save is still in progress, then the second tap has no effect and only one row is created.

**FEAT-05.SPEC-001-AC-10:** Given Maya and Sam both tap the clear action on the same "onions" row at effectively the same time, when both requests reach the system, then the row is removed once, and the request that lands second completes as a no-op with no error shown to either member.

**FEAT-05.SPEC-001-AC-11:** Given Sam has the Pantry List screen open, when Maya adds "carrots" from her own device, then the "carrots" row appears on Sam's screen without him needing to refresh.

**FEAT-05.SPEC-001-AC-12:** Given Jordan (young kid profile, no login) has no account, when any attempt is made to reach the Pantry List screen on Jordan's behalf, then no such access exists -- the profile has no sign-in and cannot open any screen.

**FEAT-05.SPEC-001-AC-13:** Given Jordan (older kid, limited login) is signed in, when this role looks for a "Pantry" entry in the main navigation, then it is not shown, and a direct link to the pantry screen redirects to the current plan (FEAT-03) with no error message.

**FEAT-05.SPEC-001-AC-14:** Given Riley (Operator) is viewing a household's pantry through an open Support Request, when Riley looks at the screen, then the add-item field, clear actions, and "used it up?" chip are not rendered -- the view is read-only.

**FEAT-05.SPEC-001-AC-15:** Given Maya's session expires while she has typed "flour" but not yet submitted it, when the session-expired dialog appears and she signs back in, then "flour" is still present in the add-item field.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 5 (empty, populated, loading, error, offline) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |

---
document_type: spec
spec_type: screen
spec_id: FEAT-01.SPEC-006
spec_name: Dietary Rules Editor
spec_slug: dietary-rules-editor
parent_feature: FEAT-01
parent_feature_name: Household Setup & Member Profiles
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Screen Spec: Dietary Rules Editor

## Overview

**Name:** Dietary Rules Editor
**ID:** FEAT-01.SPEC-006
**Type:** Screen
**Purpose:** The organiser records or edits one member's allergies, religious rules, vegetarian setting, and dislikes, and sees each rule's change history; other adult members view a kid profile's dietary rules.
**Parent Feature:** FEAT-01 -- Household Setup & Member Profiles

## Scope and Non-Goals

**In Scope:**
- Adding, editing, and viewing a member's Dietary Rule entries: allergies (from a standard list plus an optional named extra ingredient), religious rules, a per-person vegetarian setting, and dislikes
- Showing each rule's change history (who changed it and when)
- Removing a dislike directly, and routing an allergy or religious-rule removal through the explicit confirmation gate
- Read-only viewing of a kid profile's dietary rules for Sam

**Non-Goals:**
- Allergen classification logic, hard-vs-soft strength rules, and the removal confirmation gate itself -- governed by FEAT-01.SPEC-015 (Dietary Rule Classification & Allergen Matching Rules); this screen is the surface, not the rulebook
- Checking a rule against any recipe -- owned by the Dietary Rules & Allergy Safety Engine (FEAT-02), which reads this data
- Medical or diet advice of any kind -- excluded per scope-boundaries.md SC-06: this screen records rules for matching purposes only, never as nutritional guidance

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-005 (Member Profile Detail) | Organiser or Sam taps "View/Edit dietary rules" | The member whose rules are being viewed/edited |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen for every member | Add, edit, and remove any member's dietary rules, subject to the removal confirmation gate | -- |
| Sam (Other Adult Member) | Full screen, read-only, for any member's rules | View only -- no add/edit/remove controls | Attempting to interact with a rule shows it as non-interactive with the note "Only the organiser can change dietary rules" |
| Jordan (young kid profile, no login -- MVP) | No | No | N/A -- no login exists for this profile type |
| Jordan (older kid, limited login -- Later) | No | No | N/A -- dietary rule administration is not part of the older-kid login's entitlements |
| Riley (Operator, support) | View, only through FEAT-22 (from v1), and only the allergy details inside a specific open safety report -- never the full dietary-rule list at large (XBR-14) | No | Riley never reaches this screen directly; a safety report's allergy detail is surfaced only within the separate read-only support view |
| Unauthenticated | No | No | Redirected to FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." Unsaved rule entry is preserved locally per FEAT-01.SPEC-013 |

## Layout and Content

**Header:** Title "{display name}'s dietary rules," back arrow returning to FEAT-01.SPEC-005.

**Body:** Grouped list of the member's current rules, organized by kind: Allergies, Religious Rules, Vegetarian Setting, Dislikes. Each rule row shows its label (e.g., "Peanuts (allergy)"), a "History" link opening the change_history for that rule inline, and (organiser only) edit and remove controls. A single "Add a rule" button opens a rule-entry form below the list (or as an expanding panel):
- Rule kind selector (Allergy / Religious rule / Vegetarian setting / Dislike)
- For Allergy: a selector from the standard allergen list, plus an optional "Name a specific ingredient" text field
- For Religious rule and Dislike: a free-text label field
- For Vegetarian setting: a single toggle (this member is vegetarian) plus, if the household shares meals, a note that a vegetarian variant will be offered where relevant

For a kid profile, the same layout renders without edit controls for Sam, with the data-minimality note carried from FEAT-01.SPEC-005/007 restated at the top: "{display name}'s stored data: dietary rules only, alongside their name and age band."

**Footer:** None -- each rule saves independently as it is added or edited, rather than through a single form-wide Save.

### Responsive Behavior

- **Compact breakpoint:** Rule groups stack full width; the add-rule panel expands full width below the list.
- **Medium size class and above:** Rule groups render in a single column capped at the platform-wide form width, horizontally centered; the add-rule panel appears as an inline expansion rather than a separate overlay.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| "Add a rule" button (organiser only) | Tap | Opens the rule-entry panel | Panel expands | Standard expand animation |
| Rule kind selector | Select | Shows the fields relevant to that kind | Form fields change | Immediate |
| Allergen selector | Select | Sets the allergen from the standard list | Selector shows chosen allergen | Selected allergen displayed |
| "Name a specific ingredient" field | Type | Captures the named ingredient | Field shows entered text | Standard input focus state |
| "Save" (within the add-rule panel) | Tap | 1. Validate via FEAT-01.SPEC-015. 2. Create the Dietary Rule with origin set to organiser-entered. | Panel closes, list updates | New rule appears in its group with a change_history entry ("Added by Maya, {date}") |
| Rule row "History" link | Tap | Expands the change_history for that rule inline | Row expands | Shows a list of "{who} changed {what}, {when}" entries |
| Rule row "Edit" (organiser only) | Tap | Opens the rule for editing in the add-rule panel, pre-filled | Panel expands with current values | Standard expand animation |
| Rule row "Remove" (dislike, organiser only) | Tap | Removes the dislike immediately (hard delete) | Row disappears from the list | Toast: "{rule} removed" |
| Rule row "Remove" (allergy or religious rule, organiser only) | Tap | Opens the removal confirmation modal per FEAT-01.SPEC-015 | Modal appears | Modal states what will happen and requires an explicit affirmative tap before removal proceeds |

### Accessibility Notes

- **Focus order:** Rule groups in order (Allergies -> Religious Rules -> Vegetarian -> Dislikes) -> "Add a rule" -> (within the panel) Rule kind -> kind-specific fields -> Save.
- **Change announcements:** Adding, editing, or removing a rule is announced ("{rule} added" / "{rule} removed") so the list's live change is not silently missed.
- **History disclosure:** Expanding a rule's history is announced as an expansion, and its content is read in order.
- **Keyboard alternatives:** All actions are keyboard-reachable; no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (no rules yet) | Grouped headers with "No rules recorded yet" under each, "Add a rule" prominent | Member has no Dietary Rule entries | A rule is added |
| Populated | Grouped rule list with existing entries | One or more rules exist | Always the state once populated |
| Adding/Editing | Add-rule panel expanded with fields for the selected kind | User taps "Add a rule" or a rule's "Edit" | User saves or cancels the panel |
| Removal confirmation pending | Confirmation modal shown per FEAT-01.SPEC-015 | User taps "Remove" on an allergy or religious rule | User confirms (rule removed) or cancels (modal closes, rule unchanged) |
| Error | Inline error on the add-rule panel or a toast for a failed remove; entered data retained | A save or remove operation fails | User retries |
| Offline/Degraded | Banner: "You're offline -- rule changes will save when you reconnect." Add/edit remain usable and queue locally per FEAT-01.SPEC-013; removal of an allergy or religious rule is deferred until connectivity returns, since it requires the confirmation gate to complete against the current record | Connectivity lost while this screen is open | Connectivity restored -- queued changes save automatically |

## Validation Rules

Validation governed by FEAT-01.SPEC-015 (Dietary Rule Classification & Allergen Matching Rules). See that spec for allergen selection rules, hard/soft strength assignment, vegetarian logic, and the removal confirmation gate. Checked on save of each rule entry.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-01.SPEC-005 (Member Profile Detail) | -- |

## Data Model

**Creates:** Dietary Rule -- member (set to this screen's member), rule_kind, strength (derived per FEAT-01.SPEC-015), allergen (allergies only), origin set to "entered by the organiser," change_history initialized with the creation entry.
**Reads:** Dietary Rule -- all fields and change_history, for every rule belonging to this member.
**Updates:** Dietary Rule -- rule_kind-specific fields (allergen, named ingredient, strength where editable) on edit; change_history appended on every change.
**Deletes:** Dietary Rule -- dislikes removed directly; allergies and religious rules removed only after the FEAT-01.SPEC-015 confirmation gate. The change_history entry for a removed rule is retained per this feature's Data Notes (audit trail), even though the rule itself is gone.

## Business Rules

- A new or tightened hard rule (allergy or religious rule) triggers FEAT-01.SPEC-012 (Mid-Week Hard-Rule Change Trigger), which re-checks the current week's plan immediately.
- Every dietary rule recorded here is read by the Dietary Rules & Allergy Safety Engine (FEAT-02) against every candidate recipe's ingredients.
- Rule classification, strength, and the removal confirmation gate are governed entirely by FEAT-01.SPEC-015 -- this screen never duplicates that logic.
- Change history is visible to the organiser for every rule and is never deleted even when the rule itself is removed, supporting trust and any safety investigation.

## Edge Cases

- **Organiser attempts to save an allergy with no allergen selected** -- Blocked per FEAT-01.SPEC-015 with the error defined there; no rule is created.
- **Organiser removes a dislike, then immediately adds it back** -- Treated as two independent operations: a new Dietary Rule is created with a fresh change_history, not a restoration of the removed one (no restore path, per this feature's Entity-Lifecycle Coverage Matrix).
- **Organiser attempts to remove an allergy while offline** -- The removal confirmation cannot complete against the current record while offline; the "Remove" action shows "Removing an allergy needs a connection -- try again once you're back online" instead of opening the confirmation modal.
- **Two rules for the same allergen entered in quick succession** -- The second entry is treated as an edit target, not a duplicate: the add-rule panel surfaces the existing allergen entry for editing rather than creating a second Dietary Rule for the same allergen.
- **Concurrent edit from two devices (Maya on laptop and phone)** -- Rule creation and edits are last-write-wins per field, consistent with the dependency map's Contention note for Dietary Rule; an allergy can never be silently dropped by this path since removal always requires the explicit confirmation gate.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-005 (Member Profile Detail) | Navigation (inbound/outbound) | Entry and return |
| FEAT-01.SPEC-012 (Mid-Week Hard-Rule Change Trigger) | Triggers (outbound) | A new/tightened hard rule fires the mid-week re-check |
| FEAT-01.SPEC-013 (Setup Draft Persistence & Offline Queuing) | References (inbound) | Draft saving and offline queuing |
| FEAT-01.SPEC-015 (Dietary Rule Classification & Allergen Matching Rules) | References (inbound) | All classification, strength, and removal-gate logic |
| FEAT-01.SPEC-016 (Household Setup Authorization Rules) | References (inbound) | Role-gated view/edit access |
| FEAT-02 (Dietary Rules & Allergy Safety Engine) | Affects (outbound) | Every rule recorded here is read by the safety engine |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| dietary_rule_added | rule kind (allergy / religious / vegetarian / dislike) | A new rule is saved | supports success-metrics.md: "First-Session Onboarding Completion" (dietary rules are part of the setup this metric measures completing within the first session) |
| dietary_rule_changed | rule kind, change type (edit / remove) | An existing rule is edited or removed | N/A -- occurs after first-session setup in most cases; retained as an operational signal, not tied to a Stage 2 metric for this feature |

## Acceptance Criteria

**FEAT-01.SPEC-006-AC-01:** Given Maya is on Jordan's Dietary Rules Editor with no rules yet, when she taps "Add a rule", selects "Allergy," chooses "Peanuts" from the standard list, and saves, then a new hard allergy rule appears under Allergies with a change_history entry "Added by Maya, {today's date}".

**FEAT-01.SPEC-006-AC-02:** Given Maya adds an allergy with no allergen selected, when she taps Save, then the rule is not created and the error defined by FEAT-01.SPEC-015 is shown.

**FEAT-01.SPEC-006-AC-03:** Given Maya taps "Remove" on a dislike, then the dislike is removed immediately with the toast "{rule} removed" and no confirmation step.

**FEAT-01.SPEC-006-AC-04:** Given Maya taps "Remove" on an existing peanut allergy, then the removal confirmation modal from FEAT-01.SPEC-015 appears and the allergy is not removed until she taps the explicit affirmative action.

**FEAT-01.SPEC-006-AC-05:** Given Sam opens Jordan's dietary rules, when the screen loads, then every rule renders read-only with no edit or remove controls.

**FEAT-01.SPEC-006-AC-06:** Given Maya taps "History" on an existing rule, then the rule's change_history expands inline showing who changed it and when.

**FEAT-01.SPEC-006-AC-07:** Given Maya adds a new hard allergy rule for Jordan, then FEAT-01.SPEC-012 (Mid-Week Hard-Rule Change Trigger) fires to re-check the current week's plan.

**FEAT-01.SPEC-006-AC-08:** Given Maya is offline and taps "Remove" on an allergy, then she sees "Removing an allergy needs a connection -- try again once you're back online" and no confirmation modal opens.

**FEAT-01.SPEC-006-AC-09:** Given Maya adds a dislike while offline, then the entry is queued locally per FEAT-01.SPEC-013 and saves automatically once connectivity returns.

**FEAT-01.SPEC-006-AC-10:** Given Maya sets the vegetarian toggle for a member on, when she saves, then the vegetarian rule is recorded for that member and the household is told a vegetarian variant will be offered on shared meals.

**FEAT-01.SPEC-006-AC-11:** Given Maya is signed in on two devices and edits the same member's dislike list from both within moments, then the changes apply last-write-wins per field, and no allergy is ever silently dropped by this path.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 9 | 9 |
| States | 6 (empty, populated, adding/editing, removal confirmation, error, offline) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |

---
document_type: spec
spec_type: screen
spec_id: FEAT-01.SPEC-002
spec_name: Create Project
spec_slug: create-project
parent_feature: FEAT-01
parent_feature_name: Client & Project Management
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 8
---

# Screen Spec: Create Project

## Overview

**Name:** Create Project
**ID:** FEAT-01.SPEC-002
**Type:** Screen
**Purpose:** Nadia creates a project under a specific client, entering the roster at stage "Draft."
**Parent Feature:** FEAT-01 -- Client & Project Management

## Scope and Non-Goals

**In Scope:**
- Capturing a new Project record under exactly one client
- Setting the project's initial derived stage to "Draft" via FEAT-01.SPEC-011
- Preserving entered data and offering retry on a failed save
- Handling composing this screen while offline

**Non-Goals:**
- Drafting the project's proposal -- handled by Proposal Creation & Sending (FEAT-02), reached from Project Detail (FEAT-01.SPEC-005) after the project exists
- Setting milestones or a payment schedule -- handled by Milestone & Payment Schedule Setup (FEAT-04)
- Choosing the client the project belongs to from a client that does not yet exist -- if no client exists, this screen is not reachable; Nadia must first complete FEAT-01.SPEC-001 (Add Client)
- Bulk import of historical projects -- excluded per scope-boundaries.md SC-19: with 3-15 active clients and a handful of projects each, hand entry is fast, and imported records would carry no evidence trail through this feature's own creation flow

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-003 (Client & Project Roster) | Nadia taps "New Project" under a specific client | The owning client is pre-selected and shown, not editable |
| FEAT-01.SPEC-004 (Client Detail) | Nadia taps "New Project" from the client's own detail view | The owning client is pre-selected and shown, not editable |
| FEAT-20 (Onboarding / First-Run Setup) | Onboarding step "Add first client and project" continues here after the client is created | The just-created client is pre-selected |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Fill and submit the form | -- |
| Owen (Client Primary Contact) | No | No | Not shown in Owen's portal navigation; project creation is a freelancer-only action |
| Priya (Client Reviewer Contact) | No | No | Not shown in Priya's portal navigation |
| Dana (Support Operator) | No | No | Dana's read-only support session (FEAT-31) covers the roster, Client Detail, and Project Detail only; the Create Project form is not part of the session's viewable surface |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in, the user lands on the Client & Project Roster (FEAT-01.SPEC-003), not this form |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- entered form data is preserved locally and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "New Project" with a back arrow (returns to the screen the user came from) and a "Save" action button (right-aligned).

**Body:** A single-column form with the following fields, in order:
- Client (read-only display of the pre-selected client's name; not editable on this screen)
- Project Name (text input, required)

**Footer:** None -- Save is in the header.

### Responsive Behavior

- **Compact size class:** Single-column form as described, full width; Save remains in the header.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to the entry screen (roster or Client Detail) | Screen closes; unsaved-changes check runs first | If the project name is filled, a confirmation dialog appears before leaving |
| Project Name input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Project Name input | Blur (empty) | Triggers required-field validation | Error state on field | "Project name is required" below field |
| Save button | Tap | 1. Validate the project name. 2. Save the Project record under the pre-selected client. 3. Set the initial stage to "Draft" via FEAT-01.SPEC-011. | Button shows loading state during save | Success: toast "Project created" and navigate to FEAT-01.SPEC-005 (Project Detail) for the new project. Failure: inline error banner. |
| Save button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> Project Name -> Save. (Client display is read-only and not part of the interactive tab order beyond being announced on screen entry.)
- **Validation announcements:** When the Project Name field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Save feedback:** The "Project created" toast is announced on success; on validation failure, focus moves to the Project Name field.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (default) | Project Name empty, client pre-filled, Save enabled | Screen first opens | User begins typing |
| Filling | Project Name contains user input | User types | User taps Save or navigates away |
| Saving | Save button shows loading spinner, form disabled | Validation passes | Save completes or fails |
| Validation Error | Project Name field highlighted with its error message | Required-field validation fails on submit | User corrects the field and re-submits |
| Error | Error banner at top of form with retry option; entered data preserved | Save operation fails | User taps Retry or navigates away |
| Offline/Degraded | Banner "You're offline. Connect to create a project." at top; the field remains editable but Save is disabled until connectivity returns | Connectivity lost while screen is open | Connectivity restored -- Save re-enables; if Nadia had already tapped Save while offline, the attempt automatically retries and standard success feedback appears |

## Validation Rules

**Option B -- Inline (simple validation not warranting a standalone spec):**

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| Project Name | Required, non-empty | On blur and on submit | "Project name is required" |
| Client | Must reference exactly one existing, non-archived client (pre-selected, not user-editable on this screen) | On screen entry | N/A -- the screen is not reachable without a valid client context |

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap (no unsaved changes) | Roster or Client Detail (whichever the user arrived from) | -- |
| Successful save | FEAT-01.SPEC-005 (Project Detail) | -- |
| Cancel with unsaved changes, "Discard" chosen | Roster or Client Detail (whichever the user arrived from) | -- |

## Data Model

**Creates:** Project record -- project_name (from input), client (the pre-selected client, exactly one), stage (initial value "Draft," derived via FEAT-01.SPEC-011), currency/tax_label/tax_rate unset (set later via FEAT-15), completed_at and cancelled_at unset.
**Reads:** The pre-selected Client record's name and active status (an archived client cannot receive a new project; the screen is not reachable from an archived client's context).
**Updates:** None.
**Deletes:** None.

## Business Rules

- Every project belongs to exactly one client; this cannot be changed after creation on this screen (moving a project to a different client is not a capability this product defines).
- A new project's initial stage is always "Draft," derived by FEAT-01.SPEC-011 rather than set directly.
- A project cannot be created under an archived client; Create Project is not reachable from an archived client's context.

## Edge Cases

- **Nadia navigates away with an unsaved project name** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Nadia taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in loading state).
- **Network failure during save** -- Error banner: "Could not create this project. Check your connection and try again." with a Retry button. Form data preserved.
- **The owning client is archived or deleted by another session between screen open and save** -- Save is rejected with "This client is no longer active. Return to the roster and try again." and the user is returned to FEAT-01.SPEC-003; this is a creation-only screen with no existing Project record loaded, so the conflict is on the parent Client reference, not a Project update conflict.
- **Two projects created under the same client with the same name** -- No rule prevents this; each save creates an independent Project record and both appear separately under the client.
- **Nadia composes this form while offline** -- Save is disabled with a connectivity notice; if she had already submitted just as connectivity dropped, the attempt is queued and retries automatically once connectivity returns, with no silent failure and no duplicate project created.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-003 (Client & Project Roster) | Navigation (inbound/outbound) | Entry point when arriving from "New Project" on the roster; destination on cancel |
| FEAT-01.SPEC-004 (Client Detail) | Navigation (inbound/outbound) | Entry point when arriving from a client's own detail view; destination on cancel |
| FEAT-01.SPEC-005 (Project Detail) | Navigation (outbound) | Destination after a successful save |
| FEAT-01.SPEC-011 (Project Stage Derivation) | Triggers (outbound) | Sets the project's initial stage to "Draft" |
| FEAT-20 (Onboarding / First-Run Setup) | Navigation (inbound) | Onboarding's "add first client and project" step continues here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| project_create_started | entry source (roster / client detail / onboarding) | Screen opens | supports success-metrics.md: "Client and Project Setup Speed" |
| project_created | time from opening "add client" (FEAT-01.SPEC-001) to this project appearing in the roster | Save completes successfully | supports success-metrics.md: "Client and Project Setup Speed" |

## Acceptance Criteria

**FEAT-01.SPEC-002-AC-01:** Given Nadia is on the Create Project screen for client "Acme Studio," when she enters "Website Redesign" as project name and taps Save, then the project is saved with stage "Draft," a "Project created" toast appears, and she is taken to the new project's Project Detail (FEAT-01.SPEC-005).

**FEAT-01.SPEC-002-AC-02:** Given Nadia is on the Create Project screen, when she taps Save with the project name field empty, then the project name field shows the error "Project name is required" and the save does not proceed.

**FEAT-01.SPEC-002-AC-03:** Given Nadia is on the Create Project screen with an unsaved project name, when she taps the back arrow, then a confirmation dialog appears asking "You have unsaved changes. Discard?"

**FEAT-01.SPEC-002-AC-04:** Given Nadia loses connectivity while filling the form, then the "You're offline. Connect to create a project." banner appears and Save is disabled until connectivity returns.

**FEAT-01.SPEC-002-AC-05:** Given Nadia's connectivity returns after a queued Save made while offline, then the project is saved automatically and the standard "Project created" success feedback appears with no duplicate project created.

**FEAT-01.SPEC-002-AC-06:** Given a save attempt fails due to a network error, then an error banner "Could not create this project. Check your connection and try again." appears with a Retry button, and the entered project name is preserved.

**FEAT-01.SPEC-002-AC-07:** Given the owning client was archived by another session while Nadia had this screen open, when she taps Save, then the save is rejected with "This client is no longer active. Return to the roster and try again." and she is returned to the roster.

**FEAT-01.SPEC-002-AC-08:** Given Dana is in a read-only support session on Nadia's account, when she looks for a way to create a project, then no such control or screen is reachable from her session.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 6 | 6 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |

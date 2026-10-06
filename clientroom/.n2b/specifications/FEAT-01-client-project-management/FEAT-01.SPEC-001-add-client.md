---
document_type: spec
spec_type: screen
spec_id: FEAT-01.SPEC-001
spec_name: Add Client
spec_slug: add-client
parent_feature: FEAT-01
parent_feature_name: Client & Project Management
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Screen Spec: Add Client

## Overview

**Name:** Add Client
**ID:** FEAT-01.SPEC-001
**Type:** Screen
**Purpose:** Nadia records a new client company she works with, so a project can be created under it.
**Parent Feature:** FEAT-01 -- Client & Project Management

## Scope and Non-Goals

**In Scope:**
- Capturing a new Client record: client name, and optionally at this point, billing name, billing address, and tax ID
- Enforcing the active-client limit (FEAT-01.SPEC-008) before the client is saved
- Preserving entered data and offering retry on a failed save
- Handling composing this screen while offline

**Non-Goals:**
- Creating a project under the new client -- handled by FEAT-01.SPEC-002 (Create Project), reached from the roster after this save
- Setting the client's currency and tax treatment -- owned by Currency & Tax Handling (FEAT-15), set at the project level before the first invoice, not on this screen
- Bulk import of clients from another tool -- excluded per scope-boundaries.md SC-19: a freelancer has 3-15 active clients, so adding them by hand takes minutes, and imported records would carry no evidence trail through this feature's own creation flow
- Adding client contacts (Primary/Reviewer people) -- handled by Client Contact Management & Roles (FEAT-18), reached from Client Detail (FEAT-01.SPEC-004) after the client exists

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-003 (Client & Project Roster) | Nadia taps "Add Client" | None -- form starts empty |
| FEAT-20 (Onboarding / First-Run Setup) | Onboarding step "Add first client and project" lands here | None -- form starts empty |
| FEAT-23 (Subscription Plan & Billing Management), after upgrade | Nadia completes an upgrade prompted by a blocked add | The client name and any other fields she had entered before being blocked are restored |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Fill and submit the form | -- |
| Owen (Client Primary Contact) | No | No | This is a freelancer-only management surface; the screen does not exist in Owen's portal navigation |
| Priya (Client Reviewer Contact) | No | No | Not shown in Priya's portal navigation |
| Dana (Support Operator) | No | No | Dana's read-only support session (FEAT-31) covers the roster, Client Detail, and Project Detail only; the Add Client form is not part of the session's viewable surface, and attempting to reach it directly returns Dana to the session's roster view |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in, the user lands on the Client & Project Roster (FEAT-01.SPEC-003), not this form |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- entered form data is preserved locally and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Add Client" with a back arrow (returns to the Client & Project Roster) and a "Save" action button (right-aligned).

**Body:** A single-column form with the following fields, in order:
- Client Name (text input, required)
- Billing Name (text input, optional at this point -- required later before this client's first invoice can be sent, per FEAT-01.SPEC-010)
- Billing Address (multi-line text input, optional at this point -- required before the first invoice, per FEAT-01.SPEC-010)
- Tax ID (text input, always optional)

A note below the billing fields states: "Billing details can be added now or later, but are required before you send this client's first invoice."

**Footer:** None -- Save is in the header.

### Responsive Behavior

- **Compact size class:** Single-column form as described, full width; Save remains in the header.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.
- **Billing Address field:** Grows from 2 visible lines (compact) to 3 visible lines (medium and above).

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-01.SPEC-003 (Client & Project Roster) | Screen closes; unsaved-changes check runs first | If fields are filled, a confirmation dialog appears before leaving |
| Client Name input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Client Name input | Blur (empty) | Triggers required-field validation | Error state on field | "Client name is required" below field |
| Billing Name / Billing Address / Tax ID inputs | Type | Captures text input | Field shows entered text | Standard input focus state |
| Save button | Tap | 1. Validate required fields. 2. If valid, evaluate the active-client limit via FEAT-01.SPEC-008. 3. If within limit, save the Client record. | Button shows loading state during save | Success: toast "Client added" and navigate to FEAT-01.SPEC-003 with the new client visible. Blocked: see Edge Cases. Failure: inline error banner. |
| Save button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> Client Name -> Billing Name -> Billing Address -> Tax ID -> Save.
- **Validation announcements:** When a field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Save feedback:** The "Client added" toast is announced on success; on validation failure, focus moves to the first field in error; on a limit block, focus moves to the blocking message.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (default) | All fields empty, Save enabled | Screen first opens | User begins typing in any field |
| Filling | Fields contain user input | User types in any field | User taps Save or navigates away |
| Saving | Save button shows loading spinner, form disabled | Validation passes and limit check runs | Save completes, is blocked, or fails |
| Validation Error | Client Name field highlighted with its error message | Required-field validation fails on submit | User corrects the field and re-submits |
| Limit Blocked | Blocking message in place of the form's success path, with an "Upgrade" action; entered data preserved | Active-client limit check (FEAT-01.SPEC-008) returns blocked | Nadia upgrades (cross-feature FEAT-23) and returns, or navigates away |
| Error | Error banner at top of form with retry option; entered data preserved | Save operation fails for a reason other than the limit | User taps Retry or navigates away |
| Offline/Degraded | Banner "You're offline. Connect to add a client." at top; fields remain editable but Save is disabled until connectivity returns | Connectivity lost while screen is open | Connectivity restored -- Save re-enables; if Nadia had already tapped Save while offline, the attempt automatically retries and standard success feedback appears |

## Validation Rules

Validation governed by FEAT-01.SPEC-010 (Client Billing Completeness Gate) for the billing fields' eventual completeness requirement, and inline below for this screen's own fields. This screen applies validation on field blur and on form submit.

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| Client Name | Required, non-empty | On blur and on submit | "Client name is required" |
| Billing Name, Billing Address, Tax ID | None on this screen -- completeness is enforced later, at first-invoice send, by FEAT-01.SPEC-010 | -- | -- |

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap (no unsaved changes) | FEAT-01.SPEC-003 (Client & Project Roster) | -- |
| Successful save | FEAT-01.SPEC-003 (Client & Project Roster) | -- |
| Limit-blocked "Upgrade" action | Upgrade / subscribe flow | FEAT-23 (Subscription Plan & Billing Management) |
| Cancel with unsaved changes, "Discard" chosen | FEAT-01.SPEC-003 (Client & Project Roster) | -- |

## Data Model

**Creates:** Client record -- client_name (from input), billing_name, billing_address, tax_id (from input, may be left empty), status set to Active. currency and tax treatment are not set here (set later via FEAT-15, before the first invoice).
**Reads:** None (this is a creation screen -- no existing Client data loaded). Reads the freelancer's current active-client count and Subscription Plan tier only to evaluate FEAT-01.SPEC-008 at save time.
**Updates:** None.
**Deletes:** None.

## Business Rules

- The active-client limit (FEAT-01.SPEC-008) is evaluated on every save attempt; the user cannot bypass it (XBR-23).
- Billing completeness (FEAT-01.SPEC-010) is not enforced on this screen -- it is enforced at the moment the client's first invoice is sent, from Client Detail (FEAT-01.SPEC-004) or Invoicing (FEAT-09).
- A newly created client's status defaults to Active and counts toward the active-client limit immediately.

## Edge Cases

- **Nadia navigates away with unsaved changes** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Nadia taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in loading state).
- **Network failure during save** -- Error banner: "Could not add this client. Check your connection and try again." with a Retry button. Form data preserved.
- **Save blocked by the active-client limit (FEAT-01.SPEC-008)** -- The form's entered data is preserved; a blocking message replaces the success path: "You've reached your plan's active client limit. Upgrade to add more clients." with an "Upgrade" action that opens FEAT-23; "Keep Editing" returns to the form with data intact.
- **Two client records created with the same name** -- No rule prevents this; each save creates an independent Client record and both appear separately in the roster (FEAT-01.SPEC-003). This is a creation-only screen with no existing record loaded, so no update conflict is possible here.
- **Nadia composes this form while offline** -- Save is disabled with a connectivity notice; if she had already submitted just as connectivity dropped, the attempt is queued and retries automatically once connectivity returns, with no silent failure and no duplicate client created.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-003 (Client & Project Roster) | Navigation (inbound/outbound) | Entry point and destination after save or cancel |
| FEAT-01.SPEC-008 (Active Client Limit Enforcement) | References (outbound) | Save evaluates the active-client limit before committing |
| FEAT-01.SPEC-010 (Client Billing Completeness Gate) | References (outbound) | Governs when billing fields become required (not on this screen) |
| FEAT-23 (Subscription Plan & Billing Management) | Navigation (outbound) | Upgrade action when the limit blocks the add |
| FEAT-20 (Onboarding / First-Run Setup) | Navigation (inbound) | Onboarding's "add first client" step lands here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| client_add_started | entry source (roster / onboarding / post-upgrade) | Screen opens | supports success-metrics.md: "Client and Project Setup Speed" |
| client_created | billing details provided at creation (yes/no), time from client_add_started to save | Save completes successfully | supports success-metrics.md: "Client and Project Setup Speed" |
| client_add_blocked_by_limit | current active-client count, plan tier | Save is blocked by FEAT-01.SPEC-008 | supports success-metrics.md: "Client and Project Setup Speed" (a blocked add is a delay in the same setup flow this metric tracks) |

## Acceptance Criteria

**FEAT-01.SPEC-001-AC-01:** Given Nadia is on the Add Client screen, when she enters "Acme Studio" as client name and taps Save, then the active-client limit is checked, the client is saved, a "Client added" toast appears, and she returns to the roster with the new client visible.

**FEAT-01.SPEC-001-AC-02:** Given Nadia is on the Add Client screen, when she taps Save with the client name field empty, then the client name field shows the error "Client name is required" and the save does not proceed.

**FEAT-01.SPEC-001-AC-03:** Given Nadia is on the Add Client screen with unsaved input, when she taps the back arrow, then a confirmation dialog appears asking "You have unsaved changes. Discard?"

**FEAT-01.SPEC-001-AC-04:** Given Nadia is at her plan's active-client limit, when she submits a valid client name and taps Save, then the save is blocked with the message "You've reached your plan's active client limit. Upgrade to add more clients." and her entered data remains in the form.

**FEAT-01.SPEC-001-AC-05:** Given Nadia loses connectivity while filling the form, then the "You're offline. Connect to add a client." banner appears and Save is disabled until connectivity returns.

**FEAT-01.SPEC-001-AC-06:** Given Nadia's connectivity returns after a queued Save attempt made while offline, then the client is saved automatically and the standard "Client added" success feedback appears with no duplicate client created.

**FEAT-01.SPEC-001-AC-07:** Given Nadia leaves Billing Name and Billing Address empty and taps Save, when the client name is valid, then the client saves successfully with billing fields empty, since billing completeness (FEAT-01.SPEC-010) is not required on this screen.

**FEAT-01.SPEC-001-AC-08:** Given a save attempt fails due to a network error, then an error banner "Could not add this client. Check your connection and try again." appears with a Retry button, and all entered data is preserved.

**FEAT-01.SPEC-001-AC-09:** Given Dana is in a read-only support session on Nadia's account, when she looks for a way to add a client, then no such control or screen is reachable from her session.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 7 | 7 |
| Business Rules | 3 | 3 |
| Edge Cases | 6 | 6 |

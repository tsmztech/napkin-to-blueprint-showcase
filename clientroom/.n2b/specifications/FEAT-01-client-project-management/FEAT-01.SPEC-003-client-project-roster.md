---
document_type: spec
spec_type: screen
spec_id: FEAT-01.SPEC-003
spec_name: Client & Project Roster
spec_slug: client-project-roster
parent_feature: FEAT-01
parent_feature_name: Client & Project Management
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Screen Spec: Client & Project Roster

## Overview

**Name:** Client & Project Roster
**ID:** FEAT-01.SPEC-003
**Type:** Screen
**Purpose:** Nadia (and, read-only, Dana in a support session) sees every client and project she manages in one place, each showing its current stage.
**Parent Feature:** FEAT-01 -- Client & Project Management

## Scope and Non-Goals

**In Scope:**
- Listing every Client and its Projects, with each project's derived stage
- Entry points into Add Client, Create Project, Client Detail, and Project Detail
- Filtering by Active/Archived status
- Empty, loading, error, and offline states for a list of this size (3-15 clients per freelancer)

**Non-Goals:**
- Editing a client or project inline from the roster -- handled by FEAT-01.SPEC-004 (Client Detail) and FEAT-01.SPEC-005 (Project Detail)
- Free-text search across clients and projects -- handled by Global Search (FEAT-28, v1), a distinct feature the roster does not duplicate
- Sorting or grouping beyond client-then-project order -- excluded per scope-boundaries.md's positioning against heavy PM tooling (SC-09): with 3-15 clients, a fixed, simple structure serves the stated scale without configurable views
- Displaying financial totals per client or project -- owned by the Freelancer Financial Dashboard (FEAT-12), which the roster links to but does not duplicate

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| Main navigation | Nadia opens the Client & Project Management area (this is the feature's default entry screen) | None |
| FEAT-20 (Onboarding / First-Run Setup) | Onboarding completes | None -- lands here with the first client and project visible |
| FEAT-31 (Operator Support Access) | Dana opens a logged, read-only support session on Nadia's account | Session is scoped to this one freelancer account, read-only |
| FEAT-28 (Global Search, v1) | Nadia selects a client or project search result | The matching client or project is highlighted/scrolled into view |
| FEAT-01.SPEC-001 / FEAT-01.SPEC-002 / FEAT-01.SPEC-004 / FEAT-01.SPEC-005 | Successful save, archive, or delete on any of those screens | The affected client/project's row reflects its new state |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen, all her clients and projects | Add Client, New Project, open any client or project, filter Active/Archived | -- |
| Owen (Client Primary Contact) | No | No | Not shown in Owen's portal navigation; this is the freelancer's own management surface, not the client portal (FEAT-05) |
| Priya (Client Reviewer Contact) | No | No | Not shown in Priya's portal navigation |
| Dana (Support Operator) | Full screen, read-only, for the one account under an open support session | View only -- Add Client, New Project, and any edit affordances are not rendered | Attempting to reach an add/create action directly (e.g., a stale link) returns Dana to this roster with no action taken |
| Unauthenticated | No | No | Redirected to the sign-in screen |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- the roster reloads fresh after re-authentication (no in-progress state to preserve on a list screen) |

## Layout and Content

**Header:** Screen title "Clients & Projects" with an "Add Client" action button (top right). A filter control (Active / Archived / All, defaulting to Active) sits below the title.

**Body:** A list grouped by client. Each client group shows:
- Client identity block: client name plus a status badge (Active/Archived)
- A "New Project" action for that client (hidden for Dana's read-only session and for an archived client)
- Below the client row, one row per project belonging to that client: project name plus a stage badge (Draft, In Progress, Complete, Cancelled, Archived), using the same identity-block presentation as Client Detail (FEAT-01.SPEC-004) and Project Detail (FEAT-01.SPEC-005)

A client with no projects yet shows a single line under its group: "No projects yet."

**Footer:** None.

### Responsive Behavior

- **Compact size class:** Client groups and project rows stack in a single column, full width; the "New Project" action for each client collapses to an icon button.
- **Medium size class and above:** The same single-column grouped list is retained (no multi-column restructure -- the product's scale of 3-15 clients does not warrant it), capped at a consistent platform-wide content width and horizontally centered.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| "Add Client" button | Tap | Navigate to FEAT-01.SPEC-001 (Add Client) | Screen transitions | Standard navigation transition |
| Filter control | Select Active / Archived / All | Re-fetches and re-renders the list scoped to the chosen status | List content updates | Selected filter option highlighted |
| Client row (name) | Tap | Navigate to FEAT-01.SPEC-004 (Client Detail) for that client | Screen transitions | Standard navigation transition |
| "New Project" (per client) | Tap | Navigate to FEAT-01.SPEC-002 (Create Project) pre-scoped to that client | Screen transitions | Standard navigation transition |
| Project row | Tap | Navigate to FEAT-01.SPEC-005 (Project Detail) for that project | Screen transitions | Standard navigation transition |

### Accessibility Notes

- **Focus order:** Add Client -> Filter control -> each client group in list order (client name, New Project action, then its project rows in order).
- **Dynamic updates:** When the filter changes the list content, the updated result count is announced to assistive technology (e.g., "12 active clients shown").
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty | A prompt: "No clients yet -- add your first one to get started," with an "Add Client" action | Nadia has zero clients under the current filter | A client is added, or the filter is changed to a scope with results |
| Loading | Skeleton rows for client groups and project rows | Screen opens or filter changes, while data is being fetched | Data finishes loading |
| Loaded | Full list as described in Layout and Content | Data load completes with one or more results | Filter changes or a mutating action occurs elsewhere |
| Error | Error banner: "Couldn't load your clients and projects. Try again." with a Retry button | The list fails to load | Retry succeeds |
| Offline/Degraded | A connectivity banner appears at the top: "You're offline -- showing the last loaded list." The most recently loaded list remains visible and browsable; Add Client and New Project are disabled | Connectivity is lost while viewing an already-loaded roster | Connectivity restored -- the banner clears and the list refreshes |

## Validation Rules

Not applicable -- this is a read-only listing screen with no user input beyond the filter selection, which has no invalid states (a closed set of options).

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| "Add Client" tap | FEAT-01.SPEC-001 (Add Client) | -- |
| "New Project" tap (per client) | FEAT-01.SPEC-002 (Create Project) | -- |
| Client row tap | FEAT-01.SPEC-004 (Client Detail) | -- |
| Project row tap | FEAT-01.SPEC-005 (Project Detail) | -- |

## Data Model

**Creates:** None.
**Reads:** Client records -- client_name, status (Active/Archived), for every client belonging to the freelancer account (or, for Dana, the one account under her support session). Project records -- project_name, stage (derived by FEAT-01.SPEC-011), for every project belonging to each listed client.
**Updates:** None.
**Deletes:** None.

## Business Rules

- The roster's default filter is Active; Archived clients and projects are hidden unless the Archived or All filter is selected, per the dependency map's Client and Project lifecycle notes.
- A project's stage badge always reflects the value computed by FEAT-01.SPEC-011, never a value set directly by this screen.
- Dana's support session (FEAT-31) renders this screen with every edit affordance removed, per XBR-29.

## Edge Cases

- **Roster loads a large client list** -- Skeleton rows render during load rather than a blank screen, satisfying the responsiveness expectation for the product's stated scale (3-15 active clients per freelancer).
- **A client or project changes state while the roster is open (e.g., archived from another open session)** -- The roster is a snapshot per load, not live-updating; the changed row is stale until the next load or filter change, which is acceptable because this screen only reads shared entities and never writes to them, so no write conflict is possible here (any conflict is resolved at the point of the write, in FEAT-01.SPEC-004 or FEAT-01.SPEC-005).
- **A client has projects in every stage** -- All are shown; there is no cap on project rows per client at this product's scale.
- **Filter selected with zero matching results (e.g., Archived with nothing archived yet)** -- Shows "No archived clients or projects" rather than the general empty-state prompt to add a first client.
- **Nadia navigates away and returns** -- The roster re-fetches on return rather than showing a stale cached view, so recent changes from Client Detail or Project Detail are reflected.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-001 (Add Client) | Navigation (outbound) | "Add Client" action |
| FEAT-01.SPEC-002 (Create Project) | Navigation (outbound) | "New Project" action per client |
| FEAT-01.SPEC-004 (Client Detail) | Navigation (outbound) | Selecting a client row |
| FEAT-01.SPEC-005 (Project Detail) | Navigation (outbound) | Selecting a project row |
| FEAT-01.SPEC-011 (Project Stage Derivation) | References (inbound) | Supplies the stage badge shown per project |
| FEAT-28 (Global Search Across Clients & Projects) | Navigation (inbound) | Search result selection lands here with the match highlighted |
| FEAT-31 (Operator Support Access) | Navigation (inbound) | Dana's read-only session entry point |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| roster_viewed | filter selected, result count, viewer role (Nadia / Dana) | Screen loads successfully | supports success-metrics.md: "Client and Project Setup Speed" (the roster is the point at which a newly added client and project become visible, closing the loop this metric measures) |
| roster_filter_changed | filter selected | Nadia changes the Active/Archived/All filter | N/A -- no success-metrics.md metric measures roster filter usage for this feature; recorded for product-usage visibility only |

## Acceptance Criteria

**FEAT-01.SPEC-003-AC-01:** Given Nadia has no clients yet, when she opens the Client & Project Roster, then she sees "No clients yet -- add your first one to get started" with an "Add Client" action.

**FEAT-01.SPEC-003-AC-02:** Given Nadia has clients and projects, when the roster loads, then each project shows a stage badge computed by FEAT-01.SPEC-011 (Draft, In Progress, Complete, Cancelled, or Archived).

**FEAT-01.SPEC-003-AC-03:** Given Nadia is on the roster, when she taps "Add Client," then she is taken to FEAT-01.SPEC-001 (Add Client).

**FEAT-01.SPEC-003-AC-04:** Given Nadia is on the roster, when she taps "New Project" under client "Acme Studio," then she is taken to FEAT-01.SPEC-002 (Create Project) pre-scoped to Acme Studio.

**FEAT-01.SPEC-003-AC-05:** Given Nadia is on the roster with the default Active filter, when she switches the filter to Archived, then only archived clients and their projects are shown, and the change is announced to assistive technology.

**FEAT-01.SPEC-003-AC-06:** Given Nadia's roster has more clients than fit in the first render, when it loads, then skeleton rows appear while data is fetched rather than a blank screen.

**FEAT-01.SPEC-003-AC-07:** Given Nadia loses connectivity while viewing an already-loaded roster, then a "You're offline -- showing the last loaded list." banner appears, the last list remains browsable, and Add Client / New Project become disabled.

**FEAT-01.SPEC-003-AC-08:** Given the roster fails to load, then an error banner "Couldn't load your clients and projects. Try again." appears with a Retry button.

**FEAT-01.SPEC-003-AC-09:** Given Dana opens a logged support session on Nadia's account, when she views the roster, then she sees every client and project read-only, with no Add Client or New Project controls rendered.

**FEAT-01.SPEC-003-AC-10:** Given Nadia selects a client or project from Global Search results (FEAT-28), when the roster opens, then the matching row is highlighted and scrolled into view.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 5 | 5 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |

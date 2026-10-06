---
document_type: spec
spec_type: screen
spec_id: FEAT-02.SPEC-004
spec_name: Reuse Proposal Picker
spec_slug: reuse-proposal-picker
parent_feature: FEAT-02
parent_feature_name: Proposal Creation & Sending
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Screen Spec: Reuse Proposal Picker

## Overview

**Name:** Reuse Proposal Picker
**ID:** FEAT-02.SPEC-004
**Type:** Screen
**Purpose:** Nadia browses her earlier proposals across all clients and projects and selects one to start a new draft as a copy.
**Parent Feature:** FEAT-02 -- Proposal Creation & Sending

## Scope and Non-Goals

**In Scope:**
- Listing Nadia's earlier proposals across every client and project as copy sources
- Selecting a source proposal to start a new Draft
- Staying responsive as the freelancer's cross-project proposal history grows over years of use

**Non-Goals:**
- Performing the copy itself -- owned by FEAT-02.SPEC-008 (Create Draft From Copy), triggered once a source is selected here.
- Browsing or comparing voided versions of a proposal -- intentional omission per the Feature Breakdown Brief's Non-Goals; this picker lists only each project's current (non-voided) proposal as a copy source, since voided content carries no forward value as a starting point.
- A configurable proposal template library -- excluded per scope-boundaries.md (SC-11); reuse of a real earlier proposal replaces the need for a template builder.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-02.SPEC-003 (Proposal Detail) | Nadia taps "Start from a copy" on a Draft proposal | Target project reference (the project the new Draft will belong to) |
| FEAT-02.SPEC-001 (Proposal Draft Editor) | Nadia taps "Start from a copy" from within an already-open blank editor | Target project reference |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen: every earlier proposal across her clients and projects | Select a source proposal | -- |
| Owen (Client Primary Contact) | No | No | This screen exists only in the freelancer's own workspace; no route into it exists from the client portal. |
| Priya (Client Reviewer Contact) | No | No | Same as Owen. |
| Dana (Support Operator) | No | No | Reuse is a freelancer-authoring action with no support-relevant read value beyond what FEAT-02.SPEC-003 already shows per proposal; this screen is not reachable inside a support session. |
| Unauthenticated | No | No | Redirected to freelancer sign-in. |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." The target project context is preserved and restored after re-authentication succeeds. |

## Layout and Content

**Header:** Screen title "Start from a Proposal" with a back arrow (returns to the entry point) at the left. A search input sits below the title, filtering by client or project name.

**Body:** A scrollable list of earlier proposals, one row per project's current (non-voided) proposal, ordered most-recently-sent first:
- Client name and project name
- Status (Sent, Accepted, or Voided-with-current-replacement is never listed since only the current version appears) and its date (sent_at or accepted_at)
- Price and currency
- A one-line scope description excerpt

**Footer:** None -- selecting a row is the sole action.

### Responsive Behavior

- **Compact:** Single-column list, full width, each row stacked (client/project on one line, status/date/price on the next).
- **Medium size class and above:** List remains single-column, capped at a consistent platform-wide width and horizontally centered; each row lays its fields out on one line rather than stacking.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate back to the entry point (FEAT-02.SPEC-003 or FEAT-02.SPEC-001) | Screen closes | Animated transition back |
| Search input | Type | Filters the list to matching client or project names | List updates | List re-renders to filtered results, or "No proposals match {query}." if none |
| Proposal row | Tap | Triggers FEAT-02.SPEC-008 (Create Draft From Copy) with the selected proposal as source | Row shows brief loading state | On completion, navigates to FEAT-02.SPEC-001 pre-filled with the copied content |

### Accessibility Notes

- **Focus order:** Back arrow -> search input -> proposal rows in list order.
- **Filter announcements:** The result count is announced to assistive technology when the search filter changes ("12 proposals" / "No proposals match {query}.").
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loaded (with proposals) | Full list of earlier proposals | Screen opens and at least one prior proposal exists | Nadia selects a row or navigates away |
| Empty | Message: "You don't have any earlier proposals to start from yet." with no list | Screen opens and no prior Sent, Voided, or Accepted proposal exists anywhere in the freelancer's account | N/A -- remains until a first proposal is sent elsewhere |
| No Search Results | Message: "No proposals match {query}." | Search filter matches zero rows | Nadia clears or changes the search query |
| Loading | Skeleton placeholder rows | Screen is opening and the list has not yet arrived | Data loads and the Loaded (with proposals) or Empty state renders, per whether any prior proposal exists |
| Error | Error banner: "Could not load your earlier proposals." with a Retry button | Loading the list fails | Nadia taps Retry, or connectivity returns and a re-fetch succeeds |
| Offline/Degraded | Banner: "You're offline -- showing the last loaded list." Selecting a row is disabled until connectivity returns, since the copy automation requires it | Connectivity is lost while this screen is open | Connectivity returns and selection re-enables |

## Validation Rules

Not applicable -- this screen has no user input fields beyond the non-validated search filter.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | Entry point (FEAT-02.SPEC-003 or FEAT-02.SPEC-001) | -- |
| Row selected, copy completes | FEAT-02.SPEC-001 (Proposal Draft Editor), pre-filled | -- |

## Data Model

**Creates:** None.
**Reads:** Proposal -- lists the current (non-voided) proposal per project across all of Nadia's clients: scope_description (excerpt), price, currency, status, sent_at, accepted_at. Project and Client -- names for display.
**Updates:** None.
**Deletes:** None.

## Business Rules

- Only each project's current (non-voided) proposal appears as a copy source -- a project's voided prior versions are never separately listed, consistent with the feature's exclusion of a browsable version-history view.
- The list spans every client and project the freelancer has ever sent a proposal for, not only the target project's own client, per the Feature Breakdown Brief's Key Capabilities ("Reuse an earlier proposal... from any previous proposal").
- Selecting a row always creates a new Draft for the target project passed in from the entry point -- it never modifies or navigates directly to the source proposal.

## Edge Cases

- **The freelancer has 200+ proposals across years of use** -- The list stays responsive per the Feature Breakdown Brief's Non-Functional Notes; the list loads incrementally (older entries load as Nadia scrolls) rather than all at once.
- **Nadia searches for a client name that matches zero proposals** -- "No proposals match {query}." is shown; the search input remains editable.
- **Nadia selects a row and the copy automation fails** -- The row's loading state clears, an inline error appears on the row: "Could not start from this proposal. Try again.", and Nadia remains on the picker.
- **The source proposal selected is later voided or discarded in another session before the copy completes** -- The copy automation (FEAT-02.SPEC-008) reads the source proposal's content at the moment of selection; since Comment/Proposal history is immutable once sent (XBR-04), a Voided source's last-sent content is still valid to copy, so no failure occurs solely because the source has since been voided.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-001 (Proposal Draft Editor) | Navigation (inbound/outbound) | Entry point when reached from the editor; destination after a successful copy |
| FEAT-02.SPEC-003 (Proposal Detail) | Navigation (inbound) | Entry point via "Start from a copy" on a Draft |
| FEAT-02.SPEC-008 (Create Draft From Copy) | Triggers (outbound) | Row selection triggers the copy automation |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| reuse_picker_opened | entry source (proposal detail / draft editor) | Screen finishes loading | supports success-metrics.md: "Proposal Send Speed" |
| reuse_picker_source_selected | source proposal age in days | Nadia selects a row | supports success-metrics.md: "Proposal Send Speed" |

## Acceptance Criteria

**FEAT-02.SPEC-004-AC-01:** Given Nadia has 3 earlier proposals across 2 clients, when she opens the Reuse Proposal Picker, then all 3 appear, ordered most-recently-sent first, each showing client, project, status, date, price, and a scope excerpt.

**FEAT-02.SPEC-004-AC-02:** Given Nadia has never sent a proposal, when she opens the picker, then it shows "You don't have any earlier proposals to start from yet." with no list.

**FEAT-02.SPEC-004-AC-03:** Given Nadia types a client name into the search input that matches one proposal, then the list filters to show only that proposal.

**FEAT-02.SPEC-004-AC-04:** Given Nadia searches for a name matching no proposal, then "No proposals match {query}." is shown.

**FEAT-02.SPEC-004-AC-05:** Given Nadia taps a proposal row, when the copy completes, then she is navigated to FEAT-02.SPEC-001 with scope, price, and currency pre-filled from the selected proposal.

**FEAT-02.SPEC-004-AC-06:** Given Nadia taps a proposal row and the copy automation fails, then an inline error "Could not start from this proposal. Try again." appears on that row and she remains on the picker.

**FEAT-02.SPEC-004-AC-07:** Given the picker is loading, then skeleton placeholder rows appear until data arrives.

**FEAT-02.SPEC-004-AC-08:** Given loading the list fails, then the error banner "Could not load your earlier proposals." appears with a Retry button.

**FEAT-02.SPEC-004-AC-09:** Given Nadia loses connectivity while viewing the picker, then the banner "You're offline -- showing the last loaded list." appears and row selection is disabled.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 3 | 3 |
| States | 6 (loaded, empty, no search results, loading, error, offline) | 6 |
| Business Rules | 3 | 3 |
| Edge Cases | 4 | 4 |

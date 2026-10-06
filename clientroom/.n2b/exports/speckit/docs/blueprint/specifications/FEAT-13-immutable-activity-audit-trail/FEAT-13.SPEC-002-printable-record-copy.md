---
document_type: spec
spec_type: screen
spec_id: FEAT-13.SPEC-002
spec_name: Printable Record Copy
spec_slug: printable-record-copy
parent_feature: FEAT-13
parent_feature_name: Immutable Activity & Audit Trail
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Screen Spec: Printable Record Copy

## Overview

**Name:** Printable Record Copy
**ID:** FEAT-13.SPEC-002
**Type:** Screen
**Purpose:** Nadia produces a printable, unalterable copy of a project's full activity trail, or of one selected entry, to show a client during a scope dispute.
**Parent Feature:** FEAT-13 -- Immutable Activity & Audit Trail

## Scope and Non-Goals

**In Scope:**
- Rendering a print-ready copy of every entry in a project's trail
- Rendering a print-ready copy of a single selected entry
- Loading, ready, and error states for producing the copy
- The copy's own identifying header (freelancer business identity, client, project) so it is self-contained when shown or handed to someone outside the product

**Non-Goals:**
- Browsing or selecting entries from the full trail -- handled by FEAT-13.SPEC-001 (Activity Trail); this screen only renders the copy for the scope it is opened with.
- Emailing or otherwise transmitting the copy to the client -- product-features.md's Communications field for this feature states the trail sends no notifications of its own; Nadia shows or hands the printed/saved copy to the client herself, outside the product's messaging channels.
- Any capability for Owen, Priya, or Dana to produce a copy -- excluded per the Access Matrix (user-persona.md): Activity & Audit Trail is "Full" for Nadia only; Dana's access is View-only with no export path (Access Matrix, Financial Dashboard & Accounting Export row precedent: "View... no export generation"), and client contacts have "None."

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-13.SPEC-001 (Activity Trail) | Nadia taps the header "Share" button | Project reference; scope = full trail |
| FEAT-13.SPEC-001 (Activity Trail) | Nadia taps "Share this record" on one entry | Project reference and entry reference; scope = single entry |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen, her own projects only | Produce the printable copy (full trail or single entry) and print or save it | -- |
| Owen (Client Primary Contact) | No | No | The capability is not shown at all -- no path from his portal view reaches this screen; he only ever sees the copy Nadia herself chooses to show or hand him outside the product. |
| Priya (Client Reviewer Contact) | No | No | Same as Owen -- not shown at all. |
| Dana (Support Operator) | No | No | Not reachable from a support session -- support access is View-only on the trail (FEAT-13.SPEC-001) with no export or copy-generation capability, per scope-boundaries.md (SC-04): the operator never generates exports on the freelancer's behalf. |
| Unauthenticated | No | No | Redirected to the sign-in screen; no scope context is retained. |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- since this screen persists no in-progress input, Nadia simply re-opens it from FEAT-13.SPEC-001 after signing back in. |

## Layout and Content

**Header:** Screen title -- "Activity Record" for a single-entry copy, or "Activity Trail -- {Project Name}" for a full-trail copy -- with a back control (returns to FEAT-13.SPEC-001) and a "Print / Save" action button.

**Body:** A self-contained, print-ready page, laid out for both on-screen reading and printing:
- **Identifying header block:** Nadia's business name (from her Freelancer Account business details), the client company name, the project name, and the date the copy was produced.
- **Entry content, full trail scope:** every entry for the project, in the same reverse-chronological order as FEAT-13.SPEC-001, each shown at full detail: complete event description, actor's full name and role, exact date and time, and the affected record's identifying reference (e.g., invoice number, milestone name).
- **Entry content, single-entry scope:** the one selected entry, shown at the same full detail, with no other entries present.
- **Footer:** a fixed statement that the copy reflects an append-only, unalterable record as maintained by the product, plus the copy's production timestamp.

No field on this screen is editable -- every element is read-only, rendered content.

### Responsive Behavior
- **Compact breakpoint:** Single-column page; the identifying header block stacks above the entry content; each entry's description, actor, and timestamp stack vertically, matching FEAT-13.SPEC-001's compact entry-row layout at its expanded density.
- **Medium size class and above:** The page is capped at a consistent platform-wide reading/print width and centered; entry rows lay their description, actor, timestamp, and reference out along one line, matching FEAT-13.SPEC-001's row pattern at higher density (full detail rather than the trail's summary line).
- **Print output:** Produces one continuous document containing the identifying header, every included entry, and the footer statement, independent of screen size -- the print layout is not clipped to the viewport.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back control | Tap | Navigates to FEAT-13.SPEC-001 (Activity Trail) | Screen closes | Standard transition back to the trail |
| "Print / Save" button | Tap | Produces a printable file of the currently rendered copy (full trail or single entry, matching what is on screen) | Button shows a brief "Preparing your copy..." indicator | A file is offered for save or the platform's print dialog opens; on completion the button returns to its normal state |
| "Print / Save" button (while preparing) | Tap | No action -- debounced | None | Button remains in its "Preparing..." state |

### Accessibility Notes

- **Focus order:** Back control -> "Print / Save" button -> identifying header block -> entry content, top to bottom (each entry: description -> actor -> timestamp -> reference) -> footer statement.
- **Dynamic-change announcements:** The "Preparing your copy..." state and its completion (file offered, or an error) are announced to assistive technology.
- **Keyboard alternatives:** Both the back control and "Print / Save" are reachable and operable by keyboard; there are no pointer-only gestures on this screen.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Identifying header block visible; entry content area shows a lightweight loading indicator | Screen opens from FEAT-13.SPEC-001 | The scoped entry or entries finish loading |
| Ready | Full copy rendered as described in Layout and Content, "Print / Save" button enabled | Content loads successfully | Nadia navigates away, or taps "Print / Save" |
| Preparing | "Print / Save" button shows a "Preparing your copy..." indicator; rendered content remains visible underneath | Nadia taps "Print / Save" | The file is offered, the print dialog opens, or preparation fails |
| Error | Error message in place of the entry content area: "This record couldn't be loaded. Try again." with a Retry control; the identifying header block remains visible | Loading the scoped entry or entries fails | Nadia taps Retry and loading succeeds, or she navigates away |
| Offline/Degraded | N/A -- this screen only ever renders an entry or entries already loaded into the session from FEAT-13.SPEC-001's Populated state moments earlier; the same "This record couldn't be loaded. Try again." Error state covers the case where the scoped content cannot be fetched, including while offline. | -- | -- |

## Validation Rules

**Option B -- Inline (no user input on this screen):**
This screen accepts no field input; it only renders previously written, immutable Activity Log Entry content. There is nothing for the user to enter or for this screen to validate.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back control tap | FEAT-13.SPEC-001 (Activity Trail) | -- |
| "Print / Save" completes | Remains on this screen (Ready state) | -- |

## Data Model

**Creates:** None -- this screen never writes an Activity Log Entry; it renders a presentational copy only.
**Reads:** Activity Log Entry -- `event_type`, `actor`, `occurred_at`, `affected_record` for the entry or entries in scope; Project (name, owned by FEAT-01); Client (company name, owned by FEAT-01); Freelancer Account (business name, owned by FEAT-21), for the identifying header block.
**Updates:** None.
**Deletes:** None.

## Business Rules

- XBR-04: the copy this screen produces is a faithful rendering of entries that are themselves immutable (FEAT-13.SPEC-004) -- the copy carries no editing capability, and reproducing it does not alter the underlying entry in any way.
- Visibility of this screen is governed entirely by FEAT-13.SPEC-005 (Activity Trail Access & Visibility Rules); this screen enforces no separate access logic of its own.
- The scope (full trail vs. single entry) is fixed by which control Nadia tapped on FEAT-13.SPEC-001 -- this screen offers no way to change scope after opening; she returns to FEAT-13.SPEC-001 and re-launches with a different scope instead.

## Edge Cases

- **Nadia opens the full-trail copy for a project with a very long history** -- Every entry is included; the print output spans multiple pages as needed rather than truncating.
- **An entry's affected record is later deleted or superseded after the copy is produced** -- The copy already reflects the entry's permanent content (event description, actor, timestamp, reference) at production time and is never affected by later changes to the affected record, since the entry itself never changes.
- **Nadia taps "Print / Save" twice in rapid succession** -- The second tap is ignored while the first preparation is in progress (button in its "Preparing..." state).
- **The scoped entry no longer resolves (e.g., a stale link reused after navigating away and back with a different project loaded)** -- The Error state renders: "This record couldn't be loaded. Try again." with Retry, which re-fetches using the original scope context.
- **Nadia produces a copy, then the underlying project's trail gains new entries before she shows the copy to the client** -- The already-produced copy is unaffected; it is a snapshot at production time. If she wants the new entries included, she returns to FEAT-13.SPEC-001 and produces a fresh copy.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-13.SPEC-001 (Activity Trail) | Navigation (inbound) | Nadia arrives here after choosing to share the full trail or a single entry |
| FEAT-13.SPEC-004 (Entry Immutability, Content & Attribution Rules) | References (inbound) | Guarantees the content this screen renders is unaltered |
| FEAT-13.SPEC-005 (Activity Trail Access & Visibility Rules) | References (inbound) | Governs exactly who may open this screen |
| FEAT-01 (Client & Project Management) | References (inbound) | Owns Project and Client -- this screen reads the project name and client company name from FEAT-01's records for the identifying header block |
| FEAT-21 (Settings & Account Management) | References (inbound) | Owns Freelancer Account -- this screen reads Nadia's business name from FEAT-21's records for the identifying header block |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| activity_record_shared | scope (full_trail / single_entry), entry count included, project reference | "Print / Save" completes successfully (file offered or print dialog opened) | supports success-metrics.md: "Dispute Resolution Confidence" |
| activity_record_share_failed | scope, failure point (load / preparation) | The Error state renders, or "Preparing your copy..." fails to complete | N/A -- no success-metrics.md metric measures the failure path directly; retained because product-features.md's Signals field for this feature names `activity_record_shared` as a required signal, and this companion event lets the success rate implied by "Dispute Resolution Confidence" be computed from event data without inflating the cited metric's own definition |

## Acceptance Criteria

**FEAT-13.SPEC-002-AC-01:** Given Nadia taps "Share" on the full trail from FEAT-13.SPEC-001, when this screen opens, then it shows every entry for the project at full detail, most-recent-first, under an identifying header naming her business, the client, and the project.

**FEAT-13.SPEC-002-AC-02:** Given Nadia taps "Share this record" on one entry from FEAT-13.SPEC-001, when this screen opens, then it shows only that one entry at full detail, with no other entries present.

**FEAT-13.SPEC-002-AC-03:** Given Nadia is viewing a Ready copy, when she taps "Print / Save," then the button shows "Preparing your copy..." and, on completion, a file is offered for save or the print dialog opens.

**FEAT-13.SPEC-002-AC-04:** Given Nadia is viewing a Ready copy, when she taps "Print / Save" a second time while preparation is still in progress, then the second tap has no effect and the button remains in its "Preparing..." state.

**FEAT-13.SPEC-002-AC-05:** Given Nadia taps the back control, when the tap registers, then she returns to FEAT-13.SPEC-001 (Activity Trail).

**FEAT-13.SPEC-002-AC-06:** Given this screen's scoped content fails to load, when the failure occurs, then the Error state shows "This record couldn't be loaded. Try again." with a Retry control, while the identifying header block remains visible.

**FEAT-13.SPEC-002-AC-07:** Given the Error state is showing, when Nadia taps Retry and loading succeeds, then the Ready state renders with the originally requested scope.

**FEAT-13.SPEC-002-AC-08:** Given Owen or Priya is signed into the client portal, when they look for any way to reach this screen, then no path exists anywhere in their portal view.

**FEAT-13.SPEC-002-AC-09:** Given Dana has an open support session on Nadia's account, when she views the activity trail, then no control to reach this screen is rendered anywhere in her session.

**FEAT-13.SPEC-002-AC-10:** Given Nadia produces a full-trail copy for a project with hundreds of entries, when she taps "Print / Save," then the resulting output includes every entry, spanning multiple pages as needed.

**FEAT-13.SPEC-002-AC-11:** Given Nadia has already produced a copy and new entries are later written to the project's trail, when she views the previously produced copy again, then it still shows only the entries present at the time it was produced.

**FEAT-13.SPEC-002-AC-12:** Given an unauthenticated visitor attempts to reach this screen directly, when the request is made, then they are redirected to sign-in with no scope context retained.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 3 | 3 |
| States | 3 (loading, preparing, error) | 3 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |

---
document_type: spec
spec_type: screen
spec_id: FEAT-22.SPEC-003
spec_name: Household Support Access Record
spec_slug: household-support-access-record
parent_feature: FEAT-22
parent_feature_name: Operator Read-Only Support Access
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Screen Spec: Household Support Access Record

## Overview

**Name:** Household Support Access Record
**ID:** FEAT-22.SPEC-003
**Type:** Screen
**Purpose:** Maya views the record of every time support viewed her household, when and why, entered from her household settings.
**Parent Feature:** FEAT-22 -- Operator Read-Only Support Access

## Scope and Non-Goals

**In Scope:**
- Listing every access session logged by FEAT-22.SPEC-004 for this household, with its start and end time and the Support Request it belongs to
- Showing the reason (the request's kind and, where present, its note) alongside each session
- Read-only display, entered from FEAT-01.SPEC-010 (Household Settings Hub)

**Non-Goals:**
- Any control over support access itself (ending a session early, blocking future access) -- excluded per XBR-14: the organiser's Support View access level is View, not Full; only Riley's read-only view (FEAT-22.SPEC-002) and its own gating (FEAT-22.SPEC-006) govern when access opens and closes
- Sam's access to this record -- excluded per the Access Matrix's Support View column (Sam: None); this screen is Maya's alone
- The underlying household data Riley viewed -- this screen shows only that access occurred and why, never a replay of what Riley saw, since that would recreate the very household-data exposure this feature exists to bound
- Any general support Support Request that never opened an access session (e.g., resolved before Riley opened the household) -- excluded per this feature's own definition: the access record tracks sessions, not the request's status history in general, which is visible instead through the request's own resolution notice (FEAT-22.SPEC-010)

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-010 (Household Settings Hub) | Maya taps "Support access record" | None -- screen loads this household's full access history |
| FEAT-22.SPEC-009 (Support View Recorded Notification) | Maya taps "View record" | None -- screen loads this household's full access history |
| FEAT-22.SPEC-010 (Support Request Resolved Notification) | Maya taps "View record" | None -- screen loads this household's full access history |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|--------------------------|
| Maya (Organiser) | Full screen | View only -- no actions beyond viewing | -- |
| Sam (Other Adult Member) | No | No | "Support access record" is not shown as an option within Sam's household settings, per the Access Matrix's Support View column (None for Sam) |
| Jordan (young kid profile, no login -- MVP) | No | No | No login exists for this role |
| Jordan (older kid, limited login -- Later) | No | No | This login's product surface (Grocery List, Dinner Voting) has no path to household settings |
| Riley (Operator, support) | No | No | This screen is the organiser's own view of the record; Riley's read-only access is a separate screen (FEAT-22.SPEC-002) that never shows this record |
| Unauthenticated | No | No | Redirected to the sign-in screen |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- no data was being entered here, so nothing is lost |

## Layout and Content

**Header:** Screen title "Support access record" with a back arrow (returns to FEAT-01.SPEC-010).

**Body:** A single list of access sessions, most recent first. Each entry shows:
- Start and end time (or "In progress" if the session has not yet ended)
- The associated Support Request's kind ("Safety concern" or "General support") and, for safety concerns, the reported meal's name
- The request's current status (Raised, Under review, Resolved)

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Entries stack vertically, full width, each showing all fields listed above in a single card.
- **Medium size class and above:** Entries render as a table with start/end time, kind, reported meal (safety concerns only), and status in separate columns; no structural change beyond column layout.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-01.SPEC-010 (Household Settings Hub) | Screen closes | Standard transition back |

### Accessibility Notes

- **Focus order:** Back arrow -> record entries in most-recent-first order.
- **In-progress announcement:** An entry whose session has not yet ended announces "In progress" as its end-time value to assistive technology, rather than being left blank.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Populated (default) | List of access sessions as described | Screen loads with 1+ logged sessions | Maya navigates away |
| Empty | Message "Support has never accessed your household." with no list | Screen loads with zero logged sessions | A new access session is logged |
| Loading | Brief inline loading indicator in place of the list | Screen first opens, before the record resolves | Record loads (populated or empty) |
| Error | Error banner "Couldn't load the support access record. Check your connection and try again." with Retry | Record fails to load | Maya taps Retry (returns to Loading) |
| Offline/Degraded | Banner "You're offline -- the support access record needs a connection to load." List area is empty until connectivity returns | Connectivity lost while this screen is open | Connectivity restored -- the record loads automatically |

## Validation Rules

**Option B -- Inline (no user input exists on this read-only screen; no fields to validate).**

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|----------------|
| N/A -- read-only screen | This screen accepts no input | -- | -- |

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|--------------------------------------|
| Back arrow tap | FEAT-01.SPEC-010 (Household Settings Hub) | FEAT-01 (Household Setup & Member Profiles) |

## Data Model

**Creates:** None.
**Reads:** Support Request -- access_record (each logged session's start/end timestamp), kind, planned_meal/recipe (for safety concerns), status, all for Support Requests belonging to this household.
**Updates:** None.
**Deletes:** None.

## Business Rules

- This screen shows the organiser's View-level visibility into FEAT-22's access log, per XBR-14 -- Maya sees when and why Riley viewed the household, with no control over the access itself.
- A session's entry is written by FEAT-22.SPEC-004 and never edited by this screen or any other -- this screen only reads what FEAT-22.SPEC-004 has already recorded.
- Access records are retained for the life of the household account, per SC-18 and the Entity-Lifecycle Coverage Matrix's Delete/Archive: N/A finding for Support Request.

## Edge Cases

- **A new access session is logged while Maya is viewing this screen** -- The list does not auto-insert the new entry mid-view; Maya sees it on the next screen load, consistent with this being a rare, on-demand record rather than a live-updating feed.
- **The household has multiple Support Requests, each with its own sessions** -- All sessions across all of the household's Support Requests appear together in one chronological list, not grouped or filtered by request.
- **A session is still in progress (Riley has not yet closed the household view) when Maya opens this screen** -- The entry shows "In progress" as its end time rather than a blank or a stale value.
- **Network failure while loading the record** -- Error banner: "Couldn't load the support access record. Check your connection and try again." with Retry.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-010 (Household Settings Hub) | Navigation (inbound) | Entry point from household settings |
| FEAT-22.SPEC-004 (Support Access Session Logging) | References (inbound) | Writes every entry this screen displays |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| support_access_record_viewed | entry_count | Screen loads | N/A -- no success-metrics.md metric traces to FEAT-22; retained so the organiser's actual use of this trust-facing record is observable |

## Acceptance Criteria

**FEAT-22.SPEC-003-AC-01:** Given Maya taps "Support access record" from household settings, when the screen loads and 2 past sessions exist, then both entries appear, most recent first, each showing start/end time, kind, and status.

**FEAT-22.SPEC-003-AC-02:** Given a safety-concern session is logged, when Maya views its entry, then the reported meal's name is shown alongside the kind.

**FEAT-22.SPEC-003-AC-03:** Given no session has ever been logged for Maya's household, when the screen loads, then "Support has never accessed your household." is shown with no list.

**FEAT-22.SPEC-003-AC-04:** Given Riley currently has this household's view open, when Maya loads this screen, then the corresponding entry shows "In progress" as its end time.

**FEAT-22.SPEC-003-AC-05:** Given Sam attempts to reach this screen directly, then it is not accessible to him, since "Support access record" is not shown in his household settings.

**FEAT-22.SPEC-003-AC-06:** Given the record fails to load due to a network error, when Maya views the screen, then the error banner with Retry appears.

**FEAT-22.SPEC-003-AC-07:** Given Maya loses connectivity while viewing this screen, then the offline banner appears and the list area remains empty until connectivity returns.

**FEAT-22.SPEC-003-AC-08:** Given Maya's household has Support Requests of both kinds with their own sessions, when the screen loads, then all sessions from all requests appear together in one chronological list.

**FEAT-22.SPEC-003-AC-09:** Given Maya taps the back arrow, then she returns to FEAT-01.SPEC-010 (Household Settings Hub).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 1 | 1 |
| States | 5 (populated, empty, loading, error, offline) | 5 |
| Business Rules | 3 | 3 |
| Edge Cases | 4 | 4 |

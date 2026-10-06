---
document_type: spec
spec_type: screen
spec_id: FEAT-18.SPEC-001
spec_name: Client Contact List
spec_slug: client-contact-list
parent_feature: FEAT-18
parent_feature_name: Client Contact Management & Roles
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Screen Spec: Client Contact List

## Overview

**Name:** Client Contact List
**ID:** FEAT-18.SPEC-001
**Type:** Screen
**Purpose:** Nadia views and manages the contacts and roles for one client company; Dana views the same list read-only inside a logged support session.
**Parent Feature:** FEAT-18 -- Client Contact Management & Roles

## Scope and Non-Goals

**In Scope:**
- Listing every Client Contact belonging to one client company, with role and status
- Launching contact creation (FEAT-18.SPEC-002), editing (FEAT-18.SPEC-002), and removal (FEAT-18.SPEC-003)
- The empty state that blocks proposal sending until a Primary contact exists (FEAT-18.SPEC-006)
- Dana's read-only mirrored view of this same list inside a support session (FEAT-31)

**Non-Goals:**
- Entering or editing contact fields -- handled by FEAT-18.SPEC-002 (Add or Edit Client Contact); this list only launches that screen
- Confirming a removal -- handled by FEAT-18.SPEC-003 (Remove Client Contact); this list only launches that screen
- Owen inviting a Reviewer colleague -- handled by FEAT-18.SPEC-004 (Invite Reviewer Colleague), Owen's own client-side entry point; this screen is Nadia's freelancer-side surface and Owen never reaches it (Access Matrix: Owen carries None for Client & Project Management)
- Bulk import of contacts -- excluded per scope-boundaries.md (SC-19): a client carries only a handful of contacts, so individual add/invite entry is fast enough and a bulk path would risk importing contacts never verified through the product's own flow

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-004 (Client Detail) | Nadia taps "Manage Contacts" | The client reference, scoping this list to that client |
| FEAT-14 (Notifications (Email)), delivery warning | Nadia taps a delivery warning on a project flagging a bouncing contact email | The client reference and the specific contact whose email is bouncing, so the affected row is highlighted |
| FEAT-31.SPEC-002 (Operator Support Session Console) | Dana navigates to a client's contacts inside an open, read-only support session | The client reference; the session's permanent read-only banner and disabled controls (FEAT-31.SPEC-003) |
| FEAT-18.SPEC-002 (Add or Edit Client Contact) | A save completes successfully | Returns to this list, refreshed |
| FEAT-18.SPEC-003 (Remove Client Contact) | A removal completes, or is blocked | Returns to this list, refreshed |
| FEAT-18.SPEC-011 (Primary-Invited-Colleague Alert) | Nadia taps the alert email's "View contacts" CTA | The client reference for {client_company_name}, scoping this list to that client |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen, for any client she owns | Add, edit, remove contacts and change roles | -- |
| Owen (Client Primary Contact) | No | No | Not shown in Owen's portal navigation; this is the freelancer's own management surface (Access Matrix: Owen carries None for Client Contact Management on the freelancer side) |
| Priya (Client Reviewer Contact) | No | No | Not shown in Priya's portal navigation (Access Matrix: None) |
| Dana (Support Operator) | Full screen, read-only, for the one freelancer account named by her open support session (FEAT-31.SPEC-005: one account at a time) | View only -- Add, edit, and remove controls are not rendered | Attempting to reach an add/edit/remove action directly returns Dana to the read-only view with no change made, under the permanent "Read-only support session" banner (FEAT-31.SPEC-003) |
| Unauthenticated | No | No | Redirected to the sign-in screen |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- no in-progress form data exists on this list screen to preserve |

## Layout and Content

**Header:** Client name (breadcrumb back to FEAT-01.SPEC-004, Client Detail) with screen title "Contacts." An "Add Contact" action button, top right, visible to Nadia only.

**Body:** A vertical list of contact rows, one per Client Contact belonging to this client, ordered by role (Primary contacts first) then name. Each row shows:
- Name
- Email
- Role badge (Primary or Reviewer)
- Status badge (Invited, Active, or Removed contacts are never shown in this list -- see Data Model)
- Last sign-in (relative time, or "Not yet signed in" for Invited contacts)
- Row actions, visible to Nadia only: "Edit" and "Remove"

If the delivery-warning entry point (FEAT-14) brought Nadia here, the flagged contact's row is highlighted with a small inline note: "This email is bouncing."

For Dana's read-only view, row actions are not rendered; the row content is otherwise identical.

**Footer:** None.

### Responsive Behavior

- **Compact size class:** Contact rows stack in a single column, full width; role and status badges wrap below the name/email on narrow widths; row actions collapse into an overflow menu.
- **Medium size class and above:** Rows remain single-column, capped at a consistent platform-wide content width and horizontally centered; role, status, and last-sign-in display inline with name and email; row actions show as separate buttons rather than an overflow menu.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| "Add Contact" button (Nadia only) | Tap | Navigate to FEAT-18.SPEC-002 (Add or Edit Client Contact) in create mode, scoped to this client | Screen transitions | Standard navigation transition |
| Contact row (Nadia only) | Tap | Navigate to FEAT-18.SPEC-002 in edit mode, pre-filled with this contact's data | Screen transitions | Standard navigation transition |
| "Edit" row action (Nadia only) | Tap | Same as tapping the row | Screen transitions | Standard navigation transition |
| "Remove" row action (Nadia only) | Tap | Navigate to FEAT-18.SPEC-003 (Remove Client Contact) for this contact | Screen transitions | Standard navigation transition |
| "This email is bouncing" note (Nadia only) | Tap | Navigates to FEAT-18.SPEC-002 in edit mode for that contact, with the email field focused | Screen transitions | Standard navigation transition |
| Contact row (Dana) | Tap | No action -- row is display-only in the read-only view | None | None |

### Accessibility Notes

- **Focus order:** Breadcrumb -> "Add Contact" (when visible) -> contact rows in list order, each row's Edit action before its Remove action (when visible).
- **Dynamic updates:** A row appearing, disappearing, or changing role/status after returning from FEAT-18.SPEC-002 or FEAT-18.SPEC-003 is announced to assistive technology as a list-content change.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Skeleton rows in place of the contact list | Screen first opens | Data finishes loading |
| Loaded (has contacts) | Full list as described in Layout and Content | Data load completes with one or more contacts | Navigation away, or a contact is added/edited/removed |
| Empty | "No contacts yet. Add a Primary contact before you can send a proposal to this client." with the "Add Contact" action (Nadia only); Dana's read-only view shows the same message with no action | Client has zero contacts | A contact is successfully added |
| Error | Error banner: "Couldn't load contacts. Try again." with a Retry button | Initial data load fails | Retry succeeds |
| Offline/Degraded | N/A -- contact management is an infrequent, connectivity-required action (feature-overview.md, States); this screen assumes connectivity and shows the Error state on a load failure rather than a distinct offline mode | -- | -- |

## Validation Rules

Validation governed by FEAT-18.SPEC-005 (Contact Field Validation Rules). This list screen performs no direct input validation of its own -- all field entry happens on FEAT-18.SPEC-002.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Breadcrumb tap | FEAT-01.SPEC-004 (Client Detail) | FEAT-01 |
| "Add Contact" tap | FEAT-18.SPEC-002 (Add or Edit Client Contact) | -- |
| Contact row / "Edit" tap | FEAT-18.SPEC-002 (Add or Edit Client Contact) | -- |
| "Remove" tap | FEAT-18.SPEC-003 (Remove Client Contact) | -- |

## Data Model

**Creates:** None -- this screen only lists and launches other specs' create/edit/remove flows.
**Reads:** Client Contact -- name, email, role, status, last_sign_in, for every contact belonging to the scoped Client, excluding Removed contacts (a removed contact's personal details are erased per FEAT-18.SPEC-009 and it no longer appears in this list). Client -- client_name, for the header.
**Updates:** None directly.
**Deletes:** None directly.

## Business Rules

- Only Nadia may add, edit, or remove contacts from this screen (FEAT-18.SPEC-007, Role Authorization Rules).
- Removed contacts never appear in this list; a returning contact must be re-added as a new Client Contact record (feature-overview.md, Non-Goals -- no restore path).
- A client with zero Primary contacts shows the blocking prompt described in the Empty state, consistent with FEAT-18.SPEC-006 (Primary Contact Requirement Rule) and XBR-07.
- Dana's view mirrors this screen exactly except that every action control is omitted, per FEAT-31.SPEC-005's unconditional read-only rule.

## Edge Cases

- **Owen adds a Reviewer colleague (FEAT-18.SPEC-004) while Nadia has this list open** -- This screen is a snapshot loaded on open, not live-updating; the new contact does not appear until Nadia reloads the list or navigates back to it (for example, after returning from FEAT-18.SPEC-002 or FEAT-18.SPEC-003, which always re-fetch).
- **A contact's role or status changes in another of Nadia's own open sessions while this list is open** -- The list shown here can become stale; the authoritative state is re-fetched whenever this screen is (re)entered. Opening FEAT-18.SPEC-002 to edit a contact always loads the contact's current state at that moment, so a stale row here does not cause a stale edit.
- **Nadia removes the client's last Primary contact from another session while this list is open** -- No conflict is possible on this list screen itself: FEAT-18.SPEC-006 re-checks the last-Primary condition at the moment of the removal's own commit (on FEAT-18.SPEC-003), not against what this list displays.
- **The client has contacts but all are Reviewers (no Primary)** -- The list displays normally, but the same blocking prompt as the Empty state's rationale applies at proposal-send time (FEAT-18.SPEC-006); this screen itself shows an inline banner above the list: "This client has no Primary contact. A proposal cannot be sent until one is added or promoted."
- **Dana's support session closes while she is viewing this list** -- She is returned to the Operator Support Session Console queue (FEAT-31.SPEC-002); this screen becomes unreachable for that account until a new session opens.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-18.SPEC-002 (Add or Edit Client Contact) | Navigation (outbound) | "Add Contact" and row/"Edit" taps launch it |
| FEAT-18.SPEC-003 (Remove Client Contact) | Navigation (outbound) | "Remove" tap launches it |
| FEAT-18.SPEC-006 (Primary Contact Requirement Rule) | References (inbound) | Governs the no-Primary blocking banner and empty-state message |
| FEAT-18.SPEC-007 (Role Authorization Rules) | References (inbound) | Governs which roles see this screen and its actions |
| FEAT-01.SPEC-004 (Client Detail) | Navigation (inbound) | "Manage Contacts" link arrives here |
| FEAT-14 (Notifications (Email)) | Navigation (inbound) | A delivery warning on a project links here to correct a bouncing contact email |
| FEAT-31.SPEC-002 (Operator Support Session Console) | Navigation (inbound) | Dana's read-only entry point during an open session |
| FEAT-31.SPEC-003 (Support Session Open & Read-Only Enforcement) | References (inbound) | Enforces the disabled controls Dana sees |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| contact_list_viewed | viewer_role (freelancer / support_operator), contact_count, has_primary (yes/no) | Screen finishes loading | N/A -- no success-metrics.md metric is connected to Client Contact Management & Roles; retained so contact-list engagement is observable |
| contact_list_no_primary_banner_shown | contact_count | The no-Primary banner renders because the client has contacts but no Primary | N/A -- no success-metrics.md metric is connected to this feature |

## Acceptance Criteria

**FEAT-18.SPEC-001-AC-01:** Given Nadia opens a client with three contacts, when the screen loads, then she sees all three contacts listed with name, email, role badge, and status.

**FEAT-18.SPEC-001-AC-02:** Given Nadia is on the Client Contact List for a client with no contacts, when the screen loads, then she sees "No contacts yet. Add a Primary contact before you can send a proposal to this client." with an "Add Contact" action.

**FEAT-18.SPEC-001-AC-03:** Given Nadia taps "Add Contact", when the tap registers, then she is navigated to FEAT-18.SPEC-002 in create mode, scoped to this client.

**FEAT-18.SPEC-001-AC-04:** Given Nadia taps an existing contact's row, when the tap registers, then she is navigated to FEAT-18.SPEC-002 in edit mode, pre-filled with that contact's current data.

**FEAT-18.SPEC-001-AC-05:** Given Nadia taps "Remove" on a contact row, when the tap registers, then she is navigated to FEAT-18.SPEC-003 for that contact.

**FEAT-18.SPEC-001-AC-06:** Given Nadia arrives here from a delivery warning on a bouncing contact email, when the screen loads, then that contact's row is highlighted with an inline note "This email is bouncing."

**FEAT-18.SPEC-001-AC-07:** Given a client has one or more Reviewer contacts but no Primary contact, when Nadia views this list, then an inline banner reads "This client has no Primary contact. A proposal cannot be sent until one is added or promoted."

**FEAT-18.SPEC-001-AC-08:** Given Dana is inside an open support session on Nadia's account, when she navigates to a client's contacts, then she sees the same list with no "Add Contact", "Edit", or "Remove" controls rendered, under the permanent read-only banner.

**FEAT-18.SPEC-001-AC-09:** Given Dana views the Client Contact List, when she looks for a way to change a contact, then no such control exists, and there is no way to trigger a change from this screen.

**FEAT-18.SPEC-001-AC-10:** Given Owen or Priya is signed in to the client portal, when they look for any route to this screen, then none exists -- the screen is not part of the client portal's navigation.

**FEAT-18.SPEC-001-AC-11:** Given the initial data load for this screen fails, when the failure occurs, then Nadia sees "Couldn't load contacts. Try again." with a Retry button.

**FEAT-18.SPEC-001-AC-12:** Given a contact was removed in another of Nadia's sessions while this list was open, when Nadia returns to this list from FEAT-18.SPEC-002 or FEAT-18.SPEC-003, then the list re-fetches and no longer shows the removed contact.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 4 (loading, empty, error, offline-degraded N/A) | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |

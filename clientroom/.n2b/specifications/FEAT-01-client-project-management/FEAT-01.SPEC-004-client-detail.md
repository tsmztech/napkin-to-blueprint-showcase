---
document_type: spec
spec_type: screen
spec_id: FEAT-01.SPEC-004
spec_name: Client Detail
spec_slug: client-detail
parent_feature: FEAT-01
parent_feature_name: Client & Project Management
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Screen Spec: Client Detail

## Overview

**Name:** Client Detail
**ID:** FEAT-01.SPEC-004
**Type:** Screen
**Purpose:** Single view of one client -- rename it, edit billing details, view its contacts, and archive, reactivate, or delete it.
**Parent Feature:** FEAT-01 -- Client & Project Management

## Scope and Non-Goals

**In Scope:**
- Renaming the client
- Editing billing details (billing name, billing address, tax ID), governed by FEAT-01.SPEC-010 for completeness
- Archiving and reactivating the client, including the open-items confirmation (FEAT-01.SPEC-007)
- Deleting the client when eligible (FEAT-01.SPEC-009), or explaining why it is not
- Showing the client's basic contact reference and a link into full contact management

**Non-Goals:**
- Managing the full client contact list (invite, remove, change roles) -- handled by Client Contact Management & Roles (FEAT-18), reached from this screen
- Setting currency and tax treatment -- owned by Currency & Tax Handling (FEAT-15), configured at the project level
- Viewing the client's financial totals -- owned by the Freelancer Financial Dashboard (FEAT-12)
- Editing a project's own details -- handled by FEAT-01.SPEC-005 (Project Detail)

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-003 (Client & Project Roster) | Nadia selects a client row | The client's identifier |
| FEAT-31 (Operator Support Access) | Dana selects a client during a read-only support session | Same client, read-only rendering |
| FEAT-28 (Global Search, v1) | Nadia selects a client search result | The client's identifier |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Rename, edit billing details, archive/reactivate, delete (when eligible), navigate to contacts | -- |
| Owen (Client Primary Contact) | No | No | Not shown in Owen's portal navigation; this is the freelancer's own management surface |
| Priya (Client Reviewer Contact) | No | No | Not shown in Priya's portal navigation |
| Dana (Support Operator) | Full screen, read-only | View only -- Rename, edit billing, archive/reactivate, and delete controls are not rendered | Attempting to reach an edit action directly returns Dana to the read-only view with no change made |
| Unauthenticated | No | No | Redirected to the sign-in screen |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- any unsaved billing-detail edits are preserved locally and restored after re-authentication succeeds |

## Layout and Content

**Header:** Client name (editable inline via a "Rename" action) with a status badge (Active/Archived), using the same identity-block presentation as the roster (FEAT-01.SPEC-003) and Project Detail (FEAT-01.SPEC-005). An overflow menu holds Archive/Reactivate and Delete.

**Body, in order:**
- **Billing details section:** Billing Name (text input), Billing Address (multi-line text input), Tax ID (text input, optional), each editable in place with a "Save" action for the section. A completeness indicator ("Billing details complete" / "Billing details incomplete -- required before the first invoice") reflects FEAT-01.SPEC-010.
- **Contacts section:** The client's basic contact reference (name and email of the Primary contact, if any) with a "Manage Contacts" link into FEAT-18.
- **Projects section:** A list of this client's projects (name plus stage badge), each opening FEAT-01.SPEC-005, with a "New Project" action.

**Footer:** None.

### Responsive Behavior

- **Compact size class:** Header, billing section, contacts section, and projects section stack in a single column, full width.
- **Medium size class and above:** Same single-column stacking, capped at a consistent platform-wide content width and horizontally centered; the billing section's address field grows from 2 to 3 visible lines.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| "Rename" action | Tap, then edit and confirm | Updates client_name | Header reflects new name | Toast "Client renamed" |
| Billing fields | Type, then tap section "Save" | Updates billing_name, billing_address, tax_id | Completeness indicator updates | Toast "Billing details saved," or, if the underlying record changed concurrently, see Edge Cases |
| Overflow menu > Archive | Tap | Triggers FEAT-01.SPEC-007 (Archive Open-Items Check) | If no open items, client archives immediately; if open items exist, a confirmation dialog appears first | Toast "Client archived" on completion |
| Overflow menu > Reactivate (shown only when Archived) | Tap | Re-checks the active-client limit via FEAT-01.SPEC-008, then sets status to Active | Status badge updates to Active | Toast "Client reactivated," or a limit-blocked message (see Edge Cases) |
| Overflow menu > Delete | Tap | Evaluates eligibility via FEAT-01.SPEC-009 | If eligible, a confirmation dialog appears, then the client is hard-deleted; if ineligible, Delete is disabled and Archive is offered instead | Toast "Client deleted" on completion, or the ineligible explanation (see Edge Cases) |
| "Manage Contacts" link | Tap | Navigate to Client Contact Management & Roles (FEAT-18) for this client | Screen transitions | Standard navigation transition |
| "New Project" action | Tap | Navigate to FEAT-01.SPEC-002 (Create Project) pre-scoped to this client | Screen transitions | Standard navigation transition |
| Project row | Tap | Navigate to FEAT-01.SPEC-005 (Project Detail) | Screen transitions | Standard navigation transition |

### Accessibility Notes

- **Focus order:** Rename action -> overflow menu -> billing fields in order -> section Save -> Manage Contacts link -> New Project action -> project rows in order.
- **Dynamic updates:** Toasts and the completeness indicator's change are announced to assistive technology; a concurrent-edit rejection dialog receives focus immediately when it appears.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Skeleton layout for header, billing, contacts, and projects sections | Screen opens | Data finishes loading |
| Loaded | Full detail as described in Layout and Content | Data load completes | A mutating action occurs, or navigation away |
| Editing (billing) | Billing fields in edit mode with a visible section Save | Nadia begins editing a billing field | Save completes, is rejected, or edit is cancelled |
| Error | Error banner: "Couldn't load this client. Try again." with a Retry button | Initial data load fails | Retry succeeds |
| Offline/Degraded | Banner "You're offline -- changes will be saved when you reconnect." at top; the loaded detail remains viewable; Rename, billing Save, Archive, Reactivate, and Delete are disabled until connectivity returns | Connectivity lost while viewing an already-loaded client | Connectivity restored -- controls re-enable |

## Validation Rules

Validation governed by FEAT-01.SPEC-010 (Client Billing Completeness Gate) for billing-field completeness, and by FEAT-01.SPEC-009 (Client Delete Eligibility) for the Delete action. Client name follows the same "required, non-empty" rule as on Add Client (FEAT-01.SPEC-001), checked on the Rename confirm action.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| "Manage Contacts" tap | Client contact list | FEAT-18 (Client Contact Management & Roles) |
| "New Project" tap | FEAT-01.SPEC-002 (Create Project) | -- |
| Project row tap | FEAT-01.SPEC-005 (Project Detail) | -- |
| Successful archive or delete | FEAT-01.SPEC-003 (Client & Project Roster) | -- |
| Back navigation | FEAT-01.SPEC-003 (Client & Project Roster) | -- |

## Data Model

**Creates:** None.
**Reads:** Client record -- client_name, billing_name, billing_address, tax_id, status. Client Contact -- basic reference (name, email) of the Primary contact, read-only. Project records belonging to this client -- project_name, stage (derived).
**Updates:** Client record -- client_name (Rename), billing_name, billing_address, tax_id (billing edit), status (Archive/Reactivate).
**Deletes:** Client record -- only when FEAT-01.SPEC-009 confirms eligibility.

## Business Rules

- Renaming a client updates its display name everywhere it is shown; ID-based references (projects, invoices) are unaffected.
- Archive is subject to the open-items confirmation defined by FEAT-01.SPEC-007; it never erases data, and archived clients and their projects/contacts are retained indefinitely with no automatic purge (ASMP-22, SC-24).
- Reactivating an archived client re-checks the active-client limit (FEAT-01.SPEC-008, XBR-23); reactivation is blocked the same way a new-client add would be.
- Delete is permitted only when FEAT-01.SPEC-009 confirms the client has no sent proposal, invoice, or activity (XBR-24); otherwise Delete is disabled and Archive is offered as the only removal path.
- Billing completeness (FEAT-01.SPEC-010) governs when this client's first invoice can be sent; this screen shows the completeness state but does not itself block sending (that block lives in FEAT-09).

## Edge Cases

- **Client renamed or billing-edited by another session between load and save (last-write-wins per the dependency map's Contention note for Client)** -- The later save simply overwrites the earlier one; no conflict dialog for descriptive-field edits.
- **Archive or Delete attempted after the client's state changed since load (reject-with-refresh per the dependency map's Contention note)** -- The action is rejected with "This client's records changed since you loaded this page. Refresh to see the latest state before archiving/deleting." and a "Refresh" action reloads the client; this is the concurrent-edit conflict behavior for state-changing actions on this shared entity.
- **Archive attempted with unpaid invoices or pending approvals** -- FEAT-01.SPEC-007 surfaces an explicit confirmation summarizing what is still open before the archive completes; declining returns to this screen unchanged.
- **Reactivate attempted beyond the active-client limit** -- Blocked with "You've reached your plan's active client limit. Upgrade to reactivate this client." and an "Upgrade" action opening FEAT-23; the client remains Archived.
- **Delete attempted on an ineligible client** -- Delete is disabled in the overflow menu with inline text: "This client has a sent proposal, invoice, or activity and can only be archived." Archive remains available.
- **Nadia loses connectivity mid-edit** -- The offline banner appears; in-progress billing edits are preserved locally and Save remains disabled until connectivity returns, then the pending save submits automatically.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-003 (Client & Project Roster) | Navigation (inbound/outbound) | Entry point; destination after archive or delete |
| FEAT-01.SPEC-002 (Create Project) | Navigation (outbound) | "New Project" action |
| FEAT-01.SPEC-005 (Project Detail) | Navigation (outbound) | Selecting a project row |
| FEAT-01.SPEC-007 (Archive Open-Items Check) | Triggers (outbound) | Archive action runs the open-items check |
| FEAT-01.SPEC-008 (Active Client Limit Enforcement) | References (outbound) | Reactivate re-checks the active-client limit |
| FEAT-01.SPEC-009 (Client Delete Eligibility) | References (outbound) | Delete action's eligibility check |
| FEAT-01.SPEC-010 (Client Billing Completeness Gate) | References (outbound) | Billing section's completeness rules |
| FEAT-18 (Client Contact Management & Roles) | Navigation (outbound) | "Manage Contacts" link |
| FEAT-23 (Subscription Plan & Billing Management) | Navigation (outbound) | Upgrade action on a limit-blocked reactivation |
| FEAT-31 (Operator Support Access) | Navigation (inbound) | Dana's read-only session entry point |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| client_billing_details_saved | fields changed, completeness result (complete/incomplete) | Billing section save completes | N/A -- billing completeness is measured by FEAT-01.SPEC-010's own coverage, not by "Client and Project Setup Speed," which measures initial add-to-roster time only |
| client_archived | had open items at archive time (yes/no) | Archive completes | N/A -- no success-metrics.md metric tracks archive activity for this feature |
| client_deleted | -- | Delete completes | N/A -- no success-metrics.md metric tracks delete activity for this feature |

## Acceptance Criteria

**FEAT-01.SPEC-004-AC-01:** Given Nadia is on Client Detail for "Acme Studio," when she renames it to "Acme Studio Inc." and confirms, then the header updates and a "Client renamed" toast appears.

**FEAT-01.SPEC-004-AC-02:** Given Nadia edits the billing address and taps section Save, when the save succeeds, then the completeness indicator updates to reflect whether billing details are now complete.

**FEAT-01.SPEC-004-AC-03:** Given Nadia taps Archive on a client with no unpaid invoices or pending approvals, then the client archives immediately with a "Client archived" toast and no confirmation dialog.

**FEAT-01.SPEC-004-AC-04:** Given Nadia taps Archive on a client with an unpaid invoice, then FEAT-01.SPEC-007's confirmation dialog appears summarizing the open item before the archive completes.

**FEAT-01.SPEC-004-AC-05:** Given Nadia taps Reactivate on an archived client while she is already at her plan's active-client limit, then the action is blocked with "You've reached your plan's active client limit. Upgrade to reactivate this client." and the client stays Archived.

**FEAT-01.SPEC-004-AC-06:** Given Nadia opens the overflow menu on a client with a sent invoice, when she looks at Delete, then it is disabled with the text "This client has a sent proposal, invoice, or activity and can only be archived."

**FEAT-01.SPEC-004-AC-07:** Given Nadia taps Delete on a client with no sent proposal, invoice, or activity, when she confirms, then the client is permanently deleted and she returns to the roster with a "Client deleted" toast.

**FEAT-01.SPEC-004-AC-08:** Given the client's record changed in another session since Nadia loaded this screen, when she attempts to Archive, then the action is rejected with "This client's records changed since you loaded this page. Refresh to see the latest state before archiving/deleting."

**FEAT-01.SPEC-004-AC-09:** Given Dana is viewing this client in a read-only support session, when she looks for Rename, billing Save, Archive, Reactivate, or Delete, then none of those controls are rendered.

**FEAT-01.SPEC-004-AC-10:** Given Nadia loses connectivity while editing billing details, then the offline banner appears, Save is disabled, and her in-progress edits are preserved.

**FEAT-01.SPEC-004-AC-11:** Given Nadia is on Client Detail, when she taps "Manage Contacts," then she is taken to Client Contact Management & Roles (FEAT-18) for this client.

**FEAT-01.SPEC-004-AC-12:** Given Client Detail fails to load, then an error banner "Couldn't load this client. Try again." appears with a Retry button.

**FEAT-01.SPEC-004-AC-13:** Given Nadia taps "New Project" from Client Detail, then she is taken to FEAT-01.SPEC-002 (Create Project) pre-scoped to this client.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 5 | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

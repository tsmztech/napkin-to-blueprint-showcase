---
document_type: spec
spec_type: screen
spec_id: FEAT-01.SPEC-001
spec_name: Service List
spec_slug: service-list
parent_feature: FEAT-01
parent_feature_name: Service & Pricing Management
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-26
acceptance_criteria_count: 10
---

# Screen Spec: Service List

## Overview

**Name:** Service List
**ID:** FEAT-01.SPEC-001
**Type:** Screen
**Purpose:** The Pro views their full service catalog, reorders how services appear on the public booking page, switches between Active and Archived views, and reactivates an archived service.
**Parent Feature:** FEAT-01 -- Service & Pricing Management

## Scope and Non-Goals

**In Scope:**
- Displaying the Pro's services in display order, filtered by Active or Archived status
- Drag-to-reorder of Active services, persisting the new display_order
- Reactivating an Archived service back to Active
- The guided empty state for a Pro with zero services
- Entry point for creating a new service and for opening an existing one for edit

**Non-Goals:**
- Creating a new service -- handled by FEAT-01.SPEC-002 (Add Service); this screen only launches it
- Editing service fields or archiving a service -- handled by FEAT-01.SPEC-003 (Edit Service); this screen only launches it
- Multi-staff or shared service catalogs -- excluded per scope-boundaries.md SC-01: the product is strictly single-operator, so this list always shows exactly one Pro's own services with no staff-level partitioning
- Cross-pro visibility of any kind -- excluded per scope-boundaries.md SC-03: there is no shared or comparative view of any pro's service list across accounts

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| Default entry | Pro opens the Services & Pricing area from their main navigation | None -- loads the Pro's own services, Active view by default |
| FEAT-01.SPEC-002 (Add Service) | Save completes | None -- returns to Active view with the new service visible |
| FEAT-01.SPEC-003 (Edit Service) | Save completes, or an archive is confirmed | None -- returns to the view (Active/Archived) the archived or edited service now belongs to |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen -- Active and Archived views, both scoped to her own account | Reorder Active services, switch filters, reactivate Archived services, open Add Service, open any service row | -- |
| Platform Operator (Support) | Full screen, read-only, for the Pro account under an active help request | None -- drag handles, the Add Service button, and the Reactivate action are not shown | Support's attempt to act is impossible because the controls are not rendered; there is no separate denial message because nothing actionable is ever offered |
| The Client (Riley) | No | No | Service & Pricing Management screens are never reached through any Client-facing path; a Client browsing the public booking page (FEAT-05) sees only the resulting Active service list, never this setup screen |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (XBR-29); after signing in, the Pro lands on their Daily Schedule Dashboard (FEAT-12), not directly back on this screen |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- any in-progress reorder drag is discarded (the last persisted display_order stands); after re-authentication the Pro returns to the Daily Schedule Dashboard (FEAT-12) |

## Layout and Content

**Header:** Screen title "Services & Pricing." An "Add Service" action button sits at the top-right (Pro only -- not shown to Support). Below the title, a two-option filter toggle: "Active" (default selected) and "Archived."

**Body:** A vertical list of service rows, one per service, ordered by display_order (Active view) or by the order they were archived, most recent first (Archived view). Each row shows, left to right: a drag handle (Active view only, Pro only), the service name, its price, its duration, and a compact summary of its deposit rule (e.g., "$X deposit" or "X% deposit"). In the Archived view, the drag handle is replaced by a "Reactivate" action button on each row. Tapping anywhere else on a row (outside the drag handle or Reactivate button) opens that service in FEAT-01.SPEC-003 (Edit Service).

**Footer:** None -- Add Service is in the header.

### Responsive Behavior

- **Compact breakpoint:** Single-column list as described above, full width; the header's filter toggle and Add Service button remain visible without scrolling.
- **Medium size class and above:** The list remains single-column, capped at a consistent platform-wide content width and horizontally centered; no structural change beyond width capping. Row content (name, price, duration, deposit summary) lays out on one line instead of wrapping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Add Service button (Pro only) | Tap | Navigate to FEAT-01.SPEC-002 (Add Service) | Screen closes | Standard forward transition |
| Active/Archived filter toggle | Tap | Switches the displayed list between Active and Archived services | List re-renders with the other filter's services; drag handles or Reactivate buttons swap accordingly | Selected toggle option is visually indicated |
| Service row (outside drag handle/Reactivate) | Tap | Navigate to FEAT-01.SPEC-003 (Edit Service) for that service | Screen closes | Standard forward transition |
| Drag handle (Active view, Pro only) | Press and drag | Reorders the dragged service among the Active list; on drop, persists the new display_order for every Active service whose position changed | Row moves to the dropped position immediately (optimistic) | Row settles into its new position; no confirmation dialog |
| Reactivate button (Archived view, Pro only) | Tap | Sets the service's status to Active and its display_order to the end of the current Active list | Row disappears from the Archived view | Toast: "{Service name} reactivated -- it's back on your booking page." |
| Add Service button (Support) | -- | Not rendered for this role | -- | -- |

### Accessibility Notes

- **Focus order:** Filter toggle (Active, then Archived) -> Add Service button -> service rows in display order, each row's drag handle or Reactivate button reachable before the row's own open-for-edit target.
- **Reorder announcements:** A drag-to-reorder move is announced to assistive technology as "{Service name} moved to position {N} of {total}"; a keyboard-based reorder alternative (move up / move down controls, always visible when a screen reader is active) reaches the same result without requiring a pointer drag.
- **Reactivate feedback:** The reactivation toast is announced on success.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard, including reordering (see above); there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (first-time) | Guided prompt: "Add your first service to start taking bookings" with a prominent "Add Service" action, in place of a blank Active list | The Pro's account has zero services of any status | The Pro saves their first service (FEAT-01.SPEC-002) |
| Loaded (Active) | Service rows in display order, drag handles visible | Default entry, or return from Add/Edit with at least one service | Filter toggled to Archived, or screen closed |
| Loaded (Archived, empty) | Message: "No archived services" in place of a blank list | Archived filter selected and the Pro has zero archived services | A service is archived (FEAT-01.SPEC-003) or the filter is switched back |
| Loaded (Archived, populated) | Service rows most-recently-archived first, Reactivate buttons visible | Archived filter selected and at least one archived service exists | Filter toggled to Active, or screen closed |
| Error | Error banner at the top of the list: "Couldn't load your services. Try again." with a Retry action | The initial list load fails | Pro taps Retry and the load succeeds |
| Offline/Degraded | N/A -- this is a setup screen used on a stable connection between clients, not a mobile in-the-moment flow (product-features.md, States field); no offline behavior is defined for it |

## Validation Rules

No field validation applies to this screen -- it captures no free-form input. The only user-entered value is the reorder position (an implicit integer position derived from drop location), which has no format constraint: any position within the current Active list length is valid.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Add Service button tap | FEAT-01.SPEC-002 (Add Service) | -- |
| Service row tap | FEAT-01.SPEC-003 (Edit Service) | -- |

## Data Model

**Creates:** None.
**Reads:** Service -- all fields (name, price, duration, deposit_rule, display_order, status), scoped to the signed-in Pro Account (or, for Support, the Pro Account under an open help request).
**Updates:** Service.display_order (drag-to-reorder, Active view only); Service.status (Reactivate action sets Archived -> Active and appends display_order to the end of the Active list).
**Deletes:** None -- archived services are never hard-deleted while any Booking references them (dependency map, Service Lifecycle).

## Business Rules

- Only the Pro can reorder or reactivate; Support's access is View-only across every action on this screen (Access Matrix, Service & Availability Setup group).
- Reactivating a service does not restore its previous display_order -- it is appended to the end of the current Active list, so the Pro's remaining order is never disturbed by a reactivation.
- Archived services are retained indefinitely and remain reachable through this screen's Archived filter -- no automatic purge exists (dependency map's Service Lifecycle line, consistent with scope-boundaries.md SC-22's multi-year history-retention policy).
- XBR-11: a service reactivated or reordered never silently affects any confirmed booking -- reorder and reactivate only change what is offered to new clients going forward.

## Edge Cases

- **Pro reorders services from two signed-in devices at nearly the same time** -- The dependency map's Service Contention note applies: last-write-wins between the Pro's own sessions. The device whose reorder persists last determines the final display_order; the other device's list silently refreshes to the current order on its next load or reopen.
- **Pro double-taps Reactivate on the same row rapidly** -- The second tap is a no-op: the first tap's request is already in flight, and the button is disabled until the reactivation completes or fails.
- **Zero services after all have been archived** -- The Active view shows the Empty (first-time) guided state exactly as if no service had ever been added, since it is driven purely by "zero Active services," not by whether any Archived services exist.
- **List load fails while the Archived filter is selected** -- The Error state and Retry action apply identically to either filter; retrying reloads whichever filter was active when the error occurred.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-002 (Add Service) | Navigation (outbound) | Add Service button opens the creation form |
| FEAT-01.SPEC-003 (Edit Service) | Navigation (outbound) | Tapping a service row opens it for edit or archive |
| FEAT-01.SPEC-002 (Add Service) | Navigation (inbound) | Successful save returns here |
| FEAT-01.SPEC-003 (Edit Service) | Navigation (inbound) | Successful save or confirmed archive returns here |
| FEAT-01.SPEC-004 (Service Field & Deposit Rule Validation) | Enforced rule (inbound) | This screen is an enforcing spec of that rule: it applies the rule's authorization check (Pro only) to the Reorder and Reactivate actions; no field validation is checked here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| service_reordered | count of services whose display_order changed | A drag-to-reorder drop persists | N/A -- no success metric in success-metrics.md tracks list-organization behavior for this feature; success-metrics.md's "Service Setup Confidence" (this feature's only connected metric) measures the add/edit attempt itself, captured in FEAT-01.SPEC-002 and FEAT-01.SPEC-003 |
| service_reactivated | none beyond the event itself | Reactivate completes successfully | N/A -- same reason as above |

## Acceptance Criteria

**FEAT-01.SPEC-001-AC-01:** Given Talia has zero services, when she opens Services & Pricing, then she sees the guided "Add your first service to start taking bookings" prompt instead of a blank list.

**FEAT-01.SPEC-001-AC-02:** Given Talia has three Active services, when she opens Services & Pricing, then she sees all three in their saved display order with the Active filter selected by default.

**FEAT-01.SPEC-001-AC-03:** Given Talia is viewing her Active services, when she drags the third service above the first, then the new order persists and reopening the screen later shows the same new order.

**FEAT-01.SPEC-001-AC-04:** Given Talia taps the Archived filter and she has one archived service, when the view switches, then she sees that service with a Reactivate button in place of a drag handle.

**FEAT-01.SPEC-001-AC-05:** Given Talia is viewing an archived service, when she taps Reactivate, then the service disappears from the Archived view, a toast confirms "{Service name} reactivated -- it's back on your booking page," and the service reappears at the end of the Active view.

**FEAT-01.SPEC-001-AC-06:** Given Talia taps a service row, when the tap registers outside the drag handle, then she is taken to FEAT-01.SPEC-003 (Edit Service) for that service.

**FEAT-01.SPEC-001-AC-07:** Given the initial service list fails to load, when the screen opens, then an error banner reads "Couldn't load your services. Try again." with a Retry action, and tapping Retry reloads the list.

**FEAT-01.SPEC-001-AC-08:** Given Platform Operator (Support) opens a Pro's Services & Pricing screen during a help request, when the screen renders, then no drag handles, Add Service button, or Reactivate buttons appear anywhere on the screen.

**FEAT-01.SPEC-001-AC-09:** Given Talia reorders her Active services from her phone while the same list is open on her tablet, when both reorders are saved moments apart, then the order from whichever save completed last is the order both devices show after their next refresh.

**FEAT-01.SPEC-001-AC-10:** Given Talia's session has expired while she is mid-drag reordering a service, when the expiry is detected, then the dialog "Your session has expired. Sign in to continue." appears, the in-progress drag is discarded, and the last successfully persisted order stands.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 6 (empty, loaded-active, loaded-archived-empty, loaded-archived-populated, error, offline/degraded N/A) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 4 | 4 |

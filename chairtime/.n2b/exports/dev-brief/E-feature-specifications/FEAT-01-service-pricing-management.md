# FEAT-01 — Service & Pricing Management

This chapter covers Service & Pricing Management (FEAT-01), a Core-tier feature. It carries 6 specifications carrying 72 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-01.SPEC-001 | Service List | screen | 10 |
| FEAT-01.SPEC-002 | Add Service | screen | 10 |
| FEAT-01.SPEC-003 | Edit Service | screen | 11 |
| FEAT-01.SPEC-004 | Service Field & Deposit Rule Validation | logic-rule | 17 |
| FEAT-01.SPEC-005 | Price & Deposit Lock at Booking Time | logic-rule | 14 |
| FEAT-01.SPEC-006 | Archive Impact Check | automation | 10 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Service & Pricing Management

## Summary

**Feature:** Service & Pricing Management
**ID:** FEAT-01
**Description:** The Pro defines the services they offer -- name, price, duration, and the deposit rule for that service (fixed amount or percentage) -- and controls which services are currently bookable.
**Priority:** Core
**Phase:** MVP
**Type:** User-Facing
**Rationale:** Directly required by BRIEF.md's Vision: a client "picks a service" with "prices and how long each takes" stated in plain words before booking. Without this, the booking page has nothing to show. MVP: the product cannot function without at least one bookable service. [INFERRED: carried from Visionary draft]

**Key Capabilities:**
- Add a service -- name, price, duration, and its deposit rule (fixed amount or percentage of price)
- Edit a service -- update price, duration, or deposit rule for future bookings without altering past ones
- Archive a service -- hide it from new bookings while keeping history intact
- Reorder services as they appear on the booking page

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-01.SPEC-001 | Service List | Screen | The Pro, Platform Operator (Support) | Pro views, reorders, filters (Active/Archived), and reactivates services; entry point for the feature |
| FEAT-01.SPEC-002 | Add Service | Screen | The Pro | Pro creates a new service (name, price, duration, deposit rule) with an inline preview of its client-facing appearance |
| FEAT-01.SPEC-003 | Edit Service | Screen | The Pro, Platform Operator (Support) | Pro updates or archives an existing service; Support views service details read-only for troubleshooting |
| FEAT-01.SPEC-004 | Service Field & Deposit Rule Validation | Logic/Rule | The Pro | Validation rules governing name, price, duration, and deposit-rule integrity, including the minimum chargeable deposit |
| FEAT-01.SPEC-005 | Price & Deposit Lock at Booking Time | Logic/Rule | The Pro, The Client | Governs that editing or archiving a service never changes the price, duration, or deposit already agreed on a confirmed booking |
| FEAT-01.SPEC-006 | Archive Impact Check | Automation | The Pro | On an archive request, checks for upcoming bookings referencing the service and surfaces an impact warning before the Pro confirms |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Add a service -- name, price, duration, deposit rule | FEAT-01.SPEC-002 | Primary purpose of the Add Service screen | Phase 2 (Explicit) |
| Edit a service -- update price, duration, or deposit rule for future bookings without altering past ones | FEAT-01.SPEC-003, FEAT-01.SPEC-005 | Edit Service screen performs the update; the lock rule guarantees confirmed bookings keep their originally agreed price and deposit | Phase 2 (Explicit) / Phase 5 (Rule Discovery) |
| Archive a service -- hide it from new bookings while keeping history intact | FEAT-01.SPEC-003, FEAT-01.SPEC-006, FEAT-01.SPEC-001 | Archive action on the Edit screen; impact check warns of upcoming bookings before confirming; List screen's Archived filter keeps history visible and reachable | Phase 2 (Explicit) / Phase 4 (Trigger-Response) |
| Reorder services as they appear on the booking page | FEAT-01.SPEC-001 | Drag-to-reorder action on the Service List, persisting display_order | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-01.SPEC-004 | Service Field & Deposit Rule Validation | Phase 5 (Rule Discovery) | The Validation & Limits field names five or more distinct rules on the Service entity (name length, positive price, duration steps/ceiling, deposit ceiling by form, minimum chargeable deposit) -- past the inline-validation threshold, so it becomes a standalone Logic/Rule spec shared by Add and Edit |
| FEAT-01.SPEC-005 | Price & Deposit Lock at Booking Time | Phase 5 (Rule Discovery) | Elaborates XBR-04 and the Alternate flow ("existing confirmed bookings keep the price and deposit that were agreed at booking time") -- a rule shared across editing and archiving, not owned by either screen alone |
| FEAT-01.SPEC-006 | Archive Impact Check | Phase 4 (Trigger-Response) | The Archive alternate flow ("Pro archives a service with upcoming bookings; those bookings are honored... but the service disappears from new booking choices") implies a cross-entity check against Booking before the archive is confirmed -- too complex to stay inline per the standalone-Automation decision rule |

## Entity-Lifecycle Coverage Matrix

**Entity: Service**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-01.SPEC-002 | Add Service screen -- Pro fills the form and saves; validated by SPEC-004 | Also created by FEAT-15 for the first service during setup, using this same screen |
| Read (single) | FEAT-01.SPEC-003 | Edit Service screen -- loads the existing service's fields | -- |
| Read (list) | FEAT-01.SPEC-001 | Service List screen -- Active and Archived views | -- |
| Update | FEAT-01.SPEC-003 | Edit Service screen -- Pro modifies name, price, duration, or deposit rule and saves; validated by SPEC-004; governed by SPEC-005 | Reorder is a narrower update (display_order only), covered separately below |
| Delete/Archive | FEAT-01.SPEC-003, FEAT-01.SPEC-006 | Soft delete only -- status set to Archived; restore path: Pro reactivates from the Archived filter on FEAT-01.SPEC-001, resetting status to Active; cascade: none -- referencing Bookings are never altered (SPEC-005); retention/purge: retained indefinitely, never hard-deleted while any Booking references it (dependency map's Service Lifecycle line, consistent with SC-22's history-retention policy) -- no automatic purge, recorded as an explicit non-goal below | SPEC-006 runs before the archive is confirmed to warn of upcoming bookings |
| State Transition | FEAT-01.SPEC-003, FEAT-01.SPEC-001 | Active <-> Archived, toggled by Archive (SPEC-003) and Reactivate (SPEC-001) | No other Service states exist in the product definition |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Pro Account | FEAT-01.SPEC-001, FEAT-01.SPEC-002, FEAT-01.SPEC-003 | Every service belongs to and is scoped by exactly one signed-in Pro Account |
| Booking | FEAT-01.SPEC-006, FEAT-01.SPEC-005 | Archive Impact Check counts upcoming bookings referencing the service; the lock rule references each booking's fixed price/deposit/duration fields to confirm they are never rewritten by a later Service edit |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Pro saves a new service (Add) | Validate name, price, duration, and deposit rule (including minimum chargeable deposit) | Standalone Logic/Rule | FEAT-01.SPEC-004 |
| Pro saves an edited service | Validate fields; confirmed bookings that already reference this service are never recalculated | Standalone Logic/Rule | FEAT-01.SPEC-004, FEAT-01.SPEC-005 |
| Pro requests to archive a service | Check for upcoming bookings referencing the service and show an impact warning before the archive is confirmed | Standalone Automation | FEAT-01.SPEC-006 |
| Pro confirms an archive | Service status set to Archived; hidden from new booking choices; any existing or upcoming bookings keep their originally agreed price, duration, and deposit unchanged | Standalone Logic/Rule (elaboration), inline action in the triggering screen | FEAT-01.SPEC-005 / FEAT-01.SPEC-003 |
| Pro archives a service that has upcoming bookings | The resulting conflict is flagged on the Pro's booking management surface for an explicit Pro decision -- this feature never auto-resolves it | Cross-feature | FEAT-30 responsibility (per XBR-11) |
| Pro drags to reorder services | Persist new display_order; if two of the Pro's own sessions reorder concurrently, the last write wins | Inline in triggering screen | FEAT-01.SPEC-001 |
| Pro reactivates an archived service | Status reset to Active; service reappears in new booking choices | Inline in triggering screen | FEAT-01.SPEC-001 |
| Feature needs a per-account, searchable record store for Service data (ASMP-34) | Service records persist and are read back across sessions and devices | Inline in Create/Read/Update specs | FEAT-01.SPEC-001 / FEAT-01.SPEC-002 / FEAT-01.SPEC-003 |

ASMP-34 is a product-wide storage dependency spanning services, bookings, clients, and payment outcomes -- not a category-level external capability specific to this feature (unlike, say, payment processing or transactional email). Ordinary CRUD persistence in SPEC-001/002/003 satisfies it, so this feature carries no dedicated Integration spec; `integration_count: 0` is intentional, not an omission. The feature's Communications field is explicitly N/A ("this is a setup action with no notification of its own"), so `notification_count: 0` is likewise intentional -- no side-effect in this table sends a message to a person.

## Shared Context

**Shared Entities:**
- Service -- created by SPEC-002; read and updated by SPEC-003; listed, reordered, and reactivated by SPEC-001; validated by SPEC-004; governed at booking time by SPEC-005; checked for archive impact by SPEC-006. Fields: name, price, duration, deposit_rule, buffer_override (set elsewhere, in FEAT-02 -- read-only here), display_order, status.

**Shared UI Patterns:**
- Service form -- shared by SPEC-002 (create, starts empty, shows a live client-facing preview) and SPEC-003 (edit, starts pre-populated with the existing values). Same fields, same layout, same validation. Spec Writers for both screens should describe the form consistently.
- Empty state -- SPEC-001 shows a guided "add your first service" prompt when the Pro's account has zero services (States field), rather than a blank list.

**Shared Validation:**
- SPEC-004 defines all field-level and deposit-rule validation. SPEC-002 and SPEC-003 both reference it for validation behavior rather than duplicating the rules.

## Internal Dependency Map

```
SPEC-001 (Service List) -> [Pro taps "Add Service"] -> SPEC-002 (Add Service)
SPEC-001 (Service List) -> [Pro taps a service row] -> SPEC-003 (Edit Service)
SPEC-002 (Add Service) -> [Pro taps Save] -> SPEC-004 (Service Field & Deposit Rule Validation) -> [valid] -> SPEC-001
SPEC-003 (Edit Service) -> [Pro taps Save] -> SPEC-004 (Service Field & Deposit Rule Validation) -> [valid] -> SPEC-001
SPEC-003 (Edit Service) -> [Pro saves a price/duration/deposit change] -> SPEC-005 (Price & Deposit Lock at Booking Time)
SPEC-003 (Edit Service) -> [Pro taps Archive] -> SPEC-006 (Archive Impact Check) -> [Pro confirms] -> SPEC-005 (Price & Deposit Lock at Booking Time) -> SPEC-001
SPEC-001 (Service List) -> [Pro drags to reorder] -> SPEC-001 (persists display_order, inline)
SPEC-001 (Service List) -> [Pro switches to Archived filter, taps Reactivate] -> SPEC-001 (status reset to Active, inline)
```

**Default Entry:** SPEC-001 (Service List) -- the screen shown when the Pro navigates to Services & Pricing.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-01.SPEC-002 | Outbound | FEAT-05 (Public Booking Page & Booking Flow) | A newly added or edited Active service appears in the client-facing service list | Pro saves a new or edited service |
| FEAT-01.SPEC-003 | Outbound | FEAT-03 (Real-Time Slot Availability Engine) | Service duration drives slot length | Pro edits a service's duration |
| FEAT-01.SPEC-003 | Outbound | FEAT-07 (Deposit Payment at Booking) | Deposit rule (fixed amount or percentage) feeds the deposit computed at booking time | Pro sets or edits a service's deposit rule |
| FEAT-01.SPEC-003 | Outbound | FEAT-30 (Pro Booking Management) | Archiving a service that has upcoming bookings surfaces a flagged conflict for the Pro to resolve explicitly | Pro archives a service with upcoming bookings |
| FEAT-01.SPEC-002 | Inbound | FEAT-15 (Pro Onboarding & Setup Wizard) | The first service is added during setup using this feature's Add Service screen | Pro reaches the services step of first-time setup |

## Non-Functional Notes

**Data volumes / growth:** A Pro's service list stays small -- a handful to a few dozen entries (feature's States field) -- even as the account otherwise accumulates 100-500 clients and 20-40 bookings a week over multiple years (ASMP-22); no growth pattern here threatens responsiveness.

**Responsiveness:** Service lists load instantly with no loading state needed (States field: "N/A -- service lists are small... and load instantly"); because slot availability (FEAT-03) and the public booking page (FEAT-05) read service data live, a save or edit here must be reflected without perceptible delay to keep the product's under-one-minute booking benchmark intact (ASMP-21).

**Data sensitivity / privacy:** None -- service names, prices, and durations are public commercial information intentionally shown on the booking page (Data Notes field; dependency map's Service entity Data Sensitivity line). No personal data is captured or stored by this feature.

**Compliance flags:** N/A -- assumptions-constraints.md names no compliance regime specific to service or pricing configuration; timezone and currency remain per-account settings owned by Pro Account (FEAT-27, per XBR-25), not fields this feature defines.

## Non-Goals

- **Multi-staff or shared service catalogs** -- Excluded per SC-01: the product is strictly single-operator, so each Pro Account has exactly one service list with no staff-level partitioning or shared ownership.
- **Dynamic, demand-based, or time-of-day pricing** -- Excluded per SC-14: each service carries one fixed price; varying it by demand or time of day is explicitly out of scope for the setup this feature provides.
- **Handling or storing card data as part of deposit-rule configuration** -- Excluded per SC-11: this feature only defines the deposit rule (fixed amount or percentage); actual card capture is a hard boundary owned entirely by the payment-processing capability behind FEAT-07.
- **Automatic purge of archived services** -- Intentional lifecycle decision from the Entity-Lifecycle Coverage Matrix: archived services are retained indefinitely because the dependency map states a service is "never hard-deleted while bookings reference it," consistent with SC-22's multi-year history-retention policy. No purge window applies.
- **Cross-pro visibility of service catalogs** -- Excluded per SC-03: there is no shared, comparative, or cross-account view of any pro's services; each Pro Account's list is visible only to that Pro (Full access) and, read-only, to Platform Operator (Support) for troubleshooting.



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



# Screen Spec: Add Service

## Overview

**Name:** Add Service
**ID:** FEAT-01.SPEC-002
**Type:** Screen
**Purpose:** The Pro creates a new bookable service by entering its name, price, duration, and deposit rule, with an inline preview of how it will appear on the client-facing booking page.
**Parent Feature:** FEAT-01 -- Service & Pricing Management

## Scope and Non-Goals

**In Scope:**
- Capturing name, price, duration, and deposit rule (fixed amount or percentage) for a new service
- Inline field validation and a live client-facing preview as the Pro types
- Saving the new service as Active, appended to the end of the Pro's display order
- The entry path used both from the Service List and from first-time setup (FEAT-15)

**Non-Goals:**
- Editing an existing service, or archiving one -- handled by FEAT-01.SPEC-003 (Edit Service)
- Setting the per-service buffer override -- owned entirely by FEAT-02 (Availability & Working Hours Setup); this screen never shows or captures that field
- Handling or storing card data as part of the deposit rule -- excluded per scope-boundaries.md SC-11: this screen only defines the deposit rule (fixed amount or percentage); actual card capture belongs to the payment-processing capability behind FEAT-07
- Dynamic, demand-based, or time-of-day pricing -- excluded per scope-boundaries.md SC-14: each service carries exactly one fixed price

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-001 (Service List) | Pro taps "Add Service" | None -- form starts empty |
| FEAT-15.SPEC-001 (Setup Wizard Shell) (Pro Onboarding & Setup Wizard) | Pro reaches the services step of first-time setup | None -- form starts empty; on save, the Pro continues into the next onboarding step (working hours) instead of returning to FEAT-01.SPEC-001 |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Fill and save the form | -- |
| Platform Operator (Support) | No | No | Support's View access to Service & Availability Setup applies to existing service records (FEAT-01.SPEC-001, FEAT-01.SPEC-003) for troubleshooting; there is nothing yet to troubleshoot on a creation form, so this screen is never opened in a support context and no control is rendered for it |
| The Client (Riley) | No | No | Service & Pricing Management screens are never reached through any Client-facing path |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (XBR-29) |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- entered form data is preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Add Service" with a back arrow (returns to the entry source) and a "Save" action button (right-aligned).

**Body:** A single-column form with the following fields in order:
- Name (text input, required)
- Price (numeric input, required, in the Pro Account's currency)
- Duration (numeric input in minutes, required, with quick-select common durations)
- Deposit Rule (a two-way toggle between "Fixed amount" and "Percentage," followed by the corresponding numeric input)

Below the form, a live preview panel shows exactly what a client will see on the booking page for this service: name, price, duration, and the deposit amount that results from the current price and deposit rule, recalculated as the Pro types.

**Footer:** None -- Save is in the header.

### Responsive Behavior

- **Compact breakpoint:** Single-column form as described above, full width; the preview panel appears below the form, stacked.
- **Medium size class and above:** Form and preview panel appear side by side (form on the left, live preview on the right), the whole layout capped at a consistent platform-wide content width and horizontally centered.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to the entry source (FEAT-01.SPEC-001, or the next-earlier onboarding step if entered from FEAT-15) | Screen closes | Standard back transition |
| Name input | Type | Captures text input | Field shows entered text; preview panel's name updates live | Standard input focus state |
| Name input | Blur | Triggers field validation via FEAT-01.SPEC-004 | Error state on field if invalid | Field-level error message if invalid |
| Price input | Type | Captures numeric input | Field shows entered value; preview panel's price and computed deposit update live | Standard input focus state |
| Price input | Blur | Triggers field validation via FEAT-01.SPEC-004 | Error state on field if invalid | Field-level error message if invalid |
| Duration input | Type or quick-select | Captures duration in minutes | Field shows entered value; preview panel's duration updates live | Standard input focus state |
| Duration input | Blur | Triggers field validation via FEAT-01.SPEC-004 | Error state on field if invalid | Field-level error message if invalid |
| Deposit Rule toggle (Fixed/Percentage) | Tap | Switches the deposit input's mode; clears the previously entered deposit value | Deposit input relabels and resets | Preview panel's computed deposit updates to reflect the cleared value |
| Deposit value input | Type | Captures the fixed amount or percentage, per the toggle's mode | Field shows entered value; preview panel's computed deposit updates live | Standard input focus state |
| Deposit value input | Blur | Triggers field and cross-field validation via FEAT-01.SPEC-004 | Error state on field if invalid | Field-level error message if invalid |
| Save button | Tap | 1. Validate all fields and the deposit-rule cross-field rules via FEAT-01.SPEC-004. 2. If valid, create the Service record. | Button shows loading state during save | Success: toast "{Service name} added -- it's live on your booking page" and navigate per Entry Points context. Failure: inline error messages, entered values preserved. |
| Save button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> Name -> Price -> Duration -> Deposit Rule toggle -> Deposit value -> Save.
- **Validation announcements:** When a field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Preview updates:** The live preview panel is not part of the primary focus order and its updates are not separately announced on every keystroke, to avoid announcement noise; its final state is reachable on demand via a labeled "Preview" landmark.
- **Save feedback:** The success toast is announced on save; on validation failure, focus moves to the first field in error.
- **Keyboard alternatives:** Every action on this screen, including the Deposit Rule toggle, is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (default) | All form fields empty, preview panel shows a placeholder card, Save enabled | Screen first opens | Pro begins typing in any field |
| Filling | Form fields contain Pro input, preview panel reflects current values, Save enabled | Pro types in any field | Pro taps Save or navigates away |
| Validation Error | Failed fields highlighted with error messages below them | Validation fails on blur or submit | Pro corrects the field and re-triggers validation |
| Saving | Save button shows a loading state, form fields disabled | Validation passes | Save completes or fails |
| Error | Error banner at the top of the form: "Couldn't save this service. Try again." with a Retry action | Save operation fails | Pro taps Retry and the save succeeds |
| Offline/Degraded | N/A -- this is a setup screen used on a stable connection between clients, not a mobile in-the-moment flow (product-features.md, States field) |

## Validation Rules

Validation governed by FEAT-01.SPEC-004 (Service Field & Deposit Rule Validation). See that spec for all field-level and cross-field rules, including the minimum chargeable deposit. This screen applies validation on field blur and on form submit.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-01.SPEC-001 (Service List) | -- |
| Back arrow tap (entered from onboarding) | Previous onboarding step | FEAT-15 (Pro Onboarding & Setup Wizard) |
| Successful save (entered from list) | FEAT-01.SPEC-001 (Service List) | -- |
| Successful save (entered from onboarding) | Next onboarding step (working hours) | FEAT-15 (Pro Onboarding & Setup Wizard) |
| Cancel (if unsaved changes) | Entry source, after confirmation | -- |

## Data Model

**Creates:** Service record -- name, price, duration, and deposit_rule set from form input; status set to Active automatically; display_order appended to the end of the Pro's current Active list automatically; buffer_override left unset (owned by FEAT-02).
**Reads:** None -- this is a creation screen; no existing data loaded.
**Updates:** None.
**Deletes:** None.

## Business Rules

- Field validation and the minimum chargeable deposit are enforced by FEAT-01.SPEC-004 -- the Pro cannot save with invalid or unchargeable values.
- XBR-25: price is entered and displayed in the Pro Account's currency (owned by FEAT-27); this screen never lets the Pro choose a different currency per service.
- Once saved, the service is immediately visible on the public booking page (FEAT-05) -- there is no separate publish step.
- A service created here is, from that moment, subject to FEAT-01.SPEC-005 (Price & Deposit Lock at Booking Time): any future edit or archive of this service will never retroactively change a booking confirmed against it.

## Edge Cases

- **Pro navigates away with unsaved changes** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Pro taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in loading state).
- **Network or save failure** -- Error banner: "Couldn't save this service. Try again." with a Retry button. All entered values are preserved.
- **Pro switches the Deposit Rule toggle after entering a value** -- The previously entered deposit value is cleared (not silently reinterpreted under the new mode), and the preview panel's computed deposit reflects the cleared value until the Pro re-enters one.
- **Two identically named services** -- No uniqueness rule applies to the service name (FEAT-01.SPEC-004 defines no name-uniqueness rule); the save proceeds and the Pro's list simply shows two rows with the same name, distinguished by their other fields.
- **Another service is created or edited by the Pro from a second device while this form is open** -- No live conflict on this creation screen: no existing record is loaded, and this screen's own save only ever creates a new record, so it cannot collide with any other in-flight change to a different or the same service.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-004 (Service Field & Deposit Rule Validation) | References (inbound) | Validation and the minimum-chargeable-deposit rule applied to form fields |
| FEAT-01.SPEC-001 (Service List) | Navigation (inbound/outbound) | Pro arrives from the list and returns to it on save or back |
| FEAT-15.SPEC-001 (Setup Wizard Shell, Step Navigation & Guidance), FEAT-15.SPEC-004 (Setup Progress Tracking & Resume) -- within FEAT-15 (Pro Onboarding & Setup Wizard, Setup Wizard step) | Navigation (inbound/outbound) | Pro's first service is added here during setup, then continues into the next onboarding step |
| FEAT-01.SPEC-005 (Price & Deposit Lock at Booking Time) | References (outbound) | The service created here immediately becomes subject to the lock guarantee for any future booking |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| service_created | entry source (service list / onboarding), deposit rule form (fixed / percentage) | Successful save completes | supports success-metrics.md: "Service Setup Confidence" |
| service_create_validation_failed | field(s) in error, error type(s) | Save attempt is blocked by validation | supports success-metrics.md: "Service Setup Confidence" |
| service_create_abandoned | last field focused, count of fields filled | Pro discards unsaved changes via the back arrow | supports success-metrics.md: "Service Setup Confidence" |

## Acceptance Criteria

**FEAT-01.SPEC-002-AC-01:** Given Talia is on the Add Service screen, when she fills in "Classic Lash Set" as name, a positive price, a duration in 5-minute steps, and a valid deposit rule, and taps Save, then the service is created as Active, appended to the end of her service list, and she sees the toast "Classic Lash Set added -- it's live on your booking page" before returning to FEAT-01.SPEC-001.

**FEAT-01.SPEC-002-AC-02:** Given Talia is on the Add Service screen with the name field empty, when she taps Save, then the name field shows an error state and the save does not proceed.

**FEAT-01.SPEC-002-AC-03:** Given Talia enters a price and switches the Deposit Rule toggle from Fixed to Percentage, when the toggle switches, then any previously entered fixed-amount value is cleared and the preview panel's computed deposit updates to reflect the cleared value.

**FEAT-01.SPEC-002-AC-04:** Given Talia enters a price and a percentage deposit rule whose resulting deposit falls below the minimum chargeable amount, when she blurs the deposit field, then FEAT-01.SPEC-004's validation error appears and Save is blocked until she adjusts the price or the rule.

**FEAT-01.SPEC-002-AC-05:** Given Talia is on the Add Service screen with unsaved changes, when she taps the back arrow, then a confirmation dialog appears asking "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.

**FEAT-01.SPEC-002-AC-06:** Given Talia types values into the form, when each keystroke registers, then the live preview panel updates to show the name, price, duration, and computed deposit exactly as a client would see them.

**FEAT-01.SPEC-002-AC-07:** Given Talia reaches this screen from the services step of first-time setup (FEAT-15), when she saves a valid service, then she continues into the next onboarding step (working hours) rather than returning to FEAT-01.SPEC-001.

**FEAT-01.SPEC-002-AC-08:** Given Talia's save fails due to a save error, when the failure occurs, then an error banner reads "Couldn't save this service. Try again." with a Retry action, and all entered field values remain on screen.

**FEAT-01.SPEC-002-AC-09:** Given Talia's session expires while she has partially filled the form, when the expiry is detected, then the dialog "Your session has expired. Sign in to continue." appears, and her entered values are restored after she signs back in.

**FEAT-01.SPEC-002-AC-10:** Given Talia saves a second service with the same name as an existing one, when she taps Save, then the save proceeds without any duplicate-name warning, since no name-uniqueness rule exists.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 11 | 11 |
| States | 6 (empty, filling, validation error, saving, error, offline/degraded N/A) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Screen Spec: Edit Service

## Overview

**Name:** Edit Service
**ID:** FEAT-01.SPEC-003
**Type:** Screen
**Purpose:** The Pro updates an existing service's fields or archives it; Platform Operator (Support) views the same service details read-only for troubleshooting.
**Parent Feature:** FEAT-01 -- Service & Pricing Management

## Scope and Non-Goals

**In Scope:**
- Loading and displaying an existing service's fields, pre-populated
- Editing name, price, duration, or deposit rule and saving the change
- Initiating the Archive action, including the impact-check hand-off
- Support's read-only view of the same fields for troubleshooting

**Non-Goals:**
- Creating a new service -- handled by FEAT-01.SPEC-002 (Add Service), which shares this screen's form layout and validation
- Reordering services or reactivating an archived one -- both handled by FEAT-01.SPEC-001 (Service List)
- Determining whether upcoming bookings exist before an archive completes -- that check and its warning are owned by FEAT-01.SPEC-006 (Archive Impact Check); this screen only triggers it and reflects its result
- Retroactively changing the price, duration, or deposit already agreed on a confirmed booking -- explicitly prevented by FEAT-01.SPEC-005 (Price & Deposit Lock at Booking Time); no control on this screen can do this

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-001 (Service List) | Pro or Support taps a service row | The selected service's identifier; the screen loads that service's current fields |
| FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) | Support taps the Services entry during an active support session | The Pro account under review; screen renders read-only for Support with no action controls |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Edit any field, save, and Archive | -- |
| Platform Operator (Support) | Full screen, all fields, read-only | None -- no field is editable and no Save or Archive control is rendered; a persistent banner reads "Support view -- no changes can be made here" | A direct attempt to interact with a field is impossible because every input renders as static text in this role; there is no separate denial dialog because nothing actionable is ever offered |
| The Client (Riley) | No | No | Service & Pricing Management screens are never reached through any Client-facing path |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (XBR-29) |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- entered but unsaved edits are preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title showing the service's name, a back arrow (returns to FEAT-01.SPEC-001), and a "Save" action button (right-aligned, Pro only). Support sees no Save button; in its place, the "Support view -- no changes can be made here" banner spans the header's width.

**Body:** The same single-column form layout as FEAT-01.SPEC-002 (Add Service), pre-populated with the service's current values:
- Name (text input, required)
- Price (numeric input, required)
- Duration (numeric input in minutes, required)
- Deposit Rule (toggle between "Fixed amount" and "Percentage," with the corresponding value pre-filled)

Below the form, the same live client-facing preview panel as Add Service, reflecting the current (or in-progress edited) values.

For the Pro only, an "Archive this service" action appears below the preview panel, visually separated from the editable fields.

**Footer:** None -- Save is in the header; Archive is in the body.

### Responsive Behavior

- **Compact breakpoint:** Single-column form, full width; preview panel and Archive action stacked below it.
- **Medium size class and above:** Form and preview panel appear side by side, matching FEAT-01.SPEC-002's layout; the Archive action remains full-width below both, horizontally centered within the capped content width.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-01.SPEC-001 (Service List) | Screen closes | Standard back transition |
| Name/Price/Duration/Deposit Rule fields (Pro only) | Type / toggle | Same behavior as FEAT-01.SPEC-002 -- captures input, updates the preview panel live | Field shows entered value; preview updates | Standard input focus state |
| Any field (Pro only) | Blur | Triggers field validation via FEAT-01.SPEC-004 | Error state on field if invalid | Field-level error message if invalid |
| Save button (Pro only) | Tap | 1. Validate all fields via FEAT-01.SPEC-004. 2. If valid, save the changes; existing confirmed bookings are governed by FEAT-01.SPEC-005 and are never recalculated. | Button shows loading state during save | Success: toast "{Service name} updated" and navigate to FEAT-01.SPEC-001. Failure: inline error messages, entered values preserved. |
| Archive this service (Pro only) | Tap | Triggers FEAT-01.SPEC-006 (Archive Impact Check) | Screen shows a loading indicator on the Archive action while the check runs | If no upcoming bookings: proceeds directly to the archive confirmation step below. If upcoming bookings exist: an impact warning modal appears with the count and Confirm/Cancel |
| Impact warning modal -- Confirm (Pro only) | Tap | Sets the service's status to Archived per FEAT-01.SPEC-005's lock guarantee | Screen closes | Toast "{Service name} archived -- it's no longer bookable, and existing appointments are unaffected." Navigate to FEAT-01.SPEC-001 (Archived filter) |
| Impact warning modal -- Cancel (Pro only) | Tap | No action | Modal closes | Edit Service screen remains open, service remains Active |
| Fields (Support) | -- | Rendered as static text, not interactive | -- | -- |

### Accessibility Notes

- **Focus order:** Back arrow -> Name -> Price -> Duration -> Deposit Rule toggle -> Deposit value -> Archive this service -> Save. For Support, focus order skips directly from the banner to the back arrow, since no field or action is interactive.
- **Validation announcements:** Identical to FEAT-01.SPEC-002 -- error messages are announced to assistive technology and programmatically associated with their field.
- **Archive confirmation:** The impact warning modal, when shown, receives focus immediately and its full text (including the upcoming-booking count) is announced.
- **Read-only announcement:** For Support, the "Support view -- no changes can be made here" banner is announced when the screen first renders.
- **Keyboard alternatives:** Every action, including Archive and the impact-warning modal's Confirm/Cancel, is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Skeleton form while the service's current fields load | Screen first opens | Fields finish loading |
| Loaded/Filling (Pro) | Form pre-populated, editable, Save enabled | Load completes (Pro) | Pro edits, saves, archives, or navigates away |
| Read-only (Support) | Form pre-populated, all fields static text, banner visible | Load completes (Support) | Support navigates away |
| Validation Error | Failed fields highlighted with error messages below them | Validation fails on blur or submit | Pro corrects the field and re-triggers validation |
| Saving | Save button shows a loading state, form fields disabled | Validation passes | Save completes or fails |
| Archiving | Archive action shows a loading state while the impact check runs | Pro taps Archive this service | Impact check returns (no-warning path or warning modal shown) |
| Error | Error banner: "Couldn't save this service. Try again." (save) or "Couldn't check upcoming bookings for this service. Try again." (archive), each with Retry | Save operation or archive impact check fails | Pro taps Retry and the operation succeeds |
| Offline/Degraded | N/A -- this is a setup screen used on a stable connection between clients, not a mobile in-the-moment flow (product-features.md, States field) |

## Validation Rules

Validation governed by FEAT-01.SPEC-004 (Service Field & Deposit Rule Validation). See that spec for all field-level and cross-field rules, including the minimum chargeable deposit. This screen applies validation on field blur and on form submit, identically to FEAT-01.SPEC-002.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-01.SPEC-001 (Service List) | -- |
| Successful save | FEAT-01.SPEC-001 (Service List) | -- |
| Archive confirmed | FEAT-01.SPEC-001 (Service List, Archived filter) | -- |
| Cancel (if unsaved changes, Pro) | FEAT-01.SPEC-001 (Service List), after confirmation | -- |

## Data Model

**Creates:** None.
**Reads:** Service -- all fields (name, price, duration, deposit_rule, buffer_override, display_order, status). buffer_override is displayed nowhere on this screen (it is owned and shown by FEAT-02) -- the field is simply not part of this screen's data surface.
**Updates:** Service -- name, price, duration, deposit_rule (Pro only, subject to FEAT-01.SPEC-004 validation); status (Pro only, Active -> Archived via the Archive action, governed by FEAT-01.SPEC-005 and FEAT-01.SPEC-006).
**Deletes:** None -- archiving is a soft-delete (status change), never a hard delete (dependency map, Service Lifecycle).

## Business Rules

- Field validation is enforced by FEAT-01.SPEC-004 -- the Pro cannot save with invalid or unchargeable values.
- XBR-04: editing price, duration, or deposit rule here applies to future bookings only; any booking already Confirmed (or later in its lifecycle) keeps the price, duration, and deposit it was given at booking time, per FEAT-01.SPEC-005.
- Archiving always runs FEAT-01.SPEC-006 (Archive Impact Check) first -- the Pro cannot skip straight to archiving without the check completing.
- XBR-11: archiving a service with upcoming bookings never silently cancels them; those bookings are honored as-is, and the resulting conflict is flagged on the Pro's booking management surface (FEAT-30) for an explicit Pro decision -- this screen never auto-resolves it.
- Support's read-only access covers every field on this screen with no exception -- Service carries no personal data, so there is no field Support is additionally barred from (unlike, for example, a Pro's private client notes).

## Edge Cases

- **Service changed by the Pro from another device between load and save** -- Per the dependency map's Contention note for Service: resolution is last-write-wins between the Pro's own sessions (not reject-with-refresh); the save that completes last is the version that stands, and the earlier device's screen silently reflects the new values on its next load or reopen.
- **Pro archives a service with zero upcoming bookings** -- FEAT-01.SPEC-006 finds no upcoming bookings and the archive proceeds immediately with no warning modal; the Pro sees only the success toast.
- **Pro archives a service with upcoming bookings, then cancels the warning** -- The service remains Active and unarchived; no field on this screen changes.
- **Pro navigates away with unsaved field edits** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Pro taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in loading state).
- **Support opens a service that the Pro archives moments later from another device** -- Support's screen is a snapshot at load time; it does not live-update, so it continues showing the values as loaded until Support reopens the screen.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-004 (Service Field & Deposit Rule Validation) | References (inbound) | Validation and the minimum-chargeable-deposit rule applied to form fields |
| FEAT-01.SPEC-005 (Price & Deposit Lock at Booking Time) | References (inbound) | Governs that saved edits and confirmed archives never retroactively change an existing booking |
| FEAT-01.SPEC-006 (Archive Impact Check) | Triggers (outbound) | Archive action fires the impact check before any archive is confirmed |
| FEAT-01.SPEC-001 (Service List) | Navigation (inbound/outbound) | Pro or Support arrives from the list and returns to it on back, save, or confirmed archive |
| FEAT-30 (Pro Booking Management) | References (outbound) | The conflict from archiving a service with upcoming bookings is surfaced on that feature's booking management surface, per XBR-11 |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| service_updated | field(s) changed | Successful save completes | supports success-metrics.md: "Service Setup Confidence" |
| service_update_validation_failed | field(s) in error, error type(s) | Save attempt is blocked by validation | supports success-metrics.md: "Service Setup Confidence" |
| service_archived | had upcoming bookings at time of archive (yes/no) | Archive is confirmed | N/A -- success-metrics.md's "Service Setup Confidence" measures the add/edit attempt itself, not archive; no other metric connected to this feature covers archive behavior |

## Acceptance Criteria

**FEAT-01.SPEC-003-AC-01:** Given Talia opens an existing service, when the screen loads, then all fields are pre-populated with the service's current name, price, duration, and deposit rule.

**FEAT-01.SPEC-003-AC-02:** Given Talia edits the price of an existing service and taps Save, when the save succeeds, then the change is applied for future bookings, a toast confirms "{Service name} updated," and she returns to FEAT-01.SPEC-001.

**FEAT-01.SPEC-003-AC-03:** Given Talia edits a service that has an existing Confirmed booking, when she saves the change, then that booking's own price, duration, and deposit remain exactly as they were at booking time, per FEAT-01.SPEC-005.

**FEAT-01.SPEC-003-AC-04:** Given Talia taps "Archive this service" on a service with zero upcoming bookings, when FEAT-01.SPEC-006's check completes, then the archive proceeds immediately with no warning modal, and a toast confirms "{Service name} archived -- it's no longer bookable, and existing appointments are unaffected."

**FEAT-01.SPEC-003-AC-05:** Given Talia taps "Archive this service" on a service with two upcoming bookings, when FEAT-01.SPEC-006's check completes, then an impact warning modal appears naming the count, with Confirm and Cancel options, and the archive does not proceed until she taps Confirm.

**FEAT-01.SPEC-003-AC-06:** Given Talia sees the impact warning modal, when she taps Cancel, then the modal closes, the service remains Active, and no field on the screen has changed.

**FEAT-01.SPEC-003-AC-07:** Given Platform Operator (Support) opens a Pro's service during a help request, when the screen renders, then every field displays as static text, the banner "Support view -- no changes can be made here" is visible, and no Save or Archive control appears anywhere.

**FEAT-01.SPEC-003-AC-08:** Given Talia is editing a service on her phone while the same service is also open on her tablet, when she saves from the phone and then, moments later, saves a different change from the tablet, then the tablet's save is the version that stands, per the dependency map's last-write-wins resolution for Service.

**FEAT-01.SPEC-003-AC-09:** Given Talia is on the Edit Service screen with unsaved field edits, when she taps the back arrow, then a confirmation dialog appears asking "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.

**FEAT-01.SPEC-003-AC-10:** Given Talia's archive impact check fails to complete, when the failure occurs, then an error banner reads "Couldn't check upcoming bookings for this service. Try again." with a Retry action, and the service remains Active.

**FEAT-01.SPEC-003-AC-11:** Given Talia enters a deposit rule whose resulting deposit falls below the minimum chargeable amount and taps Save, then FEAT-01.SPEC-004's validation error appears and the save does not proceed.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 8 (loading, loaded/filling, read-only, validation error, saving, archiving, error, offline/degraded N/A) | 8 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Service Field & Deposit Rule Validation

## Overview

**Name:** Service Field & Deposit Rule Validation
**ID:** FEAT-01.SPEC-004
**Type:** Logic/Rule
**Purpose:** Defines every field-level validation rule, cross-field deposit-rule rule, authorization rule, and default/derivation for the Service entity, shared by the Add and Edit screens.
**Parent Feature:** FEAT-01 -- Service & Pricing Management
**Governed Entity:** Service

## Scope and Non-Goals

**In Scope:**
- Per-field validation rules for every Service field this feature owns (name, price, duration, deposit_rule, display_order, status)
- The cross-field deposit-rule rules, including the minimum chargeable deposit
- Authorization rules for every action this feature defines on the Service entity, per role
- Default values and derived fields on the Service entity
- Error messages for every validation failure

**Non-Goals:**
- Validating buffer_override -- that field is written exclusively by FEAT-02 (Availability & Working Hours Setup), through its own screen and its own Logic/Rule spec; this spec only notes its presence on the entity
- Determining whether an archive is safe to complete (counting upcoming bookings) -- that check is a standalone Automation, FEAT-01.SPEC-006 (Archive Impact Check); this spec governs field-level rules and authorization only
- Governing whether an edit or archive can retroactively change an already-confirmed booking -- excluded here and covered by FEAT-01.SPEC-005 (Price & Deposit Lock at Booking Time), which elaborates XBR-04 as its own standalone rule
- Dynamic, demand-based, or time-of-day pricing rules -- excluded per scope-boundaries.md SC-14: the product defines exactly one fixed price per service, so no rule set for variable pricing exists to validate

## Governed Entity

**Entity:** Service
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| name | text | The service's client-facing name |
| price | number | The fixed price of the service, in the Pro Account's currency |
| duration | number | The service's fixed length, in minutes |
| deposit_rule | enum + number | Either a fixed deposit amount (no greater than price) or a percentage (1-100%) of price |
| buffer_override | number (optional) | Optional per-service buffer in minutes; written only by FEAT-02, read by FEAT-03 -- out of scope for this spec's validation |
| display_order | number | The service's position on the booking page and on FEAT-01.SPEC-001 |
| status | enum (Active \| Archived) | The service's current lifecycle state |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-01.SPEC-002 | Add Service | On field blur and form submit; authorization on screen entry (Pro only) and on save |
| FEAT-01.SPEC-003 | Edit Service | On field blur and form submit; authorization on screen entry (edit rendering vs. read-only), on save, and on the Archive action |
| FEAT-01.SPEC-001 | Service List | Authorization only, for the Reorder and Reactivate actions -- no field validation is checked on this screen |
| FEAT-01.SPEC-006 | Archive Impact Check | Authorization only, for the underlying Archive action this automation gates -- the automation itself applies no field validation |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| name | Required, non-empty, 1-80 characters | Always | On blur and on submit | "Service name is required" / "Service name must be 80 characters or fewer" | Yes |
| price | Required, a positive amount greater than zero, in the Pro Account's currency | Always | On blur and on submit | "Enter a price greater than zero" | Yes |
| duration | Required, a positive number of minutes, in 5-minute steps, no greater than 720 minutes (12 hours) | Always | On blur and on submit | "Duration is required" / "Duration must be in 5-minute steps" / "Duration cannot exceed 12 hours" | Yes |
| deposit_rule (fixed form) | The fixed amount must be greater than zero and no greater than price | When the Pro has selected the Fixed amount form | On blur and on submit | "Deposit amount must be greater than $0 and no more than the service price" | Yes |
| deposit_rule (percentage form) | The percentage must be an integer between 1 and 100 | When the Pro has selected the Percentage form | On blur and on submit | "Deposit percentage must be between 1% and 100%" | Yes |
| display_order | No validation beyond data type -- assigned automatically on create (appended to the end of the Active list) and updated only by drag-reorder in FEAT-01.SPEC-001; never a raw user-entered value | Always | -- | -- | -- |
| status | No field-level validation -- this is a state transition (Active <-> Archived) governed by the Authorization Rules below and by FEAT-01.SPEC-006's archive-time processing, not by field format rules | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Fixed deposit ceiling | price, deposit_rule (fixed form) | The fixed deposit amount must be no greater than price -- a service can require up to (but never more than) full prepayment | "Deposit amount must be greater than $0 and no more than the service price" |
| Minimum chargeable deposit | price, deposit_rule | The deposit that results from the current price and deposit rule (the fixed amount as entered, or price x percentage / 100) must be at least platform parameter: `minimum-chargeable-deposit` -- the smallest amount a card payment can be taken for | "The deposit for this price and rule is below the minimum amount that can be charged. Increase the price, deposit amount, or percentage." |
| Deposit rule form exclusivity | deposit_rule (fixed form), deposit_rule (percentage form) | Exactly one form is active at a time; switching forms clears the other form's previously entered value rather than reinterpreting it | N/A -- this is a UI-state rule with no error condition, enforced as a field reset in FEAT-01.SPEC-002/FEAT-01.SPEC-003's Deposit Rule toggle interaction |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create service | The Pro | Always, for their own account only | -- |
| Create service | Platform Operator (Support) | Never | Add Service is never opened in a support context (Access Matrix: Service & Availability Setup = View); no control exists for Support to reach it |
| Create service | The Client | Never | Service & Pricing Management screens are unreachable through any Client-facing path |
| View service (list or detail) | The Pro | Always, for their own account's services only | -- |
| View service (list or detail) | Platform Operator (Support) | Always, read-only, for the Pro account under an active help request | -- |
| View service (list or detail) | The Client | Never (setup views) | The Client sees only the resulting public Active service list on the booking page (FEAT-05), never this feature's setup screens |
| Edit service fields | The Pro | Always, for their own account's services only | -- |
| Edit service fields | Platform Operator (Support) | Never | Every field renders as static text; banner reads "Support view -- no changes can be made here" |
| Edit service fields | The Client | Never | Not reachable, as above |
| Archive service | The Pro | Always, subject to completing the FEAT-01.SPEC-006 impact check and confirming | -- |
| Archive service | Platform Operator (Support) | Never | No Archive control is rendered for this role |
| Archive service | The Client | Never | Not reachable, as above |
| Reactivate service | The Pro | Always, for their own account's archived services only | -- |
| Reactivate service | Platform Operator (Support) | Never | No Reactivate control is rendered for this role |
| Reactivate service | The Client | Never | Not reachable, as above |
| Reorder services | The Pro | Always, for their own account's Active services only | -- |
| Reorder services | Platform Operator (Support) | Never | No drag handle is rendered for this role |
| Reorder services | The Client | Never | Not reachable, as above |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| status | Active | On create only | No -- a new service is always created Active; only the Archive action (via FEAT-01.SPEC-006) or Reactivate (via FEAT-01.SPEC-001) changes it afterward |
| display_order | Appended to the end of the Pro's current Active list | On create, and on Reactivate | No direct override -- the Pro repositions it afterward only via drag-reorder on FEAT-01.SPEC-001 |
| Computed deposit (preview only, not a stored field) | Fixed form: the entered amount, unchanged. Percentage form: price x percentage / 100, rounded to the currency's smallest unit | Recalculated live on every price or deposit-rule change, on both Add and Edit | No -- it is always derived from the current price and deposit_rule; the Pro can only change the inputs that feed it |

## Business Rules

- All validation rules in this spec apply identically on create (FEAT-01.SPEC-002) and edit (FEAT-01.SPEC-003) -- the product definition establishes no create-only or edit-only field rules.
- Field validation and the minimum-chargeable-deposit rule run before any save is attempted; a service is never persisted in an invalid state.
- XBR-25: price is always entered and displayed in the Pro Account's currency; this spec defines no per-service currency override, since currency is owned by FEAT-27 and locked once the first deposit is taken.
- The minimum-chargeable-deposit rule (platform parameter: `minimum-chargeable-deposit`) exists because a deposit below it could never actually be collected by the payment-processing capability behind FEAT-07 -- this is a correctness rule, not a business preference.
- Authorization for every action defined here is evaluated on every attempt, not only at screen entry -- a role change or session change mid-session is re-checked the next time an action is attempted.

## Edge Cases

- **Service name at exactly 80 characters** -- Passes validation. 81 characters shows the "must be 80 characters or fewer" error.
- **Price entered as exactly the smallest unit above zero (e.g., one cent)** -- Passes the "greater than zero" rule; whether the resulting deposit still clears the minimum-chargeable-deposit rule depends on the deposit rule chosen.
- **Duration at exactly 5 minutes** -- Passes validation (the minimum step). **Duration at exactly 720 minutes (12 hours)** -- Passes validation. 725 minutes shows the "cannot exceed 12 hours" error.
- **Fixed deposit exactly equal to price** -- Passes the fixed-deposit ceiling rule (full prepayment is allowed). One cent above price fails with the ceiling error.
- **Percentage deposit at exactly 1%** and **exactly 100%** -- Both pass validation; 0% and 101% each fail with the percentage-range error.
- **Computed deposit exactly equal to the minimum chargeable amount** -- Passes the minimum-chargeable-deposit rule. One cent below it fails.
- **Pro switches from Percentage back to Fixed after a valid percentage was entered** -- The percentage value is cleared per the deposit-rule-form-exclusivity rule; the fixed field starts empty and re-triggers the required-field rule if the Pro attempts to save before entering a new value.
- **Support's role changes mid-session (help request ends)** -- The next authorization check (e.g., an attempted view refresh) re-evaluates Support's access; a lapsed help request is out of this spec's scope to define further (owned by FEAT-19), but this spec's Authorization Rules table's "Always, read-only, for the Pro account under an active help request" condition is what gates it.

## Acceptance Criteria

**FEAT-01.SPEC-004-AC-01:** Given Talia leaves the name field empty and moves to the next field, then the name field shows the error "Service name is required."

**FEAT-01.SPEC-004-AC-02:** Given Talia enters an 81-character service name, when she blurs the field, then she sees "Service name must be 80 characters or fewer."

**FEAT-01.SPEC-004-AC-03:** Given Talia enters a price of $0, when she blurs the field, then she sees "Enter a price greater than zero."

**FEAT-01.SPEC-004-AC-04:** Given Talia enters a duration of 7 minutes, when she blurs the field, then she sees "Duration must be in 5-minute steps."

**FEAT-01.SPEC-004-AC-05:** Given Talia enters a duration of 725 minutes, when she blurs the field, then she sees "Duration cannot exceed 12 hours."

**FEAT-01.SPEC-004-AC-06:** Given Talia sets a $100 price and enters a fixed deposit of $150, when she blurs the deposit field, then she sees "Deposit amount must be greater than $0 and no more than the service price."

**FEAT-01.SPEC-004-AC-07:** Given Talia sets a $100 price and a fixed deposit of $100, when she blurs the deposit field, then no error is shown (full prepayment is valid).

**FEAT-01.SPEC-004-AC-08:** Given Talia selects the Percentage form and enters 0%, when she blurs the field, then she sees "Deposit percentage must be between 1% and 100%."

**FEAT-01.SPEC-004-AC-09:** Given Talia sets a very low price and a percentage that results in a deposit below the minimum chargeable amount, when she blurs the deposit field, then she sees "The deposit for this price and rule is below the minimum amount that can be charged. Increase the price, deposit amount, or percentage."

**FEAT-01.SPEC-004-AC-10:** Given Talia has entered a fixed deposit value and then switches the toggle to Percentage, when the toggle switches, then the previously entered fixed value is cleared rather than reinterpreted as a percentage.

**FEAT-01.SPEC-004-AC-11:** Given Talia saves a new service, then its status is set to Active and its display_order is appended to the end of her current Active list automatically, with no field on the form for either value.

**FEAT-01.SPEC-004-AC-12:** Given Talia (the Pro) attempts to edit a service, when she has valid access to her own account, then the edit is allowed unconditionally.

**FEAT-01.SPEC-004-AC-13:** Given Platform Operator (Support) views a Pro's service during a help request, when Support looks for any edit control, then none is rendered, and the "Support view -- no changes can be made here" banner is shown instead.

**FEAT-01.SPEC-004-AC-14:** Given Riley (the Client) attempts to reach any Service & Pricing Management screen, then no such path exists in the product's Client-facing navigation, and any direct attempt lands on the ordinary unauthenticated/Pro sign-in experience per FEAT-01.SPEC-001/002/003's Access and Visibility tables.

**FEAT-01.SPEC-004-AC-15:** Given Talia archives a service, when she confirms the archive, then the action is allowed because she is the Pro completing the required FEAT-01.SPEC-006 impact check; the same action attempted by Support or a Client is never possible because no Archive control exists for those roles.

**FEAT-01.SPEC-004-AC-16:** Given Talia reactivates an archived service, then its display_order is appended to the end of her current Active list, not restored to its previous position.

**FEAT-01.SPEC-004-AC-17:** Given Talia types a price and a percentage deposit rule, when either value changes, then the computed deposit preview recalculates live as price x percentage / 100, rounded to the currency's smallest unit, and the Pro cannot directly edit that computed number.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 7 | 7 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 16 | 16 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 8 | 8 |



# Logic/Rule Spec: Price & Deposit Lock at Booking Time

## Overview

**Name:** Price & Deposit Lock at Booking Time
**ID:** FEAT-01.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs that editing or archiving a service never changes the price, duration, or deposit already agreed on a confirmed booking -- this feature's elaboration of XBR-04.
**Parent Feature:** FEAT-01 -- Service & Pricing Management
**Governed Entity:** Service (specifically, the propagation behavior of each Service field toward Bookings that already reference it)

## Scope and Non-Goals

**In Scope:**
- Defining, per Service field, whether an edit to it retroactively affects a Booking that already references that service
- Defining that Booking.price_agreed, Booking.duration, and Booking.deposit_amount are captured once, at booking time, and never recalculated afterward
- Defining that archiving a service never alters, cancels, or hides any existing Booking that references it
- Authorization over the one action this spec exists to forbid: manually overriding a confirmed booking's locked values

**Non-Goals:**
- Field-format validation of Service's own fields (required, length, range, the minimum chargeable deposit) -- fully owned by FEAT-01.SPEC-004; this spec assumes those rules already passed
- Counting or warning about upcoming bookings before an archive is confirmed -- that check is FEAT-01.SPEC-006 (Archive Impact Check); this spec defines what happens to those bookings' data once the archive completes, not whether the Pro is warned first
- Computing the deposit amount from a service's deposit rule -- that computation belongs to the payment-processing capability behind FEAT-07; this spec only asserts that, once computed and captured on a Booking, it is never recalculated by a later Service edit
- Partial refunds or tiered cancellation schedules -- excluded per scope-boundaries.md SC-18: this spec concerns price/deposit immutability, not refund percentages, which stay a binary rule owned by FEAT-09

## Governed Entity

**Entity:** Service (this spec's rules concern each field's propagation behavior toward referencing Bookings; the Service field list itself is defined in FEAT-01.SPEC-004 and is not repeated here except as the subject of each propagation rule)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| name | text | The service's client-facing name |
| price | number | The fixed price of the service, in the Pro Account's currency |
| duration | number | The service's fixed length, in minutes |
| deposit_rule | enum + number | Fixed amount or percentage deposit rule |
| buffer_override | number (optional) | Owned by FEAT-02; not addressed by this feature |
| display_order | number | The service's position on the booking page |
| status | enum (Active \| Archived) | The service's current lifecycle state |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-01.SPEC-003 | Edit Service | On every field save -- the save never touches any existing Booking's own copied fields |
| FEAT-01.SPEC-003 / FEAT-01.SPEC-006 | Edit Service / Archive Impact Check | On archive confirmation -- the status change never alters, cancels, or hides any existing Booking |
| FEAT-05 (Public Booking Page & Booking Flow), FEAT-07 (Deposit Payment at Booking) | -- (cross-feature) | At the moment a Booking is created and confirmed, Service's then-current price, duration, and deposit rule are read once and copied onto the Booking; this spec's guarantee begins from that moment |
| FEAT-12 (Pro Daily Schedule Dashboard), FEAT-16 (Booking & Payment Activity Record), FEAT-30 (Pro Booking Management) | -- (cross-feature) | Every surface that displays a Booking's price, duration, or deposit reads the Booking's own locked copy, never the Service's current live value |

## Field Validation Rules

{This spec governs propagation behavior, not field format -- format rules for every Service field live in FEAT-01.SPEC-004. For each field, this table states whether an edit propagates retroactively to an existing Booking.}

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| name | Does not propagate a booking-time lock -- a service rename is reflected wherever its name is shown, including on existing bookings, since the product definition locks only price, duration, and deposit (XBR-04) | Always | Not applicable -- there is no invalid state to block | N/A | No |
| price | Locked -- editing price never changes Booking.price_agreed on any Booking already at Confirmed or later in its lifecycle | Always, from the moment a Booking reaches Confirmed | Enforced structurally: no save path exists that writes to a Booking's price_agreed from a Service edit | N/A -- there is no user-facing error; the lock is a structural guarantee, not a validation the Pro can trigger or fail | Yes (as a structural block, not a form error) |
| duration | Locked -- editing duration never changes Booking.duration on any Booking already at Confirmed or later | Always, from the moment a Booking reaches Confirmed | Enforced structurally, as above | N/A | Yes (structural) |
| deposit_rule | Locked -- editing the deposit rule never recomputes Booking.deposit_amount on any Booking already at Confirmed or later | Always, from the moment a Booking reaches Confirmed | Enforced structurally, as above | N/A | Yes (structural) |
| buffer_override | No propagation concern for this feature -- owned entirely by FEAT-02; this spec makes no claim about its effect on availability computation | Always | -- | -- | -- |
| display_order | No propagation concern -- display-only ordering field, never referenced by any Booking | Always | -- | -- | -- |
| status | Locked in the honoring sense -- setting status to Archived never cancels, hides, or alters any existing Booking that references the service; the archived service continues to display correctly on every past and upcoming Booking that references it | Always | Enforced structurally at archive time (FEAT-01.SPEC-003/FEAT-01.SPEC-006) | N/A | Yes (structural) |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Simultaneous edit has no compounding effect | price, deposit_rule, duration | Editing more than one locked field in the same save has no greater retroactive effect than editing one -- all three remain independently locked on any Confirmed-or-later Booking regardless of how many are changed together | N/A -- structural guarantee, no error state |
| Archive plus prior edit | status, price, duration, deposit_rule | A service that was edited and later archived carries both changes forward identically: existing Bookings keep the values that were locked in at their own booking time, regardless of how many edits or an eventual archive followed | N/A -- structural guarantee, no error state |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Manually override a Confirmed-or-later booking's locked price, duration, or deposit | Nobody (Never) -- not even the Pro | N/A -- no condition grants this | No control exists anywhere in the product to alter these fields on a Booking once Confirmed; every surface that displays them (FEAT-12, FEAT-16, FEAT-30) renders them read-only |
| Edit a service's price, duration, or deposit rule going forward (future bookings only) | The Pro | Always, for their own account's services | -- |
| Edit a service's price, duration, or deposit rule going forward | Platform Operator (Support) | Never | No edit control is rendered for this role (see FEAT-01.SPEC-004) |
| Edit a service's price, duration, or deposit rule going forward | The Client | Never | Not reachable, as defined in FEAT-01.SPEC-001/002/003 |
| Archive a service that has upcoming bookings | The Pro | Allowed after acknowledging the impact warning from FEAT-01.SPEC-006 (or immediately, if no upcoming bookings exist) | -- |
| Archive a service that has upcoming bookings | Platform Operator (Support) | Never | No Archive control is rendered for this role |
| Archive a service that has upcoming bookings | The Client | Never | Not reachable, as defined above |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Booking.price_agreed | Copied once from Service.price at the moment the Booking is confirmed | On Booking confirmation only (owned by FEAT-05/FEAT-07) | No -- never recalculated by a later Service edit; no Pro or Client override exists |
| Booking.duration | Copied once from Service.duration at the moment the Booking is confirmed | On Booking confirmation only | No -- never recalculated by a later Service edit |
| Booking.deposit_amount | Computed once from Service.price and Service.deposit_rule at the moment the Booking is confirmed (computation owned by FEAT-07) | On Booking confirmation only | No -- never recalculated by a later Service edit |

## Business Rules

- XBR-04 (owned by FEAT-01): service edits and archiving apply to future bookings only; confirmed bookings keep the price, duration, and deposit agreed at booking. This spec is that rule's full elaboration for the Service side.
- XBR-11: setup changes -- including a Service edit or archive -- never silently cancel a confirmed booking; any resulting conflict (for example, an archived service with upcoming bookings) is flagged on the Pro's booking management surface (FEAT-30) and resolved only by an explicit Pro choice there. This spec never auto-resolves such a conflict itself.
- This spec's guarantee begins at Confirmed and holds for every later Booking state (Awaiting Outcome, Completed, No-Show, Cancelled by Client, Cancelled by Pro, Rescheduled) -- once locked, a Booking's price, duration, and deposit are never unlocked by any later state transition either.
- A Booking that is Rescheduled (FEAT-10, FEAT-30) is still the same Booking record with the same locked price and deposit; only its time (and, per FEAT-30, potentially other fields those features own) changes -- rescheduling is never a mechanism for re-pricing.
- FEAT-01.SPEC-006's impact check runs before an archive is confirmed, but the lock this spec defines applies regardless of whether that check ran or what it found -- the lock is unconditional on every Confirmed-or-later Booking, not contingent on the Pro having been warned.

## Edge Cases

- **Client is mid-checkout (Booking state Pending Payment, not yet Confirmed) when the Pro edits the service's price** -- This spec's lock applies from Confirmed onward, not before. A Pending Payment attempt reflects the Service's values at the moment the payment step reads them (owned by FEAT-05/FEAT-07); if the price changes before payment completes, the client's outcome for that not-yet-confirmed attempt is defined by those features, not by this spec's guarantee.
- **Pro archives a service while a client is mid-checkout for it** -- Per the dependency map's Service Contention note: if the service is archived before the client's payment completes, the client is refused with a refresh back to the service list; no Booking was ever confirmed for that attempt, so this spec's lock never applied to it.
- **Service archived while a referencing Booking is later Completed or marked No-Show** -- The Booking's locked price, duration, and deposit remain exactly as captured at booking time; archiving has no effect on already-locked data regardless of the Booking's subsequent state.
- **Two Pro edits to the same service's price in quick succession, both before any new booking is confirmed against either version** -- Whichever edit is saved last (per the dependency map's Service Contention note: last-write-wins between the Pro's own sessions) is the value read the next time a Booking is confirmed; this has no bearing on any Booking already locked before either edit.
- **A goodwill refund (FEAT-30) is issued against a locked deposit** -- The refund changes the Deposit Transaction's outcome (owned by FEAT-09/FEAT-30), never the Booking's locked deposit_amount figure itself, which remains the historical record of what was agreed.
- **Support views a booking's locked price during a help request** -- Support sees the same read-only locked value the Pro sees; Support has no path, and never will, to alter it (Access Matrix: Booking & Payment = View for Support).

## Acceptance Criteria

**FEAT-01.SPEC-005-AC-01:** Given Talia has a Confirmed booking for a service priced at $80, when she later edits that service's price to $100, then the existing booking's price_agreed remains $80.

**FEAT-01.SPEC-005-AC-02:** Given Talia has a Confirmed booking for a 60-minute service, when she later edits that service's duration to 90 minutes, then the existing booking's duration remains 60 minutes.

**FEAT-01.SPEC-005-AC-03:** Given Talia has a Confirmed booking whose deposit was computed under a 20% deposit rule, when she later changes the service's deposit rule to a fixed amount, then the existing booking's deposit_amount is unchanged.

**FEAT-01.SPEC-005-AC-04:** Given Talia renames a service from "Classic Set" to "Signature Set," when the rename saves, then every existing booking referencing that service now shows "Signature Set" as its service name, since name is not a locked field.

**FEAT-01.SPEC-005-AC-05:** Given Talia archives a service that has three upcoming Confirmed bookings, when the archive completes, then all three bookings remain Confirmed with their original price, duration, and deposit intact, and none is cancelled or hidden from the Pro's schedule.

**FEAT-01.SPEC-005-AC-06:** Given Talia (the Pro) looks for any way to manually change a Confirmed booking's locked price on her dashboard, then no such control exists anywhere in the product -- the field renders read-only on FEAT-12, FEAT-16, and FEAT-30.

**FEAT-01.SPEC-005-AC-07:** Given Riley (the Client) has a Confirmed booking, when the Pro edits or archives the underlying service, then Riley's booking confirmation and reminder continue to show the original price and deposit she agreed to, unchanged.

**FEAT-01.SPEC-005-AC-08:** Given a client is mid-checkout for a service (Pending Payment, not yet Confirmed) and the Pro edits its price before payment completes, then this spec's lock does not yet apply to that attempt, since no Booking has reached Confirmed.

**FEAT-01.SPEC-005-AC-09:** Given a client is mid-checkout for a service and the Pro archives it before payment completes, then the client's payment attempt is refused and they are returned to the service list, per the dependency map's Service Contention note.

**FEAT-01.SPEC-005-AC-10:** Given Talia edits both the price and the deposit rule of a service in the same save, when an existing Confirmed booking already references it, then that booking's price_agreed and deposit_amount both remain exactly as they were at booking time.

**FEAT-01.SPEC-005-AC-11:** Given a booking referencing an archived service later reaches Completed, when Talia views its history, then the price, duration, and deposit shown are exactly what was locked in at booking time, unaffected by the archive.

**FEAT-01.SPEC-005-AC-12:** Given Talia rebooks a rescheduled appointment (FEAT-10/FEAT-30) that keeps the same booking record, then its already-locked price and deposit are unchanged by the reschedule; only its time changes.

**FEAT-01.SPEC-005-AC-13:** Given Platform Operator (Support) views a Confirmed booking's locked price during a help request, when Support looks for any way to edit it, then no edit control is rendered, consistent with Support's View-only access to Booking & Payment.

**FEAT-01.SPEC-005-AC-14:** Given Talia edits a service's price twice from two different devices in quick succession, when both saves complete, then the later-completing save's price is the value used for any Booking confirmed afterward, per the dependency map's last-write-wins resolution -- and this has no effect on any booking already locked before either edit.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 7 | 7 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 6 | 6 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Automation Spec: Archive Impact Check

## Overview

**Name:** Archive Impact Check
**ID:** FEAT-01.SPEC-006
**Type:** Automation
**Purpose:** On an archive request, checks for upcoming bookings referencing the service and surfaces an impact warning before the Pro confirms the archive.
**Parent Feature:** FEAT-01 -- Service & Pricing Management

## Scope and Non-Goals

**In Scope:**
- Counting upcoming Bookings (Pending Payment, Confirmed, or Awaiting Outcome, with a future start_time) that reference the service being archived
- Deciding whether to show an impact warning (count of 1 or more) or proceed directly (count of zero)
- Handing the Pro's confirmation or cancellation back to FEAT-01.SPEC-003 to complete or abandon the archive

**Non-Goals:**
- Setting the service's status to Archived -- that write is performed by FEAT-01.SPEC-003 once this automation's outcome and the Pro's confirmation both allow it; this automation only checks and warns
- Resolving the conflict an archived service's upcoming bookings create -- that conflict is flagged on the Pro's booking management surface (FEAT-30) per XBR-11, and resolved there by an explicit Pro decision; this automation never cancels, reschedules, or otherwise touches those bookings
- Guaranteeing the locked price, duration, and deposit of the counted bookings remain unchanged -- that guarantee is FEAT-01.SPEC-005's, not this automation's; this automation only counts and warns
- Cross-pro or cross-client visibility of the count -- excluded per scope-boundaries.md SC-03: the count is shown only to the requesting Pro, about their own account's bookings

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Pro taps "Archive this service" | FEAT-01.SPEC-003 (Edit Service) | Fires immediately on tap, before any confirmation dialog is shown | The service's identifier, the Pro Account, and the current date/time in the Pro's timezone |

## Processing Logic

1. Receive the archive request for the specified service from FEAT-01.SPEC-003, including the service's identifier and the Pro Account it belongs to.
2. Read all Bookings that reference this service and belong to this Pro Account.
3. Filter to Bookings in an upcoming state -- Pending Payment, Confirmed, or Awaiting Outcome -- whose start_time is in the future relative to the current time in the Pro's timezone.
4. Count the filtered Bookings.
5. If the count is zero, signal FEAT-01.SPEC-003 to proceed directly to archiving, with no warning shown to the Pro.
6. If the count is one or more, return that count to FEAT-01.SPEC-003 for display in an impact warning modal, and hold the archive pending the Pro's explicit confirmation.
7. On the Pro's confirmation, signal FEAT-01.SPEC-003 to set the service's status to Archived, per FEAT-01.SPEC-005's lock guarantee -- none of the counted bookings are altered.
8. On the Pro's cancellation of the warning, take no action -- the service remains Active and the archive does not proceed.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| No upcoming bookings | The filtered count is zero | None from this automation -- the archive write happens in FEAT-01.SPEC-003 | No warning modal shown; the archive confirmation flow proceeds directly to the success toast | FEAT-01.SPEC-003 |
| Upcoming bookings found | The filtered count is one or more | None yet -- pending the Pro's decision | Impact warning modal: "{count} upcoming booking(s) reference this service. They will be honored as booked; the service will no longer appear for new bookings." with Confirm and Cancel options | FEAT-01.SPEC-003 |
| Pro confirms archive | The Pro taps Confirm on the impact warning modal | Service.status set to Archived (written by FEAT-01.SPEC-003); the counted bookings are never altered, per FEAT-01.SPEC-005 | Success toast: "{Service name} archived -- it's no longer bookable, and existing appointments are unaffected." Navigate to FEAT-01.SPEC-001 (Archived filter) | FEAT-01.SPEC-001, FEAT-01.SPEC-003, FEAT-01.SPEC-005 |
| Pro cancels the warning | The Pro taps Cancel on the impact warning modal | None | Modal closes; Edit Service screen remains open; service remains Active | FEAT-01.SPEC-003 |
| Automation failure (count could not be determined) | The Booking read fails or times out | None | Blocking error banner on FEAT-01.SPEC-003: "Couldn't check upcoming bookings for this service. Try again." with a Retry action -- the archive is never permitted to proceed without a successful count, since guessing "zero" could hide a real conflict | FEAT-01.SPEC-003 |

## Data Model

**Reads:** Booking -- service reference, state, and start_time fields, scoped to the requesting Pro Account. Service -- the service's identifier and current status (to short-circuit if it is somehow already Archived).
**Creates:** None.
**Updates:** None -- Service.status is written by FEAT-01.SPEC-003, not by this automation.
**Deletes:** None.

## Business Rules

- "Upcoming" is defined as: Booking state is Pending Payment, Confirmed, or Awaiting Outcome, and start_time is in the future relative to the Pro's current time. A Booking already Completed, No-Show, Cancelled, or Expired never counts toward the impact warning, regardless of how recently it ended.
- XBR-11: this automation surfaces the count but never resolves the resulting conflict itself -- the conflict created by archiving with upcoming bookings is flagged on the Pro's booking management surface (FEAT-30) for an explicit Pro decision, independent of this automation's own outcome.
- The impact warning is informational, not a block -- the Pro may still archive a service with any number of upcoming bookings; this automation never prevents the archive outright, it only ensures the Pro sees the count first.
- A zero count is only ever reported after a successful read -- an automation failure never resolves to "assume zero," since that could silently hide a real conflict from the Pro.

## Edge Cases

- **A new booking for this service is completed by a client in the moments between the warning being shown and the Pro confirming** -- The archive proceeds based on the Pro's confirmation of the count already shown; the newly created booking is honored the same as any other upcoming booking, per FEAT-01.SPEC-005, but is simply not reflected in the count the Pro already saw, since that count is a snapshot, not a live guarantee.
- **The service is already Archived when the archive is attempted again (e.g., a stale screen)** -- The automation short-circuits with the message "This service is already archived," and FEAT-01.SPEC-003 routes the Pro back to FEAT-01.SPEC-001 without re-running the count.
- **Concurrent trigger firing (Pro taps Archive for the same service from two signed-in devices around the same time)** -- Each device runs its own independent check and, if applicable, its own warning modal. Whichever device's confirmation completes first sets the service to Archived; the second device's confirmation (if it proceeds) is a redundant no-op, since the service is already Archived -- its screen refreshes to reflect the current state, per the dependency map's Service Contention note.
- **Trigger fires while a previous run is in flight for the same service** -- FEAT-01.SPEC-003 disables the "Archive this service" action while a check is in progress, so a second run for the same service cannot start from the same screen session. Runs for different services proceed independently and never queue behind each other.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-003 (Edit Service) | Triggered by (inbound) | Fires when the Pro taps "Archive this service" |
| FEAT-01.SPEC-003 (Edit Service) | Affects (outbound) | Returns the count (or the direct-proceed signal) for the Edit Service screen to display or act on |
| FEAT-01.SPEC-001 (Service List) | Affects (outbound) | Pro is returned here, Archived filter, after a confirmed archive |
| FEAT-01.SPEC-004 (Service Field & Deposit Rule Validation) | Enforced rule (inbound) | This automation is an enforcing spec of that rule: it applies the rule's authorization check (Pro only) to the underlying Archive action it gates; it applies no field validation |
| FEAT-01.SPEC-005 (Price & Deposit Lock at Booking Time) | References (outbound) | The counted bookings' locked price, duration, and deposit are guaranteed unaffected by that spec, once the archive completes |
| FEAT-30 (Pro Booking Management) | References (outbound) | The conflict created by archiving a service with upcoming bookings is surfaced on that feature's booking management surface, per XBR-11; this automation originates that flag but does not resolve it |

## Analytics and Success Signals

- **archive_impact_checked** (result: no_upcoming / upcoming_found; upcoming_count) -- N/A -- success-metrics.md's only metric connected to this feature, "Service Setup Confidence," measures the add/edit attempt experience; no metric connected to Service & Pricing Management tracks archive-impact outcomes.
- **archive_confirmed_with_upcoming_bookings** (upcoming_count at time of confirmation) -- N/A -- same reason as above; this is a diagnostic signal for the Pro's own operational history (visible via FEAT-16), not a signal any success metric in this feature's slice tracks.
- **archive_impact_check_failed** (reason: read_failure / timeout) -- N/A -- same reason as above.

## Acceptance Criteria

**FEAT-01.SPEC-006-AC-01:** Given Talia taps "Archive this service" on a service with zero upcoming bookings, when the check completes, then the archive proceeds directly with no warning modal shown.

**FEAT-01.SPEC-006-AC-02:** Given Talia taps "Archive this service" on a service with two upcoming Confirmed bookings, when the check completes, then an impact warning modal reads "2 upcoming booking(s) reference this service. They will be honored as booked; the service will no longer appear for new bookings." with Confirm and Cancel options.

**FEAT-01.SPEC-006-AC-03:** Given Talia sees the impact warning modal showing two upcoming bookings, when she taps Confirm, then the service is set to Archived, both bookings remain unaltered, and she sees the toast "{Service name} archived -- it's no longer bookable, and existing appointments are unaffected."

**FEAT-01.SPEC-006-AC-04:** Given Talia sees the impact warning modal, when she taps Cancel, then the modal closes, the service remains Active, and no data changes.

**FEAT-01.SPEC-006-AC-05:** Given Talia's booking read fails while checking for upcoming bookings, when the failure occurs, then a blocking error banner reads "Couldn't check upcoming bookings for this service. Try again." and the archive does not proceed until a successful check completes.

**FEAT-01.SPEC-006-AC-06:** Given a service has one upcoming booking in Pending Payment and one in Completed, when Talia archives it, then the impact warning counts only the Pending Payment booking, since Completed bookings never count as upcoming.

**FEAT-01.SPEC-006-AC-07:** Given a client completes a new booking for a service in the few seconds between Talia's impact warning being shown and her confirming, when she confirms, then the archive proceeds based on the count she saw, and the new booking is honored exactly as any other upcoming booking per FEAT-01.SPEC-005.

**FEAT-01.SPEC-006-AC-08:** Given Talia attempts to archive a service that a stale screen still shows as Active but that is already Archived, when the automation runs, then it reports "This service is already archived" and returns her to FEAT-01.SPEC-001 without re-running the count.

**FEAT-01.SPEC-006-AC-09:** Given Talia taps Archive for the same service from her phone and her tablet within moments of each other, when both checks and confirmations proceed, then whichever device's confirmation completes first archives the service, and the second device's confirmation is a no-op that simply refreshes to the current Archived state.

**FEAT-01.SPEC-006-AC-10:** Given Talia archives a service with upcoming bookings, when the archive completes, then those bookings' conflicts are made available on her booking management surface (FEAT-30) per XBR-11, for her to resolve there explicitly.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 (Pro taps Archive on FEAT-01.SPEC-003) | 1 |
| Outcome Paths | 5 (no upcoming, upcoming found, confirmed, cancelled, failure) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 4 | 4 |


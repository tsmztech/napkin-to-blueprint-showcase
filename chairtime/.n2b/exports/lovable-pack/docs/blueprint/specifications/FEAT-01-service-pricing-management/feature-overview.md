---
document_type: feature-overview
feature_number: FEAT-01
feature_name: Service & Pricing Management
feature_slug: service-pricing-management
priority_tier: Core
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 6
screen_count: 3
automation_count: 1
logic_rule_count: 2
integration_count: 0
notification_count: 0
---

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

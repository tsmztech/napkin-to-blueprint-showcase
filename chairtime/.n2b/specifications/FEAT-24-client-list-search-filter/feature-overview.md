---
document_type: feature-overview
feature_number: FEAT-24
feature_name: Client List Search & Filter
feature_slug: client-list-search-filter
priority_tier: Nice-to-Have
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 2
screen_count: 1
automation_count: 0
logic_rule_count: 1
integration_count: 0
notification_count: 0
---

# Feature Breakdown Brief: Client List Search & Filter

## Summary

**Feature:** Client List Search & Filter
**ID:** FEAT-24
**Description:** As a Pro's client base grows, they can search and filter their client list by name, phone, or recent activity instead of scrolling a long list.
**Priority:** Nice-to-Have
**Phase:** v1
**Type:** User-Facing
**Rationale:** BRIEF.md's Scale & Non-Functional Expectations states a Pro may have "100–500 clients," a volume where an unfiltered list becomes genuinely unwieldy. Not required at launch (a new Pro starts with very few clients), so phased to v1 rather than MVP.

**Key Capabilities:**
- Search clients by name or phone number
- Filter by recency (e.g., booked in the last 30 days) or upcoming-booking status

**Connected Entities:** Client (read — search/filter only)

**Access (from the Access Matrix):** The Pro has Full access to search their own clients. Platform Operator (Support) has View-only access, used to locate a specific client's record during a support session (XBR-24) — the Access field names Support as an actual user of this feature's search behavior, not merely a bystander, so Support is carried as a Roles Touched value on this feature's screen and rule, exactly as the Access field states. The Client has no access to this feature (Access Matrix: None).

**Communications:** N/A — a browsing convenience with no message trigger (product-features.md Communications field for this feature). No Communications line to elaborate into a Notification spec; `notification_count: 0` is legal and expected here.

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-24.SPEC-001 | Client Search & Filter | Screen | The Pro (Talia), Platform Operator (Support) | The Pro or Support types a partial name or phone number and/or applies a recency or upcoming-booking filter, and sees the client list narrow instantly, with a plain "no clients match" state when nothing matches |
| FEAT-24.SPEC-002 | Search Match & Filter Derivation Rules | Logic/Rule | The Pro (Talia), Platform Operator (Support) | Governs partial name/phone matching, the 30-day recency window and upcoming-booking-status derivation from each client's booking history, and the fallback-to-full-list behavior when a search call fails |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Search clients by name or phone number | FEAT-24.SPEC-001, FEAT-24.SPEC-002 | Search box on the screen narrows the list as the Pro/Support types; the matching algorithm (partial, case-insensitive name and phone matching) is defined by the rule spec | Phase 2 (Explicit) |
| Filter by recency (e.g., booked in the last 30 days) or upcoming-booking status | FEAT-24.SPEC-001, FEAT-24.SPEC-002 | Filter controls on the same screen; the 30-day window and the upcoming-booking-status derivation (computed from each client's Booking history) are defined by the rule spec | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-24.SPEC-002 | Search Match & Filter Derivation Rules | Phase 5 (Rule-Constraint Discovery) | The recency window and upcoming-booking-status filter are derivation rules with non-trivial logic (a time-window calculation and a status lookup over each client's booking_history) — crossing the standalone-Logic/Rule threshold ("Derivation rules with non-trivial logic"). Phase 6's failure analysis added the "failed search falls back to the full, unfiltered list" behavior (from the feature's own States field) to the same spec, since it is a conditional rule governing the same search operation rather than a separate concern. |

## Entity-Lifecycle Coverage Matrix

**Entity: Client**

This feature's Connected Entities line scopes Client to "read — search/filter only"; this feature creates, updates, and deletes nothing.

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A | Created by FEAT-05 (a client's first booking) and FEAT-30 (Pro books a client in) per the dependency map — this feature never creates a Client record | Cross-feature |
| Read (single) | N/A | Opening one client's full record (contact details, private note, booking history) is owned by Client Record Management (FEAT-13); this feature only ever shows the searchable/filterable list row, never the single-record view | Cross-feature; tapping a result navigates to FEAT-13.SPEC-001 |
| Read (list) | FEAT-24.SPEC-001 | The Client Search & Filter screen loads the Pro's full client list and narrows it by name/phone match and by the active recency/upcoming-booking filter | This is the feature's entire reason for existing |
| Update | N/A | Contact-detail and private-note edits are owned by Client Record Management (FEAT-13, name/phone/email/note) and Client Booking Identity (FEAT-06, client's own email/consent) — this feature has no edit affordance | Cross-feature |
| Delete/Archive | N/A | Client deletion is owned by Client Record Management (FEAT-13.SPEC-003/SPEC-004), gated by FEAT-13.SPEC-006, per XBR-19 — hard delete of contact details and notes, de-identified financial/timeline history retained; this feature never deletes and simply stops matching a deleted client in future searches | Cross-feature; consistent with XBR-19: a later booking from the same person creates a new, unconnected record that search will match, never a resurrection of the deleted one |
| State Transition | N/A | The Client entity has no lifecycle states beyond existing/deleted (product-features.md Domain Entity Inventory; confirmed by FEAT-13's own matrix) | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Booking | FEAT-24.SPEC-001, FEAT-24.SPEC-002 | The recency ("booked in the last 30 days") and upcoming-booking-status filters are computed over each client's derived booking_history field — a Booking read, never a write, per the dependency map's Shared Data Entities slice |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Pro/Support types a partial name or phone number into search | List narrows instantly to matching clients (partial, case-insensitive match) | Inline in triggering screen, matching rule defined by | FEAT-24.SPEC-001 / FEAT-24.SPEC-002 |
| Pro/Support applies the recency filter (booked in the last 30 days) | List narrows to clients with a Booking within the 30-day window | Standalone Logic/Rule (derivation over Booking history) | FEAT-24.SPEC-002 |
| Pro/Support applies the upcoming-booking-status filter | List narrows to clients with an upcoming Booking | Standalone Logic/Rule (derivation over Booking history) | FEAT-24.SPEC-002 |
| Search and/or filter returns no matches | Show a plain "no clients match" state rather than an empty, unexplained list | Inline in triggering screen | FEAT-24.SPEC-001 |
| The search call fails | Fall back to the full, unfiltered client list rather than an error screen | Standalone Logic/Rule (fallback behavior) | FEAT-24.SPEC-002 |
| Pro/Support taps a client in the results | Navigate to that client's record (and, for the Pro, private note) | Cross-feature | FEAT-13.SPEC-001 (Client Record Detail) |
| Device or connection goes offline | Search and filter continue to operate against the most recently loaded client list rather than blocking | Inline in triggering screen | FEAT-24.SPEC-001 |
| Pro/Support performs a search | Log `client_search_performed` | Inline in triggering screen | FEAT-24.SPEC-001 |
| Pro/Support applies a filter | Log `client_filter_applied` | Inline in triggering screen | FEAT-24.SPEC-001 |

## Shared Context

**Shared Entities:**
- Client record -- read-only in this feature. FEAT-24.SPEC-001 reads the list (name, phone, and derived booking status for filtering); no field is ever written here. Fields relevant to this feature: name, phone (the two searchable fields per Key Capabilities), and booking_history (the derived field the recency/upcoming-status filters run over). private_note is never read or displayed by this feature — search/filter results carry only name, phone, and booking-status information, so the XBR-24 restriction on Support ever seeing private notes is satisfied by this feature's scope, not by an added exclusion rule.

**Shared UI Patterns:**
- Client result row -- the same compact identity display (name, phone, and a recency/upcoming-booking indicator) is used for every row in the search/filter results and is the client list's own row identity: FEAT-24.SPEC-001 is the canonical client list (FEAT-13 has no list screen of its own, per FEAT-13.SPEC-002), so the Spec Writer for FEAT-24.SPEC-001 should describe the row once and let FEAT-13 and FEAT-12 summary views reference it rather than inventing a second pattern.

**Shared Validation:**
- FEAT-24.SPEC-002 defines all matching and derivation logic (name/phone matching, the 30-day recency window, upcoming-booking-status derivation, and the search-failure fallback); FEAT-24.SPEC-001 references it rather than restating the rules.

**Analyst note on Automation count (spec-count-sanity self-check):** This feature has `automation_count: 0`. Every capability here is a read/filter over existing data (Connected Entities: "Client (read — search/filter only)"; Data Notes: "Derived: none — this is a view over existing data") — there is no entity creation or update for this feature to trigger a side-effecting automation from. This is a deliberate consequence of the feature's read-only scope, not an omission.

**Analyst note on Integration count (spec-count-sanity self-check):** This feature has `integration_count: 0`. The only Dependencies-slice entry touching it, ASMP-34 (a searchable, per-pro record store for services, bookings, clients, and payment outcomes), is the product's own foundational persistence layer that every feature reads from — it is not a category-level *external* capability (money movement, identity, communications, third-party data, or AI) that this feature itself needs to contract with, and no other feature's Integration spec is asked to own it on this feature's behalf.

## Internal Dependency Map

```
SPEC-001 (Client Search & Filter) -> [Pro/Support types a search term] -> matched by SPEC-002 (Search Match & Filter Derivation Rules) -> narrowed list shown inline
SPEC-001 (Client Search & Filter) -> [Pro/Support applies a recency or upcoming-booking filter] -> derived by SPEC-002 -> narrowed list shown inline
SPEC-001 (Client Search & Filter) -> [the search call fails] -> SPEC-002's fallback rule -> full unfiltered list shown, no error screen
SPEC-001 (Client Search & Filter) -> [Pro/Support taps a client result] -> FEAT-13.SPEC-001 (Client Record Detail)
```

**Default Entry:** SPEC-001 (Client Search & Filter) -- this feature has no entry point of its own; the Pro or Support opens the client list, which is FEAT-24.SPEC-001 itself (FEAT-24.SPEC-001's own Entry Points list; FEAT-13 owns no client-list screen, per FEAT-13.SPEC-002's Read (list) cell), and search and filter narrow that list in place. Entry to the list comes from the Pro's navigation to their client list, and FEAT-13.SPEC-003 returns the Pro here after a deletion when this was the originating screen.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-24.SPEC-001 | Inbound | FEAT-13 (Client Record Management) | FEAT-13.SPEC-003 returns the Pro to this list after a client deletion when this was the originating screen; FEAT-13 owns no client-list screen (FEAT-13.SPEC-002), so search and filter happen in place on this canonical list rather than being entered from a FEAT-13 list | Pro completes a client deletion started from this list |
| FEAT-24.SPEC-001 | Outbound | FEAT-13 (Client Record Management) | Tapping a search result opens that client's record and, for the Pro, private note | Pro/Support taps a client in search results |
| FEAT-24.SPEC-002 | Inbound | FEAT-13 (Client Record Management) | Reads the Client entity's derived booking_history field (Booking read-only context) to compute the recency and upcoming-booking-status filters | A recency or upcoming-booking filter is applied |
| FEAT-24 (both specs) | Inbound | FEAT-19 (Platform Support Read-Only Access) | Support uses this same search/filter view, read-only, to locate a client's record during a support session, with every view logged in the Pro's visible account activity (XBR-24) | Support opens a Pro account and searches for a client |
| FEAT-24.SPEC-002 | Inbound | FEAT-13 (Client Record Management) | Consistent with XBR-19: a deleted client's record no longer appears in search results, and a later booking from the same person creates a new, unconnected, search-matchable record rather than resurrecting the deleted one | Client deletion executes on FEAT-13 |
| FEAT-24.SPEC-001 | Inbound | FEAT-29 (Pro Sign-In & Account Lifecycle) | Consistent with XBR-29: this screen requires a signed-in Pro (or an active Support session); anyone else is sent to the Pro sign-in screen | Anyone without a signed-in Pro or Support session attempts to reach this screen |

## Non-Functional Notes

**Data volumes / growth:** A Pro may have 100–500 clients (ASMP-22), the exact volume this feature exists to make navigable; search and filter must stay equally responsive as that count and each client's booking history grow across multiple years (ASMP-22's multi-year growth expectation applies here directly, since this feature's whole purpose is scaling the client list).

**Responsiveness:** Per the feature's own States field, results appear "instant for the stated client volumes" — narrowing happens as the Pro/Support types, with no perceptible loading state to design for at 100–500 clients (consistent with ASMP-21's general responsiveness bar, though ASMP-21 itself is stated in terms of booking-slot and booking-completion speed rather than this feature directly).

**Data sensitivity / privacy:** Client records are personal data — name, phone, and (for filtering) booking status — visible only to the Pro and, for support-troubleshooting purposes, to Support (ASMP-23). This feature never surfaces the private_note field in any result, so it carries no additional exposure risk beyond what the underlying Client read already allows; no health data is involved (SC-08).

**Compliance flags:** N/A — this feature introduces no compliance obligation beyond the general Client-data privacy posture already governed by ASMP-23 and XBR-24 (Support's read-only, note-excluded access), both already elaborated above; it triggers no separate deletion, payment, or messaging obligation of its own.

**Analytics linkage (from Signals):** `client_search_performed` (SPEC-001, logged when a search is executed) and `client_filter_applied` (SPEC-001, logged when a recency or upcoming-booking filter is applied). No success metric in success-metrics.md names this feature directly — its quality bar is carried entirely by the Non-Functional Expectations above (ASMP-21, ASMP-22).

## Non-Goals

- **Searching or filtering by fields other than name, phone, recency, and upcoming-booking status (e.g., email, address, or private-note content)** -- Excluded: the feature's Key Capabilities in product-features.md name only "name or phone number" for search and "recency ... or upcoming-booking status" for filtering; Stage 2's Functional Depth fields are elaborated into specs here, never extended with new criteria the product definition did not decide.
- **Bulk actions on search/filter results (bulk message, bulk export, bulk edit)** -- Excluded per this feature's own Data Notes: "Derived: none — this is a view over existing data," and its Description frames the feature as a browsing convenience ("instead of scrolling a long list"); no bulk-action capability is named anywhere in Stage 2.
- **Editing a client's contact details or private note from within search results** -- Excluded: Connected Entities scopes this feature to "read — search/filter only." Any edit requires navigating to Client Record Management (FEAT-13), which owns Client update per the dependency map.
- **Saved searches or persistent filter presets across sessions** -- Excluded: the feature's States field names only the most-recently-loaded client list as what persists offline; no Stage 2 field describes a saved-search or preset mechanism, so each visit to this screen starts with search and filters cleared.
- **Support performing any action beyond viewing search results** -- Excluded per scope-boundaries.md SC-05: Support's access to Client Records is read-only everywhere in the product; this feature adds no exception, and Support cannot edit, message, or delete a client from this screen.
- **Automatic purge or archival driven by this feature** -- Not applicable here as a design decision: this feature owns no Client lifecycle operation (see Entity-Lifecycle Coverage Matrix), so retention and purge policy is entirely FEAT-13's and scope-boundaries.md SC-22's responsibility, not a gap in this Brief.

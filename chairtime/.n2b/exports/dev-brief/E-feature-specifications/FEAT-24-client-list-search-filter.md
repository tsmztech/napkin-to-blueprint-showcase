# FEAT-24 — Client List Search & Filter

This chapter covers Client List Search & Filter (FEAT-24), a Nice-to-Have-tier feature. It carries 2 specifications carrying 33 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-24.SPEC-001 | Client Search & Filter | screen | 18 |
| FEAT-24.SPEC-002 | Search Match & Filter Derivation Rules | logic-rule | 15 |

The feature breakdown brief follows, then every specification in full.


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



# Screen Spec: Client Search & Filter

## Overview

**Name:** Client Search & Filter
**ID:** FEAT-24.SPEC-001
**Type:** Screen
**Purpose:** The Pro or Support views the Pro's full client list and narrows it by typing a partial name or phone number and/or selecting a recency or upcoming-booking filter, so a growing client base (100–500 clients) stays navigable instead of requiring an unfiltered scroll.
**Parent Feature:** FEAT-24 -- Client List Search & Filter

## Scope and Non-Goals

**In Scope:**
- Loading and displaying the Pro's full client list
- A search box that narrows the list by partial, case-insensitive name or phone match as the Pro/Support types (matching rule governed by FEAT-24.SPEC-002)
- A filter control that narrows the list to clients booked within the recency window or clients with an upcoming booking (derivation governed by FEAT-24.SPEC-002)
- The "no clients match" state when search and/or filter produce zero results
- Offline/degraded operation against the most recently loaded client list
- Navigation from a result row into that client's full record (FEAT-13.SPEC-001)

**Non-Goals:**
- Searching or filtering by fields other than name, phone, recency, or upcoming-booking status (e.g., email, address, or private-note content) -- excluded per the Brief's Non-Goals: product-features.md's Key Capabilities name only name/phone for search and recency/upcoming-booking for filtering; this spec never extends the criteria the product definition did not decide
- Bulk actions on search/filter results (bulk message, bulk export, bulk edit) -- excluded per the Brief's Non-Goals: this feature's Data Notes state "Derived: none -- this is a view over existing data," and no bulk-action capability is named anywhere in Stage 2
- Editing a client's contact details or private note from this screen -- excluded per the Brief's Connected Entities scope ("Client (read -- search/filter only)"); any edit requires navigating to Client Record Management (FEAT-13.SPEC-002), which owns Client update
- Saved searches or persistent filter presets across sessions -- excluded per the Brief's Non-Goals: no Stage 2 field describes a saved-search mechanism, so this screen always opens with search and filter cleared
- Displaying the client's private note in any result row or preview -- excluded per XBR-24 (Support never sees private client notes) and ASMP-23 (Client personal data visible only to the Pro); this screen shows only name, phone, and derived booking-status information

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-13 (Client Record Management) -- client list context (per feature-overview.md Cross-Feature Touchpoints; FEAT-13 owns no separate spec ID for this context and defers all list browsing to this screen) | Pro or Support (in an active support session) opens the client list | None -- screen loads the Pro's full client list with search and filter cleared |
| FEAT-24.SPEC-001 (this screen) | Pro/Support types a character into the search box, or selects a filter segment, while already on this screen | None -- narrowing happens in place; this is not a screen transition |
| FEAT-13.SPEC-003 (Client Deletion Confirmation) | Pro completes a client deletion and is returned to the screen they arrived from, when that screen was this one | None -- the deleted client no longer appears when the list next loads |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen -- full client list, search box, filter control | Search, apply/clear filter, tap any result to open FEAT-13.SPEC-001 | -- |
| Platform Operator (Support) | Full screen, identical to the Pro's view, during an active support session opened after a Pro help request (XBR-24) | Search, apply/clear filter, tap any result to open FEAT-13.SPEC-001 -- read-only beyond this screen's own read-only nature (no action on this screen writes data for either role) | -- |
| The Client (Riley) | No | No | Access Matrix (user-persona.md): Client Records = None for the Client role. A Client is never signed in as a Pro, so this screen is unreachable to them; any attempt lands on the Pro sign-in screen per XBR-29 |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (XBR-29); a failed sign-in never reveals whether an account exists |
| Expired session | No | No | Redirected to the Pro sign-in screen with the message "Your session has expired. Sign in to continue."; no in-progress search or filter state is preserved -- there is nothing to preserve, since this screen never persists search or filter state (see Non-Goals) |

## Layout and Content

**Header:** Screen title "Clients," directly below it a search input (placeholder text "Search by name or phone") spanning the header width, with a clear ("x") control that appears once text is entered.

**Filter row:** Below the search input, a single-select segmented control with three segments: "All," "Recently booked," and "Upcoming booking." "All" is selected by default on every screen open.

**Body:** A vertically scrolling list of client result rows. Each row uses the same compact client-identity display as FEAT-13's client list context (per the Brief's Shared UI Patterns): the client's name, phone number, and a small recency/upcoming-booking indicator (a plain-text tag reading "Booked recently" or "Upcoming booking" when either derived condition is true for that client; no tag when neither is true). Rows are ordered by name, ascending, regardless of which filter is active. No row shows the client's email or private note.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Search input and segmented control each span the full available width, stacked as described above; the result list is single-column, full width.
- **Medium size class and above:** Layout is unchanged in structure -- search input, segmented control, and result list remain full width and single-column; only the overall content column widens uniformly with the available space. No structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Search input | Type a character | Narrows the displayed list to clients matching the current search text, per FEAT-24.SPEC-002's matching rule, combined with the active filter segment | List updates in place | Matching rows remain/appear; non-matching rows are removed from view instantly (no loading indicator at stated client volumes) |
| Search input | Tap clear ("x") | Clears the search text; list re-narrows using only the active filter segment (or shows the full list if "All" is active) | Search input empties; list updates | Full filter-only result set appears instantly |
| Segmented control -- "All" | Tap | Clears any active filter; list narrows using only the current search text (or shows the full list if search is also empty) | Segment shows selected state | List updates instantly |
| Segmented control -- "Recently booked" | Tap | Narrows the list to clients meeting FEAT-24.SPEC-002's recency-filter condition, combined with the current search text | Segment shows selected state | List updates instantly |
| Segmented control -- "Upcoming booking" | Tap | Narrows the list to clients meeting FEAT-24.SPEC-002's upcoming-booking condition, combined with the current search text | Segment shows selected state | List updates instantly |
| Client result row | Tap | Navigate to FEAT-13.SPEC-001 (Client Record Detail) for that client | Screen transitions | Standard navigation transition |

### Accessibility Notes

- **Focus order:** Search input -> clear control (when present) -> segmented control ("All" -> "Recently booked" -> "Upcoming booking") -> result rows in displayed order.
- **Dynamic-update announcements:** When the result list narrows or restores after a search or filter change, the new result count is announced to assistive technology (e.g., "12 clients" or "No clients match"), so a screen-reader user is not left reading a stale list silently.
- **Keyboard alternatives:** The segmented control and every result row are reachable and operable by keyboard/switch control; there are no pointer-only gestures on this screen.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loaded (default) | Full client list shown, "All" segment selected, search empty | Screen opens and the client list loads successfully, and the Pro has at least one existing client | Pro/Support types in search or selects a different filter segment |
| Empty (no clients yet) | Body shows the same empty-list treatment the Pro already sees on the underlying client list from FEAT-13 (per product-features.md's Empty field for this feature: "N/A — inherits the underlying client list's own empty state (FEAT-13)"); no "No clients match" message is shown, since this is the absence of any client at all, not a narrowing that produced zero results; search input and filter control remain visible but there is nothing to search or filter | Screen opens and the Pro has zero clients in total, regardless of search text (always empty on open) or filter selection -- distinct from "No Matches," which requires at least one client to exist for search/filter to have narrowed away | A client is created for this Pro (FEAT-05 or FEAT-30) and the screen is next opened, moving it to "Loaded" |
| Narrowed | A subset of the client list shown, matching the current search text and/or filter selection | Pro/Support types a non-empty search or selects "Recently booked" or "Upcoming booking," while the Pro has at least one client in total | Search is cleared and filter returned to "All" |
| No Matches | Plain message "No clients match" in the body area, in place of the list; search input and filter control remain interactive | The Pro has at least one client in total, and the active search/filter combination matches zero of them | Pro/Support changes search text or filter selection to one that matches at least one client |
| Loading | N/A -- per product-features.md's stated Loading behavior for this feature, results are instant for the stated client volumes (100–500, per ASMP-22); no perceptible loading state is designed for this screen | -- | -- |
| Error (search/filter computation fails) | N/A -- per the feature's own States field, a failed search falls back to the full, unfiltered client list (FEAT-24.SPEC-002's fallback rule) rather than an error screen; search and filter controls reset to cleared/"All" | -- | -- |
| Offline/Degraded | No offline banner; search and filter continue to operate against the most recently loaded client list already held by the screen; result rows reflect that last-loaded snapshot, not any change made elsewhere while offline | Connectivity is lost while this screen is open, or the screen is opened while already offline (using the last list loaded during a prior online session, if any) | Connectivity restored -- the client list is refreshed on the next screen open (not automatically mid-session, consistent with this screen never live-updating; see Edge Cases) |

## Validation Rules

N/A -- this is a read-only, input-free-of-validation convenience screen (per product-features.md's Validation & Limits field: "a read-only convenience feature with no input validation beyond a search box"). The search box accepts any text; there is no format, length, or required-field validation to enforce. Matching behavior itself (not validation) is governed by FEAT-24.SPEC-002.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Client result row tap | FEAT-13.SPEC-001 (Client Record Detail) | FEAT-13 (Client Record Management) |

## Data Model

**Creates:** None.
**Reads:** Client -- name, phone, and the derived booking_history field (used by FEAT-24.SPEC-002 to derive the recency and upcoming-booking indicators). email and private_note are never read or displayed by this screen.
**Updates:** None.
**Deletes:** None.

## Business Rules

- All name/phone matching and recency/upcoming-booking derivation is governed by FEAT-24.SPEC-002 (Search Match & Filter Derivation Rules) -- this screen never restates or reimplements the matching or derivation logic.
- Search text and the selected filter segment combine per FEAT-24.SPEC-002's cross-field combination rule -- both conditions must be satisfied for a client to appear in a Narrowed result set.
- If the search/filter computation fails, FEAT-24.SPEC-002's fallback rule applies: the full, unfiltered client list is shown and search/filter inputs reset, rather than an error state (see States).
- XBR-29 governs reachability of this screen entirely: it requires a signed-in Pro or an active Support session; anyone else is sent to the Pro sign-in screen.
- A client deleted via FEAT-13 (XBR-19) simply stops appearing in this screen's list on its next load; this screen performs no deletion of its own and shows no special messaging for the absence.
- A Pro with zero clients in total is shown the Empty (no clients yet) state, never "No clients match" -- the "No Clients Match" message is reserved for a search/filter combination that narrowed an existing, non-empty client list down to zero, per product-features.md's Empty field for this feature.

## Edge Cases

- **Pro/Support navigates away and returns to this screen** -- The client list is re-fetched fresh; search text and filter selection reset to cleared/"All" (consistent with the Non-Goals: no saved searches or persistent filter presets across visits).
- **A client is deleted (FEAT-13) or a new client is created (FEAT-05/FEAT-30) while this screen is open** -- This screen is a snapshot loaded at screen-open time, not live-updating; the change is not reflected until the screen is next opened. No live conflict exists because this screen never writes to the Client entity (Connected Entities: "read -- search/filter only"), so no concurrent-edit conflict handling applies here.
- **Pro/Support taps a result row for a client that was deleted by another session moments earlier** -- Navigation proceeds to FEAT-13.SPEC-001, which owns what is shown for a client record that no longer exists.
- **Search text matches zero clients while a filter segment other than "All" is also active** -- The "No Matches" state is shown; both the search input and the filter control remain interactive so the Pro/Support can adjust either.
- **Pro/Support clears the search text while a filter segment is active** -- The list re-narrows to the filter-only result set; it does not revert to the full unfiltered list unless the filter is also returned to "All."
- **A client has no bookings at all** -- Never appears under "Recently booked" or "Upcoming booking" (per FEAT-24.SPEC-002's derivation, which requires at least one qualifying Booking); still matches under name/phone search or the "All" segment.
- **A brand-new Pro who has never had a booking opens this screen** -- The client list is empty because zero Client records exist for that Pro, not because a search or filter narrowed anything away; the screen shows the Empty (no clients yet) state (inheriting FEAT-13's own empty-list treatment), never "No clients match."
- **Support's session ends while this screen is open** -- Out of scope for this spec; session-boundary behavior is owned by FEAT-19 (Platform Support Read-Only Access).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-24.SPEC-002 (Search Match & Filter Derivation Rules) | References (outbound) | Search matching, recency-window and upcoming-booking derivation, and the search-failure fallback are all defined there |
| FEAT-13.SPEC-001 (Client Record Detail) | Navigation (outbound) | Tapping a result row opens that client's full record |
| FEAT-13 (Client Record Management) -- client list context | Navigation (inbound) | Entry point into this screen, per the Brief's Cross-Feature Touchpoints |
| FEAT-19 (Platform Support Read-Only Access) | Contextual (inbound) | Support reaches this screen during an active, logged support session opened after a Pro help request (XBR-24) |
| FEAT-13.SPEC-003 (Client Deletion Confirmation) | Navigation (inbound) | Pro is returned here after completing a deletion, when this was the originating screen |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| client_search_performed | actor role (Pro or Support), result count, whether a filter segment was also active | A non-empty search text produces a narrowed (or empty) result set | N/A -- no success-metrics.md metric names this feature directly (per feature-overview.md Non-Functional Notes: "No success metric in success-metrics.md names this feature directly -- its quality bar is carried entirely by the Non-Functional Expectations"); recorded per product-features.md's Signals field for the feature's own operational visibility |
| client_filter_applied | actor role (Pro or Support), filter segment selected ("Recently booked" or "Upcoming booking"), result count, whether search text was also active | Pro/Support selects a filter segment other than "All" | N/A -- no success-metrics.md metric names this feature directly (same rationale as client_search_performed) |

## Acceptance Criteria

**FEAT-24.SPEC-001-AC-01:** Given Talia is on the Client Search & Filter screen with her full client list showing, when she types "jan" into the search box, then the list narrows instantly to clients whose name or phone matches "jan" per FEAT-24.SPEC-002's matching rule.

**FEAT-24.SPEC-001-AC-02:** Given Talia has typed a search term that narrowed the list, when she taps the clear ("x") control, then the search input empties and the list returns to the full client list (since the filter segment is still "All").

**FEAT-24.SPEC-001-AC-03:** Given Talia is on this screen with "All" selected, when she taps "Recently booked," then the list narrows to clients meeting FEAT-24.SPEC-002's recency-filter condition.

**FEAT-24.SPEC-001-AC-04:** Given Talia is on this screen with "All" selected, when she taps "Upcoming booking," then the list narrows to clients meeting FEAT-24.SPEC-002's upcoming-booking condition.

**FEAT-24.SPEC-001-AC-05:** Given Talia has "Recently booked" selected and the list is narrowed, when she taps "All," then the filter clears and the list returns to showing every client (search permitting).

**FEAT-24.SPEC-001-AC-06:** Given Talia types "555" into search while "Upcoming booking" is selected, then the list shows only clients that both have a phone number containing "555" and meet the upcoming-booking condition.

**FEAT-24.SPEC-001-AC-07:** Given Talia is on this screen, when she taps a client result row, then the screen navigates to FEAT-13.SPEC-001 (Client Record Detail) for that client.

**FEAT-24.SPEC-001-AC-08:** Given Talia's search and filter combination matches no clients, then the body shows the plain message "No clients match" and the search input and filter control remain interactive.

**FEAT-24.SPEC-001-AC-09:** Given Platform Operator Support has opened a read-only support session on Talia's account after a help request, when Support opens the client list, then Support sees the identical search-and-filter screen and can search and filter, but every action remains read-only.

**FEAT-24.SPEC-001-AC-10:** Given Riley (the Client) has no Pro sign-in, when any attempt is made to reach this screen, then the request is redirected to the Pro sign-in screen per XBR-29.

**FEAT-24.SPEC-001-AC-11:** Given Talia's sign-in session has expired while she is not on this screen, when she next attempts to reach it, then she is redirected to the Pro sign-in screen with the message "Your session has expired. Sign in to continue." and no prior search or filter state is restored, since none is ever preserved.

**FEAT-24.SPEC-001-AC-12:** Given Talia loses connectivity while this screen is open with a narrowed list showing, when she continues typing in search, then narrowing continues to operate against the client list already loaded, with no offline banner or blocking of the search/filter controls.

**FEAT-24.SPEC-001-AC-13:** Given the search/filter computation fails while Talia has an active search and filter, when the failure occurs, then the screen falls back to showing the full, unfiltered client list with search and filter reset, rather than an error screen (per FEAT-24.SPEC-002's fallback rule).

**FEAT-24.SPEC-001-AC-14:** Given Talia navigates away from this screen with "Recently booked" selected and a search term entered, when she returns to this screen later, then the screen opens fresh with the full client list, "All" selected, and search cleared.

**FEAT-24.SPEC-001-AC-15:** Given a client has no bookings at all, when Talia selects "Recently booked" or "Upcoming booking," then that client never appears in the narrowed results, though the client still appears under "All" or a matching name/phone search.

**FEAT-24.SPEC-001-AC-16:** Given Talia's client list holds 500 clients (the stated upper volume per ASMP-22), when she types into search or selects a filter segment, then the list narrows with no perceptible loading state.

**FEAT-24.SPEC-001-AC-17:** Given Talia deletes a client from FEAT-13 while this screen was her point of origin, when the deletion completes and she is returned to this screen, then the deleted client no longer appears once the list next loads.

**FEAT-24.SPEC-001-AC-18:** Given a brand-new Pro has zero clients in total, when they open the Client Search & Filter screen, then the body shows the Empty (no clients yet) state -- inheriting FEAT-13's own empty-list treatment -- and not the "No clients match" message, even though search is empty and "All" is selected.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 5 (Loaded, Empty (no clients yet), Narrowed, No Matches, Offline/Degraded) | 5 |
| Business Rules | 6 | 6 |
| Edge Cases | 8 | 8 |



# Logic/Rule Spec: Search Match & Filter Derivation Rules

## Overview

**Name:** Search Match & Filter Derivation Rules
**ID:** FEAT-24.SPEC-002
**Type:** Logic/Rule
**Purpose:** Governs how a search term matches a client's name or phone number, how the recency and upcoming-booking filter conditions are derived from a client's booking history, how search and an active filter combine, and the fallback behavior when the search/filter computation fails.
**Parent Feature:** FEAT-24 -- Client List Search & Filter
**Governed Entity:** Client (read-only, as searched and filtered by FEAT-24.SPEC-001 -- this spec never writes to Client)

## Scope and Non-Goals

**In Scope:**
- Partial, case-insensitive matching of the search term against a client's name and phone number
- Derivation of the recency-filter condition ("booked in the last N days") from a client's booking history
- Derivation of the upcoming-booking-status condition from a client's booking history
- Combination logic between an active search term and an active filter segment
- Fallback behavior when the search/filter computation cannot complete
- Authorization for the search/filter action itself, by role

**Non-Goals:**
- Matching against fields other than name and phone (e.g., email, address, or private-note content) -- excluded per the Brief's Non-Goals: product-features.md's Key Capabilities name only name/phone for search, and this spec never extends the criteria the product definition did not decide
- Any write to the Client, Booking, or any other entity -- excluded per this feature's Connected Entities: "Client (read -- search/filter only)"; this spec defines derivation over existing data, never a mutation
- Validation of client contact-detail fields (name/phone/email format, length, required-ness) -- owned by FEAT-13.SPEC-005 (Client Field Validation & Access Rules), which governs those fields when the Pro creates or edits a Client record; this spec only reads name and phone for matching, it never validates them
- Saved searches or persistent filter presets -- excluded per the Brief's Non-Goals: no Stage 2 field describes a saved-search mechanism, and this spec defines only the per-invocation matching and derivation logic, not any persistence of a chosen search or filter

## Governed Entity

**Entity:** Client
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| name | text | Client's name -- one of the two searchable fields |
| phone | text | Client's phone number, the identity key within one Pro -- the other searchable field |
| email | text | Client's email address -- not read by this feature (out of scope; see Non-Goals) |
| private_note | text | Pro-only private note -- never read or displayed by this feature (XBR-24, ASMP-23) |
| booking_notes | text | The client's optional per-booking note -- not read by this feature |
| booking_history | derived | Derived list of this client's Bookings with this Pro only -- read by this spec to derive the recency and upcoming-booking conditions; never itself matched against the search term |

**Referenced Entity (read-only):** Booking -- start_time and state fields are read (via booking_history) to derive the recency and upcoming-booking conditions. This spec never creates, updates, or deletes a Booking.

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-24.SPEC-001 | Client Search & Filter | On every search-input change (as the Pro/Support types) and on every filter-segment selection; authorization is enforced on screen entry (this spec's Authorization Rules are consistent with that screen's Access and Visibility table) |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| name | Matching rule: the search term matches when it is found as a case-insensitive substring anywhere in the client's name (e.g., "an" matches "Jane Doe") | Only when the search term is non-empty | On every search-input change | N/A -- this is a match rule, not a validation failure; a non-matching client is simply excluded from the result set, never an error | No |
| phone | Matching rule: the search term is compared against the client's phone number with all non-digit characters (spaces, dashes, parentheses, plus signs) stripped from both the search term and the stored number, then matched as a substring of the resulting digit sequence (e.g., "555" matches "(555) 123-4567" and "+1 555-123-4567") | Only when the search term is non-empty | On every search-input change | N/A -- match rule, not validation | No |
| email | No rule -- field not read by this feature (out of scope per Connected Entities: "read -- search/filter only" and this feature's Key Capabilities, which name only name and phone as searchable fields) | -- | -- | -- | -- |
| private_note | No rule -- field never read or displayed by this feature (XBR-24: Support never sees private client notes; ASMP-23: visible only to the Pro) | -- | -- | -- | -- |
| booking_notes | No rule -- field not read by this feature | -- | -- | -- | -- |
| booking_history | No validation beyond data type -- read-only derived field consumed to compute the recency and upcoming-booking conditions (see Defaults and Derivations); never itself matched against the search term | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Search-and-filter combination | search term (screen input, not a Client field), filter segment selection (screen input), name, phone, booking_history | A client is included in the result set only if BOTH conditions hold: (1) the search term is empty, OR the search term matches the client's name or phone per the rules above; AND (2) the filter segment is "All," OR the client satisfies the recency condition (if "Recently booked" is selected), OR the client satisfies the upcoming-booking condition (if "Upcoming booking" is selected) | N/A -- narrowing produces an empty result set (FEAT-24.SPEC-001's "No Matches" state), never an error message |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Search clients (enter or change search text) | The Pro (Talia) | Always, for the Pro's own clients only | -- |
| Search clients (enter or change search text) | Platform Operator (Support) | Only during an active, logged support session opened after a Pro help request (XBR-24); the Pro's own clients only | -- |
| Search clients (enter or change search text) | The Client (Riley) | Never | The screen is unreachable to the Client role; any attempt is redirected to the Pro sign-in screen per XBR-29 (Access Matrix: Client Records = None for the Client role) |
| Apply recency or upcoming-booking filter | The Pro (Talia) | Always | -- |
| Apply recency or upcoming-booking filter | Platform Operator (Support) | Only during an active support session (same condition as search) | -- |
| Apply recency or upcoming-booking filter | The Client (Riley) | Never | Same as above -- screen unreachable |
| View a narrowed or full result row (name, phone, recency/upcoming tag) | The Pro (Talia) | Always | -- |
| View a narrowed or full result row (name, phone, recency/upcoming tag) | Platform Operator (Support) | Always, during an active support session -- never including the client's private_note (XBR-24) | -- |
| View a narrowed or full result row (name, phone, recency/upcoming tag) | The Client (Riley) | Never | Same as above -- screen unreachable |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| recency-filter condition (derived, not a stored field) | True when the client has at least one Booking in booking_history whose start_time falls within the last platform parameter: `client-recency-filter-window-days` days up to and including the current moment (BRIEF.md's stated example: 30 days), and whose state is not Cancelled by Client, Cancelled by Pro, or Expired (unpaid) | Recomputed every time the "Recently booked" filter segment is evaluated (on screen load with that segment active, and on every subsequent search-input change while it remains active) | No -- always derived; not user-editable |
| upcoming-booking condition (derived, not a stored field) | True when the client has at least one Booking in booking_history whose state is Confirmed and whose start_time is after the current moment | Recomputed every time the "Upcoming booking" filter segment is evaluated (on screen load with that segment active, and on every subsequent search-input change while it remains active) | No -- always derived; not user-editable |
| result set | Derived from the Cross-Field Rules combination logic above, applied to the currently loaded client list | Recomputed on every search-input change and every filter-segment selection | N/A -- not a stored field |

## Business Rules

- Search-failure fallback: if the search/filter computation cannot complete (e.g., the booking-history lookup needed for a recency or upcoming-booking derivation fails), FEAT-24.SPEC-001 falls back to displaying the full, unfiltered client list already loaded, with the search input and filter segment reset to cleared/"All," rather than showing an error screen. This is a deliberate product decision from the feature's own States field ("Error: a failed search falls back to the full, unfiltered list rather than an error screen"), not a generic error-handling default.
- The recency window is fixed at platform parameter: `client-recency-filter-window-days` for every Pro -- it is never configured per Pro or per client.
- XBR-24 governs Support's participation in this spec entirely: Support's search, filter, and result-viewing rights are identical in function to the Pro's, but scoped to an active, logged support session, and the private_note field is never surfaced regardless of role.
- A client's recency and upcoming-booking conditions are derived fresh at evaluation time from booking_history -- they are never cached or stored on the Client record itself, so a booking created, cancelled, or completed elsewhere is reflected the next time this screen's derivation runs (on the screen's next load, per FEAT-24.SPEC-001's snapshot behavior).

## Edge Cases

- **A client's most recent qualifying Booking has start_time exactly platform parameter: `client-recency-filter-window-days` days before the current moment** -- Matches the recency condition; the window is inclusive of its boundary day.
- **A client's most recent qualifying Booking has start_time one day older than platform parameter: `client-recency-filter-window-days` days before the current moment** -- Does not match the recency condition.
- **A client has a Confirmed Booking with start_time in the past** -- Does not match the upcoming-booking condition (start_time must be after the current moment); this scenario should not occur under normal operation since a past Confirmed booking auto-completes per XBR-12, but the rule holds regardless of how it arises.
- **A client has only a Cancelled by Client or Cancelled by Pro Booking within the recency window** -- Does not match the recency condition; cancelled bookings are explicitly excluded.
- **A client has no bookings at all (booking_history is empty)** -- Never matches "Recently booked" or "Upcoming booking"; still matches "All" and any search term matching name or phone.
- **The search term contains only formatting characters (e.g., "--" or spaces)** -- After non-digit stripping for phone comparison, an empty digit sequence never matches any phone number; the term is still compared literally (case-insensitively) against name, so it may or may not match depending on the client's name.
- **Two clients share the same phone-number digit sequence after formatting is stripped (a data anomaly outside this spec's control)** -- Both are returned as matches; this spec does not deduplicate or flag anomalies in the underlying Client data, since Client record integrity (phone-number match resolving to a single record) is owned by the dependency map's Contention resolution for the Client entity, not by this spec.
- **The search/filter computation fails while a non-"All" filter segment is also active** -- The fallback clears both the search term and the filter segment back to their defaults, per the Business Rules fallback behavior; the Pro/Support is not left in a partially-reset state.

## Acceptance Criteria

**FEAT-24.SPEC-002-AC-01:** Given Talia's client "Jane Doe" exists, when the search term "jan" is entered, then that client's name is evaluated as a match (case-insensitive substring).

**FEAT-24.SPEC-002-AC-02:** Given Talia's client has phone number "(555) 123-4567," when the search term "5551234" is entered, then that client's phone is evaluated as a match after non-digit stripping.

**FEAT-24.SPEC-002-AC-03:** Given Talia's client has phone number "(555) 123-4567," when the search term "999" is entered, then that client's phone is not evaluated as a match.

**FEAT-24.SPEC-002-AC-04:** Given a client has a Booking with start_time exactly platform parameter: `client-recency-filter-window-days` days before the current moment and state Completed, when the recency condition is evaluated, then it evaluates true (inclusive boundary).

**FEAT-24.SPEC-002-AC-05:** Given a client's only Booking within the window has state Cancelled by Client, when the recency condition is evaluated, then it evaluates false.

**FEAT-24.SPEC-002-AC-06:** Given a client has a Booking in state Confirmed with start_time three days in the future, when the upcoming-booking condition is evaluated, then it evaluates true.

**FEAT-24.SPEC-002-AC-07:** Given a client has no Booking in state Confirmed with a future start_time, when the upcoming-booking condition is evaluated, then it evaluates false.

**FEAT-24.SPEC-002-AC-08:** Given Talia has entered search text "jan" and selected "Recently booked," when the result set is computed, then only clients whose name or phone matches "jan" AND who satisfy the recency condition are included.

**FEAT-24.SPEC-002-AC-09:** Given Talia has entered no search text and selected "All," when the result set is computed, then every client in the Pro's list is included.

**FEAT-24.SPEC-002-AC-10:** Given the booking-history lookup needed to evaluate an active filter fails, when the failure occurs, then FEAT-24.SPEC-001 displays the full, unfiltered client list with search and filter reset, rather than an error screen.

**FEAT-24.SPEC-002-AC-11:** Given Talia (the Pro) is searching her own client list, when she enters any search term, then the action is allowed without restriction.

**FEAT-24.SPEC-002-AC-12:** Given Platform Operator Support has an active, logged support session on Talia's account opened after a help request, when Support searches or applies a filter, then the action is allowed identically to the Pro's.

**FEAT-24.SPEC-002-AC-13:** Given Platform Operator Support is viewing a search result row, when the row renders, then it never includes that client's private_note.

**FEAT-24.SPEC-002-AC-14:** Given Riley (the Client) has no Pro sign-in or active support session, when any attempt is made to invoke this feature's search or filter action, then the action is denied and the request is redirected to the Pro sign-in screen per XBR-29.

**FEAT-24.SPEC-002-AC-15:** Given a client has no bookings at all, when either "Recently booked" or "Upcoming booking" is evaluated for that client, then both evaluate false, while the client still matches under "All" or a matching name/phone search.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 (name, phone, plus the 3 out-of-scope fields noted as one group) | 4 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 9 | 9 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 8 | 8 |


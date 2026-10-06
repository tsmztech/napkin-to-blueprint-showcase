---
document_type: spec
spec_type: screen
spec_id: FEAT-24.SPEC-001
spec_name: Client Search & Filter
spec_slug: client-search-filter
parent_feature: FEAT-24
parent_feature_name: Client List Search & Filter
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 18
---

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

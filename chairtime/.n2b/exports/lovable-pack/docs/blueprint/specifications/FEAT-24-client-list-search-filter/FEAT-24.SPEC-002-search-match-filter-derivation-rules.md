---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-24.SPEC-002
spec_name: Search Match & Filter Derivation Rules
spec_slug: search-match-filter-derivation-rules
parent_feature: FEAT-24
parent_feature_name: Client List Search & Filter
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 9
acceptance_criteria_count: 15
---

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

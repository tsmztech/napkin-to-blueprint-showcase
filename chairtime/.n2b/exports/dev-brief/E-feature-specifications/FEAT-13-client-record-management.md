# FEAT-13 — Client Record Management

This chapter covers Client Record Management (FEAT-13), a Important-tier feature. It carries 6 specifications carrying 82 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-13.SPEC-001 | Client Record Detail | screen | 15 |
| FEAT-13.SPEC-002 | Client Contact Edit | screen | 13 |
| FEAT-13.SPEC-003 | Client Deletion Confirmation | screen | 13 |
| FEAT-13.SPEC-004 | Client Deletion Execution | automation | 10 |
| FEAT-13.SPEC-005 | Client Field Validation & Access Rules | logic-rule | 17 |
| FEAT-13.SPEC-006 | Deletion Eligibility & Retention Rule | logic-rule | 14 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Client Record Management

## Summary

**Feature:** Client Record Management
**ID:** FEAT-13
**Description:** The Pro maintains a simple record for each client — contact details, private notes, and booking history with that Pro — and can permanently delete a client's record on request.
**Priority:** Important
**Phase:** MVP
**Type:** User-Facing
**Rationale:** BRIEF.md's Target Users & Roles states the Pro "sees every... client" and "can delete a client's record on request," and the Constraints section makes deletion-on-request an explicit regulatory-adjacent obligation. Ranked Important rather than Core because a client record is created automatically by the act of booking (FEAT-05) — this feature is about managing that record afterward, not about the core booking loop itself. MVP phase: the delete-on-request obligation is a launch-blocking privacy commitment, not something safe to defer.

**Key Capabilities:**
- View a client's contact details and full booking history with this Pro
- Add or edit a private note about a client (preferences, allergies noted informally, etc.)
- Permanently delete a client's record on their request
- Correct a client's name, email, or phone number (for example, a typo at booking)

**Connected Entities:** Client (read, update, delete)

**Access (from the Access Matrix):** The Pro has Full access to their own clients only. Clients have no access to this feature's screens — a Client sees their own contact details only implicitly, through their own booking history in Client Booking Identity (FEAT-06), never through this feature. Platform Operator (Support) has View-only access, explicitly excluding the Pro's private notes field, delivered through Platform Support Read-Only Access (FEAT-19) reading this feature's Client entity rather than through this feature's own screens.

**Communications:** N/A — record management itself sends no notifications; a deletion is silent to the client beyond honoring their own request. (No Communications lines to elaborate into Notification specs; `notification_count: 0` is legal and expected here.)

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-13.SPEC-001 | Client Record Detail | Screen | The Pro (Talia) | The Pro views a client's contact details, private note (including one left after a previous appointment), and full booking history with this Pro, and edits the private note directly here |
| FEAT-13.SPEC-002 | Client Contact Edit | Screen | The Pro (Talia) | The Pro corrects a client's name, email, or phone number |
| FEAT-13.SPEC-003 | Client Deletion Confirmation | Screen | The Pro (Talia) | The Pro requests permanent deletion of a client's record, sees the eligibility check result, and confirms an irreversible delete |
| FEAT-13.SPEC-004 | Client Deletion Execution | Automation | The Pro (Talia) | Hard-deletes the client's contact details and private note, cascades to remove Messaging Consent, and retains de-identified financial and timeline history |
| FEAT-13.SPEC-005 | Client Field Validation & Access Rules | Logic/Rule | The Pro (Talia), Platform Operator (Support) | Governs name/phone/email/note field validation, phone-number identity uniqueness within one Pro, and the private-note field's Pro-only visibility |
| FEAT-13.SPEC-006 | Deletion Eligibility & Retention Rule | Logic/Rule | The Pro (Talia) | Governs when a client record may be deleted (blocked by an upcoming booking), the irreversibility of deletion, and the retention/de-identification of related records afterward |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| View a client's contact details and full booking history with this Pro | FEAT-13.SPEC-001 | Primary layout of the Client Record Detail screen | Phase 2 (Explicit) |
| Add or edit a private note about a client | FEAT-13.SPEC-001, FEAT-13.SPEC-005 | Inline note field on the detail screen; validated against the 1,000-character limit and Pro-only visibility rule | Phase 2 (Explicit) |
| Permanently delete a client's record on their request | FEAT-13.SPEC-003, FEAT-13.SPEC-004, FEAT-13.SPEC-006 | Confirmation screen gated by the eligibility rule, executed by the deletion automation | Phase 2 (Explicit) + Phase 4 (Trigger-Response, for execution) |
| Correct a client's name, email, or phone number | FEAT-13.SPEC-002, FEAT-13.SPEC-005 | Dedicated contact-edit screen, validated by the field validation rules | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-13.SPEC-005 | Client Field Validation & Access Rules | Phase 5 (Rule-Constraint Discovery) | The feature's Validation & Limits field states multiple field-level rules (note length, phone-format, email-conditional-required, phone-identity-uniqueness) plus a field-level visibility rule from the Access field (Support never sees private notes) — conditional, cross-screen rules shared by SPEC-001 and SPEC-002, crossing the standalone-spec threshold |
| FEAT-13.SPEC-006 | Deletion Eligibility & Retention Rule | Phase 3 (Entity-Lifecycle Analysis) + Phase 5 (Rule-Constraint Discovery) | The CRUD matrix's Delete/Archive cell requires an explicit soft-vs-hard/restore/cascade/retention decision; the Validation & Limits field adds a conditional gate (upcoming-booking block) and the Contention line adds a concurrency rule (refuse-with-refresh after deletion) — together a rule set shared by the confirmation screen and the deletion automation |
| FEAT-13.SPEC-004 | Client Deletion Execution | Phase 4 (Trigger-Response Analysis) | Confirmed deletion is not a simple single-entity write: it cascades to Messaging Consent, must retain de-identified Booking/Deposit Transaction/Activity Event history, and must be irreversible — processing logic crossing multiple entities, which the standalone-spec vs. inline rule assigns to a standalone Automation rather than an inline screen write |

## Entity-Lifecycle Coverage Matrix

**Entity: Client**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A | Created by FEAT-05 (a client's first booking) and FEAT-30 (Pro books a client in) per the dependency map — this feature never creates a Client record | Cross-feature; a later booking after deletion also creates a new record here (FEAT-05's responsibility, not a re-creation of the old one) |
| Read (single) | FEAT-13.SPEC-001 | Client Record Detail screen loads one client's contact details, private note, and booking history | -- |
| Read (list) | N/A | Browsing/searching across a Pro's clients is owned by Client List Search & Filter (FEAT-24, v1) and shown in summary form on Pro Daily Schedule Dashboard (FEAT-12); this feature has no client-list screen of its own | Cross-feature |
| Update | FEAT-13.SPEC-001 (private note), FEAT-13.SPEC-002 (name/email/phone) | Pro edits the note directly on the detail screen; Pro corrects contact fields on the dedicated edit screen | A phone-number change cascades to Access Link invalidation and fresh consent requirement — see Cross-Feature Touchpoints; both edits are validated by FEAT-13.SPEC-005 |
| Delete/Archive | FEAT-13.SPEC-003 (confirmation) + FEAT-13.SPEC-004 (execution), gated by FEAT-13.SPEC-006 | Hard delete, not soft: contact details, private note, and Messaging Consent are permanently removed (no restore path — deletion is a deliberate, confirmed, non-reversible action per Validation & Limits); Booking and Deposit Transaction history is retained but de-identified per scope-boundaries.md SC-22 and XBR-19; a later booking from the same person creates a new, unconnected Client record rather than resurrecting the deleted one | Blocked while an upcoming booking exists; the Pro is offered a one-step cancel-with-full-refund through FEAT-30 (see Cross-Feature Touchpoints) |
| State Transition | N/A | The Client entity has no lifecycle states beyond existing/deleted — the Domain Entity Inventory in product-features.md defines no intermediate state | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Booking | FEAT-13.SPEC-001 | Client Record Detail displays this client's full booking history with this Pro (booking_history is a derived list) |
| Messaging Consent | FEAT-13.SPEC-004 | Not read for display, but cascade-deleted as part of client deletion execution (evidence retained only where law requires — see FEAT-13.SPEC-006) |
| Access Link | Cross-feature (FEAT-06) | Invalidated when the client's phone number changes (FEAT-13.SPEC-002) or when the client's record is deleted (FEAT-13.SPEC-004); this feature never reads or writes Access Link records directly |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Pro opens a client's record | Load contact details, private note, and booking history; log client_record_viewed | Inline in triggering screen | FEAT-13.SPEC-001 |
| Pro adds or edits the private note and saves | Persist note (up to 1,000 characters); log client_note_added | Inline in triggering screen | FEAT-13.SPEC-001 |
| Pro saves corrected contact details | Validate against field rules (including phone-identity uniqueness within this Pro) and persist name/email/phone | Standalone Logic/Rule (validation) + inline write in triggering screen | FEAT-13.SPEC-005 / FEAT-13.SPEC-002 |
| Pro changes a client's phone number | Invalidate that client's existing access links; require the client to re-grant texting consent for the new number before any text is sent | Cross-feature | FEAT-06 (Client Booking Identity) / FEAT-14 (Messaging Consent Management) responsibility |
| Pro attempts to delete a client with an upcoming booking | Block the deletion and show the current booking; offer a one-step cancel-with-full-refund | Standalone Logic/Rule (eligibility check), with a cross-feature action | FEAT-13.SPEC-006, routing to FEAT-30 (Pro Booking Management) |
| Pro confirms deletion of an eligible client | Hard-delete contact details and private note; cascade-delete Messaging Consent; retain de-identified Booking/Deposit Transaction/Activity Event history; log client_record_deleted | Standalone Automation | FEAT-13.SPEC-004 |
| A concurrent edit is attempted on a client record that has just been deleted | Refuse the edit and prompt the Pro to refresh | Standalone Logic/Rule | FEAT-13.SPEC-006 |
| A deleted client books again later | Create a new, unconnected Client record — never resurrect the deleted one | Cross-feature | FEAT-05 (Public Booking Page & Booking Flow) responsibility |
| Two first bookings from the same phone number arrive close together | Resolve to a single Client record via phone-number match — never a duplicate | Cross-feature | FEAT-05 responsibility (creation-side contention; this feature only maintains the resulting single record) |
| Client's device or connection is offline | Show the most recently loaded client list read-only; this feature's own screens require a live connection to load or save | Inline in triggering screen | FEAT-13.SPEC-001 |

## Shared Context

**Shared Entities:**
- Client record -- read by FEAT-13.SPEC-001; updated by FEAT-13.SPEC-001 (private_note) and FEAT-13.SPEC-002 (name, phone, email); deleted by FEAT-13.SPEC-003/FEAT-13.SPEC-004; validated by FEAT-13.SPEC-005; deletion-gated by FEAT-13.SPEC-006. Fields relevant to this feature: name (required, 1–100 characters), phone (required, valid reachable format, identity key within one Pro), email (required when texts are declined, otherwise optional), private_note (Pro-only, up to 1,000 characters). booking_notes (the client's own optional per-booking note) and booking_history (derived) are read-only here — booking_notes is client-authored via other features and never edited from this feature.

**Shared UI Patterns:**
- Client identity header -- the same contact-summary display (name, phone, email) appears atop the Client Record Detail, Client Contact Edit, and Client Deletion Confirmation screens; Spec Writers for all three should describe it once, consistently.

**Shared Validation:**
- FEAT-13.SPEC-005 defines all field validation (name/phone/email/note) and the private-note field-level visibility exclusion; FEAT-13.SPEC-001 and FEAT-13.SPEC-002 both reference it rather than restating the rules.
- FEAT-13.SPEC-006 defines deletion eligibility and retention; FEAT-13.SPEC-003 and FEAT-13.SPEC-004 both reference it rather than restating the rule.

**Note on the Client role (grounded-roles self-check):** The Access Matrix gives the Client role "None" on the Client Records capability group. A Client's implicit view of their own contact details is executed entirely by Client Booking Identity (FEAT-06), not by this feature. No spec in this Brief therefore touches "The Client (Riley)" — this is a deliberate absence, not an omission, and is consistent with the Access field's own wording.

## Internal Dependency Map

```
SPEC-001 (Client Record Detail) -> [Pro taps "edit contact"] -> SPEC-002 (Client Contact Edit)
SPEC-001 (Client Record Detail) -> [Pro taps "delete client"] -> SPEC-003 (Client Deletion Confirmation)
SPEC-001 (Client Record Detail) -> [Pro edits and saves the private note] -> validated by SPEC-005 -> inline persist, no navigation
SPEC-002 (Client Contact Edit) -> [Pro taps Save] -> validated by SPEC-005 (Client Field Validation & Access Rules) -> [valid] -> SPEC-001 (Client Record Detail)
SPEC-003 (Client Deletion Confirmation) -> [Pro confirms intent to delete] -> checked by SPEC-006 (Deletion Eligibility & Retention Rule)
SPEC-006 -> [blocked: upcoming booking exists] -> SPEC-003 (shows block message, offers FEAT-30 cancel-with-refund path)
SPEC-006 -> [eligible] -> SPEC-004 (Client Deletion Execution)
SPEC-004 (Client Deletion Execution) -> [deletion complete] -> Pro returned to the screen they came from (FEAT-12 dashboard or FEAT-24 client list)
```

**Default Entry:** SPEC-001 (Client Record Detail) -- this feature has no list screen of its own; the Pro always arrives here from Pro Daily Schedule Dashboard (FEAT-12) or Client List Search & Filter (FEAT-24).

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-13.SPEC-001 | Inbound | FEAT-12 (Pro Daily Schedule Dashboard) | Pro taps a client on a booking row to open the client record and private note | Pro taps a client on the dashboard |
| FEAT-13.SPEC-001 | Inbound | FEAT-24 (Client List Search & Filter) | Pro taps a client found via search to open the client record | Pro taps a client in search results (v1) |
| FEAT-13.SPEC-001 | Inbound | FEAT-05 (Public Booking Page & Booking Flow) | A client's contact details and record originate from that client's first booking with this Pro | Client completes first booking |
| FEAT-13.SPEC-002 | Outbound | FEAT-06 (Client Booking Identity) | A changed phone number invalidates that client's existing access links (XBR-18) | Pro saves a changed phone number |
| FEAT-13.SPEC-002 | Outbound | FEAT-14 (Messaging Consent Management) | A changed phone number requires the client to re-grant texting consent for the new number (XBR-15) | Pro saves a changed phone number |
| FEAT-13.SPEC-003 | Outbound | FEAT-30 (Pro Booking Management) | Deletion blocked by an upcoming booking offers a one-step cancel-with-full-refund | Pro attempts to delete a client with an upcoming booking |
| FEAT-13.SPEC-004 | Outbound | FEAT-16 (Booking & Payment Activity Record) | De-identified financial and timeline records are retained for dispute/audit purposes after deletion (XBR-19) | Client deletion executes |
| FEAT-13 (all specs) | Inbound | FEAT-19 (Platform Support Read-Only Access) | Support views a client record read-only, with the private-note field excluded, for troubleshooting after a Pro's help request (XBR-24) | Support opens a Pro account |

## Non-Functional Notes

**Data volumes / growth:** A few hundred pros in year one, each with roughly 100–500 clients (ASMP-22); at this scale the States field is explicit that record loading is instant — this feature has no meaningful Loading state to design for, and must stay equally responsive as a Pro's client base and booking history grow over multiple years.

**Responsiveness:** Per the feature's own States field, the Client Record Detail screen loads instantly for the stated volumes (100–500 clients); a failed save preserves the Pro's entered text with a retry option rather than forcing re-entry (ASMP-27's correctness bar against silently losing entered data).

**Data sensitivity / privacy:** Client records hold personal data — name, phone, email, and the Pro's private notes — visible only to the Pro and, for their own details, the Client (through FEAT-06, ASMP-23). The private_note field carries the feature's strictest classification: it is never shown to Support, even in a troubleshooting context (Access field; XBR-24). No health data is captured, by design (scope-boundaries.md SC-08). A client's data is deletable on request and, once deleted, is never visible to any other Pro or client (BRIEF.md Constraints; SC-03).

**Compliance flags:** Client deletion functions as the product's data-subject deletion commitment (BRIEF.md's Rationale calls it "an explicit regulatory-adjacent obligation"), executed completely in one action because Chairtime never stores card data itself (SC-11). A phone-number change requires fresh texting consent under US texting-consent rules (ASMP-24, XBR-15) before any further text is sent to that number.

**Analytics linkage (from Signals):** client_record_viewed (SPEC-001 open), client_note_added (SPEC-001 note save), client_record_deleted (SPEC-004 execution). These are this feature's contribution to the product's analytics baseline; no success metric names this feature directly, but SPEC-001's private-note display is the mechanism behind the "Check a client note" moment in Talia's Between-Clients Day journey, which itself feeds the Daily Dashboard Glance Speed metric (owned by FEAT-12).

**Offline / degraded posture:** Per ASMP-27 and this feature's own States field, the most recently loaded client list remains viewable read-only when offline; opening a specific client record, editing a note, correcting contact details, or deleting a record all require a live connection and say so plainly when it is missing. This is annotated on SPEC-001, SPEC-002, and SPEC-003 rather than restated as a separate spec.

## Non-Goals

- **Client self-service deletion or account portal** -- Excluded per scope-boundaries.md SC-04: clients have no password-style account, so a client cannot request or execute their own deletion in-product; they request it informally (per BRIEF.md's Problem Statement, e.g. a DM or a message to the Pro) and the Pro performs the deletion through this feature. There is no client-facing delete-my-data flow.
- **Health or medical intake content in the private note** -- Excluded per scope-boundaries.md SC-08: the private note is informal preference text (e.g., "prefers a lighter volume"); this feature does not structure, validate against, or invite health-data fields, consistent with the brief's exclusion of intake forms.
- **Restoring a deleted client record (undo)** -- Intentional lifecycle decision, not an omission: the feature's own Validation & Limits field states deletion "is not reversible, consistent with 'delete on request' meaning delete." No restore path or undo window exists for the Client entity itself (contrast with the de-identified financial history retained separately per SC-22).
- **A shared or cross-pro client profile** -- Excluded per scope-boundaries.md SC-04: a client who books with two Pros gets two separate, unconnected Client records; this feature never merges or links records across Pro accounts, consistent with BRIEF.md's Constraints that a client's data is "never visible to any other pro or client" (SC-03).
- **Exporting client data** -- Owned by FEAT-29 (Pro Sign-In & Account Lifecycle, XBR-29's named owner), per the dependency map's Client entity Lifecycle line ("Read by ... FEAT-29 (export)"); this feature displays and manages the record but does not build any export/download function itself.
- **Support editing, refunding, or acting on a client record** -- Excluded per scope-boundaries.md SC-05: Support's access to this feature's data is strictly View-only, delivered through FEAT-19, and never includes the private-note field; Support cannot correct contact details or perform a deletion on the Pro's behalf.



# Screen Spec: Client Record Detail

## Overview

**Name:** Client Record Detail
**ID:** FEAT-13.SPEC-001
**Type:** Screen
**Purpose:** The Pro views a client's contact details, private note, and full booking history with this Pro, and edits the private note directly on this screen.
**Parent Feature:** FEAT-13 -- Client Record Management

## Scope and Non-Goals

**In Scope:**
- Displaying the client identity header (name, phone, email)
- Displaying the private note field, editable inline, with save and error handling
- Displaying the client's full booking history with this Pro (past and upcoming bookings)
- Navigation to Client Contact Edit (FEAT-13.SPEC-002) and Client Deletion Confirmation (FEAT-13.SPEC-003)
- Offline/degraded behavior consistent with the Brief's Non-Functional Notes

**Non-Goals:**
- Editing the client's name, phone, or email -- handled by FEAT-13.SPEC-002 (Client Contact Edit); this screen only edits the private note inline
- A client-facing view of this screen -- excluded per the Access Matrix (user-persona.md), which gives the Client role "None" on Client Records; a Client's implicit view of their own contact details runs entirely through FEAT-06 (Client Booking Identity), never through this screen
- A list or search view across a Pro's clients -- owned by FEAT-24 (Client List Search & Filter, v1) and summarized on FEAT-12 (Pro Daily Schedule Dashboard); this screen has no client-list entry point of its own, per the Brief's Entity-Lifecycle Coverage Matrix
- Exporting the client's data -- owned by FEAT-29 (Pro Sign-In & Account Lifecycle), per the dependency map's Client entity Lifecycle line

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-12.SPEC-001 (Today's & Upcoming Schedule) | Pro taps a client's name on a booking row | Client ID |
| FEAT-24.SPEC-001 (Client Search & Filter) (Client List Search & Filter, v1) | Pro taps a client found via search | Client ID |
| FEAT-13.SPEC-002 (Client Contact Edit) | Pro taps Save with valid contact data | Updated contact fields reflected on return |
| FEAT-13.SPEC-003 (Client Deletion Confirmation) | Pro cancels the deletion flow before confirming | None -- client record unchanged |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | All documented actions (edit private note, navigate to contact edit, navigate to deletion) | -- |
| The Client (Riley) | No | No | Per XBR-29, redirected to the Pro sign-in screen if this screen's route is reached directly; no in-product navigation ever routes a Client here -- a Client's own view of their contact details is delivered entirely by FEAT-06 (Client Booking Identity) |
| Platform Operator (Support) | No -- Support's equivalent read-only view of this same Client entity (excluding the private_note field) is delivered by FEAT-19 (Platform Support Read-Only Access) reading this entity directly, never by this screen | No | If Support's session attempts this screen's route directly, redirected to the Pro sign-in screen per XBR-29 -- this is a Pro-facing screen, not the support view |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29); no client data is exposed before authentication |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue."; any in-progress, unsaved private-note text is preserved locally and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title showing the client's name, with a back arrow (returns to the screen the Pro arrived from -- FEAT-12 or FEAT-24) and an overflow menu containing "Edit contact" (navigates to FEAT-13.SPEC-002) and "Delete client" (navigates to FEAT-13.SPEC-003).

**Client Identity Header:** Below the screen header -- the client's name, phone number, and email address, displayed read-only (the same identity-summary display shared with FEAT-13.SPEC-002 and FEAT-13.SPEC-003, per the Brief's Shared UI Patterns). Email is shown as "-- (not on file)" when the client has no email address.

**Private Note Section:** A labeled multi-line text field ("Private note"), pre-filled with the note's current text (or empty if none exists yet). Directly below the field, a save action button, initially inactive until the text changes. A helper caption reads "Only you can see this note" (reflecting private_note's Pro-only visibility, governed by FEAT-13.SPEC-005).

**Booking History Section:** Below the private note, a heading "Booking history with [client name]" followed by a reverse-chronological list of this client's bookings with this Pro, each row showing: date and time, service name, and outcome status (Confirmed, Completed, No-Show, Cancelled by Client, Cancelled by Pro, Rescheduled). Each row is display-only on this screen -- it does not navigate elsewhere; booking-level actions belong to FEAT-12 and FEAT-30, not to this feature.

### Responsive Behavior

- **Compact breakpoint:** Single-column stack in the order described above (header, identity header, private note, booking history), full width.
- **Medium size class and above:** Content remains single-column, capped at a consistent platform-wide content width and horizontally centered; no structural change beyond width capping.
- **Booking history list:** Each row remains a single line (compact) or expands to show slightly more spacing between elements (medium and above) -- no structural change to what information each row shows.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to the screen the Pro arrived from (FEAT-12 or FEAT-24) | Screen closes | Standard navigation transition |
| Overflow menu -- "Edit contact" | Tap | Navigate to FEAT-13.SPEC-002 (Client Contact Edit) | Screen transitions | Standard navigation transition |
| Overflow menu -- "Delete client" | Tap | Navigate to FEAT-13.SPEC-003 (Client Deletion Confirmation) | Screen transitions | Standard navigation transition |
| Private note field | Type | Captures text input, up to the 1,000-character limit governed by FEAT-13.SPEC-005 | Save button becomes active; a character count appears once within 100 characters of the limit | Standard input focus state; count shown as "912/1,000" style text |
| Private note field | Type past 1,000 characters | Further input is blocked at the field level | Field stops accepting characters | Caption below the field: "Private note can be up to 1,000 characters." |
| Save button (private note) | Tap | Validates the note via FEAT-13.SPEC-005, then persists the note; logs client_note_added | Button shows a brief loading state | Success: caption changes to "Saved" and fades after a moment. Failure: inline error banner above the field, entered text preserved |
| Save button (while loading) | Tap | No action -- debounced | None | Button remains in its loading state |
| Booking history row | Tap | None -- display-only | No state change | None; no navigation or expansion occurs from this row |

### Accessibility Notes

- **Focus order:** Back arrow -> overflow menu -> client identity header (name, phone, email, read-only) -> private note field -> save button -> booking history heading -> each booking history row in date order.
- **Validation and save announcements:** A private-note validation error is announced to assistive technology and programmatically associated with the field; the "Saved" confirmation is announced once when it appears.
- **Keyboard alternatives:** Every action on this screen (navigation, note editing, saving) is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loaded (default) | Client identity header, private note (or an empty note field with placeholder "No note yet"), and booking history all populated | Screen opens and the client record loads successfully | User navigates away or edits the note |
| Note Editing | Private note field shows in-progress text; Save button active | User types in the private note field | User taps Save, or discards changes by navigating away |
| Saving Note | Save button shows a loading indicator; field remains editable but a second save is debounced | User taps Save on the private note | Save completes (success or failure) |
| Note Save Error | Inline error banner above the private note field; entered text preserved, Save button re-enabled for retry | The note save fails | User taps Save again and it succeeds, or navigates away (text remains unsaved locally only for this session) |
| Empty Booking History | Booking history section shows a plain message: "No bookings with this client yet" instead of a blank list | The client has zero bookings with this Pro (data inconsistency scenario -- FEAT-13.SPEC-001 typically only exists for a client created by a first booking, so this is rare but handled) | N/A -- this state persists until a booking exists |
| Offline/Degraded | Banner "You're offline -- reconnect to view or edit this client's record." at the top; the most recently loaded contact details, note, and booking history remain visible read-only; the private note field is disabled for editing | Connectivity is lost while the screen is open, or the screen is opened without connectivity | Connectivity is restored -- the banner clears and the private note field re-enables |

## Validation Rules

Validation governed by FEAT-13.SPEC-005 (Client Field Validation & Access Rules). See that spec for the private note's 1,000-character limit and the private-note field's Pro-only visibility rule. This screen checks the character limit on input (blocking further typing at the limit) and re-validates on save.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | The screen the Pro arrived from | FEAT-12 or FEAT-24 |
| "Edit contact" tap | FEAT-13.SPEC-002 (Client Contact Edit) | -- |
| "Delete client" tap | FEAT-13.SPEC-003 (Client Deletion Confirmation) | -- |

## Data Model

**Creates:** None.
**Reads:** Client record -- name, phone, email, private_note fields; Booking (read-only, per the dependency map's Referenced Entities) -- the derived booking_history list of this client's bookings with this Pro (service, start_time, state).
**Updates:** Client record -- private_note field only (up to 1,000 characters), per FEAT-13.SPEC-005.
**Deletes:** None -- deletion is handled by FEAT-13.SPEC-003 and FEAT-13.SPEC-004, not this screen.

## Business Rules

- Private note validation and the note field's Pro-only visibility are governed by FEAT-13.SPEC-005 (Client Field Validation & Access Rules) -- this screen enforces but does not restate those rules.
- Post-deletion concurrent-edit refusal (a save against a deleted client is refused with "This client's record no longer exists. It may have been deleted.") is governed by FEAT-13.SPEC-006 (Deletion Eligibility & Retention Rule) -- this screen enforces but does not restate that rule.
- Only the private_note field is editable here; name, phone, and email are read-only on this screen and are corrected only through FEAT-13.SPEC-002.
- This screen requires a live connection to load or save (per the Brief's Non-Functional Notes); it does not queue an offline note save for later submission.

## Edge Cases

- **Pro navigates away with an unsaved private note edit** -- No confirmation dialog is shown; the in-progress text is simply discarded, consistent with this being a low-stakes inline field rather than a multi-field form.
- **Pro taps Save twice rapidly** -- The second tap is ignored while the first save is in progress (button in loading state).
- **Network failure during note save** -- Inline error banner: "Could not save note. Check your connection and try again." with the entered text preserved in the field.
- **Client record was deleted by the Pro from another device or session while this screen is open** -- Per the dependency map's Contention note for the Client entity ("once deleted any concurrent edit is refused with refresh"), a subsequent Save attempt is rejected with the message "This client's record no longer exists. It may have been deleted." and the Pro is returned to the screen they arrived from after acknowledging.
- **Client's phone number is changed via FEAT-13.SPEC-002 while this screen is open in another tab or device** -- Per the dependency map's Contention note for the Client entity ("field edits are last-write-wins"), this screen's identity header reflects the change on next load; an in-progress but unsaved private-note edit on the stale screen is unaffected, since name/phone/email and private_note are independent fields.
- **A concurrent booking is added to this client's history while this screen is open** -- The booking history list is a snapshot as of load; it refreshes on the Pro's next visit to this screen, not live, consistent with this feature's non-live-updating design for a small per-pro client volume.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-12.SPEC-001 (Today's & Upcoming Schedule) | Navigation (inbound) | Pro arrives here by tapping a client's name on a booking row |
| FEAT-24 (Client List Search & Filter) | Navigation (inbound) | Pro arrives here by tapping a client found via search (v1) |
| FEAT-13.SPEC-002 (Client Contact Edit) | Navigation (outbound) | "Edit contact" tap navigates here; returns here on successful save |
| FEAT-13.SPEC-003 (Client Deletion Confirmation) | Navigation (outbound) | "Delete client" tap navigates here |
| FEAT-13.SPEC-005 (Client Field Validation & Access Rules) | References (inbound) | Private note validation and Pro-only visibility rules applied to the note field |
| FEAT-13.SPEC-006 (Deletion Eligibility & Retention Rule) | References (inbound) | Post-deletion concurrent-edit refusal rule applied to any private-note save attempted after the client record is gone |
| FEAT-19 (Platform Support Read-Only Access) | References (inbound) | Support's equivalent read-only view of this Client entity (excluding private_note) is delivered by that feature's own screen, not this one |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| client_record_viewed | entry source (dashboard / search), booking_history_count | Screen opens and the client record loads successfully | N/A -- no success-metrics.md metric names this feature directly; this event is the mechanism behind the "Check a client note" moment in Talia's Between-Clients Day journey, which feeds FEAT-12's "Daily Dashboard Glance Speed" metric rather than a metric of this feature's own |
| client_note_added | note_length, was_previously_empty | Private note save completes successfully | N/A -- same rationale as client_record_viewed: this feature contributes to FEAT-12's glance-speed experience but has no success-metrics.md entry of its own |

## Acceptance Criteria

**FEAT-13.SPEC-001-AC-01:** Given Talia taps a client's name on a booking row on her dashboard, when the Client Record Detail screen opens, then she sees the client's name, phone, email, current private note (or an empty field if none exists), and full booking history with her.

**FEAT-13.SPEC-001-AC-02:** Given Talia is on the Client Record Detail screen with an empty private note, when she types "Prefers a lighter volume" and taps Save, then the note is validated by FEAT-13.SPEC-005, persisted, the client_note_added event is logged, and the caption briefly shows "Saved."

**FEAT-13.SPEC-001-AC-03:** Given Talia types a private note approaching 1,000 characters, when she reaches the limit, then further typing is blocked and the caption reads "Private note can be up to 1,000 characters."

**FEAT-13.SPEC-001-AC-04:** Given Talia taps the overflow menu's "Edit contact" option, when the tap registers, then she is navigated to FEAT-13.SPEC-002 (Client Contact Edit).

**FEAT-13.SPEC-001-AC-05:** Given Talia taps the overflow menu's "Delete client" option, when the tap registers, then she is navigated to FEAT-13.SPEC-003 (Client Deletion Confirmation).

**FEAT-13.SPEC-001-AC-06:** Given Talia is on the Client Record Detail screen for a client with three past bookings and one upcoming booking, when she views the booking history section, then all four bookings appear in reverse-chronological order with date, time, service, and outcome status.

**FEAT-13.SPEC-001-AC-07:** Given Talia's note save fails due to a network error, when the failure occurs, then an inline error banner reads "Could not save note. Check your connection and try again." and her entered text remains in the field.

**FEAT-13.SPEC-001-AC-08:** Given Talia loses connectivity while viewing a client's record, when the connection drops, then the banner "You're offline -- reconnect to view or edit this client's record." appears, the previously loaded data remains visible, and the private note field becomes disabled.

**FEAT-13.SPEC-001-AC-09:** Given a client record with zero bookings is somehow reached, when Talia views the booking history section, then it reads "No bookings with this client yet" instead of a blank area.

**FEAT-13.SPEC-001-AC-10:** Given Talia taps Save on the private note twice in rapid succession, when the second tap registers, then it is ignored while the first save is still in progress.

**FEAT-13.SPEC-001-AC-11:** Given the client's record was deleted from another device while Talia has this screen open, when she attempts to save a private note edit, then the save is rejected with "This client's record no longer exists. It may have been deleted." and she is returned to the screen she arrived from.

**FEAT-13.SPEC-001-AC-12:** Given Riley (the Client) attempts to open this screen's route directly, when the access check runs, then Riley is redirected to the Pro sign-in screen and no client data is exposed.

**FEAT-13.SPEC-001-AC-13:** Given Platform Operator (Support) attempts to open this screen's route directly during a support session, when the access check runs, then Support is redirected to the Pro sign-in screen -- Support's equivalent read-only view is only ever reached through FEAT-19's own screen.

**FEAT-13.SPEC-001-AC-14:** Given Talia's session expires while she has unsaved private-note text entered, when she re-authenticates, then the dialog "Your session has expired. Sign in to continue." appeared beforehand and her unsaved note text is restored after sign-in succeeds.

**FEAT-13.SPEC-001-AC-15:** Given an unauthenticated visitor attempts to open this screen's route directly, when the access check runs, then they are redirected to the Pro sign-in screen (FEAT-29) with no client data exposed.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 6 (loaded, note editing, saving, error, empty history, offline) | 6 |
| Business Rules | 3 | 3 |
| Edge Cases | 6 | 6 |



# Screen Spec: Client Contact Edit

## Overview

**Name:** Client Contact Edit
**ID:** FEAT-13.SPEC-002
**Type:** Screen
**Purpose:** The Pro corrects a client's name, email, or phone number.
**Parent Feature:** FEAT-13 -- Client Record Management

## Scope and Non-Goals

**In Scope:**
- A dedicated form for editing the client's name, phone, and email
- Field-level and cross-field validation feedback, referencing FEAT-13.SPEC-005
- Surfacing the consequence of a phone-number change (access-link invalidation and fresh texting-consent requirement) before the Pro confirms
- Offline/degraded behavior consistent with the Brief's Non-Functional Notes

**Non-Goals:**
- Editing the private note -- handled inline on FEAT-13.SPEC-001 (Client Record Detail), not on this screen
- Re-sending or managing the client's texting consent directly -- consent is re-granted by the Client through FEAT-06 (Client Booking Identity) / FEAT-14 (Messaging Consent Management); this screen only surfaces that a phone-number change requires it, per XBR-15
- Deleting the client's record -- handled by FEAT-13.SPEC-003 (Client Deletion Confirmation); this screen has no delete action
- Editing booking-level data (booking_notes, appointment details) -- booking_notes is client-authored via other features and explicitly read-only from this feature, per the Brief's Shared Entities note

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-13.SPEC-001 (Client Record Detail) | Pro taps "Edit contact" in the overflow menu | Client ID, current name/phone/email values |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Edit and save name, phone, email for their own clients only | -- |
| The Client (Riley) | No | No | Per XBR-29, redirected to the Pro sign-in screen if this screen's route is reached directly; a Client corrects nothing here -- their own email update path is FEAT-06, not this screen |
| Platform Operator (Support) | No -- Support's read-only view of the Client entity, delivered by FEAT-19, never includes an edit path (Support's access to this feature's data is View-only and never includes correcting contact details, per scope-boundaries.md SC-05) | No | If Support's session attempts this screen's route directly, redirected to the Pro sign-in screen per XBR-29 |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29); no client data is exposed before authentication |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue."; entered but unsaved field values are preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Edit contact" with a back arrow (returns to FEAT-13.SPEC-001 without saving) and a "Save" action button (right-aligned).

**Client Identity Header:** The same read-only-style identity-summary display used on FEAT-13.SPEC-001 and FEAT-13.SPEC-003, but here showing the values as they will be edited: name, phone, and email fields sit directly beneath this header context, per the Brief's Shared UI Patterns.

**Body:** A single-column form, pre-filled with the client's current values:
- Name (text input, required)
- Phone (text input, required)
- Email (text input, required only when the client has declined texts, otherwise optional -- see FEAT-13.SPEC-005 for the exact conditional rule)

Below the phone field, a conditional inline notice appears only while the Pro has changed the phone field's value from its original: "Changing this number will require [client name] to re-confirm texting consent for the new number, and any existing access link will stop working." This notice references XBR-18 and XBR-15 in functional terms and disappears if the Pro reverts the field to its original value.

### Responsive Behavior

- **Compact breakpoint:** Single-column form as described, full width; Save remains in the header.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-13.SPEC-001 (Client Record Detail) without saving | Screen closes | If any field has changed, a confirmation dialog appears first (see Edge Cases) |
| Name input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Name input | Blur (empty) | Triggers field validation via FEAT-13.SPEC-005 | Error state on field | "Name is required" below the field |
| Phone input | Type | Captures text input; if the value differs from the original, the phone-change notice appears | Field shows entered text; notice appears/disappears as the value changes | Standard input focus state |
| Phone input | Blur | Triggers field validation via FEAT-13.SPEC-005 (format, and identity-uniqueness within this Pro) | Error state on field if invalid | Exact error message per FEAT-13.SPEC-005 |
| Email input | Blur | Triggers field validation via FEAT-13.SPEC-005 (format, and conditional-required when texts are declined) | Error state on field if invalid | Exact error message per FEAT-13.SPEC-005 |
| Save button | Tap | 1. Validate all fields via FEAT-13.SPEC-005. 2. If valid and the phone number changed, show a confirmation step for the phone-number consequence. 3. Persist name/phone/email. | Button shows a loading state during save | Success: toast "Contact updated" and navigate to FEAT-13.SPEC-001. Failure: inline error messages, entered values preserved |
| Save button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |
| Phone-change confirmation dialog -- "Confirm" | Tap | Proceeds with the save, including phone-number change | Dialog closes, save proceeds | Standard transition into the Saving state |
| Phone-change confirmation dialog -- "Keep original number" | Tap | Reverts the phone field to its original value; save does not proceed for the phone field | Dialog closes, phone field reverts | Phone field shows the original value again; other field changes remain pending |

### Accessibility Notes

- **Focus order:** Back arrow -> Name -> Phone -> Email -> phone-change notice (when present, announced but not focusable) -> Save.
- **Validation announcements:** When a field enters an error state, its error message is announced to assistive technology and programmatically associated with the field. The phone-change notice is announced when it appears.
- **Save feedback:** The "Contact updated" toast is announced on success; on validation failure, focus moves to the first field in error. The phone-change confirmation dialog traps focus until dismissed.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loaded (default) | Form pre-filled with the client's current name, phone, email; Save enabled | Screen opens and the client record loads successfully | User begins editing any field |
| Editing | Form fields contain user input, phone-change notice shown if applicable | User types in any field | User taps Save or the back arrow |
| Validating | Save button shows a loading indicator | User taps Save | Validation completes (pass or fail) |
| Validation Error | Failed fields highlighted with error messages below them | Validation fails (per FEAT-13.SPEC-005) | User corrects the field and re-triggers validation |
| Phone-Change Confirmation | Modal dialog described above is shown, form fields inert behind it | Validation passes and the phone field differs from its original value | Pro taps "Confirm" or "Keep original number" |
| Saving | Save button shows a loading indicator, form fields disabled | Validation passes (and phone-change confirmed, if applicable) | Save completes or fails |
| Error | Error banner at top of form with a retry action | Save operation fails | User taps Retry or navigates away |
| Offline/Degraded | Banner "You're offline -- this contact edit requires a live connection to save." at the top; form remains viewable with the client's last-loaded values but Save is disabled | Connectivity lost while the screen is open, or the screen is opened without connectivity | Connectivity restored -- banner clears and Save re-enables |

## Validation Rules

Validation governed by FEAT-13.SPEC-005 (Client Field Validation & Access Rules). See that spec for all field-level rules (name length, phone format and identity-uniqueness within this Pro, email format and conditional-required rule). This screen applies validation on field blur and on form submit.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap (no changes) | FEAT-13.SPEC-001 (Client Record Detail) | -- |
| Back arrow tap (unsaved changes, confirmed discard) | FEAT-13.SPEC-001 (Client Record Detail) | -- |
| Successful save | FEAT-13.SPEC-001 (Client Record Detail) | -- |

## Data Model

**Creates:** None.
**Reads:** Client record -- name, phone, email (current values, pre-filled into the form).
**Updates:** Client record -- name, phone, email fields, per FEAT-13.SPEC-005's validation rules.
**Deletes:** None.

## Business Rules

- All field validation (name/phone/email) is governed by FEAT-13.SPEC-005 -- this screen enforces but does not restate those rules.
- Post-deletion concurrent-edit refusal (a save against a deleted client is refused with "This client's record no longer exists. It may have been deleted.") is governed by FEAT-13.SPEC-006 (Deletion Eligibility & Retention Rule) -- this screen enforces but does not restate that rule.
- The consent-invalidation effect of a saved phone change is governed by FEAT-14.SPEC-008 (Phone Number Change Consent Invalidation Rule), which reacts to the save made here.
- XBR-18: A changed phone number invalidates the client's existing access links; the Pro is shown this consequence before it takes effect (Phone-Change Confirmation state).
- XBR-15: A changed phone number requires the client to re-grant texting consent for the new number before any further text is sent; this screen surfaces the requirement but the re-grant itself happens through FEAT-06/FEAT-14, outside this screen's scope.
- This screen requires a live connection to save (per the Brief's Non-Functional Notes); it does not queue an offline contact edit for later submission.

## Edge Cases

- **Pro navigates away (back arrow) with unsaved changes** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Pro taps Save twice rapidly** -- The second tap is ignored while the first save is in progress (button in loading state).
- **Network failure during save** -- Error banner: "Could not save contact. Check your connection and try again." with a Retry button. Form data preserved.
- **Pro changes the phone number to one already used by another of their own clients** -- Rejected per FEAT-13.SPEC-005's phone-identity-uniqueness rule with the exact error message defined there; the save does not proceed.
- **Client record changed by a concurrent action (e.g., the Client updated their own email via FEAT-06) between this screen's load and save** -- Per the dependency map's Contention note for the Client entity ("field edits are last-write-wins"), the Pro's save of name/phone proceeds and overwrites only the fields the Pro edited; a concurrently changed email field the Pro did not touch on this screen is not overwritten, since the form only submits the fields it displays.
- **Client record was deleted from another device or session while this screen is open** -- Per the dependency map's Contention note ("once deleted any concurrent edit is refused with refresh"), Save is rejected with "This client's record no longer exists. It may have been deleted." and the Pro is returned to the screen they arrived from after acknowledging.
- **Pro reverts the phone field to its original value after the change notice appeared** -- The notice disappears; no phone-change confirmation step occurs on save.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-13.SPEC-001 (Client Record Detail) | Navigation (inbound/outbound) | "Edit contact" tap arrives here; back arrow and successful save both return here |
| FEAT-13.SPEC-005 (Client Field Validation & Access Rules) | References (inbound) | All field-level and cross-field validation rules applied to this form |
| FEAT-13.SPEC-006 (Deletion Eligibility & Retention Rule) | References (inbound) | Post-deletion concurrent-edit refusal rule applied to any contact save attempted after the client record is gone |
| FEAT-14.SPEC-008 (Phone Number Change Consent Invalidation Rule) | References (outbound) | Saving a changed phone number is the event that rule reacts to; this screen enforces the phone-change confirmation step and the rule leaves the existing consent record's phone_number stale, invalidating it (XBR-15) |
| FEAT-06 (Client Booking Identity) | References (outbound) | A changed phone number invalidates access links owned by this feature (XBR-18) |
| FEAT-14 (Messaging Consent Management) | References (outbound) | A changed phone number requires fresh texting consent, re-granted through this feature (XBR-15) |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| client_contact_updated | fields_changed (name / phone / email, one or more), phone_changed (boolean) | Contact save completes successfully | N/A -- no success-metrics.md metric names this feature directly; contact correction is a data-integrity action rather than one of the feature's own analytics-linked moments (per the Brief's Analytics linkage note, which names only client_record_viewed, client_note_added, and client_record_deleted) |

## Acceptance Criteria

**FEAT-13.SPEC-002-AC-01:** Given Talia is on the Client Record Detail screen and taps "Edit contact," when the Client Contact Edit screen opens, then it is pre-filled with the client's current name, phone, and email.

**FEAT-13.SPEC-002-AC-02:** Given Talia clears the name field and moves focus away, when the blur validation runs, then the field shows the error "Name is required" per FEAT-13.SPEC-005.

**FEAT-13.SPEC-002-AC-03:** Given Talia changes the phone field's value, when the value differs from the original, then the notice about re-confirming texting consent and access-link invalidation appears beneath the field.

**FEAT-13.SPEC-002-AC-04:** Given Talia has changed the phone number and taps Save with all fields valid, when validation passes, then the Phone-Change Confirmation dialog appears before the save is committed.

**FEAT-13.SPEC-002-AC-05:** Given Talia sees the Phone-Change Confirmation dialog, when she taps "Confirm," then the save proceeds with the new phone number and she sees the toast "Contact updated," returning to FEAT-13.SPEC-001.

**FEAT-13.SPEC-002-AC-06:** Given Talia sees the Phone-Change Confirmation dialog, when she taps "Keep original number," then the phone field reverts to its original value and no phone change is saved.

**FEAT-13.SPEC-002-AC-07:** Given Talia enters a phone number already used by another of her own clients, when she taps Save, then the save is rejected with the phone-identity-uniqueness error defined in FEAT-13.SPEC-005.

**FEAT-13.SPEC-002-AC-08:** Given Talia has unsaved changes on this screen, when she taps the back arrow, then a confirmation dialog appears: "You have unsaved changes. Discard?"

**FEAT-13.SPEC-002-AC-09:** Given Talia's save fails due to a network error, when the failure occurs, then the error banner "Could not save contact. Check your connection and try again." appears with her entered values preserved.

**FEAT-13.SPEC-002-AC-10:** Given the client's record was deleted from another session while Talia has this screen open, when she taps Save, then the save is rejected with "This client's record no longer exists. It may have been deleted." and she is returned to the screen she arrived from.

**FEAT-13.SPEC-002-AC-11:** Given Talia loses connectivity while on this screen, when the connection drops, then the banner "You're offline -- this contact edit requires a live connection to save." appears and Save becomes disabled.

**FEAT-13.SPEC-002-AC-12:** Given Riley (the Client) attempts to open this screen's route directly, when the access check runs, then Riley is redirected to the Pro sign-in screen.

**FEAT-13.SPEC-002-AC-13:** Given Platform Operator (Support) attempts to open this screen's route directly, when the access check runs, then Support is redirected to the Pro sign-in screen -- Support's own access never includes a contact-edit path, per scope-boundaries.md SC-05.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 10 | 10 |
| States | 8 (loaded, editing, validating, validation error, phone-change confirmation, saving, error, offline) | 8 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |



# Screen Spec: Client Deletion Confirmation

## Overview

**Name:** Client Deletion Confirmation
**ID:** FEAT-13.SPEC-003
**Type:** Screen
**Purpose:** The Pro requests permanent deletion of a client's record, sees the eligibility check result, and confirms an irreversible delete.
**Parent Feature:** FEAT-13 -- Client Record Management

## Scope and Non-Goals

**In Scope:**
- Running and displaying the deletion eligibility check (FEAT-13.SPEC-006) when the screen opens
- Showing the blocking outcome (an upcoming booking exists) with the current booking and a route to cancel it with full refund
- Showing the eligible outcome with a plain explanation of what deletion does, and requiring explicit confirmation before executing it
- Triggering the deletion automation (FEAT-13.SPEC-004) on confirmation

**Non-Goals:**
- Performing the deletion itself -- executed by FEAT-13.SPEC-004 (Client Deletion Execution); this screen only confirms intent and hands off
- Cancelling the blocking upcoming booking -- handled by FEAT-30 (Pro Booking Management)'s cancel-with-full-refund flow, reached from this screen but owned there
- Restoring or undoing a completed deletion -- excluded per the Brief's Non-Goals: deletion "is not reversible, consistent with 'delete on request' meaning delete"; no undo path exists anywhere in this feature
- Editing the client's contact details or note before deleting -- handled by FEAT-13.SPEC-001 and FEAT-13.SPEC-002; this screen is deletion-only

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-13.SPEC-001 (Client Record Detail) | Pro taps "Delete client" in the overflow menu | Client ID |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Confirm deletion for their own clients only, gated by eligibility (FEAT-13.SPEC-006) | -- |
| The Client (Riley) | No | No | Per XBR-29, redirected to the Pro sign-in screen if this screen's route is reached directly; a Client cannot request or execute their own deletion in-product, per scope-boundaries.md SC-01's client-self-service exclusion -- they request it informally and the Pro performs it here |
| Platform Operator (Support) | No | No | Support cannot perform a deletion on the Pro's behalf, per scope-boundaries.md SC-05; if Support's session attempts this screen's route directly, redirected to the Pro sign-in screen per XBR-29 |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29); no client data is exposed before authentication |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue."; the deletion has not started and no state is preserved -- the Pro re-initiates from FEAT-13.SPEC-001 after signing back in |

## Layout and Content

**Header:** Screen title "Delete client" with a back arrow (returns to FEAT-13.SPEC-001 without deleting).

**Client Identity Header:** The same identity-summary display used on FEAT-13.SPEC-001 and FEAT-13.SPEC-002 (name, phone, email), per the Brief's Shared UI Patterns, so the Pro can confirm they are deleting the intended client.

**Eligibility Result Area:** Below the identity header, one of two mutually exclusive panels, determined by FEAT-13.SPEC-006's eligibility check, which runs automatically when the screen opens:

- **Blocked panel:** A plain message: "[Client name] has an upcoming booking on [date, time] for [service]. You'll need to cancel it before deleting this client." Below the message, a single button "Cancel booking with full refund," which routes to FEAT-30's cancel-with-full-refund flow for that booking.
- **Eligible panel:** A plain explanation: "Deleting [client name] permanently removes their contact details and your private note. This cannot be undone. Their booking and payment history will be kept, but no longer linked to a name." Below the explanation, a checkbox "I understand this cannot be undone" and a "Delete client" button, inactive until the checkbox is checked.

### Responsive Behavior

- **Compact breakpoint:** Single-column stack (header, identity header, eligibility panel), full width.
- **Medium size class and above:** Content remains single-column, capped at a consistent platform-wide content width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-13.SPEC-001 (Client Record Detail) without deleting | Screen closes | Standard navigation transition |
| "Cancel booking with full refund" (Blocked panel only) | Tap | Navigate to FEAT-30 (Pro Booking Management)'s cancel-with-full-refund flow for the blocking booking | Screen transitions | Standard navigation transition |
| "I understand this cannot be undone" checkbox (Eligible panel only) | Tap | Toggles the checkbox; activates the "Delete client" button when checked | Checkbox shows checked/unchecked state; button enables/disables accordingly | Standard checkbox feedback |
| "Delete client" button (Eligible panel only) | Tap | Triggers FEAT-13.SPEC-004 (Client Deletion Execution) | Button shows a loading state; screen becomes non-interactive during execution | On completion, navigates the Pro back to the screen they arrived at before opening the client record (FEAT-12 or FEAT-24), with a toast "Client deleted" |
| "Delete client" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> client identity header (read-only) -> eligibility panel content -> (Blocked panel) "Cancel booking with full refund" button, or (Eligible panel) checkbox -> "Delete client" button.
- **Eligibility announcement:** The eligibility result (blocked or eligible) is announced to assistive technology once the check completes.
- **Confirmation announcement:** The "Client deleted" toast is announced on completion.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Checking Eligibility | A brief in-place loading indicator in place of the eligibility panel | Screen opens | The eligibility check (FEAT-13.SPEC-006) returns a result |
| Blocked | Blocked panel shown, as described in Layout and Content | Eligibility check finds an upcoming booking | Pro navigates away, or later returns after that booking is cancelled or completed, re-triggering the eligibility check |
| Eligible | Eligible panel shown, checkbox unchecked, "Delete client" button inactive | Eligibility check finds no upcoming booking | Pro checks the confirmation checkbox, taps "Delete client," or navigates away |
| Ready to Confirm | Eligible panel shown, checkbox checked, "Delete client" button active | Pro checks the confirmation checkbox | Pro taps "Delete client," or unchecks the checkbox, or navigates away |
| Deleting | Screen is non-interactive; "Delete client" button shows a loading indicator | Pro taps "Delete client" | Deletion completes (success) or fails |
| Deletion Error | Error banner: "Could not delete this client. Check your connection and try again." with a Retry action; eligible panel remains, checkbox state preserved | FEAT-13.SPEC-004 reports a failure | Pro taps Retry, or navigates away |
| Offline/Degraded | Banner "You're offline -- deleting a client requires a live connection." at the top; the eligibility panel (if already loaded) remains visible but the "Delete client" and "Cancel booking" actions are disabled | Connectivity lost while the screen is open, or the screen is opened without connectivity | Connectivity restored -- banner clears and actions re-enable |

## Validation Rules

Validation governed by FEAT-13.SPEC-006 (Deletion Eligibility & Retention Rule). See that spec for the exact eligibility condition (no upcoming booking) and the confirmation requirement before deletion executes.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-13.SPEC-001 (Client Record Detail) | -- |
| "Cancel booking with full refund" tap | FEAT-30's cancel-with-full-refund flow | FEAT-30 (Pro Booking Management) |
| Successful deletion | The screen the Pro arrived at before opening the client record | FEAT-12 or FEAT-24 (FEAT-24.SPEC-001, Client Search & Filter) |

## Data Model

**Creates:** None.
**Reads:** Client record -- name, phone, email (for the identity header); Booking (read-only) -- checked by FEAT-13.SPEC-006 for an upcoming booking with this client.
**Updates:** None -- this screen only gathers confirmation; the actual write is performed by FEAT-13.SPEC-004.
**Deletes:** None directly -- deletion is executed by FEAT-13.SPEC-004 once this screen collects confirmation.

## Business Rules

- Deletion eligibility (the upcoming-booking block) is governed entirely by FEAT-13.SPEC-006 -- this screen displays but does not re-derive that rule.
- Delete-access (Pro only, own clients only; never Support or Client) is governed by FEAT-13.SPEC-005 (Client Field Validation & Access Rules) and is enforced on screen entry -- this screen enforces but does not restate that rule.
- Deletion is irreversible; the "I understand this cannot be undone" checkbox must be explicitly checked before the "Delete client" button activates, per the Brief's Validation & Limits ("deletion is a deliberate, confirmed action").
- XBR-19: deletion removes contact details and notes, cascades to Messaging Consent, and retains de-identified financial and timeline history -- this screen's Eligible-panel explanation states this plainly before the Pro confirms.

## Edge Cases

- **The blocking booking is cancelled or completed while this screen is open (e.g., from another device)** -- The eligibility result shown is a snapshot from when the screen loaded; the Pro must navigate away and back (or the screen re-checks on the "Cancel booking" flow's return) to see the updated Eligible panel. No live-updating occurs on this screen.
- **A new booking is created for this client (e.g., a return client books again) between the eligibility check and the Pro tapping "Delete client"** -- Per the dependency map's Contention note ("deletion is refused while an upcoming booking exists"), the deletion attempt in FEAT-13.SPEC-004 re-validates eligibility and refuses with refresh if a booking now exists; this screen surfaces that refusal (see FEAT-13.SPEC-006's edge cases for the exact behavior).
- **Pro taps "Delete client" twice rapidly** -- The second tap is ignored while the first deletion is in progress (button in loading state, screen non-interactive).
- **Network failure during deletion** -- Error banner: "Could not delete this client. Check your connection and try again." with a Retry button; no partial deletion occurs (FEAT-13.SPEC-004 is all-or-nothing).
- **Pro navigates away from the Blocked panel without cancelling the booking** -- No confirmation dialog is needed; nothing was changed, so the Pro simply returns to FEAT-13.SPEC-001 with the client record intact.
- **The client record was already deleted from another session before this screen's eligibility check completes** -- The check fails with "This client's record no longer exists. It may have been deleted." and the Pro is returned to the screen they arrived from.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-13.SPEC-001 (Client Record Detail) | Navigation (inbound) | "Delete client" tap arrives here; back arrow returns here |
| FEAT-13.SPEC-006 (Deletion Eligibility & Retention Rule) | References (inbound) | Eligibility check and irreversibility rule applied on screen load and on confirmation |
| FEAT-13.SPEC-005 (Client Field Validation & Access Rules) | References (inbound) | Delete-access authorization (Pro only, own clients only) enforced on screen entry; the Blocked panel, not a bare denial, is shown when ineligible |
| FEAT-13.SPEC-004 (Client Deletion Execution) | Triggers (outbound) | "Delete client" confirmation triggers the deletion automation |
| FEAT-30 (Pro Booking Management) | Navigation (outbound) | "Cancel booking with full refund" routes to that feature's cancellation flow |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| client_deletion_blocked | -- | Eligibility check finds an upcoming booking and the Blocked panel is shown | N/A -- no success-metrics.md metric names this feature directly; this event supports understanding how often the deletion flow is interrupted, but the Brief's Analytics linkage names only client_record_viewed, client_note_added, and client_record_deleted as this feature's contribution |
| client_deletion_confirmed | -- | Pro taps "Delete client" with the confirmation checkbox checked | N/A -- see client_deletion_blocked; the resulting deletion itself is measured by FEAT-13.SPEC-004's client_record_deleted event, which this screen triggers but does not itself emit |

## Acceptance Criteria

**FEAT-13.SPEC-003-AC-01:** Given Talia taps "Delete client" from a client record with no upcoming booking, when the Client Deletion Confirmation screen opens, then the eligibility check runs and the Eligible panel appears with the irreversibility explanation.

**FEAT-13.SPEC-003-AC-02:** Given Talia taps "Delete client" from a client record with an upcoming booking, when the eligibility check runs, then the Blocked panel appears showing that booking's date, time, and service, with a "Cancel booking with full refund" button.

**FEAT-13.SPEC-003-AC-03:** Given Talia is on the Blocked panel, when she taps "Cancel booking with full refund," then she is navigated to FEAT-30's cancel-with-full-refund flow for that booking.

**FEAT-13.SPEC-003-AC-04:** Given Talia is on the Eligible panel with the confirmation checkbox unchecked, when she looks at the "Delete client" button, then it is inactive and cannot be tapped.

**FEAT-13.SPEC-003-AC-05:** Given Talia checks "I understand this cannot be undone," when the checkbox is checked, then the "Delete client" button becomes active.

**FEAT-13.SPEC-003-AC-06:** Given Talia has checked the confirmation checkbox and taps "Delete client," when the tap registers, then FEAT-13.SPEC-004 (Client Deletion Execution) is triggered and the screen enters the Deleting state.

**FEAT-13.SPEC-003-AC-07:** Given the deletion completes successfully, when Talia sees the result, then she is returned to the screen she arrived at before opening the client record with the toast "Client deleted."

**FEAT-13.SPEC-003-AC-08:** Given Talia taps "Delete client" twice in rapid succession, when the second tap registers, then it is ignored while the first deletion is in progress.

**FEAT-13.SPEC-003-AC-09:** Given the deletion fails due to a network error, when the failure occurs, then the error banner "Could not delete this client. Check your connection and try again." appears with a Retry option.

**FEAT-13.SPEC-003-AC-10:** Given a new booking is created for this client between the eligibility check and Talia's confirmation, when she taps "Delete client," then the deletion is refused and this screen surfaces the refresh behavior defined in FEAT-13.SPEC-006.

**FEAT-13.SPEC-003-AC-11:** Given Talia loses connectivity on this screen, when the connection drops, then the banner "You're offline -- deleting a client requires a live connection." appears and both action buttons become disabled.

**FEAT-13.SPEC-003-AC-12:** Given Riley (the Client) attempts to open this screen's route directly, when the access check runs, then Riley is redirected to the Pro sign-in screen, consistent with scope-boundaries.md SC-01 excluding client self-service deletion.

**FEAT-13.SPEC-003-AC-13:** Given Platform Operator (Support) attempts to open this screen's route directly, when the access check runs, then Support is redirected to the Pro sign-in screen -- Support cannot perform a deletion on the Pro's behalf, per scope-boundaries.md SC-05.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 7 (checking eligibility, blocked, eligible, ready to confirm, deleting, error, offline) | 7 |
| Business Rules | 3 | 3 |
| Edge Cases | 6 | 6 |



# Automation Spec: Client Deletion Execution

## Overview

**Name:** Client Deletion Execution
**ID:** FEAT-13.SPEC-004
**Type:** Automation
**Purpose:** Hard-deletes the client's contact details and private note, cascades to remove Messaging Consent, and retains de-identified financial and timeline history.
**Parent Feature:** FEAT-13 -- Client Record Management

## Scope and Non-Goals

**In Scope:**
- Re-validating deletion eligibility at the moment of execution (not just at screen load)
- Permanently removing the Client record's name, phone, email, and private_note
- Cascade-deleting the client's Messaging Consent record(s)
- De-identifying the client's Booking, Deposit Transaction, and Activity Event history rather than deleting it
- Logging the client_record_deleted event

**Non-Goals:**
- Determining eligibility (the upcoming-booking block) -- governed by FEAT-13.SPEC-006 (Deletion Eligibility & Retention Rule); this automation re-checks but does not define that rule
- Collecting the Pro's confirmation -- handled by FEAT-13.SPEC-003 (Client Deletion Confirmation); this automation only executes once confirmation has been given
- Cancelling any booking -- if an upcoming booking exists, this automation refuses rather than cancelling on the Pro's behalf; cancellation is a separate, explicit action through FEAT-30 (Pro Booking Management)
- Deleting Booking, Deposit Transaction, or Activity Event records outright -- excluded per scope-boundaries.md SC-22: full history is retained for as long as the account exists, with only de-identification (not deletion) applied to a deleted client's records

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Pro confirms deletion | FEAT-13.SPEC-003 (Client Deletion Confirmation) | Fires when the Pro taps "Delete client" with the confirmation checkbox checked | Client ID; the eligibility result the confirmation screen last displayed |

## Processing Logic

1. Receive the client ID from the triggering confirmation screen, and re-check delete-access per FEAT-13.SPEC-005 (the requester is the Pro who owns this client); if not, refuse and change nothing.
2. Re-run the deletion eligibility check (FEAT-13.SPEC-006): read the client's bookings and determine whether any upcoming booking exists.
3. If an upcoming booking now exists (created or confirmed after the confirmation screen's own check), stop processing and return the Blocked outcome -- no data is changed.
4. If no upcoming booking exists, proceed:
   a. Permanently remove the Client record's name, phone, email, and private_note fields.
   b. Cascade-delete the client's Messaging Consent record(s) for every channel, except where evidence of a previously granted or revoked consent must be retained under applicable law (per FEAT-13.SPEC-006's retention rule) -- in that case, retain only the minimum evidence required, stripped of the phone number and any other identifying field not itself the required evidence.
   c. Mark the client's Booking, Deposit Transaction, and Activity Event records as de-identified: replace the client reference with a de-identified marker, leaving service, timing, amount, and outcome fields intact for dispute/audit purposes (per scope-boundaries.md SC-22 and XBR-19).
   d. Log the client_record_deleted event.
5. Signal the triggering screen (FEAT-13.SPEC-003) that deletion completed successfully.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Deletion succeeded | No upcoming booking at execution time | Client contact details and private_note permanently removed; Messaging Consent cascade-deleted (except retained legal-evidence remnants); Booking, Deposit Transaction, and Activity Event records de-identified but retained | Toast "Client deleted"; Pro returned to the screen they arrived at before opening the client record | FEAT-13.SPEC-003 (triggering screen), FEAT-12 / FEAT-24 (destination screens, which no longer list this client) |
| Blocked at execution (booking appeared since the confirmation screen's check) | An upcoming booking now exists that did not exist (or was not confirmed) when FEAT-13.SPEC-003 last checked | None -- no data changed | FEAT-13.SPEC-003 shows the Blocked panel with the newly discovered booking's date, time, and service, per its own refresh behavior | FEAT-13.SPEC-003 |
| Deletion failed (processing error) | The write fails partway or the automation cannot complete | None -- the operation is all-or-nothing; no partial deletion is left behind | FEAT-13.SPEC-003 shows "Could not delete this client. Check your connection and try again." with a Retry option | FEAT-13.SPEC-003 |

## Data Model

**Reads:** Client record (for the ID and current field values); Booking (to re-check eligibility and to identify records needing de-identification); Messaging Consent (to identify records to cascade-delete or retain as evidence).
**Creates:** None.
**Updates:** Booking, Deposit Transaction, and Activity Event records -- client reference replaced with a de-identified marker; all other fields (service, timing, amount, outcome) unchanged.
**Deletes:** Client record -- name, phone, email, private_note permanently removed (the record itself is retired, per the dependency map's Client entity Lifecycle: "Deleted by FEAT-13"). Messaging Consent -- cascade-deleted, except any minimum evidence retained where law requires (per FEAT-13.SPEC-006).

## Business Rules

- XBR-19: deletion is refused while an upcoming booking exists; removes contact details, notes, and consent; retains financial and timeline records only in de-identified form; a later booking creates a new, unconnected Client record.
- Delete-access is re-enforced at execution time per FEAT-13.SPEC-005 (Client Field Validation & Access Rules): only the Pro who owns the client may execute; a request from any other role or Pro is refused and changes nothing.
- This automation is all-or-nothing: it either completes every step in Processing Logic or changes nothing, so a client record is never left in a partially deleted state.
- Deletion is irreversible; no restore or undo path exists for the Client entity itself, per the Brief's Non-Goals.
- A later booking from the same person (matched or not to the deleted client's former identity) creates a new, unconnected Client record -- this automation never resurrects the deleted record, per the dependency map's Contention note for the Client entity.

## Edge Cases

- **The client has no Messaging Consent record at all (e.g., they always declined texts)** -- Step 4b has nothing to cascade-delete; processing continues normally to step 4c.
- **The client has zero Booking history (a data inconsistency scenario)** -- Step 4c has nothing to de-identify; processing continues normally, and client_record_deleted is still logged.
- **A card-issuer dispute is open on one of this client's past Deposit Transactions at the moment of deletion** -- The Deposit Transaction is de-identified like any other retained record; its Disputed status and evidence remain intact and usable for the dispute process, per XBR-22, with only the client reference removed.
- **Concurrent trigger firing (the Pro somehow confirms deletion for the same client from two devices at effectively the same time)** -- The first confirmed run to reach step 4 completes the deletion; the second run's re-validation at step 2 finds the client record already gone and returns a failure equivalent to "This client's record no longer exists," surfaced by FEAT-13.SPEC-003 as its own concurrent-edit refusal.
- **A trigger fires while a previous run for the same client is still in flight** -- A second confirmation attempt for the same client cannot start while the first is executing: FEAT-13.SPEC-003's "Delete client" button is disabled and the screen non-interactive during the Deleting state. Runs for different clients proceed independently.
- **A concurrent edit (e.g., a private-note save from FEAT-13.SPEC-001 on another device) is attempted on this client after deletion completes** -- Per the dependency map's Contention note ("once deleted any concurrent edit is refused with refresh"), that edit is refused with a message prompting the Pro to refresh, as specified in FEAT-13.SPEC-001's and FEAT-13.SPEC-002's own edge cases.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-13.SPEC-003 (Client Deletion Confirmation) | Triggered by (inbound) | Fires when the Pro confirms deletion |
| FEAT-13.SPEC-003 (Client Deletion Confirmation) | Affects (outbound) | Returns the success, blocked, or failure outcome to the confirmation screen |
| FEAT-13.SPEC-005 (Client Field Validation & Access Rules) | References (inbound) | Delete-access authorization (Pro only, own clients only) re-enforced at execution time, consistent with FEAT-13.SPEC-003 |
| FEAT-13.SPEC-006 (Deletion Eligibility & Retention Rule) | References (inbound) | Eligibility re-check and retention/de-identification rules applied during execution |
| FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) | Affects (outbound) | De-identified Deposit Transaction and Activity Event records remain available there for dispute/audit purposes, per XBR-19 |
| FEAT-14 (Messaging Consent Management) | Affects (outbound) | Messaging Consent records are cascade-deleted as part of this automation |
| FEAT-06 (Client Booking Identity) | Affects (outbound) | The client's Access Links, tied to the deleted record, cease to resolve to any client once the record is gone |

## Analytics and Success Signals

- **client_record_deleted** (had_retained_consent_evidence: boolean, booking_history_count) -- N/A -- no success-metrics.md metric names this feature directly; this event is the completion signal for the Brief's compliance-adjacent deletion-on-request obligation rather than a metric-connected signal, and it is the only event this automation emits worth measuring
- **client_deletion_failed** (reason: blocked_by_new_booking / processing_error) -- N/A -- same rationale as client_record_deleted; this event measures how often the all-or-nothing guarantee is exercised rather than feeding a named success metric

## Acceptance Criteria

**FEAT-13.SPEC-004-AC-01:** Given Talia has confirmed deletion for a client with no upcoming booking, when the automation runs, then the client's name, phone, email, and private note are permanently removed and the client_record_deleted event is logged.

**FEAT-13.SPEC-004-AC-02:** Given the deleted client had an active Messaging Consent record, when the automation runs, then that consent record is cascade-deleted along with the client's contact details.

**FEAT-13.SPEC-004-AC-03:** Given the deleted client had three past bookings and one deposit transaction, when the automation runs, then those Booking and Deposit Transaction records are de-identified (client reference removed) but retained with their service, timing, amount, and outcome fields intact.

**FEAT-13.SPEC-004-AC-04:** Given a new upcoming booking is created for this client between FEAT-13.SPEC-003's last check and this automation's execution, when the automation re-validates eligibility, then it stops without changing any data and returns the Blocked outcome to the confirmation screen.

**FEAT-13.SPEC-004-AC-05:** Given the automation's write fails partway through processing, when the failure occurs, then no partial deletion is left behind and FEAT-13.SPEC-003 shows "Could not delete this client. Check your connection and try again."

**FEAT-13.SPEC-004-AC-06:** Given a card-issuer dispute is open on one of the client's past deposit transactions, when the automation de-identifies that record, then the Disputed status and its evidence remain intact with only the client reference removed.

**FEAT-13.SPEC-004-AC-07:** Given Talia confirms deletion for the same client from two devices at effectively the same time, when both runs execute, then the first to complete succeeds and the second finds the record already gone, returning a "client no longer exists" failure.

**FEAT-13.SPEC-004-AC-08:** Given a deletion is already in flight for a client, when a second confirmation attempt is made for the same client before the first completes, then it cannot start -- FEAT-13.SPEC-003's Delete action is disabled during the in-flight run.

**FEAT-13.SPEC-004-AC-09:** Given a client with no Messaging Consent record on file is deleted, when the automation runs, then step 4b has nothing to cascade-delete and processing completes normally through to client_record_deleted.

**FEAT-13.SPEC-004-AC-10:** Given a client with zero booking history (a data inconsistency scenario) is deleted, when the automation runs, then step 4c has nothing to de-identify and client_record_deleted is still logged.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 (success, blocked, failure) | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Client Field Validation & Access Rules

## Overview

**Name:** Client Field Validation & Access Rules
**ID:** FEAT-13.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs name/phone/email/note field validation, phone-number identity uniqueness within one Pro, and the private-note field's Pro-only visibility.
**Parent Feature:** FEAT-13 -- Client Record Management
**Governed Entity:** Client record

## Scope and Non-Goals

**In Scope:**
- Per-field validation rules for name, phone, email, and private_note, as edited through this feature
- The cross-field conditional requiring email when the client has declined texts
- Phone-number identity-uniqueness within one Pro
- The private_note field's Pro-only visibility (excluded from Support's view)
- Authorization rules for viewing, editing, and deleting the Client record, for every role this feature's screens or Support's reading of this entity touch

**Non-Goals:**
- Validation of booking_notes (the client's own optional per-booking note) -- this field is client-authored via other features (the booking flow) and is read-only from this feature, per the Brief's Shared Entities note; its own validation lives with the feature that captures it (FEAT-05)
- The Client's own update of their email and texting consent through FEAT-06 -- that action is owned by FEAT-06 (Client Booking Identity) under the Access Matrix's "Booking & Payment" group, not by this feature's Authorization Rules
- Deletion eligibility (the upcoming-booking block) and retention/de-identification behavior -- handled entirely by FEAT-13.SPEC-006 (Deletion Eligibility & Retention Rule); this spec governs field validation and access, not the deletion condition itself
- Validation of derived fields (booking_history) -- excluded because this is a computed, read-only list with no user-entered values to validate

## Governed Entity

**Entity:** Client
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| name | text | Client's full name |
| phone | text | Client's phone number; required, identity key within one Pro |
| email | text | Client's email address; required when texts are declined, otherwise optional |
| private_note | text | The Pro's own private note about this client; Pro-only visibility |
| booking_notes | text | The client's own optional per-booking note, captured elsewhere; read-only from this feature |
| booking_history | derived | This client's Bookings with this Pro, derived from the Booking entity; read-only from this feature |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-13.SPEC-001 | Client Record Detail | Private note validation on blur (character limit) and on save; view-access enforced on screen entry |
| FEAT-13.SPEC-002 | Client Contact Edit | Name/phone/email validation on field blur and on form submit; edit-access enforced on screen entry and on save |
| FEAT-13.SPEC-003 | Client Deletion Confirmation | Delete-access enforced on screen entry; the deletion action itself is gated by FEAT-13.SPEC-006 |
| FEAT-13.SPEC-004 | Client Deletion Execution | Delete-access re-enforced at execution time, consistent with FEAT-13.SPEC-003 |
| FEAT-19.SPEC-004 | Support Session Scope & Access Rules | View-access (excluding private_note) enforced when Support reads this entity; that spec cites this rule as the owner of the client private-note exclusion at its hand-off points |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| name | Required, non-empty, 1--100 characters | Always | On blur and on submit | "Name is required" / "Name must be 100 characters or fewer" | Yes |
| phone | Required, non-empty, valid reachable phone format | Always | On blur and on submit | "Phone number is required" / "Please enter a valid phone number" | Yes |
| phone | Must not match another of this Pro's clients (identity-uniqueness within one Pro), as enforced within this feature's edit context | Always, when editing an existing client's phone through FEAT-13.SPEC-002 | On submit | "This phone number is already used by another client. Check for a duplicate before saving." | Yes |
| email | Valid email format | When provided (not empty) | On blur | "Please enter a valid email address" | Yes |
| email | Required | When the client has declined texts (no active texting opt-in for this Pro) | On submit | "An email address is required for clients who haven't opted in to texts" | Yes |
| private_note | Up to 1,000 characters | Always | On blur (blocks further typing at the limit) and on submit | "Private note can be up to 1,000 characters." | Yes |
| booking_notes | No validation beyond data type -- read-only from this feature | Always | -- | -- | -- |
| booking_history | No validation beyond data type -- derived, read-only from this feature | Always | -- | -- | -- |

**Note -- creation-time phone matches are out of scope for this rule:** the phone-uniqueness rule above governs only edits made through this feature (FEAT-13.SPEC-002). It is a rejection, not a merge, and applies solely when the Pro changes an existing client's phone number to one already on another of her client records. Creation of a new Client record (a first booking via FEAT-05, or a Pro booking a client in via FEAT-30) is a Non-Goal of this feature (see Scope and Non-Goals) and is not governed here. Per the Feature Dependency Map's Contention resolution for the Client entity, a phone-number match found at creation time resolves to a single Client record (merge, never a duplicate, never a rejection) -- that resolution is owned by the creating features, FEAT-05 and FEAT-30, not by this spec.

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Email conditionally required | email, texting opt-in state (on the associated Messaging Consent record) | Email must be non-empty when the client has no active texting consent for this Pro; email may be empty when active texting consent exists | "An email address is required for clients who haven't opted in to texts" |
| Phone-change consequence acknowledgment | phone | When phone is changed from its stored value, the change may only be saved after the Pro acknowledges (via FEAT-13.SPEC-002's confirmation step) that access links will be invalidated and fresh texting consent will be required (XBR-18, XBR-15) | N/A -- this is a confirmation gate, not a rejection; no error message, the save is simply held until acknowledged |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View client record (all fields, including private_note) | The Pro (Talia) | Own clients only | -- |
| View client record (all fields except private_note) | Platform Operator (Support) | Always, but only through FEAT-19's own read-only screen -- never through this feature's own screens | Support attempting this feature's own screens is redirected to the Pro sign-in screen (per XBR-29); the private_note field is never rendered anywhere Support can reach, even within FEAT-19 |
| View client record | The Client (Riley) | Never, through this feature | This feature's screens are never reached by a Client; a Client's own contact details are shown only through FEAT-06 (Client Booking Identity), which reads the entity independently of this feature's Authorization Rules |
| Edit private_note | The Pro (Talia) | Own clients only | -- |
| Edit private_note | Platform Operator (Support) | Never | The field is never shown to Support, and no edit control exists for it outside this feature |
| Edit private_note | The Client (Riley) | Never | The field does not appear anywhere a Client can reach |
| Edit contact fields (name, phone, email) | The Pro (Talia) | Own clients only | -- |
| Edit contact fields (name, phone, email) | Platform Operator (Support) | Never | Support's access is View-only and never includes correcting contact details, per scope-boundaries.md SC-05; no edit control is rendered |
| Edit contact fields (name, phone, email) via this feature | The Client (Riley) | Never | A Client's own email update runs through FEAT-06 (Client Booking Identity), not this feature; this feature exposes no client-facing edit path at all, per scope-boundaries.md SC-04 |
| Delete client record | The Pro (Talia) | Own clients only, and only when eligible per FEAT-13.SPEC-006 (no upcoming booking) | When ineligible: the Blocked panel in FEAT-13.SPEC-003, not a bare denial -- see that spec |
| Delete client record | Platform Operator (Support) | Never | Support cannot perform a deletion on the Pro's behalf, per scope-boundaries.md SC-05; no delete control exists in Support's read-only view |
| Delete client record | The Client (Riley) | Never | A Client cannot request or execute their own deletion in-product, per scope-boundaries.md SC-01; deletion is performed only by the Pro after an informal request |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| name, phone, email | Set from the values captured at the client's first booking (FEAT-05) or Pro-entered booking-in (FEAT-30) | On create only (outside this feature's scope) | Yes -- the Pro may correct any of these three fields at any time through FEAT-13.SPEC-002 |
| private_note | Empty | On create only (outside this feature's scope) | Yes -- the Pro sets or edits it at any time through FEAT-13.SPEC-001 |
| booking_history | Derived from the Booking entity: every Booking record referencing this client with this Pro | Always, recalculated on each load of FEAT-13.SPEC-001 | No -- always derived, never directly editable |

## Business Rules

- XBR-18: a changed phone number invalidates the client's existing access links; enforced at the point of save in FEAT-13.SPEC-002, not by this spec directly, but this spec's phone-change cross-field rule is what surfaces the acknowledgment gate that makes the save possible.
- XBR-15: a changed phone number requires the client to re-grant texting consent for the new number before any further text is sent; the re-grant itself happens through FEAT-06/FEAT-14, outside this spec's scope, but the email-conditional-required rule above depends on the current texting-consent state this rule change eventually produces.
- Per the dependency map's Contention note for the Client entity, field edits between the Pro's own sessions resolve last-write-wins; this spec does not alter that resolution, it only governs what values are acceptable to write.
- All field validation rules apply identically whether the field is edited from FEAT-13.SPEC-001 (private_note only) or FEAT-13.SPEC-002 (name/phone/email) -- the product definition establishes no screen-specific validation variance.

## Edge Cases

- **Phone number entered with international formatting (e.g., +1-555-123-4567)** -- Passes the valid-reachable-format rule; the format allows digits, spaces, dashes, parentheses, and a leading plus.
- **Name at exactly 100 characters** -- Passes validation. 101 characters shows the length error.
- **Private note at exactly 1,000 characters** -- Passes validation. 1,001 characters is blocked at input.
- **Client has active texting consent, then that consent is revoked (via FEAT-14) after the client record was saved with no email on file** -- The email-conditionally-required rule is not retroactively enforced against already-saved data; it is checked only when the Pro next saves a contact edit through FEAT-13.SPEC-002, at which point an empty email will be rejected if texting consent is not active at that moment.
- **Two of the Pro's clients are found to share a phone number due to a data-entry correction** -- The uniqueness rule blocks the edit-time save (via FEAT-13.SPEC-002) that would create the collision; the Pro must resolve which record is correct manually (out of scope for this spec, which only prevents the collision going forward through this feature's edit path) before either record can be saved with that number. This is distinct from a phone match found at creation time (FEAT-05/FEAT-30), which merges into a single record per the dependency map rather than being rejected.
- **Pro attempts to view a client's private_note as Platform Operator (Support) via FEAT-19** -- The field is omitted entirely from Support's view (per FEAT-12.SPEC-008's established Support-omission pattern for this same field), not shown blank or redacted -- it is not present in that screen's layout at all.
- **Ownership boundary: a client record somehow becomes associated with two Pro accounts (data inconsistency)** -- Not possible under the product definition (per the dependency map, "a client who books with two Pros gets two separate, unconnected Client records," scope-boundaries.md SC-04); this spec's "own clients only" condition is therefore always well-defined and never ambiguous.

## Acceptance Criteria

**FEAT-13.SPEC-005-AC-01:** Given Talia clears the name field on FEAT-13.SPEC-002 and moves focus away, when blur validation runs, then the error "Name is required" appears.

**FEAT-13.SPEC-005-AC-02:** Given Talia enters a name of exactly 100 characters, when she saves, then it is accepted; entering 101 characters shows "Name must be 100 characters or fewer."

**FEAT-13.SPEC-005-AC-03:** Given Talia enters an invalid phone format, when blur validation runs, then the error "Please enter a valid phone number" appears.

**FEAT-13.SPEC-005-AC-04:** Given Talia enters a phone number already used by another of her own clients, when she taps Save, then the save is rejected with "This phone number is already used by another client. Check for a duplicate before saving."

**FEAT-13.SPEC-005-AC-05:** Given Talia enters an invalid email format, when blur validation runs, then the error "Please enter a valid email address" appears.

**FEAT-13.SPEC-005-AC-06:** Given a client has no active texting consent for Talia and Talia leaves the email field empty, when she taps Save, then the save is rejected with "An email address is required for clients who haven't opted in to texts."

**FEAT-13.SPEC-005-AC-07:** Given a client has active texting consent for Talia and Talia leaves the email field empty, when she taps Save, then the save succeeds -- email is not required.

**FEAT-13.SPEC-005-AC-08:** Given Talia types a private note of exactly 1,000 characters, when she saves, then it is accepted; a 1,001st character is blocked from being typed at all.

**FEAT-13.SPEC-005-AC-09:** Given Talia changes a client's phone number, when she attempts to save, then the save is held until she acknowledges the access-link-invalidation and fresh-consent consequence via FEAT-13.SPEC-002's confirmation step.

**FEAT-13.SPEC-005-AC-10:** Given Talia (the Pro) views her own client's record, when the screen loads, then all fields including private_note are visible to her.

**FEAT-13.SPEC-005-AC-11:** Given Platform Operator (Support) views a client record through FEAT-19's read-only screen, when the record renders, then the private_note field is entirely omitted from what Support sees.

**FEAT-13.SPEC-005-AC-12:** Given Platform Operator (Support) attempts to reach this feature's own screens directly, when the access check runs, then Support is redirected to the Pro sign-in screen and never reaches an edit or delete control for this entity.

**FEAT-13.SPEC-005-AC-13:** Given Riley (the Client) attempts to reach any of this feature's screens, when the access check runs, then she is redirected to the Pro sign-in screen -- this feature exposes no client-facing view or edit path at all.

**FEAT-13.SPEC-005-AC-14:** Given Talia attempts to delete a client with an upcoming booking, when she opens FEAT-13.SPEC-003, then the Blocked panel is shown instead of a bare denial, per FEAT-13.SPEC-006's eligibility rule.

**FEAT-13.SPEC-005-AC-15:** Given Talia attempts to delete an eligible client, when she confirms, then the deletion is allowed -- consistent with the Pro having Full access to her own clients.

**FEAT-13.SPEC-005-AC-16:** Given Platform Operator (Support) has no delete control anywhere in their read-only view, when Support views any client, then no deletion action is ever presented to them.

**FEAT-13.SPEC-005-AC-17:** Given a client's private_note field is left empty (never written to), when the Pro views the client record, then no error is shown -- private_note has no "required" rule, only a maximum-length rule.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 8 | 8 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 12 | 12 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Deletion Eligibility & Retention Rule

## Overview

**Name:** Deletion Eligibility & Retention Rule
**ID:** FEAT-13.SPEC-006
**Type:** Logic/Rule
**Purpose:** Governs when a client record may be deleted (blocked by an upcoming booking), the irreversibility of deletion, and the retention/de-identification of related records afterward.
**Parent Feature:** FEAT-13 -- Client Record Management
**Governed Entity:** Client record (deletion lifecycle)

## Scope and Non-Goals

**In Scope:**
- The eligibility condition that gates deletion (no upcoming booking exists)
- The irreversibility of deletion once executed
- What is retained versus removed after deletion (contact details and notes removed; Messaging Consent cascade-deleted; financial and timeline history retained de-identified)
- The concurrency rule for an edit attempted on a record that has just been deleted

**Non-Goals:**
- Field-level validation for name, phone, email, and private_note -- governed entirely by FEAT-13.SPEC-005 (Client Field Validation & Access Rules); this spec's Field Validation Rules section below defers to it for every field
- The confirmation UI and eligibility-result display -- handled by FEAT-13.SPEC-003 (Client Deletion Confirmation); this spec defines the rule, that spec presents it
- The mechanics of performing the deletion write itself -- handled by FEAT-13.SPEC-004 (Client Deletion Execution); this spec defines what is and is not allowed, that spec carries it out
- Cancelling the blocking booking -- owned by FEAT-30 (Pro Booking Management)'s cancel-with-full-refund flow; this spec only defines that an upcoming booking blocks deletion, not how it is resolved

## Governed Entity

**Entity:** Client
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| name | text | Client's full name |
| phone | text | Client's phone number |
| email | text | Client's email address |
| private_note | text | The Pro's own private note about this client |
| booking_notes | text | The client's own optional per-booking note, captured elsewhere |
| booking_history | derived | This client's Bookings with this Pro |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-13.SPEC-003 | Client Deletion Confirmation | Eligibility checked on screen load, before the Pro can confirm |
| FEAT-13.SPEC-004 | Client Deletion Execution | Eligibility re-checked at the moment of execution; retention/de-identification rules applied during processing |
| FEAT-13.SPEC-001 | Client Record Detail | The post-deletion concurrent-edit refusal rule applies to any save attempted here after the record is gone |
| FEAT-13.SPEC-002 | Client Contact Edit | Same post-deletion concurrent-edit refusal rule applies to any save attempted here after the record is gone |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| name | No validation beyond data type in this spec -- governed by FEAT-13.SPEC-005 | Always | -- | -- | -- |
| phone | No validation beyond data type in this spec -- governed by FEAT-13.SPEC-005 | Always | -- | -- | -- |
| email | No validation beyond data type in this spec -- governed by FEAT-13.SPEC-005 | Always | -- | -- | -- |
| private_note | No validation beyond data type in this spec -- governed by FEAT-13.SPEC-005 | Always | -- | -- | -- |
| booking_notes | No validation beyond data type in this spec -- read-only from this feature, governed elsewhere | Always | -- | -- | -- |
| booking_history | No validation beyond data type in this spec -- derived, read-only | Always | -- | -- | -- |

## Cross-Field Rules

N/A -- this spec's core rule is a cross-entity eligibility gate between the Client and Booking entities (no upcoming Booking may exist), not a same-entity cross-field rule. That gate is defined under Business Rules and Authorization Rules below, since it governs an action (deletion) rather than a field's own valid values.

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Delete client record | The Pro (Talia) | Own clients only, and only when no Booking exists for this client with a state of Pending Payment, Confirmed, or Awaiting Outcome whose appointment time has not yet passed (i.e., no upcoming booking) | When an upcoming booking exists: FEAT-13.SPEC-003 shows the Blocked panel naming that booking's date, time, and service, with a "Cancel booking with full refund" route into FEAT-30 -- never a bare "cannot delete" message |
| Delete client record | Platform Operator (Support) | Never | No delete control exists anywhere in Support's read-only view, per scope-boundaries.md SC-05 |
| Delete client record | The Client (Riley) | Never | A Client cannot request or execute their own deletion in-product, per scope-boundaries.md SC-01; deletion happens only after an informal request to the Pro |
| Undo or restore a deleted client record | The Pro (Talia) | Never -- no role may reverse a completed deletion | No undo control exists anywhere in the product; the Brief's Non-Goals state deletion "is not reversible, consistent with 'delete on request' meaning delete" |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Deletion eligibility (derived, not a stored field) | True when zero Bookings for this client are in a Pending Payment, Confirmed, or Awaiting Outcome state with a future or in-progress appointment time; false otherwise | Recalculated every time FEAT-13.SPEC-003 loads and again immediately before FEAT-13.SPEC-004 executes | No -- the Pro cannot override an ineligible result; they must first cancel the blocking booking through FEAT-30 |
| De-identified client reference (on retained Booking, Deposit Transaction, and Activity Event records) | A marker that replaces the deleted client's identity while leaving service, timing, amount, and outcome fields intact | Applied once, at the moment of successful deletion (FEAT-13.SPEC-004) | No -- de-identification is never reversed and never re-linked to a new client record, even if the same person books again |
| Messaging Consent retention exception | Where law requires evidence of a consent action to be kept, the minimum required evidence is retained stripped of the phone number and any other identifying field beyond that evidence itself | Applied once, at the moment of successful deletion (FEAT-13.SPEC-004) | No |

## Business Rules

- XBR-19: client deletion is refused while an upcoming booking exists (the Pro is offered cancel-with-full-refund); removes contact details, notes, and consent; retains financial and timeline records only in de-identified form; a later booking creates a new record.
- A completed or no-show booking, a cancelled booking, and a rescheduled-away-from booking do not count as "upcoming" for this rule -- only Pending Payment, Confirmed, and Awaiting Outcome states with a not-yet-passed appointment time block deletion, since those are the states in which cancelling and refunding the client still matters (per XBR-12's booking outcome windows).
- Per the dependency map's Contention note for the Client entity, once a client record is deleted, any concurrent edit attempted against it (a private-note save, a contact-field save) is refused and the Pro is prompted to refresh -- there is no partial or stale-write path into a deleted record.
- Per the dependency map's Contention note, a later booking from the same person after deletion always creates a new, unconnected Client record; the deletion automation (FEAT-13.SPEC-004) never re-links a new booking to the de-identified history left behind by a previous deletion.
- Deletion is all-or-nothing (FEAT-13.SPEC-004): a client record is never left partially deleted, and eligibility is re-checked at execution time, not trusted from the confirmation screen's earlier check alone.

## Edge Cases

- **A booking is cancelled by the Client themselves (via FEAT-10) between the Pro opening FEAT-13.SPEC-003 and confirming deletion** -- The confirmation screen's eligibility snapshot is now stale (it may still show Blocked); the Pro must navigate away and back to see the updated Eligible result, since this rule's eligibility check is not live-updating.
- **The blocking booking passes its appointment time and becomes Awaiting Outcome while the Blocked panel is displayed** -- It still counts as blocking under this rule (Awaiting Outcome is one of the three blocking states) until it resolves to Completed or No-Show or is otherwise cancelled.
- **Exactly one booking exists and it is in the Awaiting Outcome state (appointment has passed but outcome not yet marked)** -- Deletion remains blocked; the Pro must wait for FEAT-12's auto-completion sweep or manually mark the outcome (FEAT-11/FEAT-12) before the client becomes eligible, or cancel is no longer offered once the appointment has passed (per XBR-12, a passed appointment can no longer be cancelled, only completed or marked no-show).
- **Deletion is attempted twice from two devices for the same client at effectively the same time** -- The first execution to reach FEAT-13.SPEC-004's write step succeeds; the second re-checks eligibility, finds the client record already gone, and fails with "This client's record no longer exists," per FEAT-13.SPEC-004's own concurrency handling.
- **A private-note edit (FEAT-13.SPEC-001) is saved a moment after deletion completes on another device** -- The save is refused per this rule's post-deletion concurrent-edit refusal, with the exact message defined in FEAT-13.SPEC-001's edge cases ("This client's record no longer exists. It may have been deleted.").
- **The Pro deletes a client, then that same phone number books again a year later** -- A brand-new Client record is created by FEAT-05 with no reference to the deleted record or its de-identified history; the de-identified Booking and Deposit Transaction records from the deletion remain permanently unconnected to the new record.

## Acceptance Criteria

**FEAT-13.SPEC-006-AC-01:** Given Talia opens FEAT-13.SPEC-003 for a client with a Confirmed booking three days from now, when the eligibility check runs, then deletion is blocked and the Blocked panel names that booking.

**FEAT-13.SPEC-006-AC-02:** Given Talia opens FEAT-13.SPEC-003 for a client with only Completed and Cancelled bookings, when the eligibility check runs, then deletion is eligible.

**FEAT-13.SPEC-006-AC-03:** Given a client's only booking is in the Awaiting Outcome state (appointment time passed, outcome not yet marked), when the eligibility check runs, then deletion remains blocked.

**FEAT-13.SPEC-006-AC-04:** Given Talia cancels the blocking booking through FEAT-30's cancel-with-full-refund route and returns to FEAT-13.SPEC-003, when the eligibility check re-runs, then deletion is now eligible.

**FEAT-13.SPEC-006-AC-05:** Given Talia confirms deletion for an eligible client, when FEAT-13.SPEC-004 executes, then the deletion is irreversible and no undo control is ever presented to her afterward.

**FEAT-13.SPEC-006-AC-06:** Given a deleted client's Messaging Consent required no legally mandated evidence retention, when deletion executes, then that consent record is fully removed along with the contact details.

**FEAT-13.SPEC-006-AC-07:** Given a deleted client's Deposit Transaction history exists, when deletion executes, then those records are retained with a de-identified client reference, keeping service, timing, amount, and outcome intact.

**FEAT-13.SPEC-006-AC-08:** Given a client record was deleted, when the Pro attempts to save a private-note edit against it from a stale screen, then the save is refused with "This client's record no longer exists. It may have been deleted."

**FEAT-13.SPEC-006-AC-09:** Given a client record was deleted a year ago, when that same phone number books again through FEAT-05, then a brand-new, unconnected Client record is created rather than resurrecting the old one.

**FEAT-13.SPEC-006-AC-10:** Given Platform Operator (Support) views any client's record, when Support looks for a delete control, then none exists, per scope-boundaries.md SC-05.

**FEAT-13.SPEC-006-AC-11:** Given Riley (the Client) wants her record deleted, when she looks for an in-product way to request or execute it herself, then none exists -- she must ask the Pro informally, per scope-boundaries.md SC-01.

**FEAT-13.SPEC-006-AC-12:** Given two devices attempt to confirm deletion for the same client at effectively the same time, when both executions run, then the first to complete succeeds and the second fails with "This client's record no longer exists."

**FEAT-13.SPEC-006-AC-13:** Given a client is cancelled by the client themselves between FEAT-13.SPEC-003's load and Talia's confirmation tap, when Talia confirms deletion anyway, then FEAT-13.SPEC-004 re-checks eligibility at execution and proceeds since the booking is now Cancelled by Client, not upcoming.

**FEAT-13.SPEC-006-AC-14:** Given a new upcoming booking is created for a client between FEAT-13.SPEC-003's load and Talia's confirmation tap, when Talia confirms deletion anyway, then FEAT-13.SPEC-004 re-checks eligibility at execution, finds the new booking, and refuses the deletion, leaving all data unchanged.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 6 (all deferred to FEAT-13.SPEC-005) | 6 |
| Cross-Field Rules | 1 (N/A, cross-entity gate explained) | 1 |
| Authorization Rules | 4 | 4 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |


---
document_type: feature-overview
feature_number: FEAT-13
feature_name: Client Record Management
feature_slug: client-record-management
priority_tier: Important
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

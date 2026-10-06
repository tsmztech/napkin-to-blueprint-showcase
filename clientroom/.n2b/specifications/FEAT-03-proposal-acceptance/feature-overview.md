---
document_type: feature-overview
feature_number: FEAT-03
feature_name: Proposal Acceptance
feature_slug: proposal-acceptance
priority_tier: Core
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 7
screen_count: 2
automation_count: 2
logic_rule_count: 1
integration_count: 0
notification_count: 2
---

# Feature Breakdown Brief: Proposal Acceptance

## Summary

**Feature:** Proposal Acceptance
**ID:** FEAT-03
**Description:** The client's Primary Contact reviews a sent proposal and accepts it with one click; the acceptance is recorded with a timestamp that can never be silently altered, and a deposit invoice appears immediately if the payment schedule calls for one.
**Priority:** Core
**Phase:** MVP
**Type:** User-Facing
**Rationale:** BRIEF.md's Experience narrative: "clicks 'Accept.' The acceptance is recorded with a timestamp, a deposit invoice appears" — this is the moment a project becomes real and billable. MVP phase: required for the deposit-invoice trigger and the evidence record the brief calls out as a Success Criterion. [RESEARCH-INFORMED: all 5 profiled competitors bundle proposals with e-signature; the brief's presumptive default (a recorded, timestamped Accept) is kept for MVP and the e-signature upgrade (FEAT-26) is brought forward to v1] [INFERRED: carried from Visionary draft]

**Key Capabilities:**
- Review the proposal — client reads scope and price before deciding
- Accept with one click — records a timestamped, permanent acceptance
- Automatic deposit invoicing — a deposit invoice is generated immediately when the schedule includes one
- Request changes — instead of accepting, the Primary Contact sends Nadia a short note asking for changes

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-03.SPEC-001 | Proposal Review & Accept | Screen | Owen | Owen reads the sent proposal's scope and price and either accepts it or opens Request Changes |
| FEAT-03.SPEC-002 | Request Changes | Screen | Owen | Owen composes and sends a short change-request note to Nadia instead of accepting |
| FEAT-03.SPEC-003 | Acceptance Recording | Automation | Owen, Nadia | Writes the immutable acceptance record and fires the downstream deposit-invoice and audit-trail effects |
| FEAT-03.SPEC-004 | Change-Request Recording | Automation | Owen, Nadia | Writes the change-request note as a Comment on the proposal and notifies Nadia |
| FEAT-03.SPEC-005 | Acceptance & Access Rules | Logic/Rule | Owen, Nadia, Priya, Dana | Governs who may accept or request changes, single-acceptance and voided-proposal enforcement, and concurrent-action resolution |
| FEAT-03.SPEC-006 | Acceptance Confirmation Notification | Notification | Owen, Nadia | Emails Owen and Nadia when the acceptance is recorded |
| FEAT-03.SPEC-007 | Change-Request Notification | Notification | Nadia | Emails Nadia immediately when Owen submits a change-request note |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Review the proposal | FEAT-03.SPEC-001 | Primary purpose of the review screen — scope and price render fully before Accept activates | Phase 2 (Explicit) |
| Accept with one click | FEAT-03.SPEC-001, FEAT-03.SPEC-003 | Screen exposes the Accept control; the Acceptance Recording automation writes the immutable record | Phase 2 (Explicit) |
| Automatic deposit invoicing | FEAT-03.SPEC-003 | Acceptance Recording fires the deposit-invoice trigger (XBR-01) to FEAT-09 when the Payment Schedule includes a deposit | Phase 2 (Explicit) |
| Request changes | FEAT-03.SPEC-002, FEAT-03.SPEC-004 | Screen exposes the Request Changes control; the Change-Request Recording automation writes the note and notifies Nadia | Phase 2 (Explicit) |

**Analyst-Discovered Specs** — Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-03.SPEC-005 | Acceptance & Access Rules | Phase 5 (Rule Discovery) | The Validation & Limits field (accept-once, voided-cannot-accept, note length), the Access field's four differentiated role behaviors, and the Proposal entity's Contention entry (reject-with-refresh, double-accept collapse) together exceed the 5-rule / shared-across-specs threshold for a standalone Logic/Rule spec |
| FEAT-03.SPEC-006 | Acceptance Confirmation Notification | Phase 4 (Notification surfacing) | The Communications field names a confirmation email to Owen and Nadia with a defined audience and delivery behavior — not a same-screen toast — so it needs a standalone Notification spec |
| FEAT-03.SPEC-007 | Change-Request Notification | Phase 4 (Notification surfacing) | The Communications field names an immediate email to Nadia when a change-request note is submitted, with its own audience and delivery behavior |

## Entity-Lifecycle Coverage Matrix

**Entity: Proposal** *(this feature updates it; creation and versioning belong to FEAT-02)*

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A | Owned by FEAT-02 (proposal drafting and sending). This feature only acts on an already-sent proposal. | — |
| Read (single) | FEAT-03.SPEC-001 | Review screen loads the sent proposal's scope, price, and status for the accepting contact | — |
| Read (list) | N/A | A project has at most one active (non-voided) proposal (dependency map, Proposal Relationships), so no list view exists in this feature | — |
| Update | FEAT-03.SPEC-003 | Acceptance Recording writes `status: Accepted`, `accepted_at`, and `accepted_by` once, never altered afterward | Also updated by FEAT-02 (edit/void) and FEAT-26 (signature record, v1) — not this feature's concern |
| Delete/Archive | N/A | This feature never deletes or archives a Proposal. Voiding is owned by FEAT-02 (XBR-06); permanent deletion is owned by FEAT-24. No retention/purge decision belongs to this feature. | — |
| State Transition | FEAT-03.SPEC-003 | Sent → Accepted, enforced as a one-way, one-time transition by FEAT-03.SPEC-005's eligibility rule | A Voided proposal cannot transition to Accepted (XBR-06); SPEC-001 redirects the client to the current version instead |

**Entity: Comment** *(this feature creates the change-request variant only; general comment lifecycle belongs to FEAT-07)*

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-03.SPEC-004 | Change-Request Recording writes a Comment with `target: Proposal`, `text` (1–2,000 characters), `author: Owen`, `posted_at` | Distinct from deliverable/milestone comments, which FEAT-07 creates |
| Read (single) | N/A | Within this feature, no screen re-displays a past change-request note; it is read by FEAT-02 when Nadia opens the proposal to revise it | See Cross-Feature Touchpoints |
| Read (list) | N/A | Same as above — this feature does not list change-request history | — |
| Update | N/A | Edit-within-grace-window and retraction are owned by FEAT-07; a request-changes note never alters the proposal itself (Validation & Limits) | — |
| Delete/Archive | N/A | Owned by FEAT-24 only (Comment lifecycle, dependency map); no soft-delete or purge decision belongs to this feature | — |
| State Transition | N/A | Posted/Retracted transitions are owned by FEAT-07 | — |

**Referenced Entities (read-only or triggered, not owned by this feature):**

| Entity | Read By | Context |
|--------|---------|---------|
| Payment Schedule | FEAT-03.SPEC-003 | Read at the moment of acceptance to determine whether a deposit trigger fires, using the schedule as it stood at that moment (dependency map, Payment Schedule Contention) |
| Client Contact | FEAT-03.SPEC-001, FEAT-03.SPEC-003, FEAT-03.SPEC-005 | Identifies the accepting contact (`accepted_by`) and enforces that only a Primary contact at the owning client can view, accept, or request changes (Access Matrix, XBR-08) |
| Invoice | FEAT-09 (not this feature) | FEAT-03.SPEC-003 fires the deposit-invoice trigger (XBR-01); FEAT-09 owns Invoice creation, fields, and its own notification. This feature neither creates nor reads Invoice records. |
| Project | FEAT-03.SPEC-001, FEAT-03.SPEC-003 | Acceptance is scoped to the Proposal's owning Project; Project stage label (e.g., "Proposal accepted") is derived elsewhere (FEAT-01) from this feature's state change |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Owen clicks Accept | Validate the proposal is current and unaccepted, then write the immutable acceptance record | Standalone Automation | FEAT-03.SPEC-003 |
| Acceptance is recorded | Fire the deposit-invoice trigger to FEAT-09 when the Payment Schedule includes a deposit | Cross-feature — logged in touchpoints | FEAT-03.SPEC-003 → FEAT-09 |
| Acceptance is recorded | Write an Activity Log Entry for the acceptance event | Cross-feature — logged in touchpoints | FEAT-03.SPEC-003 → FEAT-13 (XBR-05) |
| Acceptance is recorded | Email confirmation to Owen and Nadia | Standalone Notification | FEAT-03.SPEC-006 |
| Owen clicks Accept a second time, or against a version voided in the meantime | Reject-with-refresh: show "already accepted" or redirect to the current version, without a duplicate acceptance record | Standalone Logic/Rule | FEAT-03.SPEC-005 |
| Accept action fails mid-flight (e.g., a connectivity drop after tap) | Retry the action without recording a duplicate acceptance or losing the client's click intent | Standalone Logic/Rule (enforced within FEAT-03.SPEC-003's write) | FEAT-03.SPEC-005 |
| Owen submits a Request Changes note | Validate length (1–2,000 characters), then write it as a Comment on the proposal | Standalone Automation | FEAT-03.SPEC-004 |
| Change-request note is recorded | Email Nadia immediately | Standalone Notification | FEAT-03.SPEC-007 |
| Change-request note is recorded | Write an Activity Log Entry for the change-request event | Cross-feature — logged in touchpoints | FEAT-03.SPEC-004 → FEAT-13 (XBR-05) |
| Owen accepts or requests changes successfully | Show an on-screen confirmation state (no separate delivery rules) | Inline in triggering screen | FEAT-03.SPEC-001 / FEAT-03.SPEC-002 |
| Owen opens a proposal that was edited after sending | Show the prior version as voided and redirect to the current one | Inline in triggering screen, using the eligibility rule | FEAT-03.SPEC-001 (rule from FEAT-03.SPEC-005) |
| Priya or an unauthorized contact opens the proposal link | Show no proposal content; portal home shows only the project's stage label | Standalone Logic/Rule (authorization) | FEAT-03.SPEC-005 |
| Dana opens a proposal inside a support session | Show read-only content with no Accept or Request Changes control, inside the logged session | Standalone Logic/Rule (authorization) | FEAT-03.SPEC-005 |

## Shared Context

**Shared Entities:**
- Proposal — read by FEAT-03.SPEC-001, updated (accepted state only) by FEAT-03.SPEC-003, governed by FEAT-03.SPEC-005's eligibility rule. Fields relevant to this feature: `status`, `sent_at`, `accepted_at`, `accepted_by`, `payment_schedule_reference`.
- Comment (change-request variant) — created by FEAT-03.SPEC-004. Fields: `text` (1–2,000 characters), `author`, `posted_at`, `target: Proposal`.
- Client Contact — read by all specs in this feature to establish the acting contact's role (Primary vs. Reviewer) and identity for `accepted_by` / comment `author`.

**Shared UI Patterns:**
- Decision controls — FEAT-03.SPEC-001's Accept and Request Changes controls are large, clearly labeled tap targets, usable by keyboard and screen reader (Access field, Accessibility). FEAT-03.SPEC-002 reuses the same control sizing and labeling convention for its Send control.
- Immutable-record marker — once accepted, FEAT-03.SPEC-001 displays an "Accepted on {date}" marker in place of the Accept control; this same display convention (a permanent, non-editable marker replacing an action control) is the pattern other evidentiary records in the product should follow (XBR-04), noted here for consistency though only this feature's screen is in scope.

**Shared Validation:**
- FEAT-03.SPEC-005 defines the single source of truth for: who may act on a given proposal (Access field), whether an accept or request-changes attempt is currently valid (accept-once, voided-cannot-accept, note length), and how simultaneous actions resolve. FEAT-03.SPEC-001 and FEAT-03.SPEC-002 both reference FEAT-03.SPEC-005 rather than duplicating these checks, and FEAT-03.SPEC-003 and FEAT-03.SPEC-004 both enforce it at write time.

## Internal Dependency Map

```
FEAT-03.SPEC-001 (Proposal Review & Accept) -> [Owen taps Accept] -> FEAT-03.SPEC-005 (Acceptance & Access Rules) -> [eligible] -> FEAT-03.SPEC-003 (Acceptance Recording)
FEAT-03.SPEC-003 (Acceptance Recording) -> [acceptance written] -> FEAT-03.SPEC-006 (Acceptance Confirmation Notification)
FEAT-03.SPEC-003 (Acceptance Recording) -> [acceptance written, deposit due] -> FEAT-09 (Invoice Generation & Sending) [cross-feature]
FEAT-03.SPEC-003 (Acceptance Recording) -> [acceptance written] -> FEAT-13 (Immutable Activity & Audit Trail) [cross-feature]
FEAT-03.SPEC-001 (Proposal Review & Accept) -> [Owen taps Request Changes] -> FEAT-03.SPEC-002 (Request Changes)
FEAT-03.SPEC-002 (Request Changes) -> [Owen taps Send] -> FEAT-03.SPEC-005 (Acceptance & Access Rules) -> [note valid] -> FEAT-03.SPEC-004 (Change-Request Recording)
FEAT-03.SPEC-004 (Change-Request Recording) -> [note written] -> FEAT-03.SPEC-007 (Change-Request Notification)
FEAT-03.SPEC-004 (Change-Request Recording) -> [note written] -> FEAT-13 (Immutable Activity & Audit Trail) [cross-feature]
FEAT-03.SPEC-004 (Change-Request Recording) -> [note recorded] -> FEAT-02 (Proposal Creation & Sending) [cross-feature: Nadia opens the change request and revises]
FEAT-05 (Client Portal Access) -> [Owen opens the proposal waiting on him] -> FEAT-03.SPEC-001 (Proposal Review & Accept) [cross-feature entry point]
```

**Default Entry:** FEAT-03.SPEC-001 (Proposal Review & Accept) — the screen shown when Owen navigates to this feature, whether from the portal home (FEAT-05) or an emailed proposal link (FEAT-02).

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-03.SPEC-001 | Inbound | FEAT-05 (Client Portal Access) | Owen opens the proposal waiting on him from his portal home | Owen navigates from portal home |
| FEAT-03.SPEC-004 | Outbound | FEAT-02 (Proposal Creation & Sending) | Nadia opens the change-request email and revises the proposal | Change-request note recorded |
| FEAT-03.SPEC-003 | Outbound | FEAT-09 (Invoice Generation & Sending) | Acceptance triggers the deposit invoice when the schedule includes one (XBR-01); FEAT-09 owns invoice creation and its own notification | Acceptance recorded |
| FEAT-03.SPEC-003 | Outbound | FEAT-13 (Immutable Activity & Audit Trail) | Acceptance writes an append-only trail entry (XBR-05) | Acceptance recorded |
| FEAT-03.SPEC-004 | Outbound | FEAT-13 (Immutable Activity & Audit Trail) | Change-request note writes an append-only trail entry (XBR-05) | Change-request note recorded |
| FEAT-03.SPEC-001, FEAT-03.SPEC-005 | Inbound | FEAT-02 (Proposal Creation & Sending) | A proposal edited after sending arrives at this feature already marked Voided; this feature only displays and redirects, it does not void | Proposal re-sent after edit (XBR-06) |
| FEAT-03.SPEC-005 | Inbound | FEAT-18 (Client Contact Management) | Role (Primary vs. Reviewer) and contact status feed every authorization check in this feature (XBR-08) | Contact role assigned or changed |
| FEAT-03.SPEC-005 | Inbound | FEAT-31 (Operator Support Access) | Dana's read-only, logged support session sees this feature's screen with no action controls | Support session opened |
| FEAT-03 (feature-level) | Outbound | FEAT-26 (Legally Binding E-Signature) | v1: a legally binding signature replaces the plain Accept click with the same Primary-only access and immutability (XBR-34); FEAT-03 owns the acceptance record that FEAT-26 extends | Signature capability enabled for a proposal |
| FEAT-03 (feature-level) | Outbound | FEAT-14 (Notifications & Delivery) | Both this feature's Notification specs (FEAT-03.SPEC-006, FEAT-03.SPEC-007) rely on FEAT-14's transactional email delivery capability (ASMP-29) for actual sending and delivery/bounce status | Notification queued |

## Non-Functional Notes

**Data volumes / growth:** N/A — this feature does not introduce new data-volume concerns beyond the Proposal and Comment entities' own growth (bounded at one active proposal per project and one change-request note per request), which is already addressed under FEAT-02 and FEAT-07.

**Responsiveness:** Client-facing pages become interactive within roughly 2 seconds on a typical mobile connection (ASMP-21); the Review & Accept screen renders scope and price fully before the Accept control activates, so the perceived responsiveness requirement is that this render completes within that same window rather than leaving Owen waiting on an inert control.

**Data sensitivity / privacy:** The Proposal's scope and price are commercially confidential and, once accepted, evidentiary and immutable (dependency map, Proposal Data Sensitivity; ASMP-25); the accepting contact's identity (`accepted_by`) is personal data (ASMP-24). Strict client isolation applies (ASMP-23; XBR-09): only Owen's own company's proposal is ever reachable through this feature, and an out-of-scope or expired link shows a plain explanation and a fresh-link option, never another company's data.

**Compliance flags:** GDPR-class handling applies to the accepting contact's identity and to the change-request note's author (ASMP-24); if that contact is later erased (FEAT-18), the acceptance and the note remain on the record under their name as evidence (XBR-27, ASMP-20). The acceptance record itself must never be silently altered once written (ASMP-25).

## Non-Goals

- **A formal "decline" state** — Excluded per this feature's own Primary Flows & Alternates: there is no in-product decline; a proposal the client is not ready to accept simply stays open until Nadia revises and re-sends it, or the project is cancelled (FEAT-25, XBR-25). Request Changes is the product's only structured "not yet" path.
- **General-purpose messaging or chat for change requests** — Excluded per scope-boundaries.md (SC-15): the request-changes note is a single, contextual note attached to the proposal, not a back-and-forth chat channel; further discussion happens through Nadia revising and re-sending the proposal.
- **Reviewer (Priya) visibility into proposal content** — Excluded per the Access Matrix (Proposals & Acceptance: None for Reviewers) and this feature's Access field: Priya sees only the project's stage label, never scope, price, or the Accept/Request Changes controls, per scope-boundaries.md's SC-02 two-role limit on client-side roles.
- **Legally binding e-signature at MVP** — Deferred per scope-boundaries.md's Deferral note: the timestamped, immutable Accept click is the launch default; e-signature is FEAT-26, brought forward to v1 but not part of this feature's MVP scope.
- **A standalone Integration spec for email delivery or payment processing in this feature** — The transactional email capability that both this feature's Notification specs rely on is owned by FEAT-14 (External Touchpoints table); the payment-processing capability behind the deposit invoice's pay link is owned by FEAT-32/FEAT-10 and invoiced by FEAT-09, not integrated by this feature (context package, Dependencies slice). Both are recorded as cross-feature touchpoints rather than duplicated as Integration specs here.
- **Retention/purge policy for the acceptance record** — Not applicable to this feature: the Proposal (and its acceptance fields) is never deleted or archived by this feature; retention and eventual deletion are owned entirely by FEAT-24 (account deletion, subject to legal retention), so no lifecycle gap exists here to resolve.

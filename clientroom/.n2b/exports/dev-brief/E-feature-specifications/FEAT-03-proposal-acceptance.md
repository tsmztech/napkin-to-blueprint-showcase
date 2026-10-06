# FEAT-03 — Proposal Acceptance

This chapter covers Proposal Acceptance, a Core-tier feature. It contains the feature breakdown brief followed by every specification in full: 7 specifications carrying 81 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-03.SPEC-001 | Proposal Review & Accept | screen | 15 |
| FEAT-03.SPEC-002 | Request Changes | screen | 12 |
| FEAT-03.SPEC-003 | Acceptance Recording | automation | 10 |
| FEAT-03.SPEC-004 | Change-Request Recording | automation | 8 |
| FEAT-03.SPEC-005 | Acceptance & Access Rules | logic-rule | 17 |
| FEAT-03.SPEC-006 | Acceptance Confirmation Notification | notification | 10 |
| FEAT-03.SPEC-007 | Change-Request Notification | notification | 9 |

The feature breakdown brief follows, then every specification in full.


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



# Screen Spec: Proposal Review & Accept

## Overview

**Name:** Proposal Review & Accept
**ID:** FEAT-03.SPEC-001
**Type:** Screen
**Purpose:** Owen, the client's Primary Contact, reads a sent proposal's scope and price and either accepts it (recording a timestamped, permanent acceptance) or opens Request Changes to send Nadia a note instead.
**Parent Feature:** FEAT-03 -- Proposal Acceptance

## Scope and Non-Goals

**In Scope:**
- Rendering the sent proposal's scope description and price fully before the Accept control activates
- The Accept control and its confirmation
- The Request Changes control that opens FEAT-03.SPEC-002
- Displaying the permanent "Accepted on {date}" marker once acceptance is recorded
- Redirecting to the current proposal version when the opened version has been voided (edited-after-sending)
- Showing Priya's stage-only view and Dana's read-only support view of this same entry point

**Non-Goals:**
- A formal "decline" state -- excluded per the Feature Breakdown Brief's Non-Goals: there is no in-product decline; a proposal the client is not ready to accept simply stays open until Nadia revises and re-sends it or the project is cancelled (FEAT-25, XBR-25). Request Changes is the product's only structured "not yet" path.
- Editing or voiding the proposal -- owned entirely by Proposal Creation & Sending (FEAT-02); this screen only reads and reacts to the proposal's current state.
- Legally binding e-signature -- deferred per scope-boundaries.md's Deferral note; the timestamped, immutable Accept click is the launch default, and e-signature (FEAT-26) is out of this feature's MVP scope.
- Listing past proposal versions or a change-request history -- the Feature Breakdown Brief's Entity-Lifecycle Coverage Matrix confirms a project has at most one active proposal and this feature does not list history; only the current version is ever shown here.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-05 (Client Portal Access, portal home) | Owen navigates from the "proposal waiting on him" item on his portal home | The proposal reference for his company's active or most recently accepted proposal |
| External -- emailed proposal link (FEAT-02.SPEC-011, Proposal Sent/Resent Email) | Owen opens the proposal-sent email and follows the link | The specific proposal reference from the email; if that version has since been voided, the screen redirects to the current version |
| FEAT-03.SPEC-002 (Request Changes) | Owen taps "Back" or completes sending a change-request note | Confirmation that the note was sent; proposal state re-loaded |
| FEAT-03.SPEC-006 (Acceptance Confirmation Notification) | Owen taps the confirmation email's "View proposal" CTA | The accepted proposal reference; the screen shows "Accepted on {accepted_at_date}" |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | None -- this screen is not part of Nadia's own workspace; she sees the resulting acceptance status on the project view (FEAT-01), not this screen | No | Attempting to open this screen's link is treated as an out-of-scope link: plain explanation and a fresh-link option (XBR-09), never proposal content |
| Owen (Client Primary Contact) | Full screen -- scope, price, and current status of his own company's proposal (Own-only) | Accept the proposal; open Request Changes (FEAT-03.SPEC-002) | -- |
| Priya (Client Reviewer Contact) | No proposal content -- portal home shows only the project's stage label (e.g., "Proposal accepted"), per the Access Matrix (Proposals & Acceptance: None for Reviewers) | No | Attempting to reach this screen's link shows the project's stage label only, with no scope, price, or Accept/Request Changes controls |
| Dana (Support Operator) | Full screen content, read-only, inside a logged support session (FEAT-31) | No actions -- Accept and Request Changes controls are not rendered | If Dana attempts an action outside the support session's read-only bounds, the action is not available on screen; there is no control to attempt it with |
| Unauthenticated | No | No | Redirected to the client portal's magic-link sign-in (FEAT-05); after signing in as a recognized contact, the user lands on this screen if they are Owen, or on the stage-only view if Priya |
| Expired session | No | No | Magic link is single-use and time-limited (XBR-28); an expired link shows a plain explanation and a "request a fresh link" option (FEAT-05); no proposal content is shown in the meantime |

## Layout and Content

**Header:** Project name and client company name at the top, with the proposal's status label ("Sent" or "Accepted on {date}") shown beside it.

**Body:** A single-column, read-only presentation of the proposal, in this order:
- Scope description -- full text, rendered before any control below it activates
- Price -- the proposal's price in the project's set currency
- Payment schedule summary -- a short, plain-language statement of the payment structure (deposit, per-milestone, on completion, or a mix) referenced from the Payment Schedule, shown for Owen's context ahead of deciding
- Two primary controls, grouped below the price: an "Accept" button and a "Request Changes" button, both large, clearly labeled tap targets (Shared UI Patterns: Decision controls)
- Once accepted: the "Accept" and "Request Changes" controls are replaced in place by a single "Accepted on {date}" marker (Shared UI Patterns: Immutable-record marker), and the price and scope remain visible below it, now in a visually settled, non-editable presentation

**Footer:** None -- both decision controls sit in the body, directly below the price.

### Responsive Behavior

- **Compact breakpoint:** Single-column body as described, full width; the Accept and Request Changes controls stack vertically, Accept above Request Changes, each spanning the full content width for an easy mobile tap target.
- **Medium size class and above:** Body content is capped at a consistent platform-wide reading width and horizontally centered; the Accept and Request Changes controls sit side by side rather than stacked, Accept on the left.
- **Payment schedule summary:** Uniform scaling, no structural change across breakpoints.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Scope description and price | Screen loads | Reads the current Proposal record's `scope_description`, `price`, `currency`, and `payment_schedule_reference` | Content renders fully; Accept and Request Changes remain inactive until render completes | Loading indicator while content renders, then full content appears |
| Accept button | Tap | 1. Checks eligibility via FEAT-03.SPEC-005 (Acceptance & Access Rules). 2. If eligible, triggers FEAT-03.SPEC-003 (Acceptance Recording). | Button shows a brief loading state during the check and write | Success: button and Request Changes are both replaced by the "Accepted on {date}" marker. Failure: see Edge Cases (already accepted, voided, connectivity). |
| Request Changes button | Tap | Navigates to FEAT-03.SPEC-002 (Request Changes) | Screen transitions to the Request Changes form | Standard navigation transition |
| "Accepted on {date}" marker | None -- display-only | None | None | Non-interactive; communicates the permanent record |
| Payment schedule summary | None -- display-only | None | None | Non-interactive; provides context ahead of the decision |

### Accessibility Notes

- **Focus order:** Header status label -> scope description -> price -> payment schedule summary -> Accept button -> Request Changes button (or, once accepted, the "Accepted on {date}" marker in their place).
- **Content-ready announcement:** When the scope and price finish rendering and the Accept and Request Changes controls become active, this transition is announced to assistive technology so a screen-reader user is not left waiting on a silently inert control.
- **Acceptance announcement:** When acceptance succeeds, the "Accepted on {date}" marker replacing the controls is announced as a live-region update.
- **Keyboard alternatives:** Accept and Request Changes are both standard activatable controls reachable and operable by keyboard; there are no pointer-only gestures on this screen.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Empty | N/A -- this screen exists only once a proposal has been sent; there is no zero-data variant of this screen to render, since a Proposal reference is required to reach it at all | Never entered | Never entered |
| Loading | Scope, price, and payment schedule summary show a loading indicator; Accept and Request Changes are not yet rendered | Screen first opens | Proposal content finishes loading |
| Ready to decide (default) | Full scope, price, and payment schedule summary shown; Accept and Request Changes are both active | Content load completes and the proposal's status is Sent | Owen taps Accept (success) or navigates to Request Changes |
| Accepted | Scope and price remain visible; Accept and Request Changes are replaced by the "Accepted on {date}" marker | Acceptance is successfully recorded (this session or a prior one) | Never exits -- this is a permanent, terminal state for this proposal version |
| Error (accept failed) | Inline message below the controls: "We couldn't record your acceptance. Check your connection and try again." Controls remain active for retry. | The accept action fails mid-flight | Owen retries successfully, or navigates away |
| Voided -- redirected | Brief message "This proposal has been updated" before the current version loads | Owen opens a proposal version that was edited and re-sent after this link was generated | Redirect completes and the current version's Ready-to-decide (or Accepted) state loads |
| Offline/Degraded | Scope and price remain visible from the last successful load; Accept and Request Changes are visibly disabled with the message "Reconnect to accept" beneath them | Connectivity is lost while this screen is open | Connectivity returns -- controls re-activate automatically, no false success is ever shown |

## Validation Rules

Validation and eligibility governed by FEAT-03.SPEC-005 (Acceptance & Access Rules). See that spec for accept-once enforcement, voided-proposal enforcement, and role-based access rules. This screen calls that eligibility check at the moment Accept is tapped, not before.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|-------------------------------------|
| Request Changes button tap | FEAT-03.SPEC-002 (Request Changes) | -- |
| Successful acceptance | Stays on this screen, now in the Accepted state | -- |
| Voided-version redirect | This same screen, reloaded against the current proposal version | -- |
| Return navigation from FEAT-03.SPEC-002 | This screen, reloaded | -- |

## Data Model

**Creates:** None.
**Reads:** Proposal -- `scope_description`, `price`, `currency`, `payment_schedule_reference`, `status`, `sent_at`, `accepted_at`, `accepted_by`. Client Contact -- the signed-in contact's `role` and `status`, to confirm the viewer is a Primary contact in Active status.
**Updates:** None directly -- Accept triggers FEAT-03.SPEC-003, which performs the Proposal update.
**Deletes:** None.

## Business Rules

- Eligibility to accept (accept-once, voided-cannot-accept) is governed by FEAT-03.SPEC-005 -- this screen does not duplicate that logic, it calls the check.
- XBR-06: A voided proposal cannot be accepted; this screen redirects Owen to the current version rather than showing the stale one as actionable.
- XBR-08: Only a Primary contact at the owning client may view, accept, or request changes; Priya (Reviewer) never reaches this screen's content.
- XBR-09: Client isolation -- Owen only ever reaches his own company's proposal through this screen; an out-of-scope link shows a plain explanation and a fresh-link option.
- The scope and price render fully before the Accept control activates, preventing an accidental early tap (Feature Breakdown Brief, States).

## Edge Cases

- **Owen taps Accept a second time, or against a version voided in the meantime** -- Reject-with-refresh per FEAT-03.SPEC-005: the screen shows "This proposal has already been accepted" (if already accepted) or redirects to the current version (if voided), without recording a duplicate acceptance. Resolution: reject-with-refresh, per the dependency map's Contention note for the Proposal entity.
- **Two Primary contacts at the same client accept at the same moment** -- Only one acceptance is recorded; the second contact's screen shows "This proposal has already been accepted" instead of a second confirmation, per the dependency map's Proposal Contention note ("two Primary contacts accepting at the same moment yield one acceptance and the second sees 'already accepted'").
- **Connectivity drops immediately after Owen taps Accept** -- The action is retried without recording a duplicate acceptance or losing the click's intent (Feature Breakdown Brief, Side-Effect Inventory); the screen shows the Error state with a retry option rather than a false success.
- **Owen navigates away and returns before deciding** -- The screen re-fetches the proposal's current state; no local draft exists to preserve since this is a read-and-decide screen, not a form.
- **Priya follows a proposal link meant for Owen** -- No proposal content is shown; her portal home shows only the project's stage label, per the Access Matrix.
- **Dana opens this screen inside a support session** -- Full content renders read-only; Accept and Request Changes controls are not rendered at all, so there is no control for Dana to attempt.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|-------------------|--------------|
| FEAT-03.SPEC-002 (Request Changes) | Navigation (outbound) | Request Changes button navigates here |
| FEAT-03.SPEC-003 (Acceptance Recording) | Triggers (outbound) | Accept button, once eligible, triggers the immutable acceptance write |
| FEAT-03.SPEC-005 (Acceptance & Access Rules) | References (outbound) | Eligibility, access, and concurrent-action rules for the Accept action |
| FEAT-05 (Client Portal Access) | Navigation (inbound) | Owen arrives from his portal home |
| FEAT-02 (Proposal Creation & Sending) | Navigation (inbound) | Owen arrives from the emailed proposal link; a voided-and-resent proposal redirects here to the current version |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|---------------|-------------------|
| proposal_viewed_by_client | proposal reference, contact role, time since sent | Screen finishes loading and content is shown to Owen | supports success-metrics.md: "Time to Proposal Acceptance" |
| proposal_accepted | proposal reference, time elapsed since sent_at | FEAT-03.SPEC-003 confirms the acceptance was written | supports success-metrics.md: "Time to Proposal Acceptance" |
| proposal_accept_failed | failure reason (connectivity, already accepted, voided) | The accept action does not complete successfully | N/A -- no connected success metric measures acceptance failures; recorded here as diagnostic-only exhaust from the accept flow |

## Acceptance Criteria

**FEAT-03.SPEC-001-AC-01:** Given Owen opens the proposal link from his portal home, when the screen loads, then the scope description and price render fully before the Accept and Request Changes controls become active.

**FEAT-03.SPEC-001-AC-02:** Given Owen is on this screen with the proposal in Sent status, when he taps Accept, then the acceptance is recorded and the Accept and Request Changes controls are replaced by an "Accepted on {date}" marker.

**FEAT-03.SPEC-001-AC-03:** Given Owen is on this screen, when he taps Request Changes, then he is navigated to FEAT-03.SPEC-002 (Request Changes).

**FEAT-03.SPEC-001-AC-04:** Given Owen is on a proposal already showing "Accepted on {date}", when he views the screen, then no Accept or Request Changes control is shown.

**FEAT-03.SPEC-001-AC-05:** Given Owen taps Accept on a proposal that was already accepted by another Primary contact moments earlier, when the eligibility check runs, then the screen shows "This proposal has already been accepted" and no duplicate acceptance is recorded.

**FEAT-03.SPEC-001-AC-06:** Given Owen opens a proposal link for a version that Nadia has since edited and re-sent, when the screen loads, then he is redirected to the current version of the proposal.

**FEAT-03.SPEC-001-AC-07:** Given Owen taps Accept and loses connectivity mid-flight, when connectivity drops, then the screen shows an inline error message with a retry option and no acceptance is recorded until a retry succeeds.

**FEAT-03.SPEC-001-AC-08:** Given Owen loses connectivity while viewing the screen, when he attempts to tap Accept, then the controls are visibly disabled with the message "Reconnect to accept."

**FEAT-03.SPEC-001-AC-09:** Given Priya (Reviewer) follows a proposal link meant for Owen, when her portal home loads, then she sees only the project's stage label and no scope, price, Accept, or Request Changes control.

**FEAT-03.SPEC-001-AC-10:** Given Dana (Support Operator) opens this screen inside a logged support session, when the screen loads, then the full proposal content renders but no Accept or Request Changes control is shown.

**FEAT-03.SPEC-001-AC-11:** Given an unauthenticated visitor opens the proposal link, when the screen would otherwise load, then they are redirected to the client portal's magic-link sign-in.

**FEAT-03.SPEC-001-AC-12:** Given Owen's sign-in session has expired, when he opens the proposal link, then he sees a plain explanation and a "request a fresh link" option, with no proposal content shown.

**FEAT-03.SPEC-001-AC-13:** Given Owen is on the screen in the Ready-to-decide state, when he views the layout at a compact breakpoint, then the Accept and Request Changes controls are stacked vertically, Accept above Request Changes.

**FEAT-03.SPEC-001-AC-14:** Given Owen successfully accepts the proposal, when the acceptance is recorded, then a proposal_accepted analytics event is emitted with the proposal reference and time elapsed since sent_at.

**FEAT-03.SPEC-001-AC-15:** Given an out-of-scope contact opens a proposal link that does not belong to their own company, when the screen would otherwise load, then they see a plain explanation and a fresh-link option, never another company's proposal data.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 7 (empty, loading, ready to decide, accepted, error, voided-redirected, offline) | 7 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Screen Spec: Request Changes

## Overview

**Name:** Request Changes
**ID:** FEAT-03.SPEC-002
**Type:** Screen
**Purpose:** Owen composes and sends a short change-request note to Nadia instead of accepting the proposal.
**Parent Feature:** FEAT-03 -- Proposal Acceptance

## Scope and Non-Goals

**In Scope:**
- A single text field for Owen's change-request note (1--2,000 characters)
- The Send control and its validation
- Confirmation feedback once the note is sent
- Returning to the proposal review screen

**Non-Goals:**
- General-purpose messaging or chat -- excluded per scope-boundaries.md (SC-15): the request-changes note is a single, contextual note attached to the proposal, not a back-and-forth chat channel; further discussion happens through Nadia revising and re-sending the proposal.
- Displaying past change-request notes -- excluded per the Feature Breakdown Brief's Entity-Lifecycle Coverage Matrix: within this feature, no screen re-displays a past change-request note; only FEAT-02 reads it when Nadia revises.
- Editing or retracting a sent note -- owned by FEAT-07 (Deliverable Review & Feedback), which governs the general Comment edit-within-grace-window and retraction lifecycle; this screen only creates the note.
- Attaching files or images to the note -- excluded per the Feature Breakdown Brief's Validation & Limits: a change-request note is defined as 1--2,000 characters of text and does not alter the proposal itself; no attachment capability is part of this feature's scope.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-03.SPEC-001 (Proposal Review & Accept) | Owen taps "Request Changes" | Proposal reference; the note field starts empty |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | None -- this is a client-side composition screen | No | Attempting to open this screen's link is treated as an out-of-scope link: plain explanation and a fresh-link option (XBR-09) |
| Owen (Client Primary Contact) | Full screen (Own-only) | Compose and send a change-request note on his own company's proposal | -- |
| Priya (Client Reviewer Contact) | No -- Reviewers have no access to proposal content, per the Access Matrix | No | Cannot reach this screen; her portal home shows only the project's stage label |
| Dana (Support Operator) | No -- this is a write-only action screen with no view-only mode defined for it, consistent with Dana never sending anything on a freelancer's or client's behalf | No | If reached inside a support session, the screen shows no Send control; Dana's read-only view of the proposal is limited to FEAT-03.SPEC-001, which never links here for her |
| Unauthenticated | No | No | Redirected to the client portal's magic-link sign-in (FEAT-05) |
| Expired session | No | No | Plain explanation and a "request a fresh link" option (FEAT-05); any note in progress is not preserved across the expired session |

## Layout and Content

**Header:** Screen title "Request Changes" with a back arrow (returns to FEAT-03.SPEC-001, Proposal Review & Accept).

**Body:** A single-column form with:
- A short instructional line: a brief statement that this note goes to Nadia and does not change the proposal itself
- A multi-line text input for the change-request note (required, 1--2,000 characters), with a live character count shown beneath it
- A "Send" action button below the text input

**Footer:** None -- Send is in the body, directly below the text input.

### Responsive Behavior

- **Compact breakpoint:** Single-column form as described, full width; Send spans the full content width, consistent with the Shared UI Patterns: Decision controls sizing convention from FEAT-03.SPEC-001.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; the multi-line text input grows to show more visible lines.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-03.SPEC-001 (Proposal Review & Accept) | Screen closes; unsent note is discarded | Confirmation dialog if the note field is non-empty (see Edge Cases) |
| Note text input | Type | Captures text input | Character count updates live | Standard input focus state; character count shown beneath the field |
| Note text input | Exceeds 2,000 characters | Blocks further input beyond the limit | Field shows the limit reached | Character count shows "2,000 / 2,000" and further typing is not accepted |
| Send button | Tap | 1. Validates the note length via FEAT-03.SPEC-005. 2. If valid, triggers FEAT-03.SPEC-004 (Change-Request Recording). | Button shows a loading state during send | Success: confirmation message "Your note has been sent to Nadia" and navigation to FEAT-03.SPEC-001. Failure: inline error message, note preserved. |
| Send button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> instructional line -> note text input -> Send button.
- **Character count announcement:** The live character count is available to assistive technology on request but does not interrupt typing; the "2,000 / 2,000" limit-reached state is announced when input is blocked.
- **Validation announcements:** When the note is empty or invalid at Send, the resulting error message is announced and programmatically associated with the text input.
- **Send feedback:** The "Your note has been sent to Nadia" confirmation is announced on success.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Empty (default) | Note field empty, Send button enabled but will fail on tap until text is entered | Screen first opens | Owen begins typing |
| Filling | Note field contains user input, live character count updates, Send button enabled | Owen types in the note field | Owen taps Send or navigates away |
| Sending | Send button shows a loading spinner, note field disabled | Owen taps Send with a valid note | Send completes or fails |
| Validation Error | Note field shows an error state below it | Send is tapped with an empty note or one exceeding 2,000 characters | Owen corrects the note and re-taps Send |
| Sent (confirmation) | Confirmation message shown, then navigation back to FEAT-03.SPEC-001 | Send completes successfully | Screen transitions away after the confirmation is shown |
| Error | Inline error banner: "Could not send your note. Check your connection and try again." with a Retry option; note text preserved | The send action fails after passing validation | Owen retries successfully, or navigates away |
| Offline/Degraded | Banner "You're offline -- your note will be sent when you reconnect." at top; note field remains editable; Send queues the note locally | Connectivity is lost while this screen is open | Connectivity restored -- queued note sends automatically and the standard success feedback appears |

## Validation Rules

Validation governed by FEAT-03.SPEC-005 (Acceptance & Access Rules), which defines the note-length rule (1--2,000 characters) shared with FEAT-03.SPEC-004's write-time enforcement. This screen checks the rule on Send.

| Field | Condition | When Checked | Error Message |
|-------|-----------|---------------|-----------------|
| Note text input | Must not be empty | On Send | "Enter a note before sending." |
| Note text input | Must not exceed 2,000 characters | On change and on Send | "Your note can be up to 2,000 characters." |

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|-------------------------------------|
| Back arrow tap | FEAT-03.SPEC-001 (Proposal Review & Accept) | -- |
| Successful send | FEAT-03.SPEC-001 (Proposal Review & Accept) | -- |
| Cancel with unsent note (via confirmation) | FEAT-03.SPEC-001 (Proposal Review & Accept) | -- |

## Data Model

**Creates:** None directly -- Send triggers FEAT-03.SPEC-004, which creates the Comment record.
**Reads:** Proposal -- `status` (to confirm the proposal is still eligible for a change request at Send time, per FEAT-03.SPEC-005).
**Updates:** None.
**Deletes:** None.

## Business Rules

- Note length validation (1--2,000 characters) is governed by FEAT-03.SPEC-005 -- this screen enforces it at Send but does not own the rule.
- XBR-26: A request-changes note from the Primary contact is recorded as a comment on the proposal, notifies the freelancer immediately, and never alters the proposal itself.
- XBR-08: Only a Primary contact at the owning client may request changes; this screen is unreachable to Priya (Reviewer).
- The note never alters the proposal itself -- sending a change request does not change the proposal's `status`, `scope_description`, or `price`.

## Edge Cases

- **Owen navigates back with an unsent, non-empty note** -- Confirmation dialog: "Discard this note?" with "Discard" and "Keep Editing" options.
- **Owen taps Send twice rapidly** -- Second tap is ignored while the first send is in progress (button in loading state).
- **The proposal is voided (edited and re-sent by Nadia) while Owen is composing his note** -- Send is rejected with a message directing Owen to the current version, per FEAT-03.SPEC-005's eligibility check running again at Send time; the note text is preserved so Owen can re-submit it against the current version if he still wants to.
- **Network failure during send** -- Error banner: "Could not send your note. Check your connection and try again." with a Retry button. Note text preserved.
- **Owen submits exactly 2,000 characters** -- Accepted; the limit is inclusive.
- **The proposal is accepted by Owen through another session while this screen is open** -- Send is rejected per FEAT-03.SPEC-005 (an accepted proposal is no longer eligible for a change request); the screen shows a message that the proposal has already been accepted and offers navigation back to FEAT-03.SPEC-001, which now shows the Accepted state. No concurrent-edit conflict on the Comment entity itself arises here, since this screen only creates a new Comment and never edits an existing one (dependency map, Comment Contention: "None").

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|-------------------|--------------|
| FEAT-03.SPEC-001 (Proposal Review & Accept) | Navigation (inbound) | Owen arrives here from the Request Changes control |
| FEAT-03.SPEC-004 (Change-Request Recording) | Triggers (outbound) | Send button, once valid, triggers the note write |
| FEAT-03.SPEC-005 (Acceptance & Access Rules) | References (outbound) | Note length and eligibility rules |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|---------------|-------------------|
| proposal_changes_requested | proposal reference, note character count | FEAT-03.SPEC-004 confirms the note was recorded | N/A -- no success metric in success-metrics.md is connected to the change-request path; "Time to Proposal Acceptance" measures only the accept path, and this feature has no other connected metric to attribute a change-request signal to |

## Acceptance Criteria

**FEAT-03.SPEC-002-AC-01:** Given Owen is on the Proposal Review & Accept screen, when he taps Request Changes, then he is navigated to this screen with an empty note field.

**FEAT-03.SPEC-002-AC-02:** Given Owen is on this screen, when he types a note and taps Send, then the note is recorded as a Comment on the proposal and Nadia is notified, and Owen sees the confirmation "Your note has been sent to Nadia" before returning to FEAT-03.SPEC-001.

**FEAT-03.SPEC-002-AC-03:** Given Owen is on this screen with an empty note field, when he taps Send, then the note field shows the error "Enter a note before sending." and no note is recorded.

**FEAT-03.SPEC-002-AC-04:** Given Owen has typed 2,000 characters into the note field, when he attempts to type further, then no additional characters are accepted and the character count shows "2,000 / 2,000".

**FEAT-03.SPEC-002-AC-05:** Given Owen has typed a note and not yet sent it, when he taps the back arrow, then a confirmation dialog "Discard this note?" appears with "Discard" and "Keep Editing" options.

**FEAT-03.SPEC-002-AC-06:** Given Owen taps Send and loses connectivity mid-flight, when the send fails, then an inline error banner appears with a Retry option and the note text is preserved.

**FEAT-03.SPEC-002-AC-07:** Given Owen loses connectivity while composing his note, when he attempts to tap Send, then the banner "You're offline -- your note will be sent when you reconnect." appears and the note is submitted automatically once connectivity returns.

**FEAT-03.SPEC-002-AC-08:** Given the proposal Owen is viewing is voided by Nadia editing and re-sending it while he is composing his note, when he taps Send, then the send is rejected and Owen is directed to the current version, with his note text preserved.

**FEAT-03.SPEC-002-AC-09:** Given the proposal Owen is viewing is accepted through another session while he is composing his note, when he taps Send, then the send is rejected with a message that the proposal has already been accepted.

**FEAT-03.SPEC-002-AC-10:** Given Priya (Reviewer) attempts to reach this screen, when she follows any link toward it, then she cannot reach it and her portal home shows only the project's stage label.

**FEAT-03.SPEC-002-AC-11:** Given an unauthenticated visitor attempts to open this screen, when the screen would otherwise load, then they are redirected to the client portal's magic-link sign-in.

**FEAT-03.SPEC-002-AC-12:** Given Owen successfully sends a change-request note, when the note is recorded, then a proposal_changes_requested analytics event is emitted with the proposal reference and note character count.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 7 (empty, filling, sending, validation error, sent, error, offline) | 7 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Automation Spec: Acceptance Recording

## Overview

**Name:** Acceptance Recording
**ID:** FEAT-03.SPEC-003
**Type:** Automation
**Purpose:** Writes the immutable acceptance record on the Proposal when Owen accepts, and fires the downstream deposit-invoice and audit-trail effects.
**Parent Feature:** FEAT-03 -- Proposal Acceptance

## Scope and Non-Goals

**In Scope:**
- Validating that the proposal is current (not voided) and unaccepted at the moment of write
- Writing the Proposal's `status`, `accepted_at`, and `accepted_by` fields exactly once
- Firing the deposit-invoice trigger to FEAT-09 when the Payment Schedule includes a deposit
- Firing the audit-trail entry to FEAT-13
- Firing the confirmation notification (FEAT-03.SPEC-006)

**Non-Goals:**
- Generating or sending the deposit invoice itself -- owned entirely by FEAT-09 (Invoice Generation & Sending); this automation only fires the trigger per XBR-01, using the Payment Schedule as it stood at the moment of acceptance.
- Composing or sending the confirmation email's content -- owned by FEAT-03.SPEC-006 (Acceptance Confirmation Notification), which this automation only triggers.
- Legally binding e-signature capture -- deferred per scope-boundaries.md's Deferral note; the timestamped Accept click, written by this automation, is the launch default, and e-signature (FEAT-26) extends this record in v1 without changing this automation's scope.
- Voiding or editing the proposal -- owned by FEAT-02 (Proposal Creation & Sending); this automation only reads the proposal's current status to determine eligibility.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|-------------------|
| Owen taps Accept | FEAT-03.SPEC-001 (Proposal Review & Accept) | Fires after FEAT-03.SPEC-005's eligibility check passes (proposal is Sent, not yet Accepted, not Voided, and the actor is a Primary contact at the owning client) | Proposal reference, accepting Client Contact's identity, current timestamp |

## Processing Logic

1. Receive the proposal reference and the accepting Client Contact's identity from FEAT-03.SPEC-001, after FEAT-03.SPEC-005's eligibility check has passed on that screen.
2. Re-check eligibility at write time against the Proposal's current state: `status` must still be `Sent` (not `Accepted`, not `Voided`). This re-check is the authoritative one -- the screen-time check in step 1 only gates the user's initial tap.
3. If the re-check fails because the proposal is already `Accepted`, stop and return the "already accepted" outcome (no write performed).
4. If the re-check fails because the proposal is `Voided`, stop and return the "voided -- redirect" outcome (no write performed).
5. If the re-check passes, write the acceptance in a single, atomic step: set `status` to `Accepted`, `accepted_at` to the current timestamp, and `accepted_by` to the accepting Client Contact's reference. This write is exactly-once: the atomic step itself is what prevents two concurrent passes of step 2-5 from both succeeding.
6. Read the Project's Payment Schedule as it stood at this exact moment (dependency map, Payment Schedule Contention: "a trigger uses the schedule as it stood at the moment of the triggering action").
7. If the Payment Schedule's structure includes a deposit, fire the deposit-invoice trigger to FEAT-09 (XBR-01), passing the Payment Schedule's deposit terms as read in step 6.
8. If the Payment Schedule's structure does not include a deposit, take no invoicing action.
9. Fire the audit-trail entry to FEAT-13 (XBR-05) with event type "proposal accepted," the accepting contact as actor, the timestamp from step 5, and the Proposal as the affected record.
10. Fire FEAT-03.SPEC-006 (Acceptance Confirmation Notification) to notify Owen and Nadia.
11. Return the success outcome to FEAT-03.SPEC-001, which displays the "Accepted on {date}" marker.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|-----------------|-------------------|
| Acceptance recorded, no deposit due | Write succeeds; Payment Schedule has no deposit trigger | Proposal `status` set to Accepted, `accepted_at` and `accepted_by` written | FEAT-03.SPEC-001 shows the "Accepted on {date}" marker | FEAT-03.SPEC-001, FEAT-03.SPEC-006, FEAT-13 |
| Acceptance recorded, deposit invoice triggered | Write succeeds; Payment Schedule includes a deposit | Proposal `status` set to Accepted, `accepted_at` and `accepted_by` written; deposit-invoice trigger fired to FEAT-09 | FEAT-03.SPEC-001 shows the "Accepted on {date}" marker; the deposit invoice appears shortly after via FEAT-09's own notification | FEAT-03.SPEC-001, FEAT-03.SPEC-006, FEAT-09, FEAT-13 |
| Already accepted | Write-time re-check finds `status` already `Accepted` (a concurrent acceptance won the race, or the client re-submitted) | None -- no duplicate write | FEAT-03.SPEC-001 shows "This proposal has already been accepted" | FEAT-03.SPEC-001 |
| Voided -- redirect | Write-time re-check finds `status` is `Voided` (Nadia edited and re-sent between screen load and this write) | None | FEAT-03.SPEC-001 redirects Owen to the current proposal version | FEAT-03.SPEC-001 |
| Write failure (connectivity or processing error) | The write itself does not complete (e.g., a connectivity drop mid-flight) | None -- the write either fully completes or leaves no partial acceptance record | FEAT-03.SPEC-001 shows an inline error with a retry option; retrying re-runs this automation from step 2 | FEAT-03.SPEC-001 |

## Data Model

**Reads:** Proposal -- `status`, `sent_at` (to confirm eligibility). Payment Schedule -- `structure`, `deposit_amount` (read as it stands at the exact moment of acceptance). Client Contact -- the accepting contact's identity and `role`.
**Creates:** None directly -- the deposit-invoice trigger causes FEAT-09 to create an Invoice; the audit-trail trigger causes FEAT-13 to create an Activity Log Entry. Neither record is created by this automation itself.
**Updates:** Proposal -- `status` (Sent to Accepted), `accepted_at`, `accepted_by`. Written exactly once; never altered afterward (XBR-04).
**Deletes:** None.

## Business Rules

- XBR-01: Accepting a proposal immediately generates and sends a deposit invoice when the Payment Schedule includes a deposit, using the schedule as it stood at acceptance. This automation owns firing that trigger; FEAT-09 owns the invoice itself.
- XBR-04: The acceptance record is never silently altered once written -- `accepted_at` and `accepted_by` are set exactly once and are never updated by any later process.
- XBR-05: Acceptance writes an append-only Activity Log Entry with actor and timestamp.
- The Proposal Contention resolution (dependency map): acceptance is recorded exactly once; two Primary contacts accepting at the same moment yield one acceptance, and the second sees "already accepted" -- enforced by this automation's atomic write in Processing Logic step 5, not by the triggering screen.
- The write-time eligibility re-check (step 2) is authoritative over the screen-time check performed by FEAT-03.SPEC-005 on FEAT-03.SPEC-001 -- the screen check only prevents an obviously stale tap; this automation's own re-check is what actually guarantees exactly-once acceptance.

## Edge Cases

- **Two Primary contacts at the same client tap Accept within the same instant** -- Concurrent trigger firing: both invocations reach step 2 near-simultaneously, but the atomic write in step 5 admits only one. The first to complete the atomic write succeeds; the second's re-check (step 2) then finds `status` already `Accepted` and returns the "already accepted" outcome. Neither invocation blocks the other; there is no queuing.
- **Owen taps Accept, the automation begins, and he taps Accept again before the first run finishes (e.g., a slow connection)** -- Trigger fires while a previous run is in flight: FEAT-03.SPEC-001 disables the Accept control while the automation is running (screen-level debounce), so a second automation run for the same proposal from the same session cannot start until the first completes. If it did reach this automation regardless, step 2's re-check on the second run would find the first run's write already applied (once it commits) and return "already accepted," or would race the still-in-flight first run under the same exactly-once write guarantee as the concurrent-trigger case.
- **Nadia edits and re-sends the proposal between Owen's tap and this automation's write** -- The write-time re-check (step 2) is the authority here, not the screen-time check: if the void completes before this automation's re-check runs, the re-check finds `status: Voided` and returns "voided -- redirect" with no acceptance recorded.
- **The Payment Schedule is being adjusted by Nadia at the exact moment of acceptance** -- Per the dependency map's Payment Schedule Contention, this automation reads the schedule as it stood at the moment of acceptance (step 6); a schedule edit saved afterward is dated and applies to later triggers only, never retroactively to this acceptance's deposit determination.
- **The deposit-invoice trigger to FEAT-09 fails to fire after the acceptance write has already succeeded** -- The acceptance record itself is not rolled back (it is already the evidentiary, immutable record per XBR-04); the deposit-invoice trigger is retried by FEAT-09's own retry handling. Owen still sees "Accepted on {date}" -- the deposit invoice's own appearance is FEAT-09's concern, not this automation's failure path.
- **The audit-trail trigger to FEAT-13 fails to fire** -- Non-blocking: the acceptance record and any deposit-invoice trigger already fired are unaffected; the audit-trail write is retried by FEAT-13's own handling, consistent with FEAT-13's append-only, never-silently-lost design intent.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|-------------------|--------------|
| FEAT-03.SPEC-001 (Proposal Review & Accept) | Triggered by (inbound) | Accept button tap, after eligibility passes |
| FEAT-03.SPEC-005 (Acceptance & Access Rules) | References (inbound) | Eligibility, accept-once, and voided-proposal rules this automation re-checks and enforces at write time |
| FEAT-03.SPEC-006 (Acceptance Confirmation Notification) | Triggers (outbound) | Fires the confirmation email to Owen and Nadia once acceptance is written |
| FEAT-09 (Invoice Generation & Sending) | Triggers (outbound) | Fires the deposit-invoice trigger (XBR-01) when the schedule includes a deposit |
| FEAT-13 (Immutable Activity & Audit Trail) | Triggers (outbound) | Fires the append-only trail entry for the acceptance event (XBR-05) |

## Analytics and Success Signals

- **proposal_accepted** (proposal reference, deposit invoice triggered: yes/no, time elapsed since sent_at) -- supports success-metrics.md: "Time to Proposal Acceptance"
- **deposit_invoice_auto_generated** (proposal reference, deposit amount reference) -- N/A -- no success metric in success-metrics.md is connected to deposit-invoice generation itself; this event is recorded here as the trigger-side signal, with the invoice's own lifecycle measured under FEAT-09's metrics
- **proposal_accept_write_failed** (failure reason: connectivity / concurrent-write-lost) -- N/A -- no connected success metric measures write failures; recorded as diagnostic-only exhaust from this automation's write path

## Acceptance Criteria

**FEAT-03.SPEC-003-AC-01:** Given Owen taps Accept on a proposal in Sent status with a Payment Schedule that includes no deposit, when this automation runs, then the Proposal's `status` becomes Accepted with `accepted_at` and `accepted_by` set, and no invoice trigger fires.

**FEAT-03.SPEC-003-AC-02:** Given Owen taps Accept on a proposal in Sent status with a Payment Schedule that includes a deposit, when this automation runs, then the acceptance is recorded and the deposit-invoice trigger fires to FEAT-09 (XBR-01), using the schedule as it stood at that moment.

**FEAT-03.SPEC-003-AC-03:** Given the acceptance write succeeds, when this automation completes, then it fires FEAT-03.SPEC-006 to notify Owen and Nadia, and fires the FEAT-13 audit-trail entry for the acceptance event.

**FEAT-03.SPEC-003-AC-04:** Given two Primary contacts at the same client both tap Accept on the same proposal at effectively the same moment, when this automation's write-time re-check runs for each, then exactly one acceptance is recorded and the other invocation returns "already accepted."

**FEAT-03.SPEC-003-AC-05:** Given Owen taps Accept on a proposal that Nadia voids by editing and re-sending before this automation's write-time re-check runs, when the re-check finds `status: Voided`, then no acceptance is recorded and FEAT-03.SPEC-001 redirects Owen to the current version.

**FEAT-03.SPEC-003-AC-06:** Given Owen taps Accept and connectivity drops before the write completes, when the failure occurs, then no partial acceptance record is left and FEAT-03.SPEC-001 shows a retry option.

**FEAT-03.SPEC-003-AC-07:** Given Owen retries Accept after a failed write, when this automation re-runs, then it re-checks eligibility from the current Proposal state rather than assuming the prior attempt's context still holds.

**FEAT-03.SPEC-003-AC-08:** Given Nadia adjusts the Payment Schedule at the exact moment Owen's acceptance is being written, when this automation reads the schedule in step 6, then it uses the schedule as it stood at the moment of acceptance, and Nadia's adjustment applies only to later triggers.

**FEAT-03.SPEC-003-AC-09:** Given the acceptance write succeeds but the deposit-invoice trigger to FEAT-09 fails to fire, when this automation completes, then the acceptance record remains valid and unaffected, and the invoice trigger is retried by FEAT-09's own handling.

**FEAT-03.SPEC-003-AC-10:** Given the acceptance is recorded, when the analytics signal is emitted, then a proposal_accepted event carries the proposal reference, whether a deposit invoice was triggered, and the time elapsed since the proposal was sent.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 5 (no deposit, deposit triggered, already accepted, voided-redirect, write failure) | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 (concurrent tap, run-in-flight, void-race, schedule-race, invoice-trigger-failure, audit-trigger-failure) | 6 |



# Automation Spec: Change-Request Recording

## Overview

**Name:** Change-Request Recording
**ID:** FEAT-03.SPEC-004
**Type:** Automation
**Purpose:** Writes Owen's change-request note as a Comment on the proposal and notifies Nadia immediately.
**Parent Feature:** FEAT-03 -- Proposal Acceptance

## Scope and Non-Goals

**In Scope:**
- Validating the note's length (1--2,000 characters) at write time
- Re-checking that the proposal is still eligible to receive a change request (not Voided, not Accepted) at the moment of write
- Writing the Comment record with `target: Proposal`, `text`, `author`, `posted_at`
- Firing the change-request notification (FEAT-03.SPEC-007) and the audit-trail entry (FEAT-13)

**Non-Goals:**
- Altering the proposal itself -- excluded per XBR-26: a request-changes note never changes the Proposal's `scope_description`, `price`, or `status`; it only creates a Comment referencing the proposal.
- Composing or sending the notification email's content -- owned by FEAT-03.SPEC-007 (Change-Request Notification), which this automation only triggers.
- General comment lifecycle (editing within a grace window, retraction) -- owned entirely by FEAT-07 (Deliverable Review & Feedback), per the Feature Breakdown Brief's Entity-Lifecycle Coverage Matrix; this automation only creates the change-request variant of a Comment, once.
- Nadia's revision of the proposal in response to the note -- owned by FEAT-02 (Proposal Creation & Sending); this automation's responsibility ends once the note is recorded and Nadia is notified.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|-------------------|
| Owen taps Send | FEAT-03.SPEC-002 (Request Changes) | Fires after the screen's own length check passes (1--2,000 characters, non-empty) | Proposal reference, note text, Owen's Client Contact identity, current timestamp |

## Processing Logic

1. Receive the proposal reference, note text, and Owen's Client Contact identity from FEAT-03.SPEC-002.
2. Re-validate the note length at write time: must be 1--2,000 characters. This re-check is authoritative over the screen-time check, in case the text was altered in transit or the screen check was bypassed.
3. If the length re-check fails, stop and return the "invalid note" outcome (no write performed) -- this should not normally occur since the screen already checked, but guards against a stale or tampered submission.
4. Re-check the proposal's current `status`: it must not be `Voided` and must not be `Accepted` (a change request against an already-decided proposal has nothing left to request changes on).
5. If the proposal is `Voided`, stop and return the "voided -- redirect" outcome (no write performed).
6. If the proposal is `Accepted`, stop and return the "already accepted" outcome (no write performed).
7. If both checks pass, write a new Comment record: `target: Proposal` (the proposal reference), `text` (the note), `author` (Owen's Client Contact reference), `posted_at` (current timestamp), `status: Posted`. This is a create-only write -- no existing Comment is modified.
8. Fire the audit-trail entry to FEAT-13 (XBR-05) with event type "change request submitted," Owen as actor, the timestamp from step 7, and the Proposal as the affected record.
9. Fire FEAT-03.SPEC-007 (Change-Request Notification) to notify Nadia immediately.
10. Return the success outcome to FEAT-03.SPEC-002, which shows the confirmation and navigates back to FEAT-03.SPEC-001.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|-----------------|-------------------|
| Note recorded | Both re-checks pass | New Comment created (target: Proposal, text, author, posted_at) | FEAT-03.SPEC-002 shows "Your note has been sent to Nadia" and returns to FEAT-03.SPEC-001 | FEAT-03.SPEC-002, FEAT-03.SPEC-007, FEAT-13 |
| Invalid note (length) | Write-time length re-check fails | None | FEAT-03.SPEC-002 shows the field-level length error and does not navigate away | FEAT-03.SPEC-002 |
| Voided -- redirect | Write-time status re-check finds `Voided` | None | FEAT-03.SPEC-002 shows a message directing Owen to the current proposal version; note text preserved for re-submission | FEAT-03.SPEC-002 |
| Already accepted | Write-time status re-check finds `Accepted` | None | FEAT-03.SPEC-002 shows a message that the proposal has already been accepted, with navigation back to FEAT-03.SPEC-001 (now showing Accepted) | FEAT-03.SPEC-002, FEAT-03.SPEC-001 |
| Write failure (connectivity or processing error) | The write itself does not complete | None | FEAT-03.SPEC-002 shows an inline error with a retry option; note text preserved | FEAT-03.SPEC-002 |

## Data Model

**Reads:** Proposal -- `status` (to confirm eligibility at write time). Client Contact -- Owen's identity and `role`.
**Creates:** Comment -- `target: Proposal`, `text` (1--2,000 characters), `author` (Owen's Client Contact reference), `posted_at`, `status: Posted`.
**Updates:** None -- the proposal itself is never modified by this automation (XBR-26).
**Deletes:** None.

## Business Rules

- XBR-26: A request-changes note from the Primary contact is recorded as a comment on the proposal, notifies the freelancer immediately, and never alters the proposal itself.
- XBR-05: The change-request event writes an append-only Activity Log Entry with actor and timestamp.
- Note length (1--2,000 characters) is the single validation rule this automation enforces at write time, mirroring FEAT-03.SPEC-005 and FEAT-03.SPEC-002's screen-level check.
- A change request can only be submitted against a proposal that is neither Voided nor Accepted -- consistent with FEAT-03.SPEC-005's eligibility rules for actions on a proposal.
- The Comment created here follows the dependency map's Comment Contention note ("None -- each comment is written and retracted only by its own author"): this automation only ever creates a new Comment, never modifies an existing one, so no concurrent-write conflict on the Comment entity itself can arise from this automation.

## Edge Cases

- **Owen submits a note exceeding 2,000 characters via a route that bypassed the screen's own live check (e.g., a stale form state)** -- The write-time re-check in step 2 catches this and returns "invalid note"; no Comment is created.
- **The proposal is voided by Nadia between Owen tapping Send on FEAT-03.SPEC-002 and this automation's write-time check** -- The re-check in step 4 is authoritative: it finds `Voided` and returns "voided -- redirect," with no Comment created.
- **The proposal is accepted (by Owen through another session, or by another Primary contact) between Send and this automation's write-time check** -- The re-check in step 4 finds `Accepted` and returns "already accepted," with no Comment created.
- **Two change-request notes are submitted in quick succession by Owen (e.g., a double-tap on Send)** -- Concurrent trigger firing: FEAT-03.SPEC-002 debounces the Send button while a send is in flight, so a second automation run for the same submission cannot start until the first completes; if both nonetheless reached this automation, each independently creates its own Comment (append-only, no conflict), consistent with the dependency map's Comment Contention note that many authors' comments on the same thread need no resolution beyond ordering by posted time.
- **A change-request submission is still in flight (processing) when Owen navigates back to FEAT-03.SPEC-001 and returns to FEAT-03.SPEC-002 to submit another note before the first completes** -- Trigger fires while a previous run is in flight: each submission is processed independently against its own note text; the first run's write, once it completes, does not block or invalidate the second's eligibility re-check, since the proposal's eligibility for a change request does not change as a result of a Comment being created (Comments do not affect Proposal `status`).
- **The audit-trail trigger to FEAT-13 fails to fire after the Comment write has already succeeded** -- Non-blocking: the Comment record and the notification to Nadia are unaffected; the audit-trail write is retried by FEAT-13's own handling.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|-------------------|--------------|
| FEAT-03.SPEC-002 (Request Changes) | Triggered by (inbound) | Send button tap, after screen-level length validation passes |
| FEAT-03.SPEC-005 (Acceptance & Access Rules) | References (inbound) | Note length and eligibility rules this automation re-checks and enforces at write time |
| FEAT-03.SPEC-007 (Change-Request Notification) | Triggers (outbound) | Fires the notification email to Nadia once the note is written |
| FEAT-13 (Immutable Activity & Audit Trail) | Triggers (outbound) | Fires the append-only trail entry for the change-request event (XBR-05) |
| FEAT-02 (Proposal Creation & Sending) | References (outbound) | Nadia later opens the recorded note when she revises and re-sends the proposal |

## Analytics and Success Signals

- **proposal_changes_requested** (proposal reference, note character count) -- N/A -- no success metric in success-metrics.md is connected to the change-request path; "Time to Proposal Acceptance" measures only the accept path, and this feature has no other connected metric to attribute this signal to
- **change_request_write_failed** (failure reason: invalid-note / voided / already-accepted / connectivity) -- N/A -- no connected success metric measures write failures; recorded as diagnostic-only exhaust from this automation's write path

## Acceptance Criteria

**FEAT-03.SPEC-004-AC-01:** Given Owen submits a valid note (1--2,000 characters) on a proposal in Sent status, when this automation runs, then a Comment is created with `target: Proposal`, the note text, Owen as author, and the current timestamp.

**FEAT-03.SPEC-004-AC-02:** Given the Comment write succeeds, when this automation completes, then it fires FEAT-03.SPEC-007 to notify Nadia and fires the FEAT-13 audit-trail entry for the change-request event.

**FEAT-03.SPEC-004-AC-03:** Given a note somehow reaches this automation exceeding 2,000 characters, when the write-time length re-check runs, then no Comment is created and the "invalid note" outcome is returned.

**FEAT-03.SPEC-004-AC-04:** Given Owen submits a note on a proposal that Nadia voids by editing and re-sending before this automation's write-time re-check runs, when the re-check finds `status: Voided`, then no Comment is recorded and FEAT-03.SPEC-002 directs Owen to the current version.

**FEAT-03.SPEC-004-AC-05:** Given Owen submits a note on a proposal that is accepted (by himself in another session, or by another Primary contact) before this automation's write-time re-check runs, when the re-check finds `status: Accepted`, then no Comment is recorded and FEAT-03.SPEC-002 shows that the proposal has already been accepted.

**FEAT-03.SPEC-004-AC-06:** Given Owen submits a note and connectivity drops before the write completes, when the failure occurs, then no Comment is created and FEAT-03.SPEC-002 shows a retry option with the note text preserved.

**FEAT-03.SPEC-004-AC-07:** Given Owen submits two change-request notes on the same proposal in quick succession, when both reach this automation, then each is recorded as its own independent Comment with no conflict, ordered by posted time.

**FEAT-03.SPEC-004-AC-08:** Given a change-request note is successfully recorded, when the analytics signal is emitted, then a proposal_changes_requested event carries the proposal reference and the note's character count.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 5 (recorded, invalid note, voided-redirect, already accepted, write failure) | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 (invalid-note-race, void-race, accept-race, double-submit, run-in-flight, audit-trigger-failure) | 6 |



# Logic/Rule Spec: Acceptance & Access Rules

## Overview

**Name:** Acceptance & Access Rules
**ID:** FEAT-03.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs who may accept a proposal or request changes on it, enforces single-acceptance and voided-proposal eligibility, defines the change-request note's length rule, and resolves concurrent-action conflicts.
**Parent Feature:** FEAT-03 -- Proposal Acceptance
**Governed Entity:** Proposal (acceptance and eligibility fields); the change-request note field on the Comment record this feature creates is governed here as a secondary field, since the Feature Breakdown Brief groups it with this spec's Validation & Limits.

## Scope and Non-Goals

**In Scope:**
- Eligibility rules for accepting a proposal (accept-once, voided-cannot-accept)
- Eligibility rules for submitting a change-request note (voided-cannot-request, accepted-cannot-request)
- The change-request note's length rule (1--2,000 characters)
- Authorization rules for view, accept, and request-changes actions on the Proposal, per role
- Concurrent-action resolution for the Proposal entity (reject-with-refresh) as it applies to acceptance and change requests

**Non-Goals:**
- Proposal creation, editing, and voiding rules -- owned entirely by FEAT-02 (Proposal Creation & Sending); this spec only reads the Proposal's current `status` to determine eligibility for this feature's actions.
- General Comment field rules (edit-within-grace-window, retraction, reply threading) -- owned by FEAT-07 (Deliverable Review & Feedback); this spec governs only the length rule for the change-request note at the moment it is created.
- Payment Schedule rules and the deposit-invoice trigger's own eligibility -- owned by FEAT-04 (Milestone & Payment Schedule Setup) and FEAT-09 (Invoice Generation & Sending) respectively; this spec's concern ends at the Proposal's acceptance eligibility.
- Legally binding e-signature eligibility -- deferred per scope-boundaries.md's Deferral note; FEAT-26 (v1) defines its own eligibility rules when signature is enabled for a proposal, extending rather than replacing this spec's MVP rules (XBR-34).

## Governed Entity

**Entity:** Proposal
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| status | enum (Draft, Sent, Voided, Accepted) | The proposal's current lifecycle state; this spec governs eligibility transitions out of Sent |
| sent_at | date/time | When the proposal was sent; no validation rule beyond data type in this spec |
| accepted_at | date/time | Written once on acceptance, never altered; this spec defines when it may be written |
| accepted_by | reference (Client Contact) | The accepting contact's identity; this spec defines who may cause this field to be set |
| payment_schedule_reference | reference (Payment Schedule) | Link to the project's Payment Schedule; no validation rule beyond data type in this spec -- read-only reference for FEAT-03.SPEC-003's deposit determination |

**Secondary governed field (Comment record created by this feature):**

| Field | Data Type | Description |
|-------|-----------|-------------|
| text (change-request variant) | text | The 1--2,000 character note Owen sends to Nadia instead of accepting; governed here per the Feature Breakdown Brief's Validation & Limits grouping |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|----------------------|
| FEAT-03.SPEC-001 | Proposal Review & Accept | Access and Visibility on screen entry; eligibility check on Accept tap (screen-level, non-authoritative) |
| FEAT-03.SPEC-002 | Request Changes | Access and Visibility on screen entry; note length check on Send tap and on change (screen-level, non-authoritative) |
| FEAT-03.SPEC-003 | Acceptance Recording | Authoritative write-time re-check of proposal eligibility (accept-once, voided-cannot-accept) before writing the acceptance |
| FEAT-03.SPEC-004 | Change-Request Recording | Authoritative write-time re-check of note length and proposal eligibility (voided-cannot-request, accepted-cannot-request) before writing the Comment |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|-------------------------------|---------------|---------------------|-----------|
| status | No validation beyond data type -- this spec reads it, FEAT-02 owns writing it (except the Accepted transition, governed below) | Always | -- | -- | -- |
| sent_at | No validation beyond data type | Always | -- | -- | -- |
| accepted_at | Must be set exactly once, only by the Acceptance Recording write (FEAT-03.SPEC-003); never altered afterward | Proposal transitions Sent -> Accepted | On write (FEAT-03.SPEC-003) | N/A -- system-set field, no user-facing error; a second attempted write is intercepted by the accept-once rule below, not a field-level message | Yes |
| accepted_by | Must reference a Client Contact with role Primary and status Active at the client owning the proposal | Proposal transitions Sent -> Accepted | On write (FEAT-03.SPEC-003) | N/A -- system-set field; the gating condition surfaces as the authorization denial below, not a field-level message | Yes |
| payment_schedule_reference | No validation beyond data type | Always | -- | -- | -- |
| text (change-request note) | Required, non-empty, 1--2,000 characters | Always | On Send tap (screen) and on write (FEAT-03.SPEC-004, authoritative) | "Enter a note before sending." (empty) / "Your note can be up to 2,000 characters." (over length) | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------------|
| Accept-once | status, accepted_at | Accept may only proceed when `status` is `Sent`; once `status` is `Accepted`, `accepted_at` is fixed and no further accept attempt may change it | "This proposal has already been accepted." |
| Voided-cannot-accept | status | Accept may only proceed when `status` is `Sent`, never when `status` is `Voided` | N/A -- the client is redirected to the current proposal version rather than shown an inline error (FEAT-03.SPEC-001) |
| Voided-cannot-request-changes | status | Request Changes may only proceed when `status` is `Sent`, never when `status` is `Voided` | N/A -- the client is directed to the current proposal version (FEAT-03.SPEC-002) |
| Accepted-cannot-request-changes | status | Request Changes may only proceed when `status` is `Sent`, never when `status` is `Accepted` | "This proposal has already been accepted." |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View proposal content (scope, price) | Owen (Client Primary Contact) | Own-only -- only his own company's proposal, and only when `status` is `Sent` or `Accepted` (XBR-08, XBR-09) | -- |
| View proposal content (scope, price) | Priya (Client Reviewer Contact) | Never | Full screen is not shown; portal home shows only the project's stage label (e.g., "Proposal accepted"), per the Access Matrix (Proposals & Acceptance: None for Reviewers) |
| View proposal content (scope, price) | Nadia (Freelancer) | Never through this feature's screens -- she sees the resulting status on the project view (FEAT-01), not FEAT-03.SPEC-001 | Attempting to open FEAT-03.SPEC-001's link is treated as an out-of-scope link (XBR-09): plain explanation and a fresh-link option |
| View proposal content (scope, price) | Dana (Support Operator) | Always, read-only, inside a logged support session (FEAT-31) | -- |
| Accept proposal | Owen (Client Primary Contact) | Own-only, and only when `status` is `Sent` (accept-once and voided-cannot-accept, above) | If `status` is `Accepted`: "This proposal has already been accepted." If `status` is `Voided`: redirected to the current version, no error text shown |
| Accept proposal | Priya (Client Reviewer Contact) | Never | Accept control is not shown -- she never reaches proposal content |
| Accept proposal | Nadia (Freelancer) | Never | No Accept control exists on any screen Nadia can reach; acceptance is exclusively a client-side action |
| Accept proposal | Dana (Support Operator) | Never | Accept control is not rendered inside a support session (FEAT-31) |
| Request changes | Owen (Client Primary Contact) | Own-only, and only when `status` is `Sent` (voided-cannot-request and accepted-cannot-request, above) | If `status` is `Voided`: directed to the current version. If `status` is `Accepted`: "This proposal has already been accepted." |
| Request changes | Priya (Client Reviewer Contact) | Never | Request Changes control is not shown -- she never reaches proposal content |
| Request changes | Nadia (Freelancer) | Never | No Request Changes control exists on any screen Nadia can reach |
| Request changes | Dana (Support Operator) | Never | Request Changes control is not rendered inside a support session (FEAT-31) |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------------------|---------------|---------------------|
| accepted_at | Derived as the current timestamp at the moment the Acceptance Recording write (FEAT-03.SPEC-003) succeeds | On the Sent -> Accepted transition, once | No |
| accepted_by | Derived as the Client Contact reference of whichever eligible Primary contact's accept attempt wins the exactly-once write (FEAT-03.SPEC-003) | On the Sent -> Accepted transition, once | No |

## Business Rules

- XBR-06: A voided proposal cannot be accepted; a project has at most one active proposal at a time, and a voided proposal directs the client to the current one.
- XBR-08: Only Primary contacts accept proposals and request changes; Reviewer contacts never see proposal or invoice content.
- XBR-09: Client isolation -- a contact reaches only their own company's proposal; an out-of-scope or expired link shows a plain explanation and a fresh-link option, never another company's data.
- Acceptance is recorded exactly once, per the dependency map's Proposal Contention note: two Primary contacts accepting at the same moment yield one acceptance, and the second sees "already accepted." This spec defines the eligibility rule that FEAT-03.SPEC-003 enforces atomically at write time; this spec does not itself perform the write.
- The Proposal Contention resolution for a race between Nadia's edit-and-void (FEAT-02) and Owen's accept or request-changes attempt (this feature) is reject-with-refresh: an accept or request-changes attempt against a version voided in the meantime is refused and the client is shown the current version.
- If the accept action fails mid-flight (e.g., a connectivity drop after tap), the action is retried without recording a duplicate acceptance or losing the client's click intent -- enforced within FEAT-03.SPEC-003's atomic write, per this spec's accept-once rule.
- Dana's support session sees this feature's screens read-only, per XBR-29: no Accept or Request Changes control is ever rendered for her, regardless of the proposal's state.

## Edge Cases

- **Owen accepts a proposal at the exact instant its `status` field is Sent, with `accepted_at` and `accepted_by` both unset** -- Standard eligible-accept path; no boundary issue since this is the normal Sent state.
- **A change-request note of exactly 2,000 characters is submitted** -- Passes validation; the limit is inclusive per the field rule above.
- **A change-request note of exactly 1 character is submitted** -- Passes validation; the minimum is inclusive.
- **A change-request note of 0 characters (empty string) is submitted** -- Fails validation with "Enter a note before sending." -- whitespace-only input is treated as empty for this purpose, since it carries no substantive note content.
- **Two Primary contacts at the same client both attempt Accept within the same instant** -- The accept-once rule is enforced atomically by FEAT-03.SPEC-003's write, not by this spec directly; this spec defines that exactly one may succeed and the other must see "This proposal has already been accepted."
- **A Primary contact's role is changed to Reviewer, or their status is set to Removed, while they are mid-session on FEAT-03.SPEC-001** -- The authorization condition (Own-only, Primary, Active) is re-checked at the moment of the Accept or Request Changes action, not only at screen load; if the role or status has changed, the action is denied with the same "not a Primary contact" experience as if they had never had access, consistent with XBR-27 (a removed contact's access ends immediately).
- **Owen attempts to accept a proposal for a project belonging to a different client than the one his contact record is scoped to** -- Denied per XBR-09: this is treated as an out-of-scope link, showing a plain explanation and a fresh-link option, never another company's proposal.
- **The proposal transitions from Sent to Voided between FEAT-03.SPEC-001's screen-level eligibility check and Owen's Accept tap reaching FEAT-03.SPEC-003** -- The screen-level check is non-authoritative; FEAT-03.SPEC-003's write-time re-check is what actually catches this and returns the voided-redirect outcome.

## Acceptance Criteria

**FEAT-03.SPEC-005-AC-01:** Given Owen is viewing a proposal with `status: Sent`, when he taps Accept, then the eligibility check passes and the accept proceeds to FEAT-03.SPEC-003.

**FEAT-03.SPEC-005-AC-02:** Given a proposal already has `status: Accepted`, when Owen (or any Primary contact) attempts to accept it again, then the attempt is denied with "This proposal has already been accepted."

**FEAT-03.SPEC-005-AC-03:** Given a proposal has `status: Voided`, when Owen attempts to accept it, then the attempt is denied and he is redirected to the current proposal version, with no error text shown.

**FEAT-03.SPEC-005-AC-04:** Given Owen submits a change-request note of exactly 2,000 characters, when the length rule is checked, then the note passes validation.

**FEAT-03.SPEC-005-AC-05:** Given Owen submits a change-request note of exactly 1 character, when the length rule is checked, then the note passes validation.

**FEAT-03.SPEC-005-AC-06:** Given Owen submits an empty change-request note, when the length rule is checked, then the note is denied with "Enter a note before sending."

**FEAT-03.SPEC-005-AC-07:** Given a proposal has `status: Voided`, when Owen attempts to submit a change-request note, then the attempt is denied and he is directed to the current proposal version.

**FEAT-03.SPEC-005-AC-08:** Given a proposal has `status: Accepted`, when Owen attempts to submit a change-request note, then the attempt is denied with "This proposal has already been accepted."

**FEAT-03.SPEC-005-AC-09:** Given Owen (Client Primary Contact) at the owning client, when he attempts to view the proposal, then he can view the full screen.

**FEAT-03.SPEC-005-AC-10:** Given Priya (Client Reviewer Contact), when she attempts to view the proposal, then no proposal content is shown and her portal home shows only the project's stage label.

**FEAT-03.SPEC-005-AC-11:** Given Nadia (Freelancer), when she attempts to open this feature's proposal review screen link, then the link is treated as out-of-scope and she sees a plain explanation with a fresh-link option, never proposal content through this feature's own screen.

**FEAT-03.SPEC-005-AC-12:** Given Dana (Support Operator) inside a logged support session, when she views the proposal, then she sees the full content read-only, with no Accept or Request Changes control rendered.

**FEAT-03.SPEC-005-AC-13:** Given two Primary contacts at the same client both attempt Accept at effectively the same moment, when the write-time eligibility check runs for each, then exactly one succeeds and the other is denied with "This proposal has already been accepted."

**FEAT-03.SPEC-005-AC-14:** Given a Primary contact's status is changed to Removed while they are mid-session on the proposal screen, when they attempt to accept, then the action is denied with the same experience as a contact who never had access.

**FEAT-03.SPEC-005-AC-15:** Given a proposal transitions from Sent to Voided between the screen's own eligibility check and the write-time re-check, when FEAT-03.SPEC-003 re-checks eligibility, then the voided-cannot-accept rule denies the write and the client is redirected to the current version.

**FEAT-03.SPEC-005-AC-16:** Given a contact attempts to reach a proposal belonging to a different client than their own, when the access check runs, then the attempt is denied as out-of-scope with a plain explanation and a fresh-link option, never another company's data.

**FEAT-03.SPEC-005-AC-17:** Given the acceptance write succeeds for one Primary contact's attempt, when `accepted_at` and `accepted_by` are derived, then they are set exactly once from that attempt's timestamp and contact reference, with no user override available.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 6 | 6 |
| Cross-Field Rules | 4 | 4 |
| Authorization Rules | 12 | 12 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 7 | 7 |
| Edge Cases | 8 | 8 |



# Notification Spec: Acceptance Confirmation Notification

## Overview

**Name:** Acceptance Confirmation Notification
**ID:** FEAT-03.SPEC-006
**Type:** Notification
**Purpose:** Emails Owen and Nadia a confirmation the instant a proposal's acceptance is recorded, giving both parties a durable, timestamped record that the project is now real and billable.
**Parent Feature:** FEAT-03 -- Proposal Acceptance

## Scope and Non-Goals

**In Scope:**
- The confirmation email sent to Owen (the accepting contact) and to Nadia when acceptance is recorded
- Content, delivery rules, and edge cases for this single notification

**Non-Goals:**
- The deposit invoice's own notification -- excluded per the Feature Breakdown Brief's Communications field: "the auto-generated deposit invoice sends its own notification (FEAT-09)"; this spec covers only the acceptance confirmation itself, never invoice content.
- An in-app notification channel -- product-features.md phases the In-App Notification Center (FEAT-29) as Later, outside MVP; at this feature's priority tier (Core, MVP) the only notification channel the product defines is transactional email (ASMP-29), so this spec uses no other channel.
- A preference to turn this notification off -- excluded per XBR-30: transactional emails core to the record (this is the evidentiary confirmation of a proposal's acceptance) always send and cannot be disabled; only optional notifications carry an on/off preference.
- Notifying Priya (Client Reviewer Contact) -- excluded per the Access Matrix (Proposals & Acceptance: None for Reviewers) and XBR-08: notification recipients are limited to contacts entitled to the event, and Priya has no entitlement to proposal content.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, to both Owen and Nadia, the instant acceptance is recorded | Owen's sessions are short and triggered by a specific email, not habitual browsing (user-persona.md, Client Primary Contact, Behavioral Context); Nadia is not necessarily inside the product at the moment a client accepts, and needs to know the project just became billable without watching a screen. Email is the product's sole notification channel at MVP (ASMP-29); no in-app channel exists yet (FEAT-29 is Later). |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|-------------------|
| Acceptance recorded | FEAT-03.SPEC-003 (Acceptance Recording) | Fires immediately once the acceptance write succeeds | Proposal reference, `accepted_at`, `accepted_by` (Owen's Client Contact reference), project name, client company name, whether a deposit invoice was triggered |

## Audience and Preferences

**Recipients:** Owen (Client Primary Contact, the accepting contact) and Nadia (Freelancer, the project owner) -- both are entitled to the acceptance event per the Access Matrix (Proposals & Acceptance: Full for Nadia, Own-only accept/view for Owen) and per the Feature Breakdown Brief's Communications field ("Confirmation email to Owen and Nadia when acceptance is recorded").

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|----------------------|
| N/A | N/A | Always on -- transactional | N/A -- per XBR-30, this is a transactional email core to the evidentiary record and cannot be disabled by either recipient |

**Quiet Hours:** N/A -- the product defines no quiet-hours window for transactional confirmation emails; this notification is the timestamped record of a moment that just happened for both parties, and holding it would misrepresent when the record was actually confirmed.

## Content Definition

**Email (to Owen):**
- **Subject:** You've accepted the proposal for {project_name}
- **Body:**
  Hi {owen_first_name},

  You accepted the proposal for {project_name} on {accepted_at_date}. This confirms the record on file with {freelancer_business_name}.

  {deposit_invoice_line}
- **CTA (button):** View proposal -- deep-links to FEAT-03.SPEC-001 (Proposal Review & Accept) for this proposal, now showing "Accepted on {accepted_at_date}"

**Email (to Nadia):**
- **Subject:** {client_company_name} accepted the proposal for {project_name}
- **Body:**
  Hi {nadia_first_name},

  {owen_full_name} at {client_company_name} accepted the proposal for {project_name} on {accepted_at_date}.

  {deposit_invoice_line_nadia}
- **CTA (button):** View project -- deep-links to FEAT-01 (Client & Project Management), project view, for this project

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|--------------------------|-----------------|--------------------------|
| {project_name} | Project -- project_name | Brand Refresh Q1 | Never empty -- required at project creation (FEAT-01) |
| {owen_first_name} | Client Contact -- name (first token) | Owen | Greeting renders as "Hi," |
| {owen_full_name} | Client Contact -- name | Owen Carter | "the client's Primary contact" |
| {nadia_first_name} | Freelancer Account -- name (first token) | Nadia | Greeting renders as "Hi," |
| {client_company_name} | Client -- client_name | Carter & Co | Never empty -- required at client creation (FEAT-01) |
| {accepted_at_date} | Proposal -- accepted_at (formatted in the recipient's own time zone, FEAT-15) | March 4, 2026 | Never empty -- `accepted_at` is written by FEAT-03.SPEC-003 before this notification fires |
| {freelancer_business_name} | Freelancer Account -- business_name | Nadia Voss Design | Falls back to Nadia's account name if business_name is not yet set |
| {deposit_invoice_line} | Derived -- whether FEAT-03.SPEC-003 triggered a deposit invoice | "A deposit invoice has been sent to you separately." | "No deposit is due under this project's payment schedule." |
| {deposit_invoice_line_nadia} | Derived -- whether FEAT-03.SPEC-003 triggered a deposit invoice | "A deposit invoice has been generated and sent automatically." | "This project's payment schedule has no deposit, so no invoice was generated yet." |

## Delivery Rules

**Batching:** None -- each acceptance is a single, discrete evidentiary event; it is never combined with any other notification, even if Owen or Nadia has other pending emails.
**Deduplication:** At most one confirmation email per recipient per acceptance. Because FEAT-03.SPEC-003 writes the acceptance exactly once (accept-once, enforced atomically), this notification's trigger fires exactly once per Proposal; a retried Accept attempt that returns "already accepted" never re-fires this notification.
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window` per FEAT-14's transactional email delivery capability (ASMP-29). After the final failure, the failure is surfaced to Nadia as a delivery warning on the affected project (XBR-30); the "Accepted on {date}" marker on FEAT-03.SPEC-001 stands as the enduring in-product record regardless of email delivery outcome.
**Expiry:** This notification never expires undelivered in the sense of being withdrawn -- it is retried per the rule above, and if all retries fail, the delivery warning (XBR-30) is the surviving signal rather than a silently dropped message, since the underlying acceptance record itself never disappears.

## Edge Cases

- **Owen's email address bounces** -- The failure is retried per the Retry on failure rule; after the final failure, Nadia sees a delivery warning on the project (XBR-30) and is advised to correct Owen's contact email (FEAT-18). The acceptance record itself is unaffected -- it does not depend on this email's delivery.
- **The deposit invoice trigger fails after the acceptance write succeeds (FEAT-03.SPEC-003's own failure path)** -- This notification still fires with the "no deposit due" fallback line only if the Payment Schedule genuinely has no deposit; if a deposit was due but the invoice trigger failed, the {deposit_invoice_line} placeholder still reflects that a deposit invoice was triggered (this notification reports what FEAT-03.SPEC-003 attempted, not whether FEAT-09's own send later succeeds) -- FEAT-09 owns communicating its own delivery outcome.
- **Owen's Client Contact record is removed (access revoked) between acceptance and this notification's delivery** -- The notification still delivers to the email address captured in `accepted_by` at the moment of acceptance, since that identity is preserved as evidence even after removal (XBR-27); the notification's content is a record of what happened, not a live view of current access.
- **Nadia's account has no `business_name` set yet (pre-FEAT-21 configuration)** -- The {freelancer_business_name} placeholder falls back to her account name, so the email to Owen never renders an empty business name.
- **Two acceptance confirmation attempts race due to a transient duplicate trigger fire** -- Deduplication holds: since FEAT-03.SPEC-003 only ever writes the acceptance once and only fires this notification's trigger once per successful write, no second instance of this notification is ever queued for the same acceptance.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|-------------------|----------------|
| FEAT-03.SPEC-003 (Acceptance Recording) | Triggered by (inbound) | Fires this notification immediately once the acceptance write succeeds |
| FEAT-03.SPEC-001 (Proposal Review & Accept) | Navigation (outbound) | Owen's CTA deep-links back to the now-Accepted proposal screen |
| FEAT-01 (Client & Project Management) | Navigation (outbound) | Nadia's CTA deep-links to the project view |
| FEAT-14 (Notifications (Email)) | References (outbound) | Owns the transactional email delivery capability and delivery/bounce status reporting this notification relies on |
| FEAT-09 (Invoice Generation & Sending) | References (outbound) | The {deposit_invoice_line} placeholder reflects whether FEAT-03.SPEC-003 triggered FEAT-09's deposit invoice; FEAT-09 sends its own separate notification |

## Analytics and Success Signals

- **acceptance_confirmation_sent** (recipient: owen / nadia, deposit invoice triggered: yes / no) -- supports success-metrics.md: "Time to Proposal Acceptance"
- **acceptance_confirmation_delivery_failed** (recipient: owen / nadia, retries exhausted: yes / no) -- N/A -- no Stage 2 metric measures delivery failures directly; retained so a silently undelivered evidentiary confirmation is observable via the project's delivery warning (XBR-30) rather than invisible

## Acceptance Criteria

**FEAT-03.SPEC-006-AC-01:** Given Owen accepts a proposal with no deposit due, when FEAT-03.SPEC-003 records the acceptance, then Owen and Nadia each receive an email confirmation, and both bodies include "No deposit is due under this project's payment schedule" (Owen's variant) / "no invoice was generated yet" (Nadia's variant).

**FEAT-03.SPEC-006-AC-02:** Given Owen accepts a proposal whose Payment Schedule includes a deposit, when FEAT-03.SPEC-003 records the acceptance and triggers the deposit invoice, then Owen's confirmation email states "A deposit invoice has been sent to you separately" and Nadia's states "A deposit invoice has been generated and sent automatically."

**FEAT-03.SPEC-006-AC-03:** Given Owen receives the confirmation email, when he taps "View proposal", then he lands on FEAT-03.SPEC-001 showing "Accepted on {accepted_at_date}".

**FEAT-03.SPEC-006-AC-04:** Given Nadia receives the confirmation email, when she taps "View project", then she lands on the project view in FEAT-01.

**FEAT-03.SPEC-006-AC-05:** Given the acceptance is recorded, when this notification's trigger fires, then no on/off preference is available to either recipient to suppress it -- it always sends.

**FEAT-03.SPEC-006-AC-06:** Given Owen's email address bounces on the first delivery attempt, when the delivery capability retries, then up to platform parameter: `transactional-email-retry-count` retries occur over platform parameter: `transactional-email-retry-window` before Nadia sees a delivery warning on the project.

**FEAT-03.SPEC-006-AC-07:** Given all retries for Owen's email are exhausted, when the final failure occurs, then Nadia sees a delivery warning on the affected project and the "Accepted on {date}" marker remains the enduring in-product record regardless.

**FEAT-03.SPEC-006-AC-08:** Given the acceptance is recorded exactly once (per FEAT-03.SPEC-003's accept-once guarantee), when a second, redundant Accept attempt returns "already accepted", then this notification's trigger does not fire a second time.

**FEAT-03.SPEC-006-AC-09:** Given Owen's Client Contact record is later removed, when this notification is still pending delivery, then it still delivers to the email address captured in `accepted_by` at the moment of acceptance.

**FEAT-03.SPEC-006-AC-10:** Given the acceptance confirmation is successfully delivered to both recipients, when the analytics signal is emitted, then an acceptance_confirmation_sent event is recorded once per recipient with the deposit-invoice-triggered flag.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on -- transactional) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |



# Notification Spec: Change-Request Notification

## Overview

**Name:** Change-Request Notification
**ID:** FEAT-03.SPEC-007
**Type:** Notification
**Purpose:** Emails Nadia immediately when Owen submits a change-request note, so she can revise and re-send the proposal without a separate status check.
**Parent Feature:** FEAT-03 -- Proposal Acceptance

## Scope and Non-Goals

**In Scope:**
- The email sent to Nadia the instant a change-request note is recorded
- Content, delivery rules, and edge cases for this single notification

**Non-Goals:**
- Notifying Owen back with any confirmation beyond the in-screen feedback already defined in FEAT-03.SPEC-002 -- the Feature Breakdown Brief's Side-Effect Inventory lists Owen's confirmation as "an on-screen confirmation state (no separate delivery rules), inline in triggering screen," not a standalone notification.
- An in-app notification channel -- product-features.md phases the In-App Notification Center (FEAT-29) as Later, outside MVP; this notification uses the product's sole MVP channel, transactional email (ASMP-29).
- A preference to turn this notification off -- excluded per XBR-30: this is the freelancer's signal that a client is waiting on her, core to the product's promise of removing the "email back-and-forth" (BRIEF.md, Problem Statement); it is not an optional marketing-style email a freelancer would reasonably disable.
- Carrying the note's full text as a distinct, separately-consumable object -- the note is quoted inline in the email body as the change-request content itself; there is no separate record-viewing screen for it within this feature (Feature Breakdown Brief, Entity-Lifecycle Coverage Matrix: no screen re-displays a past change-request note within FEAT-03).

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, to Nadia, the instant the change-request note is recorded | Nadia's Behavioral Context (user-persona.md) has her checking Clientroom to see whether a client has approved or responded, not watching a live feed; a change request is exactly the kind of moment BRIEF.md's Problem Statement says used to arrive as a disconnected WhatsApp screenshot -- email delivers it into the same inbox she already monitors for client business, immediately. Email is the product's sole notification channel at MVP (ASMP-29). |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|-------------------|
| Change-request note recorded | FEAT-03.SPEC-004 (Change-Request Recording) | Fires immediately once the Comment write succeeds | Proposal reference, project name, client company name, Owen's identity, the note text, `posted_at` |

## Audience and Preferences

**Recipients:** Nadia (Freelancer) -- the sole recipient, per the Access Matrix (Client Contact Management: Full for Nadia) and the Feature Breakdown Brief's Communications field ("A request-changes note emails Nadia immediately"). Owen and Priya are never recipients of this notification: Owen already sees his own submission confirmed on-screen (FEAT-03.SPEC-002), and Priya has no entitlement to proposal content (Access Matrix, XBR-08).

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|----------------------|
| N/A | N/A | Always on -- transactional | N/A -- per XBR-30, this is a transactional email core to the product's record-and-respond loop and cannot be disabled |

**Quiet Hours:** N/A -- the product defines no quiet-hours window for this notification; a change request is time-sensitive to Nadia's own workflow (she decides when to revise), and holding it would delay her seeing that a client is waiting on a response, contradicting the immediacy the Feature Breakdown Brief specifies ("emails Nadia immediately").

## Content Definition

**Email (to Nadia):**
- **Subject:** {client_company_name} requested changes to the proposal for {project_name}
- **Body:**
  Hi {nadia_first_name},

  {owen_full_name} at {client_company_name} sent a note about the proposal for {project_name} instead of accepting it:

  "{note_text}"

  The proposal itself has not been changed. Open it to revise and re-send.
- **CTA (button):** Open proposal -- deep-links to FEAT-02 (Proposal Creation & Sending), proposal detail, for this proposal, where the note is visible for Nadia to act on

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|--------------------------|-----------------|--------------------------|
| {client_company_name} | Client -- client_name | Carter & Co | Never empty -- required at client creation (FEAT-01) |
| {project_name} | Project -- project_name | Brand Refresh Q1 | Never empty -- required at project creation (FEAT-01) |
| {nadia_first_name} | Freelancer Account -- name (first token) | Nadia | Greeting renders as "Hi," |
| {owen_full_name} | Client Contact -- name | Owen Carter | "The client's Primary contact" |
| {note_text} | Comment -- text (change-request variant, 1--2,000 characters) | "Could we swap the second concept for a lighter palette?" | Never empty -- FEAT-03.SPEC-004 rejects a note with zero characters before this notification's trigger can fire |

## Delivery Rules

**Batching:** None -- each change-request note is a single, discrete moment a freelancer needs to know about promptly; it is never combined with other notifications, even if several arrive close together for different proposals.
**Deduplication:** At most one notification per recorded Comment. FEAT-03.SPEC-004 creates one Comment per submitted note and fires this notification's trigger exactly once per successful write; a submission rejected for invalid length, a voided proposal, or an already-accepted proposal never reaches the trigger.
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window` per FEAT-14's transactional email delivery capability (ASMP-29). After the final failure, the failure is surfaced to Nadia as a delivery warning on the affected project (XBR-30); the Comment itself remains visible to Nadia when she next opens the proposal in FEAT-02, regardless of this email's delivery outcome.
**Expiry:** This notification is never withdrawn once triggered -- delivery is retried per the rule above, and the underlying Comment record persists indefinitely as the change-request evidence even if every retry fails.

## Edge Cases

- **Nadia's email address bounces** -- The failure is retried per the Retry on failure rule; after the final failure, Nadia would not see the delivery warning by email (since her own email is the one failing), but the warning still appears on the affected project inside the product the next time she opens it (XBR-30), and the Comment remains visible there regardless.
- **Owen submits a second change-request note on the same proposal shortly after the first, before Nadia has acted** -- Each note produces its own Comment (dependency map, Comment Contention: append-only, no resolution needed beyond ordering by posted time) and its own separate notification instance; the two are never merged into one email, since each is evidence of a distinct moment.
- **Nadia is already viewing the proposal in FEAT-02 when the note is recorded** -- The notification still sends; this feature defines no live-updating suppression rule, since the email is the evidentiary trail of the moment the note arrived, not merely a live-view convenience.
- **The proposal is accepted by Owen in a separate session moments after he also tried to submit a change-request note (a race already resolved by FEAT-03.SPEC-005 in favor of one outcome)** -- If the change-request write itself succeeded before the accept won the race, this notification still fires normally for that recorded note; if the change-request write was rejected because the proposal was already Accepted (per FEAT-03.SPEC-004's outcome), no Comment was created and this notification's trigger never fires.
- **Nadia's account has multiple client contacts named similarly at the same client** -- {owen_full_name} always renders the specific Client Contact's name captured as the note's `author`, never a generic "a contact," so Nadia always knows exactly who sent the note.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|-------------------|----------------|
| FEAT-03.SPEC-004 (Change-Request Recording) | Triggered by (inbound) | Fires this notification immediately once the Comment write succeeds |
| FEAT-02 (Proposal Creation & Sending) | Navigation (outbound) | Nadia's CTA deep-links to the proposal detail, where she revises and re-sends |
| FEAT-14 (Notifications (Email)) | References (outbound) | Owns the transactional email delivery capability and delivery/bounce status reporting this notification relies on |

## Analytics and Success Signals

- **change_request_notification_sent** (recipient: nadia, note character count) -- N/A -- no success metric in success-metrics.md is connected to the change-request path; "Time to Proposal Acceptance" measures only the accept path, and this feature has no other connected metric to attribute this signal to
- **change_request_notification_delivery_failed** (retries exhausted: yes / no) -- N/A -- no Stage 2 metric measures delivery failures directly; retained so a silently undelivered change-request alert is observable via the project's delivery warning (XBR-30) rather than invisible

## Acceptance Criteria

**FEAT-03.SPEC-007-AC-01:** Given Owen submits a valid change-request note, when FEAT-03.SPEC-004 records it, then Nadia receives an email with the subject naming the client company and project, quoting the note text in the body.

**FEAT-03.SPEC-007-AC-02:** Given Nadia receives the change-request email, when she taps "Open proposal", then she lands on the proposal detail in FEAT-02 with the note visible for her to act on.

**FEAT-03.SPEC-007-AC-03:** Given the change-request note is recorded, when this notification's trigger fires, then no on/off preference is available to Nadia to suppress it -- it always sends.

**FEAT-03.SPEC-007-AC-04:** Given the change-request note is recorded during Nadia's local nighttime hours, when this notification's trigger fires, then it sends immediately with no quiet-hours hold.

**FEAT-03.SPEC-007-AC-05:** Given Owen's change-request submission is rejected because the proposal was already Accepted (FEAT-03.SPEC-004's outcome), when no Comment is created, then this notification's trigger never fires.

**FEAT-03.SPEC-007-AC-06:** Given Nadia's email address bounces on the first delivery attempt, when the delivery capability retries, then up to platform parameter: `transactional-email-retry-count` retries occur over platform parameter: `transactional-email-retry-window` before a delivery warning appears on the affected project.

**FEAT-03.SPEC-007-AC-07:** Given all retries for Nadia's email are exhausted, when the final failure occurs, then a delivery warning appears on the affected project and the Comment remains visible to Nadia inside the product regardless.

**FEAT-03.SPEC-007-AC-08:** Given Owen submits two change-request notes on the same proposal in succession, when each is recorded, then Nadia receives two separate emails, one per note, never merged into one.

**FEAT-03.SPEC-007-AC-09:** Given the change-request notification is successfully delivered, when the analytics signal is emitted, then a change_request_notification_sent event is recorded with the note's character count.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on -- transactional) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |

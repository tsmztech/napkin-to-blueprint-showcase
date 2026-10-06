# FEAT-02 — Proposal Creation & Sending

This chapter covers Proposal Creation & Sending, a Core-tier feature. It contains the feature breakdown brief followed by every specification in full: 11 specifications carrying 116 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-02.SPEC-001 | Proposal Draft Editor | screen | 12 |
| FEAT-02.SPEC-002 | Proposal Preview | screen | 10 |
| FEAT-02.SPEC-003 | Proposal Detail | screen | 14 |
| FEAT-02.SPEC-004 | Reuse Proposal Picker | screen | 9 |
| FEAT-02.SPEC-005 | Proposal Send | automation | 9 |
| FEAT-02.SPEC-006 | Proposal Edit-Before-Acceptance Void & Resend | automation | 9 |
| FEAT-02.SPEC-007 | Proposal Resend | automation | 7 |
| FEAT-02.SPEC-008 | Create Draft From Copy | automation | 7 |
| FEAT-02.SPEC-009 | Discard Draft | automation | 6 |
| FEAT-02.SPEC-010 | Proposal Validation & Business Rules | logic-rule | 22 |
| FEAT-02.SPEC-011 | Proposal Sent/Resent Email | notification | 11 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Proposal Creation & Sending

## Summary

**Feature:** Proposal Creation & Sending
**ID:** FEAT-02
**Description:** The freelancer drafts a proposal with scope and price for a project and sends it to the client's Primary Contact as a link, replacing the "PDF in email" workflow described in the brief.
**Priority:** Core
**Phase:** MVP
**Type:** User-Facing
**Rationale:** BRIEF.md's Experience narrative opens with "You send a proposal to a new client from Clientroom" — this is the product's entry point into every project. MVP phase: the core loop cannot start without it. Proposals bundled with invoicing appear in all five profiled competitors, confirming this as table stakes; heavy template and workflow configuration is the category's top complaint, so this feature starts a proposal from a plain form or a copy of an earlier one rather than a template builder.

**Key Capabilities:**
- Draft a proposal — freelancer writes scope and sets a price for the project
- Send the proposal — client's Primary Contact receives a link by email
- Edit before acceptance — freelancer can revise a sent-but-unaccepted proposal
- Reuse an earlier proposal — start a new draft from a copy of any previous proposal
- Discard a draft — delete a proposal that was never sent

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-02.SPEC-001 | Proposal Draft Editor | Screen | Nadia | Freelancer writes or revises scope, price, and currency for a project's proposal |
| FEAT-02.SPEC-002 | Proposal Preview | Screen | Nadia | Freelancer reviews a branded, client-facing preview of the draft before sending |
| FEAT-02.SPEC-003 | Proposal Detail | Screen | Nadia, Dana | Freelancer (and, read-only, the support operator) views the current proposal's status, history, and available actions for a project |
| FEAT-02.SPEC-004 | Reuse Proposal Picker | Screen | Nadia | Freelancer browses and selects an earlier proposal from any client/project to start a new draft as a copy |
| FEAT-02.SPEC-005 | Proposal Send | Automation | Nadia, Owen | Validates and transitions a Draft proposal to Sent, recording the send timestamp and locking the payment schedule reference |
| FEAT-02.SPEC-006 | Proposal Edit-Before-Acceptance Void & Resend | Automation | Nadia, Owen | Editing a Sent-but-unaccepted proposal voids the prior version and creates and sends the new one (XBR-06) |
| FEAT-02.SPEC-007 | Proposal Resend | Automation | Nadia, Owen | Re-sends the link for an already-Sent proposal without creating a new version |
| FEAT-02.SPEC-008 | Create Draft From Copy | Automation | Nadia | Creates a new Draft proposal pre-filled from a selected earlier proposal's scope, price, and currency |
| FEAT-02.SPEC-009 | Discard Draft | Automation | Nadia | Permanently removes an unsent Draft proposal |
| FEAT-02.SPEC-010 | Proposal Validation & Business Rules | Logic/Rule | Nadia, Owen | Governs required fields, price positivity/currency, one-active-proposal-per-project, send eligibility (XBR-07), immutability after send/accept (XBR-04), and the accept-vs-void contention resolution |
| FEAT-02.SPEC-011 | Proposal Sent/Resent Email | Notification | Nadia, Owen | Emails the client's Primary Contact the proposal link whenever a proposal is sent, resent, or re-sent after an edit |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Draft a proposal | FEAT-02.SPEC-001 | Primary purpose of the draft editor screen | Phase 2 (Explicit) |
| Send the proposal | FEAT-02.SPEC-005, FEAT-02.SPEC-011 | Send automation transitions the proposal to Sent; Notification spec delivers the link email | Phase 2 (Explicit) / Phase 4 (Notification Surfacing) |
| Edit before acceptance | FEAT-02.SPEC-001, FEAT-02.SPEC-006 | Draft editor reused in edit mode for a Sent-but-unaccepted proposal; void-and-resend automation implements XBR-06 | Phase 2 (Explicit) / Phase 4 (Trigger-Response) |
| Reuse an earlier proposal | FEAT-02.SPEC-004, FEAT-02.SPEC-008 | Picker screen selects a source proposal; automation copies its fields into a new draft | Phase 2 (Explicit) |
| Discard a draft | FEAT-02.SPEC-009 | Automation hard-deletes an unsent draft, bounded by the immutability rule for sent/voided proposals | Phase 2 (Explicit) / Phase 3 (Entity-Lifecycle) |

**Analyst-Discovered Specs** — Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-02.SPEC-002 | Proposal Preview | Phase 2 (Explicit, elaborating Data Notes) | The feature's Data Notes field states "Displayed: a proposal preview before sending" — an explicit depth-field line with no home in the draft/send capabilities alone |
| FEAT-02.SPEC-003 | Proposal Detail | Phase 3 (Entity-Lifecycle: Read) / Phase 6 (Empty State) | The Proposal entity needs a single-record read surface distinct from the editor (viewing a Sent/Voided/Accepted proposal is not "drafting"); the feature's States field requires an explicit empty-state prompt when a project has no proposal, and navigation connections from FEAT-01, FEAT-12, FEAT-20, and FEAT-28 all land on a proposal view rather than the editor |
| FEAT-02.SPEC-007 | Proposal Resend | Phase 4 (Trigger-Response) | The Primary Flows & Alternates field names a "Resend" flow distinct from edit-and-resend (Owen simply cannot find the original email); this is a separate trigger-response pair from XBR-06's void-and-resend |
| FEAT-02.SPEC-010 | Proposal Validation & Business Rules | Phase 5 (Rule-Constraint Discovery) | More than five validation/authorization rules apply across the Proposal entity (required fields, positive price in project currency, one-active-proposal cap, send-gated-on-Primary-contact, post-send/post-accept immutability, accept-vs-void contention) — past the inline-validation threshold, and several are shared across SPEC-001, SPEC-005, and SPEC-006 |
| FEAT-02.SPEC-011 | Proposal Sent/Resent Email | Phase 4 (Notification Surfacing) | The Communications field names an email with a channel, an audience (Primary Contact), and delivery behavior, which the methodology requires to be a standalone Notification spec rather than an inline confirmation |

**External Dependencies note:** This feature's Dependencies slice names the Transactional email delivery capability (ASMP-29). That capability's Integration spec is owned by FEAT-14 (per the dependency map's External Touchpoints row); FEAT-02.SPEC-011 is the Notification spec that calls into it and is affected by its delivery-status and degraded-delivery behavior. This feature therefore carries `integration_count: 0` by design, not by omission.

## Entity-Lifecycle Coverage Matrix

**Entity: Proposal**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-02.SPEC-001, FEAT-02.SPEC-008 | Draft editor creates a blank Draft; Create-Draft-From-Copy creates a Draft pre-filled from a chosen earlier proposal | -- |
| Read (single) | FEAT-02.SPEC-001, FEAT-02.SPEC-002, FEAT-02.SPEC-003 | Editor loads the current draft for editing; Preview renders it read-only; Detail shows its current status | -- |
| Read (list) | FEAT-02.SPEC-004 | Reuse Picker lists the freelancer's earlier proposals across all clients/projects as copy sources | A project itself never lists multiple proposals -- at most one active proposal exists at a time (Validation & Limits) |
| Update | FEAT-02.SPEC-001, FEAT-02.SPEC-006 | Editor captures the revised scope/price; Void & Resend automation applies the edit as a new version | FEAT-03 also updates the entity (accepted state), outside this feature |
| Delete/Archive | FEAT-02.SPEC-009 | Hard delete of unsent Draft rows only: no soft-delete or restore path (a discarded draft was never sent and carries no evidentiary value), no cascade (nothing else references an unsent draft), no retention/purge window applies once removed. Sent, Voided, and Accepted proposals are excluded from deletion entirely -- they are evidentiary (XBR-04) and retained for the life of the account, removed only by FEAT-24 account deletion | -- |
| State Transition | FEAT-02.SPEC-005, FEAT-02.SPEC-006 | Draft -> Sent (Send); Sent -> Voided with a new Sent version created (Void & Resend, XBR-06); Sent -> Accepted is owned by FEAT-03; Voided is terminal -- a voided proposal cannot be accepted | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Project | FEAT-02.SPEC-001, FEAT-02.SPEC-003, FEAT-02.SPEC-005 | Owning project supplies currency (FEAT-15) and identifies the at-most-one-active-proposal scope |
| Client | FEAT-02.SPEC-005 | Send validates that the owning client has a Primary contact |
| Client Contact | FEAT-02.SPEC-005, FEAT-02.SPEC-011 | Send/resend targets the client's Primary Contact(s); blocked if none exists (XBR-07) |
| Payment Schedule | FEAT-02.SPEC-001, FEAT-02.SPEC-002 | Draft editor and preview show the project's Payment Schedule reference |
| Branding Profile | FEAT-02.SPEC-002, FEAT-02.SPEC-011 | Preview and email apply the freelancer's logo/colour (XBR-31) |
| Comment | FEAT-02.SPEC-003 | Detail screen surfaces a request-changes note (FEAT-03) attached to the proposal |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Freelancer saves a proposal draft | Persist scope/price/currency as a Draft; no notification | Inline in triggering screen | FEAT-02.SPEC-001 |
| Freelancer taps Send | Validate required fields, positive price, and that the client has a Primary contact | Standalone Logic/Rule | FEAT-02.SPEC-010 |
| Send validation passes | Transition Draft -> Sent, set sent_at, lock the payment schedule reference | Standalone Automation | FEAT-02.SPEC-005 |
| Proposal is sent, resent, or void-and-resent | Email the Primary Contact the proposal link | Standalone Notification | FEAT-02.SPEC-011 |
| Freelancer edits a Sent-but-unaccepted proposal | Void the prior version, create and send the new version (XBR-06) | Standalone Automation | FEAT-02.SPEC-006 |
| Freelancer taps Resend (no content change) | Re-send the link for the current Sent version | Standalone Automation | FEAT-02.SPEC-007 |
| Owen accepts a version that was voided in the meantime | Reject the acceptance; show Owen the current version | Cross-feature | FEAT-03 responsibility (contention rule, dependency map) |
| Freelancer starts an edit after the proposal was already accepted | Refuse the edit -- accepted proposals are immutable (XBR-04) | Standalone Logic/Rule | FEAT-02.SPEC-010 |
| Freelancer selects "start from a copy" | Copy scope, price, and currency from the chosen source proposal into a new Draft, recording copied_from | Standalone Automation | FEAT-02.SPEC-008 |
| Freelancer discards an unsent draft | Hard-delete the Draft record | Standalone Automation | FEAT-02.SPEC-009 |
| Freelancer submits a draft with missing scope, missing price, or a non-positive price | Show per-field validation errors; nothing is saved as Sent | Standalone Logic/Rule (referenced inline by the editor) | FEAT-02.SPEC-010 |
| Send fails (network/server error) | Preserve the draft's content unchanged and offer a retry | Inline in triggering screen | FEAT-02.SPEC-001 |
| Nadia opens a change-request email (FEAT-03) | Navigate to the proposal detail with the request-changes note shown | Inline in triggering screen | FEAT-02.SPEC-003 |

## Shared Context

**Shared Entities:**
- Proposal record -- created by SPEC-001 and SPEC-008, read by SPEC-001/SPEC-002/SPEC-003, listed (as copy sources) by SPEC-004, updated by SPEC-001/SPEC-006, transitioned by SPEC-005/SPEC-006, deleted by SPEC-009, validated by SPEC-010. Fields: scope_description, price, currency, payment_schedule_reference, status (Draft/Sent/Voided/Accepted), sent_at, copied_from, accepted_at/accepted_by (written by FEAT-03), signature record (FEAT-26, v1).

**Shared UI Patterns:**
- Proposal form -- SPEC-001 uses the same scope/price/currency fields whether starting blank, starting from SPEC-008's copy, or editing a Sent-but-unaccepted version; only the entry path and the post-save consequence (create-as-Draft vs. void-and-resend) differ. Spec Writers should describe the form once and reference it from both flows.
- Branded client-facing rendering -- SPEC-002 (preview) and SPEC-011 (email) both render the proposal's content through the freelancer's Branding Profile (XBR-31) and should describe that rendering consistently.
- Status-driven action set -- SPEC-003 shows a different action set per proposal status (Draft: Edit, Send, Discard, Start from Copy; Sent: Edit, Resend; Voided/Accepted: view only), all governed by SPEC-010.

**Shared Validation:**
- SPEC-010 defines all Proposal validation, authorization, and immutability rules. SPEC-001, SPEC-005, SPEC-006, and SPEC-009 all reference SPEC-010 rather than duplicating its rules (required fields and positive price on save; Primary-contact gate and one-active-proposal cap on send; post-send/post-accept immutability on edit and delete attempts).

## Internal Dependency Map

```
SPEC-003 (Proposal Detail) -> [no proposal exists for the project] -> SPEC-001 (Proposal Draft Editor, empty-state prompt)
SPEC-003 (Proposal Detail) -> [freelancer taps Edit] -> SPEC-001 (Proposal Draft Editor)
SPEC-003 (Proposal Detail) -> [freelancer taps "Start from a copy"] -> SPEC-004 (Reuse Proposal Picker)
SPEC-004 (Reuse Proposal Picker) -> [freelancer selects a source proposal] -> SPEC-008 (Create Draft From Copy) -> SPEC-001 (Proposal Draft Editor)
SPEC-001 (Proposal Draft Editor) -> [freelancer taps Preview] -> SPEC-002 (Proposal Preview)
SPEC-001 (Proposal Draft Editor) -> [validates fields using] -> SPEC-010 (Proposal Validation & Business Rules)
SPEC-002 (Proposal Preview) -> [freelancer taps Send] -> SPEC-010 (validation) -> [pass] -> SPEC-005 (Proposal Send)
SPEC-005 (Proposal Send) -> [send succeeds] -> SPEC-011 (Proposal Sent/Resent Email)
SPEC-005 (Proposal Send) -> [send succeeds] -> SPEC-003 (Proposal Detail, Sent state)
SPEC-003 (Proposal Detail) -> [freelancer taps Resend] -> SPEC-007 (Proposal Resend) -> SPEC-011 (Proposal Sent/Resent Email)
SPEC-003 (Proposal Detail) -> [freelancer edits a Sent-but-unaccepted proposal] -> SPEC-001 (Proposal Draft Editor, edit mode)
SPEC-001 (Proposal Draft Editor, edit mode) -> [freelancer taps Save & Resend] -> SPEC-010 (validation) -> [pass] -> SPEC-006 (Void & Resend) -> SPEC-011 (Proposal Sent/Resent Email)
SPEC-006 (Void & Resend) -> [prior version voided, new version sent] -> SPEC-003 (Proposal Detail, updated)
SPEC-001 (Proposal Draft Editor) -> [freelancer taps Discard Draft] -> SPEC-010 (validation: Draft-only check) -> [pass] -> SPEC-009 (Discard Draft) -> SPEC-003 (Proposal Detail, empty state)
```

**Default Entry:** SPEC-003 (Proposal Detail) -- the screen shown when Nadia opens the proposal area of a project.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-02.SPEC-001 | Inbound | FEAT-20 (Onboarding / First-Run Setup) | Guided onboarding step "Draft the first proposal" opens the draft editor for the new project | Onboarding reaches the proposal step |
| FEAT-02.SPEC-001 | Inbound | FEAT-01 (Client & Project Management) | Nadia drafts, edits, or opens the project's proposal from the project view | Nadia opens the proposal area from a project |
| FEAT-02.SPEC-003 | Inbound | FEAT-12 (Dashboard) | Empty-dashboard zero-state prompts toward sending a first proposal | Nadia selects the zero-state CTA |
| FEAT-02.SPEC-003 | Inbound | FEAT-28 (Search, v1) | A search result for a proposal opens its detail screen | Nadia selects a proposal search result |
| FEAT-02.SPEC-005, FEAT-02.SPEC-010 | Outbound | FEAT-18 (Client Contact Management) | Sending is blocked until the client has at least one Primary contact (XBR-07) | Freelancer taps Send |
| FEAT-02.SPEC-011 | Outbound | FEAT-05 (Client Portal Access / Sign-In) | The proposal-sent email's link leads Owen through magic-link sign-in to the proposal view; an expired link is refreshed there without losing proposal state | Owen opens the email link |
| FEAT-02.SPEC-003, FEAT-02.SPEC-006 | Inbound | FEAT-03 (Proposal Acceptance) | A request-changes note is recorded as a Comment on the proposal and shown on its detail screen (XBR-26); Nadia edits and re-sends via the Void & Resend automation | Owen sends a request-changes note |
| FEAT-02.SPEC-010 | Inbound/Outbound | FEAT-03 (Proposal Acceptance) | The accept-vs-void contention rule is enforced jointly: an Accept against a version voided in the meantime is refused by FEAT-03; an edit started by Nadia after acceptance is refused by SPEC-010 | Concurrent accept/edit on the same proposal |
| FEAT-02.SPEC-001, FEAT-02.SPEC-005 | Outbound | FEAT-15 (Currency & Tax Handling) | Proposal price is set and displayed in the project's currency; the currency cannot change once the first invoice is sent (XBR-17) | Drafting or sending a proposal |
| FEAT-02.SPEC-001, FEAT-02.SPEC-002 | Outbound | FEAT-04 (Milestone & Payment Schedule Setup) | The proposal carries a reference to the project's Payment Schedule | Drafting a proposal |
| FEAT-02.SPEC-005, FEAT-02.SPEC-006, FEAT-02.SPEC-007, FEAT-02.SPEC-011 | Outbound | FEAT-14 (Notifications delivery) | Every send, resend, and void-resend calls the transactional email delivery capability whose Integration spec is owned by FEAT-14 (ASMP-29) | Proposal sent, resent, or void-resent |
| FEAT-02.SPEC-002, FEAT-02.SPEC-011 | Inbound | FEAT-19 (Branding Profile) | Preview and email rendering apply the freelancer's logo and brand colour, falling back to a neutral default (XBR-31) | Rendering the preview or the sent email |
| FEAT-02 (all specs) | Outbound | FEAT-13 (Activity & Audit Trail) | Proposal lifecycle events (sent, edited-and-resent, resent, discarded) are recorded as immutable Activity Log Entries | On each lifecycle transition |
| FEAT-02.SPEC-003 | Outbound | FEAT-31 (Support Access) | Dana's logged, read-only support session can view proposal status, never its send/edit actions | Support session opened on the freelancer's account |
| FEAT-02 (Proposal entity) | Outbound | FEAT-24 (Account Deletion) | Sent/Voided/Accepted proposal records are removed only as part of full account deletion, never by this feature | Account deletion request |

## Non-Functional Notes

**Data volumes / growth:** A solo freelancer with 5-8 active clients accumulates one proposal per project plus its voided prior versions over years of use; the Reuse Proposal Picker (SPEC-004) must stay equally responsive as this cross-project history grows (assumptions-constraints.md, Non-Functional Expectations; user-persona.md persona summary).

**Responsiveness:** Success-metrics.md's "Proposal Send Speed" targets drafting and sending a typical proposal in under 10 minutes, and "Time to Proposal Acceptance" targets 70% of accepted proposals within 24 hours of sending -- both depend on the draft editor, preview, and send action responding without perceptible delay; per ASMP-27, every screen shows real progress while loading and never pretends an offline send succeeded.

**Data sensitivity / privacy:** Scope and price are commercially confidential; once Sent, Voided, or Accepted, the content is evidentiary and immutable (ASMP-25, XBR-04). The accepting contact's identity is personal data, GDPR-class (ASMP-24). Priya (Client Reviewer Contact) has no access to proposal content at all, per the Access Matrix; Dana's support access is read-only and logged (ASMP-23).

**Compliance flags:** ASMP-24 (GDPR-class handling of client-contact identity on send/accept records) and ASMP-25 (correctness and immutability of the sent/accepted proposal record) both apply directly to this feature; no card or payment data is ever captured here (ASMP-24).

**Analytics signals:** proposal_drafted, proposal_sent, proposal_edited_before_acceptance, proposal_resent, proposal_created_from_copy, proposal_draft_discarded (product-features.md, Signals) -- emitted respectively by SPEC-001 (draft save), SPEC-005 (send), SPEC-006 (edit-and-resend), SPEC-007 (resend), SPEC-008 (copy), and SPEC-009 (discard).

## Non-Goals

- **A configurable proposal template or workflow builder** -- Excluded per scope-boundaries.md (SC-11): configurable workflow/form builders are the documented source of the category's 15-25+ hour setup burden; this feature instead offers a plain form or copy-an-earlier-proposal (product-features.md Rationale).
- **A library of legal contract templates or clauses** -- Excluded per scope-boundaries.md (SC-13): providing jurisdiction-specific legal documents is outside the founder's capacity and outside BRIEF.md's scope; freelancers write their own scope text and may reuse an earlier proposal (SPEC-004/SPEC-008) instead of a legal template library.
- **A client-side counter-proposal or price-editing capability** -- Excluded per scope-boundaries.md (SC-02): the Access Matrix gives Owen only view/accept/request-changes/sign rights on a proposal, with no additional client-side role or edit capability defined; price and scope changes remain exclusively a freelancer (Nadia) action, surfaced through the request-changes flow (FEAT-03) instead.
- **A browsable version-history or diff view of voided proposal versions** -- Intentional omission surfaced by Phase 3/7 analysis: the dependency map's XBR-04/XBR-06 retain voided versions only as immutable evidence, and no Stage 2 depth field or journey step calls for a UI to browse or compare them; SPEC-003 shows only the current version's status.
- **Automatic purge or retention window for Sent/Voided/Accepted proposals** -- Intentional lifecycle decision from the CRUD matrix: these statuses are evidentiary (ASMP-25, XBR-04) and are retained for the life of the account, removed only by FEAT-24 account deletion; no automatic purge applies.



# Screen Spec: Proposal Draft Editor

## Overview

**Name:** Proposal Draft Editor
**ID:** FEAT-02.SPEC-001
**Type:** Screen
**Purpose:** Nadia writes scope, price, and currency for a project's proposal from a blank form, from a copied earlier proposal, or by revising a Sent-but-unaccepted proposal.
**Parent Feature:** FEAT-02 -- Proposal Creation & Sending

## Scope and Non-Goals

**In Scope:**
- Blank-draft creation for a project with no existing proposal
- Pre-filled draft creation when opened from a copy (FEAT-02.SPEC-008)
- Edit mode for a Sent-but-unaccepted proposal, culminating in a void-and-resend rather than a silent update
- Inline field validation via FEAT-02.SPEC-010
- Local draft persistence and offline-tolerant drafting
- Navigating to the branded preview before sending
- Discarding an unsent draft

**Non-Goals:**
- Sending the proposal -- owned by FEAT-02.SPEC-005 (Proposal Send); this screen only prepares content and hands off to Send from the Preview screen.
- Browsing earlier proposals to copy from -- owned by FEAT-02.SPEC-004 (Reuse Proposal Picker); this screen only receives already-selected copied content from FEAT-02.SPEC-008.
- A configurable proposal template or workflow builder -- excluded per scope-boundaries.md (SC-11): the category's 15-25+ hour setup burden comes from configurable builders, so this product ships one fixed form instead.
- A library of legal contract clauses -- excluded per scope-boundaries.md (SC-13): freelancers write their own scope text or reuse an earlier proposal (FEAT-02.SPEC-004/008) rather than draw on a maintained legal template library.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-02.SPEC-003 (Proposal Detail) | Nadia taps "Edit" on a Draft, or on a Sent-but-unaccepted proposal | Existing proposal id, current scope/price/currency, mode flag (Draft edit vs. Sent-but-unaccepted edit) |
| FEAT-02.SPEC-003 (Proposal Detail) | Empty-state prompt when the project has no proposal yet | Project reference; blank form; project currency (FEAT-15) |
| FEAT-02.SPEC-008 (Create Draft From Copy) | Automation finishes copying an earlier proposal and opens the editor | New Draft's scope/price/currency pre-filled from the source; copied_from reference |
| FEAT-01 (Client & Project Management), project view proposal area | Nadia opens the proposal area of a project with no existing proposal | Project reference; blank form |
| FEAT-20 (Onboarding / First-Run Setup) | Onboarding step "Draft the first proposal" | New project reference; blank form |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Create, edit, save draft, preview, save & resend, discard -- all actions | -- |
| Owen (Client Primary Contact) | No | No | This screen exists only in the freelancer's own workspace; no route into it exists from the client portal, so Owen never sees it. |
| Priya (Client Reviewer Contact) | No | No | Same as Owen -- no client-portal route to this screen exists. |
| Dana (Support Operator) | Full screen, read-only (all field values visible) | None -- no Save, Preview, Save & Resend, or Discard controls appear | Reached only inside a logged, read-only support session (FEAT-31); every input renders disabled, so there is nothing to attempt beyond viewing. |
| Unauthenticated | No | No | Redirected to freelancer sign-in; no proposal content renders before authentication succeeds. |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." Unsaved field values are preserved locally and restored automatically once re-authentication succeeds. |

## Layout and Content

**Header:** Screen title -- "New Proposal" (blank or copied-draft entry) or "Edit Proposal" (Sent-but-unaccepted edit entry) -- with a back arrow (returns to FEAT-02.SPEC-003) at the left. In Draft mode, a secondary "Save Draft" text action sits at the top-right.

**Body:** A single-column form:
- **Copied-from banner** (shown only when the editor opened from FEAT-02.SPEC-008): "Started from a copy of {source project name}'s proposal, {source proposal's sent or accepted date}." Display-only.
- **Scope Description** -- multi-line text input, required
- **Price** -- number input, required, positive amount
- **Currency** -- read-only display field showing the project's currency (FEAT-15); never editable on this screen, since XBR-17 fixes it once the first invoice is sent and it is set for the project elsewhere
- **Payment Schedule** -- a read-only summary card showing the project's Payment Schedule (FEAT-04) structure if one exists, or the text "No payment schedule set yet" with a link into FEAT-04 if none exists

**Footer:** No footer region. A bottom action bar carries the primary actions: in Draft mode, "Discard Draft" (destructive text button, left) and "Preview" (primary button, right); in Sent-but-unaccepted edit mode, "Save & Resend" (primary button, right) replaces "Preview" and "Discard Draft" is not shown (discard applies only to unsent drafts, per FEAT-02.SPEC-010).

### Responsive Behavior

- **Compact:** Single-column form, full width; header shows only the title and back arrow, with "Save Draft" moving into an overflow menu if space is constrained.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; the bottom action bar remains fixed and visible without a structural change. Scope Description grows from 4 visible lines (compact) to 8 visible lines (medium and above).

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-02.SPEC-003 (Proposal Detail); if unsaved changes exist, confirm first | Dialog if unsaved changes exist | "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" |
| Scope Description input | Type | Captures text | Field shows entered text | Standard input focus state |
| Scope Description input | Blur (empty) | Triggers validation via FEAT-02.SPEC-010 | Error state on field | "Scope description is required" |
| Price input | Type | Captures numeric value | Field shows entered value | Standard input focus state |
| Price input | Blur (empty or non-positive) | Triggers validation via FEAT-02.SPEC-010 | Error state on field | Exact message per FEAT-02.SPEC-010 |
| Payment Schedule card | Tap | Navigate to FEAT-04 (Milestone & Payment Schedule Setup) | Screen transitions | -- |
| Save Draft (Draft mode only) | Tap | Validates required-field rules via FEAT-02.SPEC-010; persists content as Draft | Button shows brief saved confirmation | Toast "Draft saved" |
| Preview (Draft mode only) | Tap | Validates via FEAT-02.SPEC-010; if valid, saves current content as Draft and navigates to FEAT-02.SPEC-002 (Proposal Preview); if invalid, blocks navigation | Button shows loading state briefly | Success: navigates to Preview. Failure: inline field errors shown, focus moves to first error |
| Save & Resend (edit mode only) | Tap | Validates via FEAT-02.SPEC-010; if valid, triggers FEAT-02.SPEC-006 (Void & Resend) | Button shows loading state during the operation | Success: navigates to FEAT-02.SPEC-003 showing the updated Sent state. Failure: error banner, edits preserved |
| Discard Draft (Draft mode only) | Tap | Confirmation dialog, then triggers FEAT-02.SPEC-009 (Discard Draft) | Confirmation dialog appears | Dialog: "Discard this draft? This cannot be undone." with "Discard" and "Keep Editing" |

### Accessibility Notes

- **Focus order:** Back arrow -> Save Draft (if shown) -> Scope Description -> Price -> Currency (read-only, announced but not editable) -> Payment Schedule card -> Discard Draft (if shown) -> Preview/Save & Resend.
- **Validation announcements:** When a field enters an error state, its error message is announced to assistive technology and programmatically associated with the field; on Preview/Save & Resend failure, focus moves to the first field in error.
- **Save feedback:** The "Draft saved" toast is announced on success.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (default) | All fields empty except the read-only Currency (populated from the project) | Screen opens for a new blank proposal | Nadia begins typing in any field |
| Prefilled | Scope Description, Price, and Currency populated from a copy or from the existing Draft/Sent-but-unaccepted proposal | Screen opens from FEAT-02.SPEC-008 or in edit mode | Nadia edits a field |
| Filling | Form fields contain user input | Nadia types in any field | Nadia navigates away or completes an action |
| Validating | Preview/Save Draft/Save & Resend button shows a brief loading indicator | Nadia taps Preview, Save Draft, or Save & Resend | Validation completes (pass or fail) |
| Validation Error | Failed fields highlighted with error messages below them | Validation fails | Nadia corrects the field and re-triggers validation |
| Saving | Active button shows a loading spinner, form fields disabled | Validation passes | Save (or Save & Resend) completes or fails |
| Error | Error banner at the top of the form with a retry option | Save, Preview-save, or Save & Resend fails | Nadia taps Retry or navigates away |
| Offline/Degraded | Banner: "You're offline -- your changes are kept on this device until you reconnect." Drafting continues locally; Preview, Save Draft, and Save & Resend are disabled until connectivity returns (sending requires connectivity per the feature definition) | Connectivity is lost while the screen is open | Connectivity returns; disabled actions re-enable |

## Validation Rules

Validation governed by FEAT-02.SPEC-010 (Proposal Validation & Business Rules). See that spec for all field-level rules (required scope description, positive price) and for the immutability rule that blocks a Save & Resend attempt on an already-accepted proposal. This screen applies validation on field blur and on Preview/Save Draft/Save & Resend.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-02.SPEC-003 (Proposal Detail) | -- |
| Preview tap (validation passes) | FEAT-02.SPEC-002 (Proposal Preview) | -- |
| Save & Resend success | FEAT-02.SPEC-003 (Proposal Detail) | -- |
| Discard Draft confirmed | FEAT-02.SPEC-003 (Proposal Detail), empty state | -- |
| Payment Schedule card tap | Milestone & Payment Schedule Setup | FEAT-04 |

## Data Model

**Creates:** Proposal record (Draft) -- scope_description, price, currency (read from the owning Project), payment_schedule_reference (read from the Project's Payment Schedule), status set to Draft, copied_from set when opened from FEAT-02.SPEC-008.
**Reads:** Project -- currency and project_name (FEAT-15). Payment Schedule -- structure summary (FEAT-04). The existing Proposal record's scope_description, price, and status when opened in edit mode.
**Updates:** Proposal record's scope_description and price when Save Draft is used on an existing Draft; a Sent-but-unaccepted proposal's content is never updated directly here -- Save & Resend hands the new content to FEAT-02.SPEC-006, which creates the new version.
**Deletes:** None directly -- the Discard Draft action hands off to FEAT-02.SPEC-009, which performs the deletion.

## Business Rules

- Field requirements and positive-price validation are governed by FEAT-02.SPEC-010 -- this screen never saves a Draft or triggers a send with an invalid scope description or price.
- Currency is never editable on this screen; it always reflects the owning project's currency and is fixed once the project's first invoice is sent (XBR-17).
- Editing a Sent-but-unaccepted proposal never updates the proposal in place -- Save & Resend triggers FEAT-02.SPEC-006, which voids the prior version and creates and sends the new one (XBR-06).
- Discard is available only on unsent Drafts, per FEAT-02.SPEC-010 -- the action is not shown once a proposal has been sent.

## Edge Cases

- **Nadia navigates away with unsaved changes** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Nadia taps Preview or Save & Resend twice rapidly** -- The second tap is ignored while the first operation is in progress (button in loading state).
- **Network failure during Save Draft** -- Error banner: "Could not save your draft. Check your connection and try again." with a Retry button; form content is preserved.
- **Nadia edits a Sent-but-unaccepted proposal in two open sessions and saves from both** -- Reject-with-refresh, per the dependency map's Contention note for the Proposal entity: the second Save & Resend is checked against the proposal's current version at commit; if the first session's edit already voided and re-sent it, the second save is refused with "This proposal was already edited and resent. Review the current version." and the screen reloads the now-current version's content.
- **Nadia attempts to edit a proposal that Owen accepted while the editor was open** -- Save & Resend is refused per XBR-04 with the dialog "This proposal has already been accepted and can no longer be edited." and a "View Current Status" option that navigates to FEAT-02.SPEC-003.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-002 (Proposal Preview) | Navigation (outbound) | Preview tap, after validation, opens the branded preview |
| FEAT-02.SPEC-003 (Proposal Detail) | Navigation (inbound/outbound) | Entry point for Edit and the empty-state prompt; return destination after save, resend, or discard |
| FEAT-02.SPEC-004 (Reuse Proposal Picker) | References (inbound, indirect) | Reached only through FEAT-02.SPEC-008 after a source proposal is selected |
| FEAT-02.SPEC-006 (Void & Resend) | Triggers (outbound) | Save & Resend, after validation, triggers the void-and-resend automation |
| FEAT-02.SPEC-008 (Create Draft From Copy) | Navigation (inbound) | Opens this screen pre-filled once the copy automation completes |
| FEAT-02.SPEC-009 (Discard Draft) | Triggers (outbound) | Discard Draft, after confirmation, triggers the hard-delete automation |
| FEAT-02.SPEC-010 (Proposal Validation & Business Rules) | References (inbound) | All field validation and the edit-eligibility rule applied on this screen |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| proposal_drafted | entry mode (blank / from copy), time from screen open to first save (seconds) | Save Draft completes successfully on a new Draft | supports success-metrics.md: "Proposal Send Speed" |
| proposal_draft_save_failed | failure reason (validation / connectivity) | Save Draft or Preview-triggered save fails | N/A -- no Stage 2 metric measures failed saves; retained so drafting friction is observable rather than invisible |

## Acceptance Criteria

**FEAT-02.SPEC-001-AC-01:** Given Nadia opens the proposal area of a project with no existing proposal, when the screen loads, then the form shows empty Scope Description and Price fields with Currency pre-filled from the project.

**FEAT-02.SPEC-001-AC-02:** Given Nadia fills in a scope description and a positive price and taps "Save Draft", then the system saves the Proposal as Draft and shows the toast "Draft saved".

**FEAT-02.SPEC-001-AC-03:** Given Nadia leaves the Scope Description field empty and moves focus away, then the field shows the error "Scope description is required" per FEAT-02.SPEC-010.

**FEAT-02.SPEC-001-AC-04:** Given Nadia has filled in valid scope and price and taps "Preview", then validation passes, the content saves as Draft, and she is navigated to FEAT-02.SPEC-002 (Proposal Preview).

**FEAT-02.SPEC-001-AC-05:** Given Nadia opens the editor from FEAT-02.SPEC-008 after selecting a source proposal, when the screen loads, then Scope Description, Price, and Currency are pre-filled from the source and the copied-from banner shows the source project's name.

**FEAT-02.SPEC-001-AC-06:** Given Nadia opens a Sent-but-unaccepted proposal in edit mode, when she revises the price and taps "Save & Resend", then FEAT-02.SPEC-006 is triggered and, on success, she is navigated to FEAT-02.SPEC-003 showing the updated Sent state.

**FEAT-02.SPEC-001-AC-07:** Given Nadia is on a Draft and taps "Discard Draft", when she confirms in the dialog, then FEAT-02.SPEC-009 is triggered and she is navigated to FEAT-02.SPEC-003 showing the empty state.

**FEAT-02.SPEC-001-AC-08:** Given Nadia has unsaved changes on the form, when she taps the back arrow, then the dialog "You have unsaved changes. Discard?" appears with "Discard" and "Keep Editing".

**FEAT-02.SPEC-001-AC-09:** Given Nadia loses connectivity while filling the form, then the banner "You're offline -- your changes are kept on this device until you reconnect." appears and Preview, Save Draft, and Save & Resend are disabled until connectivity returns.

**FEAT-02.SPEC-001-AC-10:** Given Dana (Support Operator) opens this screen inside a logged support session, when she views the form, then all field values render read-only with no Save, Preview, Save & Resend, or Discard controls.

**FEAT-02.SPEC-001-AC-11:** Given Nadia edited and saved-and-resent a Sent-but-unaccepted proposal from a second open session while this session's edit was also pending, when this session's "Save & Resend" completes its check, then the save is refused with "This proposal was already edited and resent. Review the current version." and the screen reloads the current version's content.

**FEAT-02.SPEC-001-AC-12:** Given Owen accepted the proposal while Nadia's edit screen was open, when Nadia taps "Save & Resend", then the save is refused with the dialog "This proposal has already been accepted and can no longer be edited." and a "View Current Status" option to FEAT-02.SPEC-003.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 10 | 10 |
| States | 8 (empty, prefilled, filling, validating, validation error, saving, error, offline) | 8 |
| Business Rules | 4 | 4 |
| Edge Cases | 4 | 4 |



# Screen Spec: Proposal Preview

## Overview

**Name:** Proposal Preview
**ID:** FEAT-02.SPEC-002
**Type:** Screen
**Purpose:** Nadia reviews a branded, client-facing rendering of the current draft's scope, price, and payment schedule before sending it to the client.
**Parent Feature:** FEAT-02 -- Proposal Creation & Sending

## Scope and Non-Goals

**In Scope:**
- A read-only, branded rendering of the draft's content exactly as Owen will see it
- Initiating the Send action from this screen
- Returning to the editor to make further changes before sending

**Non-Goals:**
- Editing content on this screen -- editing happens only in FEAT-02.SPEC-001 (Proposal Draft Editor); this screen is read-only by design so Nadia reviews the exact content that will be sent.
- Performing the send itself -- the send transition, its validation, and its side effects are owned by FEAT-02.SPEC-010 (validation) and FEAT-02.SPEC-005 (Proposal Send); this screen only initiates that flow.
- Rendering the sent email -- the email's own layout and copy are owned by FEAT-02.SPEC-011 (Proposal Sent/Resent Email); this screen previews the in-portal proposal content, not the email.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-02.SPEC-001 (Proposal Draft Editor) | Nadia taps "Preview" after entering valid scope and price | The Draft's current scope_description, price, currency, and payment_schedule_reference |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Send the proposal, return to edit | -- |
| Owen (Client Primary Contact) | No | No | This screen is Nadia's own pre-send review; no route into it exists from the client portal. Once sent, Owen reviews the proposal through his own portal view (FEAT-03), not this screen. |
| Priya (Client Reviewer Contact) | No | No | Same as Owen -- no client-portal route to this screen. |
| Dana (Support Operator) | Full screen, read-only (Send control not shown) | None | Reached only inside a logged, read-only support session (FEAT-31). |
| Unauthenticated | No | No | Redirected to freelancer sign-in. |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." The Draft that was being previewed remains saved and is reachable again from FEAT-02.SPEC-003 after re-authentication. |

## Layout and Content

**Header:** Screen title "Preview" with a back arrow (returns to FEAT-02.SPEC-001, editor pre-filled with the same content) at the left, and a "Send" primary action button at the right.

**Body:** A single-column, branded rendering styled with the freelancer's Branding Profile (logo and brand colour, FEAT-19, per XBR-31; falls back to a neutral default when unset), showing, in order:
- Freelancer's logo and brand colour applied to the page header band
- Project name and client name
- Scope Description, rendered as formatted text
- Price and Currency, prominently displayed
- Payment Schedule summary (structure and amounts, from FEAT-04), or "No payment schedule set yet" if none exists

**Footer:** None -- Send is in the header.

### Responsive Behavior

- **Compact:** Single-column rendering, full width; header condenses to title, back arrow, and Send button only.
- **Medium size class and above:** Rendering remains single-column, capped at a consistent platform-wide reading width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-02.SPEC-001 (Proposal Draft Editor), pre-filled with the previewed content | Screen closes | Animated transition back to the editor |
| Send button | Tap | Re-validates via FEAT-02.SPEC-010 (send-eligibility rules, including the Primary-contact check) and, if eligible, triggers FEAT-02.SPEC-005 (Proposal Send) | Button shows loading state during send | Success: navigates to FEAT-02.SPEC-003 (Proposal Detail) showing Sent state, with confirmation "Proposal sent to {Primary Contact name}." Failure: error banner, content preserved |
| Send button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> Send button (the rendered content is read via normal document order between them).
- **Send feedback:** The success confirmation and any send-eligibility error are announced to assistive technology when they appear.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty | N/A -- not reachable | N/A -- Preview is only reachable from a Draft that has already passed field validation (FEAT-02.SPEC-001 requires valid scope and price before offering "Preview"), so an empty-content state is never possible here | N/A |
| Loading | Skeleton placeholders for the branded rendering (header band, scope block, price block, payment schedule block) | Screen opens from FEAT-02.SPEC-001, before the Branding Profile (FEAT-19), Payment Schedule (FEAT-04), and Client/Project names finish loading -- none of these are handed off by the Entry Point, so this screen fetches them itself on open | The fetch completes and the Loaded state renders |
| Loaded (default) | Full branded rendering of the draft's content | The Loading fetch completes successfully | Nadia navigates away or taps Send |
| Sending | Send button shows a loading spinner; content remains visible but read-only (already the default) | Nadia taps Send and validation passes | Send completes or fails |
| Send Blocked | Inline banner above the Send button naming the unmet send-eligibility condition (e.g., "This client has no Primary contact yet.") with a link to resolve it | Send-eligibility validation fails (FEAT-02.SPEC-010) | Nadia resolves the condition and returns, or navigates away |
| Error | Error banner at the top with a retry option; the draft's content is unchanged. Two causes share this appearance: (a) the initial Loading fetch fails ("Could not load the preview." banner, Send hidden until retried) or (b) the send operation itself fails after eligibility passed ("Could not send this proposal." banner, content still shown) | (a) The Loading fetch fails, or (b) the send operation fails after eligibility passed (e.g., connectivity lost mid-send) | Nadia taps Retry (re-runs the failed fetch or the failed send) or navigates back to the editor |
| Offline/Degraded | Banner: "You're offline -- sending requires a connection." Send is disabled; the preview content itself remains viewable from local state | Connectivity is lost while this screen is open | Connectivity returns; Send re-enables |

## Validation Rules

Validation governed by FEAT-02.SPEC-010 (Proposal Validation & Business Rules). The Send button re-runs send-eligibility checks (required fields, positive price, and the client Primary-contact requirement, XBR-07) at the moment of tap -- not only at the time the editor was left -- since eligibility can change between preview and send.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-02.SPEC-001 (Proposal Draft Editor) | -- |
| Successful send | FEAT-02.SPEC-003 (Proposal Detail) | -- |
| Send Blocked banner link (no Primary contact) | Client contact list | FEAT-18 (Client Contact Management & Roles) |

## Data Model

**Creates:** None -- this screen renders existing Draft content; it does not persist new data.
**Reads:** Proposal -- scope_description, price, currency, payment_schedule_reference (carried directly from the Entry Point's context, no fetch needed). Payment Schedule -- structure and amounts (FEAT-04); Branding Profile -- logo and brand colour (FEAT-19); Client and Project -- names for display: none of these three are handed off by the Entry Point, so the screen fetches them itself on open, which is what the Loading state covers.
**Updates:** None directly -- a successful Send hands the transition to FEAT-02.SPEC-005.
**Deletes:** None.

## Business Rules

- The rendering shown here must exactly match what FEAT-02.SPEC-011 (Proposal Sent/Resent Email) and Owen's portal view (FEAT-03) will present, per the Feature Breakdown Brief's Shared Context: preview and email both apply the freelancer's Branding Profile consistently (XBR-31).
- Send eligibility (required fields, positive price, one-active-proposal cap, client Primary-contact requirement per XBR-07) is fully governed by FEAT-02.SPEC-010 -- this screen never bypasses those checks even though the editor validated once already.
- Sending from this screen is the sole trigger for FEAT-02.SPEC-005 within this feature's happy path (the Draft editor never sends directly).

## Edge Cases

- **Nadia taps Send twice rapidly** -- The second tap is ignored while the first send is in progress (button in loading state).
- **The client's last Primary contact is removed between opening Preview and tapping Send** -- Send is blocked with the Send Blocked state: "This client has no Primary contact yet." and a link into FEAT-18 to add one; the Draft is unaffected.
- **Network failure during send** -- Error banner: "Could not send this proposal. Check your connection and try again." with a Retry button; the Draft's content is preserved and the proposal remains in Draft status (no partial Sent state is created).
- **Nadia navigates back to the editor and changes the price, then returns to Preview** -- The rendering reflects the updated content; there is no stale-preview state because Preview always reads the Draft's current saved values.
- **Fetching the Branding Profile, Payment Schedule, or Client/Project names fails on open** -- The Loading state resolves to the Error state ("Could not load the preview.") rather than rendering with any of the three missing; Send is not offered until the retry succeeds, since Send re-validation depends on data this screen has not yet confirmed it can display correctly.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-001 (Proposal Draft Editor) | Navigation (inbound/outbound) | Entry point from Preview tap; return destination via the back arrow |
| FEAT-02.SPEC-003 (Proposal Detail) | Navigation (outbound) | Destination after a successful send |
| FEAT-02.SPEC-005 (Proposal Send) | Triggers (outbound) | Send button, after eligibility passes, triggers the send automation |
| FEAT-02.SPEC-010 (Proposal Validation & Business Rules) | References (inbound) | Send-eligibility rules re-checked at the moment of Send |
| FEAT-19 (Freelancer Branding) | References (inbound) | Branding Profile applied to the rendering |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| proposal_preview_viewed | time since draft last saved (seconds) | Screen finishes loading | supports success-metrics.md: "Proposal Send Speed" |
| proposal_send_blocked | blocking reason (no_primary_contact / other) | Send-eligibility validation fails on Send tap | N/A -- no Stage 2 metric measures blocked sends; retained so the send-eligibility gate's real-world frequency is observable |

## Acceptance Criteria

**FEAT-02.SPEC-002-AC-01:** Given Nadia taps "Preview" from a valid Draft, when the Preview screen loads, then it shows the branded rendering of the scope, price, currency, and payment schedule exactly as saved on the Draft.

**FEAT-02.SPEC-002-AC-02:** Given Nadia is viewing the Preview and the client has a Primary contact, when she taps "Send", then FEAT-02.SPEC-005 is triggered and, on success, she is navigated to FEAT-02.SPEC-003 with the confirmation "Proposal sent to {Primary Contact name}."

**FEAT-02.SPEC-002-AC-03:** Given the client has no Primary contact, when Nadia taps "Send" from Preview, then the Send Blocked banner "This client has no Primary contact yet." appears with a link into FEAT-18, and no send occurs.

**FEAT-02.SPEC-002-AC-04:** Given Nadia taps the back arrow on Preview, then she is returned to FEAT-02.SPEC-001 with the same content still populated.

**FEAT-02.SPEC-002-AC-05:** Given Nadia taps "Send" and the operation fails due to a network error, then an error banner appears with a Retry option and the proposal remains in Draft status.

**FEAT-02.SPEC-002-AC-06:** Given Nadia loses connectivity while viewing Preview, then the banner "You're offline -- sending requires a connection." appears and Send is disabled.

**FEAT-02.SPEC-002-AC-07:** Given Dana (Support Operator) opens this screen inside a logged support session, when she views it, then the content renders read-only with no Send control shown.

**FEAT-02.SPEC-002-AC-08:** Given Nadia taps "Send" twice in rapid succession, then the second tap is ignored while the first send is in progress.

**FEAT-02.SPEC-002-AC-09:** Given Nadia returns to Preview after changing the price in the editor, when the screen reloads, then it shows the updated price, not the previously previewed value.

**FEAT-02.SPEC-002-AC-10:** Given Nadia taps "Preview" and the Branding Profile, Payment Schedule, and Client/Project names have not yet finished loading, when the screen opens, then skeleton placeholders appear until the fetch completes, after which the full branded rendering displays.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 3 | 3 |
| States | 7 (empty [N/A], loading, loaded, sending, send blocked, error, offline) | 7 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



# Screen Spec: Proposal Detail

## Overview

**Name:** Proposal Detail
**ID:** FEAT-02.SPEC-003
**Type:** Screen
**Purpose:** Nadia (and, read-only, Dana) views a project's current proposal -- its status, history, and the actions available for that status -- and Dana's support view is limited to status only.
**Parent Feature:** FEAT-02 -- Proposal Creation & Sending

## Scope and Non-Goals

**In Scope:**
- The single current proposal's status, content summary, and available status-driven actions for a project
- The empty state when a project has no proposal yet
- Surfacing a request-changes note (FEAT-03) attached to the proposal
- Entry points into editing, sending-adjacent flows, and reuse

**Non-Goals:**
- A browsable version-history or diff view of voided proposal versions -- intentional omission per the Feature Breakdown Brief's Non-Goals: no Stage 2 depth field or journey step calls for browsing or comparing voided versions; only the current version's status is shown.
- Editing proposal content directly on this screen -- editing happens in FEAT-02.SPEC-001 (Proposal Draft Editor), reached via the Edit action.
- Accepting the proposal or recording a request-changes note -- both are Owen's actions, owned by FEAT-03 (Proposal Acceptance); this screen only displays their resulting state and the note's content.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01 (Client & Project Management), project view proposal area | Nadia opens the proposal area of a project | Project reference |
| FEAT-12 (Freelancer Financial Dashboard) | Empty-dashboard zero-state prompt toward sending a first proposal | Project reference (the freelancer's first project) |
| FEAT-28 (Global Search Across Clients & Projects) | A search result for a proposal | Proposal/project reference |
| FEAT-02.SPEC-005 (Proposal Send) | Send succeeds | Project reference, updated to Sent state |
| FEAT-02.SPEC-006 (Void & Resend) | Prior version voided and new version sent | Project reference, updated to Sent state (new version) |
| FEAT-02.SPEC-009 (Discard Draft) | Draft discarded | Project reference, updated to empty state |
| FEAT-03 (Proposal Acceptance), request-changes email | Nadia opens the change-request email | Proposal reference, with the request-changes note shown |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen: status, content summary, request-changes note, and all status-driven actions | Edit, Send, Discard, Start from Copy (Draft); Edit, Resend (Sent); none (Voided, Accepted -- view only) | -- |
| Owen (Client Primary Contact) | No | No | This screen is Nadia's own workspace view; Owen's equivalent is his own portal proposal view (FEAT-03), not this screen. |
| Priya (Client Reviewer Contact) | No | No | Same as Owen -- Priya has no access to proposal content at all, per the Access Matrix; her portal home shows only the project's stage. |
| Dana (Support Operator) | Status and history only -- no scope description, price, or request-changes note text; no send/edit/discard actions are shown or reachable | None | Reached only inside a logged, read-only support session (FEAT-31); content beyond status is not rendered, so there is nothing further to attempt. |
| Unauthenticated | No | No | Redirected to freelancer sign-in. |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." The screen reloads showing the same proposal once re-authentication succeeds. |

## Layout and Content

**Header:** Screen title "Proposal" with a back arrow (returns to the project view, FEAT-01) at the left. When a proposal exists, a status badge (Draft / Sent / Voided / Accepted) sits beside the title.

**Body (proposal exists):**
- **Status summary card:** current status, sent_at (if sent), accepted_at and accepted_by (if accepted, read from the record FEAT-03 writes)
- **Content summary:** scope description (truncated with "Show more"), price and currency, payment schedule reference summary
- **Request-changes note** (shown only when one exists, attached by FEAT-03): note text, posted date, with a label "Change requested by {Primary Contact name}"
- **Action bar:** status-driven action set (see Business Rules)

**Body (no proposal exists -- empty state):** A prompt: "This project has no proposal yet." with a primary "Draft a Proposal" button.

**Footer:** None -- actions live in the body's action bar.

### Responsive Behavior

- **Compact:** Single-column stacked cards, full width; action bar becomes a bottom-fixed bar.
- **Medium size class and above:** Content remains single-column, capped at a consistent platform-wide width and horizontally centered; action bar remains inline at the top of the body rather than fixed to the bottom.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| "Draft a Proposal" (empty state) | Tap | Navigate to FEAT-02.SPEC-001 (Proposal Draft Editor), blank | Screen closes | Animated transition to the editor |
| "Show more" on scope description | Tap | Expands the truncated text inline | Text expands | Text reflows to show full scope description |
| Edit (Draft or Sent status) | Tap | Navigate to FEAT-02.SPEC-001 in the corresponding mode | Screen closes | Animated transition to the editor |
| Send (Draft status) | Tap | Navigate to FEAT-02.SPEC-002 (Proposal Preview) | Screen closes | Animated transition to Preview |
| Discard (Draft status) | Tap | Confirmation dialog, then triggers FEAT-02.SPEC-009 (Discard Draft) | Confirmation dialog appears | Dialog: "Discard this draft? This cannot be undone." |
| Start from Copy (Draft status, when the Draft is otherwise unsent and blank-equivalent) | Tap | Navigate to FEAT-02.SPEC-004 (Reuse Proposal Picker) | Screen closes | Animated transition to the picker |
| Resend (Sent status) | Tap | Triggers FEAT-02.SPEC-007 (Proposal Resend) | Button shows brief loading state | Toast "Proposal link resent to {Primary Contact name}." |
| Request-changes note | Tap | Expands the full note text if truncated | Text expands | Text reflows |

### Accessibility Notes

- **Focus order:** Back arrow -> status badge (announced) -> content summary -> request-changes note (if present) -> action bar buttons in the order shown.
- **Status announcements:** A status change reached by navigating back to this screen after an action (send, resend, discard, void-and-resend) is announced on load, e.g., "Proposal sent."
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty | Prompt "This project has no proposal yet." with "Draft a Proposal" button | Project has no Proposal record, or its only Draft was just discarded | A Draft is created for the project |
| Draft | Status badge "Draft"; Edit, Send, Discard, and (when eligible) Start from Copy actions shown | A Draft Proposal exists for the project | The Draft is sent or discarded |
| Sent | Status badge "Sent"; sent_at shown; Edit and Resend actions shown; request-changes note shown if present | The proposal transitions to Sent (FEAT-02.SPEC-005 or FEAT-02.SPEC-006) | The proposal is accepted, or edited-and-resent again |
| Voided | Not directly displayed as a distinct screen state | N/A -- this row exists only to document the Proposal entity's status enum (FEAT-02.SPEC-010); this screen queries and always renders the project's single current (non-voided) proposal, so when FEAT-02.SPEC-006 voids a version and creates the new Sent version, the screen's next load reads and shows that new Sent version directly -- the intermediate Voided status is never itself rendered here, even momentarily | N/A -- not a reachable screen state |
| Accepted | Status badge "Accepted"; accepted_at and accepted_by shown; no edit/send/discard/resend actions shown | Owen accepts the proposal (FEAT-03) | N/A -- Accepted is terminal |
| Loading | Skeleton placeholders for the status card and content summary | Screen is opening and proposal data has not yet arrived | Data loads and one of Empty, Draft, Sent, or Accepted renders, per the proposal's current, resolved status |
| Error | Error banner: "Could not load this proposal." with a Retry button | Loading the proposal's data fails | Nadia taps Retry, or connectivity returns and a re-fetch succeeds |
| Offline/Degraded | Banner: "You're offline -- showing the last loaded version of this proposal." Content shown is the last successfully loaded snapshot; Edit/Send/Discard/Resend actions are disabled until connectivity returns | Connectivity is lost while viewing a previously loaded proposal | Connectivity returns and the screen re-fetches |

## Validation Rules

Not applicable -- this screen has no user input fields. Action eligibility (which buttons appear per status) is governed by FEAT-02.SPEC-010 (Proposal Validation & Business Rules).

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | Project view | FEAT-01 (Client & Project Management) |
| "Draft a Proposal" / Edit tap | FEAT-02.SPEC-001 (Proposal Draft Editor) | -- |
| Send tap | FEAT-02.SPEC-002 (Proposal Preview) | -- |
| Start from Copy tap | FEAT-02.SPEC-004 (Reuse Proposal Picker) | -- |
| Discard confirmed | This screen, empty state | -- |
| Resend tap | Stays on this screen (toast confirmation) | -- |

## Data Model

**Creates:** None.
**Reads:** Proposal -- status, scope_description, price, currency, sent_at, accepted_at, accepted_by, copied_from. Comment -- request-changes note text, author, and posted_at when one is attached to the proposal (FEAT-03). Payment Schedule -- summary (FEAT-04).
**Updates:** None directly -- all status transitions happen through the triggered automations (FEAT-02.SPEC-005, 006, 007, 009) and FEAT-03's acceptance flow.
**Deletes:** None directly -- Discard hands off to FEAT-02.SPEC-009.

## Business Rules

- Status-driven action set (FEAT-02.SPEC-010 governs eligibility for each): Draft -- Edit, Send, Discard, Start from Copy; Sent -- Edit, Resend; Voided/Accepted -- view only, no actions.
- "Start from Copy" from this screen behaves identically to reaching FEAT-02.SPEC-004 from any other entry point -- see the Feature Breakdown Brief's Shared UI Patterns.
- The screen always shows the project's single current (non-voided) proposal -- a project has at most one active proposal per the dependency map, so no list or version switcher is needed.
- A request-changes note (XBR-26) is display-only here -- it never alters the proposal, and Reviewer contacts' inability to see proposal content (Access Matrix) has no bearing on this screen, which Reviewers cannot reach at all.

## Edge Cases

- **Nadia opens this screen while the proposal is being voided-and-resent in another session** -- Reject-with-refresh is not applicable to a read-only screen; the screen simply re-fetches on next load or resume and shows the current state, which may differ from what was last shown. No stale-write conflict exists here because this screen performs no writes of its own.
- **The project's proposal was just discarded in another session** -- On next load or resume, the screen re-fetches and shows the Empty state rather than a stale Draft.
- **Owen accepts the proposal while Nadia is viewing this screen** -- On next load or resume, the screen shows Accepted with accepted_at/accepted_by; no live-push update is implied (this is a snapshot view, re-fetched on open/resume).
- **A request-changes note is very long** -- The note is truncated to a preview with "Show more", consistent with the scope description's truncation treatment.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-001 (Proposal Draft Editor) | Navigation (outbound) | "Draft a Proposal" (empty state) and Edit (Draft/Sent) both open the editor |
| FEAT-02.SPEC-002 (Proposal Preview) | Navigation (outbound) | Send (Draft status) opens Preview |
| FEAT-02.SPEC-004 (Reuse Proposal Picker) | Navigation (outbound) | Start from Copy opens the picker |
| FEAT-02.SPEC-005 (Proposal Send) | References (inbound) | Shows the Sent state this automation produces |
| FEAT-02.SPEC-006 (Void & Resend) | References (inbound) | Shows the updated Sent state after a void-and-resend |
| FEAT-02.SPEC-007 (Proposal Resend) | Triggers (outbound) | Resend (Sent status) triggers this automation |
| FEAT-02.SPEC-009 (Discard Draft) | Triggers (outbound) | Discard, after confirmation, triggers this automation |
| FEAT-02.SPEC-010 (Proposal Validation & Business Rules) | References (inbound) | Action-set eligibility per status |
| FEAT-03 (Proposal Acceptance) | References (inbound) | Accepted state and the request-changes note both originate here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| proposal_detail_viewed | current status | Screen finishes loading | N/A -- no Stage 2 metric measures detail-screen views; retained for baseline usage visibility |
| proposal_start_from_copy_selected | -- | Nadia taps "Start from Copy" | supports success-metrics.md: "Proposal Send Speed" (a faster path into a new draft than a blank form) |

## Acceptance Criteria

**FEAT-02.SPEC-003-AC-01:** Given Nadia opens the proposal area of a project with no proposal, when the screen loads, then it shows "This project has no proposal yet." with a "Draft a Proposal" button.

**FEAT-02.SPEC-003-AC-02:** Given Nadia taps "Draft a Proposal", then she is navigated to FEAT-02.SPEC-001 with a blank form.

**FEAT-02.SPEC-003-AC-03:** Given a project has a Draft proposal, when Nadia opens the Detail screen, then she sees the "Draft" status badge with Edit, Send, Discard, and Start from Copy actions.

**FEAT-02.SPEC-003-AC-04:** Given a project has a Sent proposal, when Nadia opens the Detail screen, then she sees the "Sent" status badge, the sent_at timestamp, and Edit and Resend actions (no Discard, no Start from Copy).

**FEAT-02.SPEC-003-AC-05:** Given a project has an Accepted proposal, when Nadia opens the Detail screen, then she sees the "Accepted" status badge with accepted_at and accepted_by, and no action buttons.

**FEAT-02.SPEC-003-AC-06:** Given a Sent proposal has a request-changes note attached, when Nadia opens the Detail screen, then the note's text, author, and posted date are shown.

**FEAT-02.SPEC-003-AC-07:** Given Nadia taps "Resend" on a Sent proposal, then FEAT-02.SPEC-007 is triggered and the toast "Proposal link resent to {Primary Contact name}." appears.

**FEAT-02.SPEC-003-AC-08:** Given Nadia taps "Discard" on a Draft, when she confirms in the dialog, then FEAT-02.SPEC-009 is triggered and the screen shows the empty state afterward.

**FEAT-02.SPEC-003-AC-09:** Given Nadia opens the change-request email from FEAT-03, when she follows the link, then she lands on this screen with the request-changes note visible.

**FEAT-02.SPEC-003-AC-10:** Given Dana (Support Operator) opens this screen inside a logged support session, when she views it, then she sees only the status and history, with no scope description, price, request-changes note text, or action buttons.

**FEAT-02.SPEC-003-AC-11:** Given the screen is loading, then skeleton placeholders appear for the status card and content summary until data arrives.

**FEAT-02.SPEC-003-AC-12:** Given loading the proposal's data fails, then the error banner "Could not load this proposal." appears with a Retry button.

**FEAT-02.SPEC-003-AC-13:** Given Nadia loses connectivity after this screen has loaded, then the banner "You're offline -- showing the last loaded version of this proposal." appears and all action buttons are disabled.

**FEAT-02.SPEC-003-AC-14:** Given the proposal was discarded in another session, when Nadia returns to this screen, then it re-fetches and shows the empty state rather than the stale Draft.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 8 (empty, draft, sent, voided, accepted, loading, error, offline) | 8 |
| Business Rules | 4 | 4 |
| Edge Cases | 4 | 4 |



# Screen Spec: Reuse Proposal Picker

## Overview

**Name:** Reuse Proposal Picker
**ID:** FEAT-02.SPEC-004
**Type:** Screen
**Purpose:** Nadia browses her earlier proposals across all clients and projects and selects one to start a new draft as a copy.
**Parent Feature:** FEAT-02 -- Proposal Creation & Sending

## Scope and Non-Goals

**In Scope:**
- Listing Nadia's earlier proposals across every client and project as copy sources
- Selecting a source proposal to start a new Draft
- Staying responsive as the freelancer's cross-project proposal history grows over years of use

**Non-Goals:**
- Performing the copy itself -- owned by FEAT-02.SPEC-008 (Create Draft From Copy), triggered once a source is selected here.
- Browsing or comparing voided versions of a proposal -- intentional omission per the Feature Breakdown Brief's Non-Goals; this picker lists only each project's current (non-voided) proposal as a copy source, since voided content carries no forward value as a starting point.
- A configurable proposal template library -- excluded per scope-boundaries.md (SC-11); reuse of a real earlier proposal replaces the need for a template builder.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-02.SPEC-003 (Proposal Detail) | Nadia taps "Start from a copy" on a Draft proposal | Target project reference (the project the new Draft will belong to) |
| FEAT-02.SPEC-001 (Proposal Draft Editor) | Nadia taps "Start from a copy" from within an already-open blank editor | Target project reference |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen: every earlier proposal across her clients and projects | Select a source proposal | -- |
| Owen (Client Primary Contact) | No | No | This screen exists only in the freelancer's own workspace; no route into it exists from the client portal. |
| Priya (Client Reviewer Contact) | No | No | Same as Owen. |
| Dana (Support Operator) | No | No | Reuse is a freelancer-authoring action with no support-relevant read value beyond what FEAT-02.SPEC-003 already shows per proposal; this screen is not reachable inside a support session. |
| Unauthenticated | No | No | Redirected to freelancer sign-in. |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." The target project context is preserved and restored after re-authentication succeeds. |

## Layout and Content

**Header:** Screen title "Start from a Proposal" with a back arrow (returns to the entry point) at the left. A search input sits below the title, filtering by client or project name.

**Body:** A scrollable list of earlier proposals, one row per project's current (non-voided) proposal, ordered most-recently-sent first:
- Client name and project name
- Status (Sent, Accepted, or Voided-with-current-replacement is never listed since only the current version appears) and its date (sent_at or accepted_at)
- Price and currency
- A one-line scope description excerpt

**Footer:** None -- selecting a row is the sole action.

### Responsive Behavior

- **Compact:** Single-column list, full width, each row stacked (client/project on one line, status/date/price on the next).
- **Medium size class and above:** List remains single-column, capped at a consistent platform-wide width and horizontally centered; each row lays its fields out on one line rather than stacking.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate back to the entry point (FEAT-02.SPEC-003 or FEAT-02.SPEC-001) | Screen closes | Animated transition back |
| Search input | Type | Filters the list to matching client or project names | List updates | List re-renders to filtered results, or "No proposals match {query}." if none |
| Proposal row | Tap | Triggers FEAT-02.SPEC-008 (Create Draft From Copy) with the selected proposal as source | Row shows brief loading state | On completion, navigates to FEAT-02.SPEC-001 pre-filled with the copied content |

### Accessibility Notes

- **Focus order:** Back arrow -> search input -> proposal rows in list order.
- **Filter announcements:** The result count is announced to assistive technology when the search filter changes ("12 proposals" / "No proposals match {query}.").
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loaded (with proposals) | Full list of earlier proposals | Screen opens and at least one prior proposal exists | Nadia selects a row or navigates away |
| Empty | Message: "You don't have any earlier proposals to start from yet." with no list | Screen opens and no prior Sent, Voided, or Accepted proposal exists anywhere in the freelancer's account | N/A -- remains until a first proposal is sent elsewhere |
| No Search Results | Message: "No proposals match {query}." | Search filter matches zero rows | Nadia clears or changes the search query |
| Loading | Skeleton placeholder rows | Screen is opening and the list has not yet arrived | Data loads and the Loaded (with proposals) or Empty state renders, per whether any prior proposal exists |
| Error | Error banner: "Could not load your earlier proposals." with a Retry button | Loading the list fails | Nadia taps Retry, or connectivity returns and a re-fetch succeeds |
| Offline/Degraded | Banner: "You're offline -- showing the last loaded list." Selecting a row is disabled until connectivity returns, since the copy automation requires it | Connectivity is lost while this screen is open | Connectivity returns and selection re-enables |

## Validation Rules

Not applicable -- this screen has no user input fields beyond the non-validated search filter.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | Entry point (FEAT-02.SPEC-003 or FEAT-02.SPEC-001) | -- |
| Row selected, copy completes | FEAT-02.SPEC-001 (Proposal Draft Editor), pre-filled | -- |

## Data Model

**Creates:** None.
**Reads:** Proposal -- lists the current (non-voided) proposal per project across all of Nadia's clients: scope_description (excerpt), price, currency, status, sent_at, accepted_at. Project and Client -- names for display.
**Updates:** None.
**Deletes:** None.

## Business Rules

- Only each project's current (non-voided) proposal appears as a copy source -- a project's voided prior versions are never separately listed, consistent with the feature's exclusion of a browsable version-history view.
- The list spans every client and project the freelancer has ever sent a proposal for, not only the target project's own client, per the Feature Breakdown Brief's Key Capabilities ("Reuse an earlier proposal... from any previous proposal").
- Selecting a row always creates a new Draft for the target project passed in from the entry point -- it never modifies or navigates directly to the source proposal.

## Edge Cases

- **The freelancer has 200+ proposals across years of use** -- The list stays responsive per the Feature Breakdown Brief's Non-Functional Notes; the list loads incrementally (older entries load as Nadia scrolls) rather than all at once.
- **Nadia searches for a client name that matches zero proposals** -- "No proposals match {query}." is shown; the search input remains editable.
- **Nadia selects a row and the copy automation fails** -- The row's loading state clears, an inline error appears on the row: "Could not start from this proposal. Try again.", and Nadia remains on the picker.
- **The source proposal selected is later voided or discarded in another session before the copy completes** -- The copy automation (FEAT-02.SPEC-008) reads the source proposal's content at the moment of selection; since Comment/Proposal history is immutable once sent (XBR-04), a Voided source's last-sent content is still valid to copy, so no failure occurs solely because the source has since been voided.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-001 (Proposal Draft Editor) | Navigation (inbound/outbound) | Entry point when reached from the editor; destination after a successful copy |
| FEAT-02.SPEC-003 (Proposal Detail) | Navigation (inbound) | Entry point via "Start from a copy" on a Draft |
| FEAT-02.SPEC-008 (Create Draft From Copy) | Triggers (outbound) | Row selection triggers the copy automation |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| reuse_picker_opened | entry source (proposal detail / draft editor) | Screen finishes loading | supports success-metrics.md: "Proposal Send Speed" |
| reuse_picker_source_selected | source proposal age in days | Nadia selects a row | supports success-metrics.md: "Proposal Send Speed" |

## Acceptance Criteria

**FEAT-02.SPEC-004-AC-01:** Given Nadia has 3 earlier proposals across 2 clients, when she opens the Reuse Proposal Picker, then all 3 appear, ordered most-recently-sent first, each showing client, project, status, date, price, and a scope excerpt.

**FEAT-02.SPEC-004-AC-02:** Given Nadia has never sent a proposal, when she opens the picker, then it shows "You don't have any earlier proposals to start from yet." with no list.

**FEAT-02.SPEC-004-AC-03:** Given Nadia types a client name into the search input that matches one proposal, then the list filters to show only that proposal.

**FEAT-02.SPEC-004-AC-04:** Given Nadia searches for a name matching no proposal, then "No proposals match {query}." is shown.

**FEAT-02.SPEC-004-AC-05:** Given Nadia taps a proposal row, when the copy completes, then she is navigated to FEAT-02.SPEC-001 with scope, price, and currency pre-filled from the selected proposal.

**FEAT-02.SPEC-004-AC-06:** Given Nadia taps a proposal row and the copy automation fails, then an inline error "Could not start from this proposal. Try again." appears on that row and she remains on the picker.

**FEAT-02.SPEC-004-AC-07:** Given the picker is loading, then skeleton placeholder rows appear until data arrives.

**FEAT-02.SPEC-004-AC-08:** Given loading the list fails, then the error banner "Could not load your earlier proposals." appears with a Retry button.

**FEAT-02.SPEC-004-AC-09:** Given Nadia loses connectivity while viewing the picker, then the banner "You're offline -- showing the last loaded list." appears and row selection is disabled.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 3 | 3 |
| States | 6 (loaded, empty, no search results, loading, error, offline) | 6 |
| Business Rules | 3 | 3 |
| Edge Cases | 4 | 4 |



# Automation Spec: Proposal Send

## Overview

**Name:** Proposal Send
**ID:** FEAT-02.SPEC-005
**Type:** Automation
**Purpose:** Validates and transitions a Draft proposal to Sent, recording the send timestamp and locking the payment schedule reference so the client always sees the schedule as it stood at send.
**Parent Feature:** FEAT-02 -- Proposal Creation & Sending

## Scope and Non-Goals

**In Scope:**
- Re-validating send eligibility at the moment of send (required fields, positive price, client Primary-contact requirement)
- Transitioning the Proposal from Draft to Sent
- Recording sent_at and locking the payment_schedule_reference
- Triggering the Proposal Sent email

**Non-Goals:**
- Editing or discarding the proposal -- owned by FEAT-02.SPEC-001, FEAT-02.SPEC-006, and FEAT-02.SPEC-009; this automation only performs the Draft-to-Sent transition.
- Composing or delivering the email itself -- owned by FEAT-02.SPEC-011 (Proposal Sent/Resent Email); this automation only triggers it on success.
- Re-sending an already-Sent proposal's link -- owned by FEAT-02.SPEC-007 (Proposal Resend); this automation only handles the first Draft-to-Sent transition for a given version.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nadia taps Send on the Preview screen | FEAT-02.SPEC-002 (Proposal Preview) | The proposal is currently in Draft status | Proposal id, scope_description, price, currency, payment_schedule_reference, owning project and client |

## Processing Logic

1. Read the Proposal's current record by id. If no record is found, the Draft was discarded (FEAT-02.SPEC-009) since Preview was opened -- stop and report the discarded-draft outcome (see Edge Cases). If a record is found but its status is not Draft, stop and report the state-mismatch failure -- these are reported as distinct outcomes so the message Nadia sees always matches what actually happened, never assuming "already sent" when the Draft was instead discarded.
2. Re-run the required-field and positive-price checks defined in FEAT-02.SPEC-010.
3. Check that the project's owning Client has at least one Primary contact (XBR-07); if not, stop and report the Primary-contact failure.
4. Check that the project has no other active (non-voided, non-Draft-being-sent) proposal -- confirming the one-active-proposal-per-project rule still holds at the moment of send.
5. If all checks pass, transition the Proposal's status from Draft to Sent.
6. Record sent_at as the current date and time.
7. Lock the proposal's payment_schedule_reference to the project's Payment Schedule as it currently stands, so later schedule adjustments do not retroactively change what this sent version references (consistent with the Payment Schedule's own non-retroactive adjustment rule).
8. Trigger FEAT-02.SPEC-011 (Proposal Sent/Resent Email) to the client's Primary contact(s).
9. Write an Activity Log Entry recording the send (XBR-05), owned by FEAT-13.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Send succeeds | All eligibility checks pass | Proposal status -> Sent; sent_at set; payment_schedule_reference locked | Preview navigates to FEAT-02.SPEC-003 showing Sent status; confirmation "Proposal sent to {Primary Contact name}." | FEAT-02.SPEC-003, FEAT-02.SPEC-011, FEAT-13 |
| Blocked -- no Primary contact | Client has zero Primary contacts (XBR-07) | None | Preview shows the Send Blocked banner "This client has no Primary contact yet." with a link into FEAT-18 | FEAT-02.SPEC-002 |
| Blocked -- invalid fields | Scope description empty, or price missing/non-positive | None | Preview (or, if reached directly, the editor) shows the specific field error per FEAT-02.SPEC-010 | FEAT-02.SPEC-001, FEAT-02.SPEC-002 |
| Blocked -- state mismatch | The proposal record still exists but is no longer in Draft status (e.g., already sent from another session) | None | Preview shows: "This proposal has already been sent. Viewing the current version." and navigates to FEAT-02.SPEC-003 | FEAT-02.SPEC-002, FEAT-02.SPEC-003 |
| Blocked -- draft discarded | The Draft's record no longer exists because it was discarded (FEAT-02.SPEC-009) concurrently, before this Send call's status check | None | Preview shows: "This draft no longer exists -- it was discarded." and navigates to FEAT-02.SPEC-003 showing the empty state | FEAT-02.SPEC-002, FEAT-02.SPEC-003, FEAT-02.SPEC-009 |
| Failure | Processing error after eligibility passed (e.g., connectivity lost mid-operation) | No partial state -- the proposal remains Draft; nothing is sent | Preview shows an error banner with Retry; the Draft is unaffected | FEAT-02.SPEC-002 |

## Data Model

**Reads:** Proposal -- status, scope_description, price, currency, payment_schedule_reference. Client -- Client Contact roster (for the Primary-contact check). Payment Schedule -- current structure, to lock into the reference.
**Creates:** Activity Log Entry (via FEAT-13) recording the send event.
**Updates:** Proposal -- status (Draft -> Sent), sent_at, payment_schedule_reference (locked to the current schedule).
**Deletes:** None.

## Business Rules

- Send eligibility is fully re-checked at send time, not only when the Draft was last saved -- conditions (client Primary contact, field validity, one-active-proposal cap) can change between drafting and sending.
- A proposal cannot be sent until the client has at least one Primary contact (XBR-07).
- The payment_schedule_reference is locked at the moment of send; later, non-retroactive Payment Schedule adjustments (FEAT-04) never change what an already-sent proposal references.
- Sending is the only path that transitions a Proposal from Draft to Sent -- FEAT-02.SPEC-006 (Void & Resend) creates a new Sent version through its own path rather than reusing this automation on an existing Draft.

## Edge Cases

- **The proposal is no longer in Draft status when Send is invoked because it was already sent from another session** -- Reported as the state-mismatch outcome ("This proposal has already been sent."); the Preview screen redirects to FEAT-02.SPEC-003 to show the current Sent version.
- **The proposal's Draft record no longer exists when Send is invoked because it was discarded concurrently (FEAT-02.SPEC-009)** -- Reported as the distinct discarded-draft outcome ("This draft no longer exists -- it was discarded."), never the state-mismatch message, since "already been sent" would misinform Nadia about what actually happened. The Preview screen redirects to FEAT-02.SPEC-003's empty state.
- **The client's only Primary contact is removed between Preview load and Send tap** -- Blocked with the Primary-contact outcome; no partial Sent state is created.
- **Concurrent trigger firing (Send tapped from two open Preview sessions for the same Draft at effectively the same time)** -- Only the first to pass the Draft-status check transitions the proposal to Sent; the second finds the proposal already Sent and receives the state-mismatch outcome, redirected to view the current (now Sent) version. No duplicate Sent version and no duplicate email are produced.
- **A trigger fires while a previous send for the same proposal is still in flight** -- The triggering Preview screen's Send button is disabled during the operation (FEAT-02.SPEC-002), so a second send for the same Draft cannot be initiated from the same session; a second session is covered by the concurrent-trigger-firing case above.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-002 (Proposal Preview) | Triggered by (inbound) | Send tap, after screen-level eligibility passes, triggers this automation |
| FEAT-02.SPEC-003 (Proposal Detail) | Affects (outbound) | Shows the resulting Sent status, or the state-mismatch redirect |
| FEAT-02.SPEC-010 (Proposal Validation & Business Rules) | References (inbound) | Field validation and send-eligibility rules re-checked here |
| FEAT-02.SPEC-011 (Proposal Sent/Resent Email) | Affects (outbound) | Triggered on send success to deliver the proposal link |
| FEAT-18 (Client Contact Management & Roles) | References (outbound) | Primary-contact requirement checked against this feature's contact roster |
| FEAT-13 (Immutable Activity & Audit Trail) | Affects (outbound) | Writes the send event to the trail (XBR-05) |
| FEAT-02.SPEC-009 (Discard Draft) | References (inbound) | A concurrent discard produces the distinct discarded-draft outcome rather than the state-mismatch outcome |

## Analytics and Success Signals

- **proposal_sent** (project id, price, currency, time from draft creation to send in minutes) -- supports success-metrics.md: "Proposal Send Speed"
- **proposal_send_blocked_no_primary_contact** (client id) -- N/A -- no Stage 2 metric measures this specific block reason; retained so the XBR-07 gate's real-world frequency is observable
- **proposal_send_state_mismatch** (-- ) -- N/A -- no Stage 2 metric measures concurrent-send collisions; retained for operational visibility into how often the race occurs
- **proposal_send_draft_discarded** (-- ) -- N/A -- no Stage 2 metric measures this specific race; retained for operational visibility into how often a concurrent discard is the cause of a blocked send, as distinct from an already-sent collision

## Acceptance Criteria

**FEAT-02.SPEC-005-AC-01:** Given Nadia's Draft proposal has valid scope, a positive price, and the client has a Primary contact, when Send is triggered from FEAT-02.SPEC-002, then the proposal transitions to Sent, sent_at is recorded, and FEAT-02.SPEC-011 is triggered.

**FEAT-02.SPEC-005-AC-02:** Given the client has no Primary contact, when Send is triggered, then the proposal remains Draft and the Blocked -- no Primary contact outcome is returned.

**FEAT-02.SPEC-005-AC-03:** Given the proposal's scope description is empty, when Send is triggered, then the proposal remains Draft and the Blocked -- invalid fields outcome is returned.

**FEAT-02.SPEC-005-AC-04:** Given the proposal was already sent from another session before this Send call reaches the status check, when this automation runs, then it returns the state-mismatch outcome ("This proposal has already been sent.") and no second Sent version or email is created.

**FEAT-02.SPEC-005-AC-05:** Given the Draft was discarded (FEAT-02.SPEC-009) from another session before this Send call reaches the status check, when this automation runs, then it returns the distinct discarded-draft outcome ("This draft no longer exists -- it was discarded."), not the state-mismatch message, and Preview navigates to FEAT-02.SPEC-003's empty state.

**FEAT-02.SPEC-005-AC-06:** Given Send succeeds, then the proposal's payment_schedule_reference is locked to the Payment Schedule as it stood at that moment.

**FEAT-02.SPEC-005-AC-07:** Given a processing failure occurs after eligibility checks pass but before the transition completes, then the proposal remains Draft and no email is sent.

**FEAT-02.SPEC-005-AC-08:** Given two Preview sessions for the same Draft trigger Send at effectively the same time, then only the first transitions the proposal to Sent and the second receives the state-mismatch outcome.

**FEAT-02.SPEC-005-AC-09:** Given Send succeeds, then an Activity Log Entry recording the send is written (FEAT-13).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 6 (success, no primary contact, invalid fields, state mismatch, draft discarded, failure) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 4 | 4 |



# Automation Spec: Proposal Edit-Before-Acceptance Void & Resend

## Overview

**Name:** Proposal Edit-Before-Acceptance Void & Resend
**ID:** FEAT-02.SPEC-006
**Type:** Automation
**Purpose:** When Nadia edits a Sent-but-unaccepted proposal, voids the prior version and creates and sends the new one, so the client is never shown an outdated price (XBR-06).
**Parent Feature:** FEAT-02 -- Proposal Creation & Sending

## Scope and Non-Goals

**In Scope:**
- Re-validating the edited content and send eligibility
- Voiding the prior Sent version
- Creating and sending the new version with the edited content
- Preserving the void-then-send sequence as a single atomic outcome from the user's perspective

**Non-Goals:**
- Editing an unsent Draft -- an unsent Draft is simply updated in place by FEAT-02.SPEC-001; this automation only applies once a proposal has been Sent.
- Handling the case where the proposal has already been accepted -- an accepted proposal is immutable (XBR-04); this automation refuses to run against an Accepted proposal rather than voiding accepted, evidentiary content.
- Presenting a version-history or diff view of the voided version -- excluded per the Feature Breakdown Brief's Non-Goals; the voided version is retained only as immutable evidence, never surfaced for browsing.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nadia taps "Save & Resend" on the editor in edit mode | FEAT-02.SPEC-001 (Proposal Draft Editor) | The target proposal is currently in Sent status (not yet Accepted or already Voided) | Existing Sent proposal id, edited scope_description, edited price, currency (read-only, unchanged), payment_schedule_reference |

## Processing Logic

1. Read the target Proposal's current status; if it is not Sent, stop and report a state-mismatch failure (see Edge Cases -- covers both the already-Accepted and already-Voided cases).
2. Re-run the required-field and positive-price checks on the edited content, per FEAT-02.SPEC-010.
3. Re-check that the project's owning Client still has at least one Primary contact (XBR-07).
4. If all checks pass, mark the existing Sent proposal's status as Voided (an immutable, permanent state -- XBR-04).
5. Create a new Proposal record for the same project with the edited scope_description, price, and currency, and the project's current payment_schedule_reference.
6. Transition the new Proposal directly to Sent, recording its own sent_at.
7. Record the new Proposal's predecessor reference so the void-and-resend relationship is traceable (distinct from copied_from, which records only freelancer-initiated reuse via FEAT-02.SPEC-008).
8. Trigger FEAT-02.SPEC-011 (Proposal Sent/Resent Email) to the client's Primary contact(s) with the new version's content.
9. Write an Activity Log Entry recording both the void and the new send as one logged event (XBR-05), owned by FEAT-13.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Void & resend succeeds | All eligibility checks pass and the target proposal is Sent | Prior proposal -> Voided; new Proposal created and set to Sent with its own sent_at | Editor navigates to FEAT-02.SPEC-003 showing the new Sent version; confirmation "Proposal updated and resent to {Primary Contact name}." | FEAT-02.SPEC-003, FEAT-02.SPEC-011, FEAT-13 |
| Blocked -- already accepted | The target proposal's status is Accepted | None | Editor shows: "This proposal has already been accepted and can no longer be edited." with a "View Current Status" link to FEAT-02.SPEC-003 | FEAT-02.SPEC-001 |
| Blocked -- already voided or superseded | The target proposal's status is already Voided (a second concurrent edit lost the race) | None | Editor shows: "This proposal was already edited and resent. Review the current version." and reloads the current (new) version's content | FEAT-02.SPEC-001, FEAT-02.SPEC-003 |
| Blocked -- invalid fields or no Primary contact | Scope description empty, price missing/non-positive, or the client has no Primary contact | None | Editor (or, if applicable, the send-eligibility banner) shows the specific error per FEAT-02.SPEC-010 | FEAT-02.SPEC-001 |
| Failure | Processing error after eligibility passed (e.g., connectivity lost mid-operation) | No partial state -- the prior Sent proposal remains Sent and unvoided; no new version is created | Editor shows an error banner with Retry; the edited content is preserved locally for retry | FEAT-02.SPEC-001 |

## Data Model

**Reads:** Proposal -- current status, scope_description, price, currency of the target version. Client -- Primary contact roster. Payment Schedule -- current structure for the new version's reference.
**Creates:** A new Proposal record (status Sent, its own sent_at, payment_schedule_reference locked at this moment) with a predecessor reference to the voided version. Activity Log Entry (via FEAT-13) recording the void-and-resend.
**Updates:** The prior Proposal record -- status set to Voided (terminal, immutable thereafter).
**Deletes:** None -- the voided version is retained as evidence (XBR-04), never deleted.

## Business Rules

- A Sent-but-unaccepted proposal is never updated in place -- every edit after send produces a new version through void-and-resend (XBR-06).
- A voided proposal cannot be accepted; if Owen attempts to accept the version that was just voided, FEAT-03 refuses the acceptance and shows him the current version (joint contention rule, dependency map).
- An edit attempt against an already-Accepted proposal is always refused -- accepted proposals are immutable (XBR-04) -- regardless of how the edit was initiated.
- A project retains at most one active (non-voided) proposal at any time -- the new version becomes the sole active proposal for the project the moment it is created.
- The void and the new send are treated as one logged event in the Activity Log Entry, not two independent entries, so the trail reads as a single edit-and-resend action.

## Edge Cases

- **Owen accepts the proposal in the moment between Nadia tapping "Save & Resend" and this automation's status check completing** -- The status check reads Accepted and the automation reports the "already accepted" blocked outcome; Owen's acceptance stands as recorded by FEAT-03, and nothing is voided.
- **Concurrent trigger firing (Nadia edits the same Sent proposal from two open sessions and both trigger Save & Resend at effectively the same time)** -- Only the first to pass the Sent-status check proceeds; it voids the original and creates the new Sent version. The second finds the proposal already Voided and receives the "already voided or superseded" blocked outcome, with its editor reloading the new current version's content -- the second session's edits are not silently merged or lost, but the user must re-apply them against the new version if still wanted.
- **A trigger fires while a previous void-and-resend run for the same proposal is still in flight** -- The triggering editor's "Save & Resend" button is disabled during the operation (FEAT-02.SPEC-001), preventing a second run for the same session; a second session racing in is covered by the concurrent-trigger-firing case above.
- **The client's Primary contact changes between the original send and this edit** -- The new version is sent to the client's current Primary contact(s) at the moment this automation runs, not to whoever received the original Sent version.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-001 (Proposal Draft Editor) | Triggered by (inbound) | "Save & Resend" in edit mode triggers this automation |
| FEAT-02.SPEC-003 (Proposal Detail) | Affects (outbound) | Shows the new Sent version, or the blocked-outcome redirect |
| FEAT-02.SPEC-010 (Proposal Validation & Business Rules) | References (inbound) | Field validation, send eligibility, and the post-acceptance immutability rule |
| FEAT-02.SPEC-011 (Proposal Sent/Resent Email) | Affects (outbound) | Triggered on success to deliver the new version's link |
| FEAT-03 (Proposal Acceptance) | References (inbound/outbound) | Joint owner of the accept-vs-void contention resolution; the request-changes flow (XBR-26) that often precedes this automation |
| FEAT-13 (Immutable Activity & Audit Trail) | Affects (outbound) | Writes the void-and-resend event to the trail (XBR-05) |

## Analytics and Success Signals

- **proposal_edited_before_acceptance** (project id, days between original send and this edit) -- supports success-metrics.md: "Proposal Send Speed" (a fast edit-and-resend keeps the overall proposal-to-acceptance loop quick)
- **proposal_void_resend_blocked_already_accepted** (-- ) -- N/A -- no Stage 2 metric measures this specific race outcome; retained so the accept-vs-edit contention's real-world frequency is observable
- **proposal_void_resend_blocked_already_voided** (-- ) -- N/A -- no Stage 2 metric measures concurrent-edit collisions; retained for operational visibility

## Acceptance Criteria

**FEAT-02.SPEC-006-AC-01:** Given Nadia edits a Sent-but-unaccepted proposal's price and taps "Save & Resend", when validation passes, then the prior version is set to Voided, a new Proposal is created and Sent with the edited content, and FEAT-02.SPEC-011 is triggered.

**FEAT-02.SPEC-006-AC-02:** Given the target proposal's status is Accepted at the moment of the check, when this automation runs, then it is refused with "This proposal has already been accepted and can no longer be edited." and no void occurs.

**FEAT-02.SPEC-006-AC-03:** Given the target proposal was already voided by a concurrent edit, when this automation runs for the second session, then it is refused with "This proposal was already edited and resent. Review the current version." and the second session's editor reloads the new current version.

**FEAT-02.SPEC-006-AC-04:** Given the edited scope description is empty, when Save & Resend is triggered, then the void does not occur and the field-level error is shown per FEAT-02.SPEC-010.

**FEAT-02.SPEC-006-AC-05:** Given the client has no Primary contact at the moment of this edit, when Save & Resend is triggered, then the void does not occur and the Primary-contact eligibility error is shown.

**FEAT-02.SPEC-006-AC-06:** Given Owen accepts the proposal in the instant before this automation's status check runs, when the automation proceeds, then it reads Accepted and refuses with the already-accepted outcome, leaving Owen's acceptance intact.

**FEAT-02.SPEC-006-AC-07:** Given two sessions trigger Save & Resend on the same Sent proposal at effectively the same time, then only the first succeeds and the second receives the already-voided-or-superseded outcome.

**FEAT-02.SPEC-006-AC-08:** Given a processing failure occurs after eligibility passes but before the transition completes, then the prior proposal remains Sent and unvoided, and no new version is created.

**FEAT-02.SPEC-006-AC-09:** Given void-and-resend succeeds, then a single Activity Log Entry records both the void and the new send as one event (FEAT-13).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 5 (success, already accepted, already voided, invalid/blocked, failure) | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 4 | 4 |



# Automation Spec: Proposal Resend

## Overview

**Name:** Proposal Resend
**ID:** FEAT-02.SPEC-007
**Type:** Automation
**Purpose:** Re-sends the link for an already-Sent proposal without creating a new version, for the case where Owen simply cannot find the original email.
**Parent Feature:** FEAT-02 -- Proposal Creation & Sending

## Scope and Non-Goals

**In Scope:**
- Re-triggering delivery of the current Sent version's link to the client's Primary contact(s)
- Rate-limiting rapid repeated resends

**Non-Goals:**
- Creating a new proposal version or changing its content -- that is FEAT-02.SPEC-006 (Void & Resend), triggered only by an actual content edit; this automation never alters scope, price, or currency.
- Composing the email content -- owned by FEAT-02.SPEC-011 (Proposal Sent/Resent Email); this automation only triggers delivery of the existing version's content.
- Resending a Draft, Voided, or Accepted proposal -- resend applies only to a currently Sent proposal, since those are the only statuses awaiting the client's decision.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nadia taps "Resend" on the Detail screen | FEAT-02.SPEC-003 (Proposal Detail) | The proposal is currently in Sent status | Proposal id, existing scope_description, price, currency, sent_at, owning client's Primary contact(s) |

## Processing Logic

1. Read the Proposal's current status; if it is not Sent, stop and report a state-mismatch failure (see Edge Cases).
2. Check the resend rate limit for this proposal (see Business Rules); if exceeded, stop and report the rate-limit outcome.
3. Confirm the client still has at least one Primary contact (XBR-07); if not, stop and report the Primary-contact failure.
4. Trigger FEAT-02.SPEC-011 (Proposal Sent/Resent Email) to the client's current Primary contact(s), carrying the existing version's content unchanged.
5. Record the resend event (timestamp) against the proposal for rate-limit tracking.
6. Write an Activity Log Entry recording the resend (XBR-05), owned by FEAT-13.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Resend succeeds | Proposal is Sent, rate limit not exceeded, client has a Primary contact | Resend event recorded for rate-limit tracking; no change to proposal content, status, or sent_at | Toast on FEAT-02.SPEC-003: "Proposal link resent to {Primary Contact name}." | FEAT-02.SPEC-003, FEAT-02.SPEC-011, FEAT-13 |
| Blocked -- state mismatch | The proposal is no longer Sent (e.g., already voided, discarded, or accepted since the Detail screen loaded) | None | Detail screen re-fetches and shows the current status; the Resend action is not available in that status | FEAT-02.SPEC-003 |
| Blocked -- rate limited | A resend for this proposal was already sent within the rate-limit window | None | Toast: "You already resent this proposal recently. Try again in a few minutes." | FEAT-02.SPEC-003 |
| Blocked -- no Primary contact | Client's last Primary contact was removed since the original send | None | Detail screen shows: "This client has no Primary contact yet." with a link into FEAT-18 | FEAT-02.SPEC-003 |
| Failure | Processing error (e.g., connectivity lost mid-operation) | None | Toast: "Could not resend the proposal link. Try again." | FEAT-02.SPEC-003 |

## Data Model

**Reads:** Proposal -- status, scope_description, price, currency (unchanged, read-only here). Client -- Primary contact roster.
**Creates:** A resend-event record used for rate-limit tracking (not a new Proposal version). Activity Log Entry (via FEAT-13).
**Updates:** None on the Proposal record itself -- status, content, and sent_at are all untouched by a resend.
**Deletes:** None.

## Business Rules

- Resend never creates a new proposal version and never changes sent_at -- it only re-delivers the existing Sent version's link (distinguishing it from FEAT-02.SPEC-006, which resends because content changed).
- Resend is limited to one successful resend per proposal per platform parameter: `proposal-resend-cooldown-minutes`, to prevent accidental repeated sends from generating multiple emails for the same unchanged content.
- Resend requires the same client Primary-contact eligibility as the original send (XBR-07).

## Edge Cases

- **The proposal is voided or accepted between the Detail screen loading and Nadia tapping Resend** -- Reported as the state-mismatch outcome; Resend is not available once the Detail screen's next load reflects the new status.
- **Nadia taps Resend twice within the cooldown window** -- The second attempt is blocked with the rate-limited outcome; no second email is sent.
- **Concurrent trigger firing (Resend tapped from two open Detail sessions for the same proposal at effectively the same time)** -- Only the first passes the rate-limit check within the window; the second receives the rate-limited outcome. At most one email is delivered for the pair.
- **A trigger fires while a previous resend for the same proposal is still in flight** -- The rate-limit check (Business Rules) also functions as the in-flight guard: a resend in progress is treated as having just occurred, so a same-instant second trigger is rejected as rate-limited rather than double-sending.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-003 (Proposal Detail) | Triggered by (inbound) | "Resend" action on a Sent proposal triggers this automation |
| FEAT-02.SPEC-011 (Proposal Sent/Resent Email) | Affects (outbound) | Triggered on success to re-deliver the existing version's link |
| FEAT-18 (Client Contact Management & Roles) | References (outbound) | Primary-contact requirement checked against this feature's contact roster |
| FEAT-13 (Immutable Activity & Audit Trail) | Affects (outbound) | Writes the resend event to the trail (XBR-05) |

## Analytics and Success Signals

- **proposal_resent** (project id, days since original sent_at) -- supports success-metrics.md: "Notification Delivery Reliability" (measures how often a client-facing email had to be manually re-delivered)
- **proposal_resend_rate_limited** (-- ) -- N/A -- no Stage 2 metric measures rate-limit hits; retained for operational visibility into resend friction

## Acceptance Criteria

**FEAT-02.SPEC-007-AC-01:** Given a proposal is Sent and the client has a Primary contact, when Nadia taps "Resend", then FEAT-02.SPEC-011 is triggered and the toast "Proposal link resent to {Primary Contact name}." appears, with the proposal's status, content, and sent_at unchanged.

**FEAT-02.SPEC-007-AC-02:** Given the proposal was voided since the Detail screen loaded, when Nadia taps "Resend", then the automation reports the state-mismatch outcome and the screen re-fetches to show the current status.

**FEAT-02.SPEC-007-AC-03:** Given Nadia resent the proposal moments ago, when she taps "Resend" again within the cooldown window, then the toast "You already resent this proposal recently. Try again in a few minutes." appears and no email is sent.

**FEAT-02.SPEC-007-AC-04:** Given the client's last Primary contact was removed since the original send, when Nadia taps "Resend", then "This client has no Primary contact yet." appears with a link into FEAT-18.

**FEAT-02.SPEC-007-AC-05:** Given a processing failure occurs during resend, then the toast "Could not resend the proposal link. Try again." appears.

**FEAT-02.SPEC-007-AC-06:** Given two Detail sessions trigger Resend for the same proposal at effectively the same time, then at most one email is delivered and the second trigger receives the rate-limited outcome.

**FEAT-02.SPEC-007-AC-07:** Given Resend succeeds, then an Activity Log Entry recording the resend is written (FEAT-13).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 5 (success, state mismatch, rate limited, no primary contact, failure) | 5 |
| Business Rules | 3 | 3 |
| Edge Cases | 4 | 4 |



# Automation Spec: Create Draft From Copy

## Overview

**Name:** Create Draft From Copy
**ID:** FEAT-02.SPEC-008
**Type:** Automation
**Purpose:** Creates a new Draft proposal for the target project, pre-filled from a selected earlier proposal's scope, price, and currency.
**Parent Feature:** FEAT-02 -- Proposal Creation & Sending

## Scope and Non-Goals

**In Scope:**
- Copying scope_description and price from a selected source proposal into a new Draft
- Recording the copied_from reference for traceability
- Handling a source proposal in a different currency than the target project

**Non-Goals:**
- Selecting the source proposal -- owned by FEAT-02.SPEC-004 (Reuse Proposal Picker); this automation only acts once a source is chosen.
- Editing the copied content -- the freelancer edits the resulting Draft in FEAT-02.SPEC-001 like any other Draft; this automation only performs the initial copy.
- Copying the payment_schedule_reference -- the new Draft references the target project's own Payment Schedule (FEAT-04), never the source proposal's, since payment schedules are project-specific and the target project may have a different schedule or none yet.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nadia selects a source proposal row | FEAT-02.SPEC-004 (Reuse Proposal Picker) | The target project has no other active (non-voided) proposal | Source proposal id (scope_description, price, currency), target project reference |

## Processing Logic

1. Read the target project's current proposal state; if an active (Draft, Sent, or Accepted) proposal already exists for it, stop and report the one-active-proposal-cap failure.
2. Read the source proposal's scope_description and price.
3. Read the target project's own currency (FEAT-15).
4. Create a new Proposal record for the target project with status Draft: scope_description copied verbatim from the source; price copied as a numeric amount, expressed in the target project's currency without any conversion (see Business Rules); payment_schedule_reference set to the target project's own Payment Schedule (or left unset if the target project has none yet, matching a blank-start Draft).
5. Set copied_from to the source proposal's id.
6. Open the new Draft in FEAT-02.SPEC-001 (Proposal Draft Editor) for the freelancer to review and adjust before saving or sending.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Copy succeeds | Target project has no active proposal | New Proposal created (Draft), copied_from set | Editor opens pre-filled, with the copied-from banner shown | FEAT-02.SPEC-001 |
| Blocked -- active proposal exists | Target project already has a Draft, Sent, or Accepted proposal | None | Reuse Picker shows an inline error on the row: "This project already has a proposal. Open it from the project instead." | FEAT-02.SPEC-004 |
| Failure | Processing error (e.g., connectivity lost mid-copy) | None | Reuse Picker shows an inline error on the row: "Could not start from this proposal. Try again." and Nadia remains on the picker | FEAT-02.SPEC-004 |

## Data Model

**Reads:** Proposal (source) -- scope_description, price, currency. Project (target) -- currency, existing proposal state, Payment Schedule reference.
**Creates:** Proposal (new Draft) -- scope_description, price, currency (target project's own), payment_schedule_reference (target project's own), status Draft, copied_from (source proposal id).
**Updates:** None.
**Deletes:** None.

## Business Rules

- The one-active-proposal-per-project cap (dependency map: Project relationships) is checked before the copy is created -- a project already carrying a Draft, Sent, or Accepted proposal cannot receive a second one from this automation.
- Price is copied as a numeric amount only; if the source proposal's currency differs from the target project's currency, the number is carried over unconverted (no automatic currency conversion exists in the product, per XBR-18) and the freelancer is expected to review and adjust it in the editor before sending -- the copied-from banner and the visible currency field make the source's original currency and the target's current currency both apparent for that review.
- copied_from is set once at creation and is never altered afterward -- it is a permanent provenance reference to the source proposal, not a live link.

## Edge Cases

- **The source proposal's currency differs from the target project's currency** -- The price value is copied as-is; the editor displays the target project's currency next to the copied number so Nadia can adjust it before sending (see Business Rules). No automatic conversion is performed or implied.
- **The target project already has a Draft, Sent, or Accepted proposal by the time this automation runs (created concurrently in another session)** -- Reported as the blocked outcome; no duplicate active proposal is created for the project.
- **Concurrent trigger firing (two copy operations targeting the same project fire at effectively the same time)** -- Only the first to pass the one-active-proposal check creates the Draft; the second finds an active proposal already present and receives the blocked outcome.
- **A trigger fires while a previous copy for the same target project is still in flight** -- The Reuse Picker's row shows a loading state during the operation (FEAT-02.SPEC-004), preventing a second selection from the same session; a second session racing in is covered by the concurrent-trigger-firing case above.
- **The source proposal is later voided or discarded after this copy completes** -- Has no effect on the new Draft; copied_from is a point-in-time provenance reference, not a live dependency, and the new Draft is fully independent once created.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-004 (Reuse Proposal Picker) | Triggered by (inbound) | Row selection triggers this automation with the chosen source |
| FEAT-02.SPEC-001 (Proposal Draft Editor) | Affects (outbound) | The resulting Draft opens here for review before save or send |
| FEAT-15 (Currency & Tax Handling) | References (inbound) | Target project's currency governs the new Draft's currency |
| FEAT-04 (Milestone & Payment Schedule Setup) | References (inbound) | Target project's own Payment Schedule is referenced, never the source's |

## Analytics and Success Signals

- **proposal_created_from_copy** (source proposal age in days, currency match: same / different) -- supports success-metrics.md: "Proposal Send Speed"
- **proposal_copy_blocked_active_proposal_exists** (-- ) -- N/A -- no Stage 2 metric measures this specific block; retained for operational visibility into how often the one-active-proposal cap is hit via the reuse path

## Acceptance Criteria

**FEAT-02.SPEC-008-AC-01:** Given Nadia selects a source proposal for a target project with no existing proposal, when the copy completes, then a new Draft is created with the source's scope description and price, the target project's own currency and payment schedule reference, and copied_from set to the source.

**FEAT-02.SPEC-008-AC-02:** Given the target project already has a Draft proposal, when Nadia selects a source to copy from, then the copy is blocked with "This project already has a proposal. Open it from the project instead." and no new Draft is created.

**FEAT-02.SPEC-008-AC-03:** Given the source proposal's currency differs from the target project's currency, when the copy completes, then the price number is carried over unconverted and the editor displays the target project's own currency alongside it.

**FEAT-02.SPEC-008-AC-04:** Given a processing failure occurs during the copy, then the Reuse Picker shows "Could not start from this proposal. Try again." and Nadia remains on the picker.

**FEAT-02.SPEC-008-AC-05:** Given two copy operations for the same target project fire at effectively the same time, then only the first creates a Draft and the second receives the active-proposal-exists blocked outcome.

**FEAT-02.SPEC-008-AC-06:** Given the copy succeeds, when the editor opens, then it shows the copied-from banner naming the source project.

**FEAT-02.SPEC-008-AC-07:** Given the source proposal is voided in another session after this copy has already completed, then the resulting Draft is unaffected and remains fully editable.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 (success, blocked, failure) | 3 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



# Automation Spec: Discard Draft

## Overview

**Name:** Discard Draft
**ID:** FEAT-02.SPEC-009
**Type:** Automation
**Purpose:** Permanently removes an unsent Draft proposal that Nadia no longer wants to keep.
**Parent Feature:** FEAT-02 -- Proposal Creation & Sending

## Scope and Non-Goals

**In Scope:**
- Hard-deleting a Draft-status Proposal record
- Refusing to delete a proposal that is not in Draft status

**Non-Goals:**
- Soft-deleting or archiving a Sent, Voided, or Accepted proposal -- excluded per the Feature Breakdown Brief's Entity-Lifecycle Coverage Matrix: those statuses are evidentiary (XBR-04) and are retained for the life of the account, removed only by FEAT-24 account deletion.
- Providing a restore or undo path after discard -- a discarded draft was never sent and carries no evidentiary value, so no retention or recovery window applies; the confirmation dialog (FEAT-02.SPEC-001, FEAT-02.SPEC-003) is the only safeguard before permanent removal.
- Cascading deletes to other entities -- nothing references an unsent Draft (dependency map: Entity-Lifecycle Coverage Matrix), so no cascade logic exists in this automation.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nadia confirms "Discard Draft" | FEAT-02.SPEC-001 (Proposal Draft Editor) | The proposal is currently in Draft status | Proposal id, owning project reference |
| Nadia confirms "Discard" | FEAT-02.SPEC-003 (Proposal Detail) | The proposal is currently in Draft status | Proposal id, owning project reference |

## Processing Logic

1. Read the Proposal's current status; if it is not Draft, stop and report a state-mismatch failure (see Edge Cases).
2. Permanently delete the Proposal record.
3. Write an Activity Log Entry recording the discard (XBR-05), owned by FEAT-13.
4. Signal the triggering screen to show the project's empty proposal state.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Discard succeeds | Proposal is currently Draft | Proposal record permanently removed | Triggering screen navigates to FEAT-02.SPEC-003 showing the empty state | FEAT-02.SPEC-001, FEAT-02.SPEC-003, FEAT-13 |
| Blocked -- state mismatch | The proposal is no longer Draft (e.g., sent from another session in the moment before this ran) | None | Triggering screen shows: "This proposal was just sent and can no longer be discarded." and reloads to show the current (Sent) state | FEAT-02.SPEC-001, FEAT-02.SPEC-003 |
| Failure | Processing error (e.g., connectivity lost mid-operation) | None | Error banner: "Could not discard this draft. Try again." with a Retry option; the Draft is preserved | FEAT-02.SPEC-001, FEAT-02.SPEC-003 |

## Data Model

**Reads:** Proposal -- current status.
**Creates:** Activity Log Entry (via FEAT-13) recording the discard.
**Updates:** None.
**Deletes:** Proposal record -- permanent, hard delete, Draft-status records only.

## Business Rules

- Discard is available only against a Draft-status proposal, per FEAT-02.SPEC-010 -- Sent, Voided, and Accepted proposals can never be discarded through this automation.
- The delete is permanent and immediate on confirmation -- there is no soft-delete state, retention window, or restore path for a discarded Draft, since it was never sent and carries no evidentiary value.
- Discarding a Draft does not affect any other proposal for the same project, since a project has at most one active proposal at a time -- after discard, the project simply has none.

## Edge Cases

- **The proposal was sent from another session in the instant before this automation's status check runs** -- Reported as the state-mismatch outcome; the now-Sent proposal is never deleted, and the triggering screen reloads to show it as Sent.
- **Concurrent trigger firing (Discard confirmed from two open sessions for the same Draft at effectively the same time)** -- Only the first to pass the Draft-status check performs the delete; the second finds the proposal already gone and receives the state-mismatch outcome (surfaced as the empty-state redirect, since a deleted record and a "someone else already discarded it" outcome are indistinguishable to the user and equally resolved by returning to the empty state).
- **A trigger fires while a previous discard for the same proposal is still in flight** -- The triggering screen's Discard control is disabled during the operation (FEAT-02.SPEC-001, FEAT-02.SPEC-003), preventing a second discard attempt from the same session; a second session racing in is covered by the concurrent-trigger-firing case above.
- **Nadia discards the only Draft that was pre-filled from a copy (FEAT-02.SPEC-008)** -- The discard proceeds normally; the source proposal that was copied from is entirely unaffected, since copied_from is a one-time provenance reference, not a live dependency.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-001 (Proposal Draft Editor) | Triggered by (inbound) | "Discard Draft" action, after confirmation, triggers this automation |
| FEAT-02.SPEC-003 (Proposal Detail) | Triggered by (inbound) / Affects (outbound) | "Discard" action triggers this automation; shows the resulting empty state |
| FEAT-02.SPEC-010 (Proposal Validation & Business Rules) | References (inbound) | Draft-only eligibility rule for the discard action |
| FEAT-13 (Immutable Activity & Audit Trail) | Affects (outbound) | Writes the discard event to the trail (XBR-05) |

## Analytics and Success Signals

- **proposal_draft_discarded** (project id, minutes since draft creation) -- supports success-metrics.md: "Proposal Send Speed" (a discarded draft that never sent is a data point on the drafting funnel, distinguishing abandoned attempts from the sent-proposal timing this metric targets)

## Acceptance Criteria

**FEAT-02.SPEC-009-AC-01:** Given a project has a Draft proposal, when Nadia confirms "Discard Draft" from FEAT-02.SPEC-001, then the Proposal record is permanently deleted and the screen navigates to the empty state.

**FEAT-02.SPEC-009-AC-02:** Given a project has a Draft proposal, when Nadia confirms "Discard" from FEAT-02.SPEC-003, then the Proposal record is permanently deleted and the empty state is shown.

**FEAT-02.SPEC-009-AC-03:** Given the proposal was sent from another session in the moment before this automation's status check runs, when the discard is attempted, then it is refused with "This proposal was just sent and can no longer be discarded." and the proposal is not deleted.

**FEAT-02.SPEC-009-AC-04:** Given a processing failure occurs during discard, then the error banner "Could not discard this draft. Try again." appears and the Draft is preserved.

**FEAT-02.SPEC-009-AC-05:** Given two sessions confirm Discard for the same Draft at effectively the same time, then only the first deletes the record and the second is redirected to the empty state without error.

**FEAT-02.SPEC-009-AC-06:** Given a Draft that was created from a copy (FEAT-02.SPEC-008) is discarded, then the source proposal it was copied from is unaffected.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 | 2 |
| Outcome Paths | 3 (success, state mismatch, failure) | 3 |
| Business Rules | 3 | 3 |
| Edge Cases | 4 | 4 |



# Logic/Rule Spec: Proposal Validation & Business Rules

## Overview

**Name:** Proposal Validation & Business Rules
**ID:** FEAT-02.SPEC-010
**Type:** Logic/Rule
**Purpose:** Governs required fields, price positivity and currency, the one-active-proposal-per-project cap, send eligibility (XBR-07), immutability after send and after acceptance (XBR-04), and the accept-vs-void contention resolution for the Proposal entity.
**Parent Feature:** FEAT-02 -- Proposal Creation & Sending
**Governed Entity:** Proposal

## Scope and Non-Goals

**In Scope:**
- Field-level validation for every Proposal field this feature writes
- The one-active-proposal-per-project cap
- Send eligibility, including the client Primary-contact requirement (XBR-07)
- Post-send and post-acceptance immutability (XBR-04) and the edit-eligibility rule that enforces it
- Authorization for every action on the Proposal entity across all four Access Matrix roles
- Default values and derived fields on the Proposal record
- The accept-vs-void contention rule as it applies from this feature's side (the edit path); FEAT-03 owns the acceptance-side half of the same rule

**Non-Goals:**
- The mechanics of voiding and creating a new version -- owned by FEAT-02.SPEC-006 (Void & Resend); this spec defines the eligibility gate that automation checks, not the versioning steps themselves.
- The acceptance action itself and its own validation (a proposal can be accepted exactly once, a voided proposal cannot be accepted) -- owned by FEAT-03 (Proposal Acceptance); this spec only defines how the Proposal entity's fields and status constrain what FEAT-02's own actions may do.
- Electronic-signature validation for the accept action -- owned by FEAT-26 (Legally Binding E-Signature for Proposals, v1); this spec's Governed Entity table lists the signature record field only for completeness of the field inventory.

## Governed Entity

**Entity:** Proposal
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| scope_description | text | The work scope the freelancer is proposing |
| price | number | The proposed amount, in the project's currency |
| currency | text (derived) | The project's billing currency (FEAT-15); not set independently on the Proposal |
| payment_schedule_reference | reference | Link to the project's Payment Schedule (FEAT-04); locked at the moment of send |
| status | enum | Draft, Sent, Voided, or Accepted |
| sent_at | date/time | Timestamp of the most recent send for the current version |
| copied_from | reference (optional) | The earlier proposal this Draft was started from, if any (FEAT-02.SPEC-008) |
| accepted_at | date/time (optional) | Timestamp of acceptance, written once by FEAT-03, never altered |
| accepted_by | reference (optional) | The accepting Client Contact, written once by FEAT-03, never altered |
| signature record | composite (optional, v1) | Signer identity, signature data, and timestamp, written by FEAT-26 when e-signature is enabled for the proposal |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-02.SPEC-001 | Proposal Draft Editor | Field validation on blur and on Preview/Save Draft/Save & Resend; edit-eligibility check on entering edit mode and on Save & Resend |
| FEAT-02.SPEC-002 | Proposal Preview | Send-eligibility re-check on Send tap |
| FEAT-02.SPEC-003 | Proposal Detail | Authorization rules on screen entry (status-driven action set) |
| FEAT-02.SPEC-005 | Proposal Send | Field validation and send-eligibility checks during processing |
| FEAT-02.SPEC-006 | Proposal Edit-Before-Acceptance Void & Resend | Field validation, send-eligibility, and the post-acceptance immutability check during processing |
| FEAT-02.SPEC-009 | Discard Draft | Draft-only eligibility check during processing |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| scope_description | Required, non-empty | Always | On blur and on submit | "Scope description is required" | Yes |
| scope_description | Maximum 10,000 characters | Always | On blur and on submit | "Scope description must be 10,000 characters or fewer" | Yes |
| price | Required, non-empty | Always | On blur and on submit | "Price is required" | Yes |
| price | Must be a positive amount (greater than zero) | Always | On blur and on submit | "Price must be a positive amount" | Yes |
| currency | No validation beyond data type -- always derived from the project, never entered directly | Always | -- | -- | -- |
| payment_schedule_reference | No validation beyond data type -- a Draft may reference no schedule yet; only locked (not required) at send | Always | -- | -- | -- |
| status | No validation beyond data type -- transitions are governed by the Business Rules and Authorization Rules below, not by field-level input validation | Always | -- | -- | -- |
| sent_at | No validation beyond data type -- system-set, never user-entered | Always | -- | -- | -- |
| copied_from | No validation beyond data type -- system-set, never user-entered | Always | -- | -- | -- |
| accepted_at | No validation beyond data type -- written once by FEAT-03, never entered or altered here | Always | -- | -- | -- |
| accepted_by | No validation beyond data type -- written once by FEAT-03, never entered or altered here | Always | -- | -- | -- |
| signature record | No validation beyond data type -- owned by FEAT-26; out of scope for this spec | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Currency follows the project | price, currency | price is always interpreted and displayed in currency, which is always the owning project's currency (FEAT-15) and can never be set independently on the Proposal | N/A -- currency is never user-entered on this entity, so no error state exists for a mismatch |
| One active proposal per project | status, project reference | A project may have at most one Proposal in Draft, Sent, or Accepted status at any time; a new Draft (blank or from copy) cannot be created while one already exists | "This project already has a proposal. Open it from the project instead." |
| Send requires a client Primary contact | status, payment_schedule_reference (n/a to this rule directly, listed for completeness), client's Primary contact roster | A Draft cannot transition to Sent (directly or via void-and-resend) unless the owning client has at least one Primary contact (XBR-07) | "This client has no Primary contact yet." |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create proposal (blank or from copy) | Nadia (Freelancer) | Only when the target project has no active (Draft, Sent, or Accepted) proposal | Create action is not offered; attempting it via the reuse path shows "This project already has a proposal. Open it from the project instead." |
| Create proposal | Owen (Client Primary Contact), Priya (Client Reviewer Contact), Dana (Support Operator) | Never | No route to create a proposal exists in the client portal or the support session; the action is never shown |
| View proposal (full content: scope, price, status, history) | Nadia (Freelancer) | Always, for any of her own proposals | -- |
| View proposal (full content) | Owen (Client Primary Contact) | Own-only -- only proposals belonging to his own client company | A proposal outside Owen's own client company is never reachable; an out-of-scope link shows the plain explanation defined by XBR-09 |
| View proposal | Priya (Client Reviewer Contact) | Never -- Priya has no access to proposal content at all, per the Access Matrix | Priya's portal home shows only the project's stage (e.g., "Proposal accepted"), never the proposal's scope or price |
| View proposal (status and history only, no scope/price/request-changes text) | Dana (Support Operator) | Always, inside a logged, read-only support session (FEAT-31) | -- |
| Edit proposal (Draft, in place) | Nadia (Freelancer) | Only while status is Draft | Edit controls are not shown once status leaves Draft |
| Edit proposal (Sent-but-unaccepted, via void-and-resend) | Nadia (Freelancer) | Only while status is Sent (not yet Accepted or Voided) | If status is Accepted: "This proposal has already been accepted and can no longer be edited." (XBR-04). If status is already Voided by a concurrent edit: "This proposal was already edited and resent. Review the current version." |
| Edit proposal | Owen, Priya, Dana | Never | No edit control is ever shown to these roles |
| Send proposal | Nadia (Freelancer) | Only while status is Draft, and only when the owning client has at least one Primary contact (XBR-07) | If status is not Draft: "This proposal has already been sent. Viewing the current version." If no Primary contact: "This client has no Primary contact yet." |
| Send proposal | Owen, Priya, Dana | Never | No send control is ever shown to these roles |
| Resend proposal (unchanged link) | Nadia (Freelancer) | Only while status is Sent, subject to the resend rate limit (FEAT-02.SPEC-007) | If status is not Sent: the Resend action is not shown. If rate-limited: "You already resent this proposal recently. Try again in a few minutes." |
| Resend proposal | Owen, Priya, Dana | Never | No resend control is ever shown to these roles |
| Discard proposal | Nadia (Freelancer) | Only while status is Draft | Discard control is not shown once status leaves Draft; a direct attempt against a non-Draft status shows "This proposal was just sent and can no longer be discarded." |
| Discard proposal | Owen, Priya, Dana | Never | No discard control is ever shown to these roles |
| Accept proposal (owned by FEAT-03) | Owen (Client Primary Contact) | Own-only, only while status is Sent, exactly once per proposal | If status is Voided or already Accepted: refused per FEAT-03's rules, with the current version shown |
| Accept proposal | Nadia, Priya, Dana | Never | No accept control is ever shown to these roles |
| Request changes (owned by FEAT-03) | Owen (Client Primary Contact) | Own-only, only while status is Sent | No request-changes control is ever shown to these roles when status is not Sent |
| Request changes | Nadia, Priya, Dana | Never | No request-changes control is ever shown to these roles |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| status | "Draft" | On create (blank or from copy) | No -- always starts Draft |
| currency | The owning project's current currency (FEAT-15) | Always (on create and whenever displayed) | No -- never editable on the Proposal itself |
| payment_schedule_reference | The owning project's current Payment Schedule, or unset if none exists yet | On create; re-derived (re-read) on every save while status is Draft | No -- locked automatically at send (FEAT-02.SPEC-005) and never user-set directly |
| sent_at | Current date and time | On the Draft -> Sent transition (send or void-and-resend) | No |
| copied_from | The source proposal's id | On create, only when created via FEAT-02.SPEC-008 | No -- permanent provenance reference |
| accepted_at, accepted_by | Set once by FEAT-03 on acceptance | On acceptance | No -- written once, never altered (XBR-04) |

## Business Rules

- A project has at most one active (Draft, Sent, or Accepted) proposal at any time; Voided versions do not count against this cap, since a Voided version's replacement is always the new active one.
- A Sent-but-unaccepted proposal is never updated in place -- every content edit after send produces a new version through FEAT-02.SPEC-006's void-and-resend, never a direct field update (XBR-06).
- An Accepted proposal is immutable in every respect -- no field on it may ever be changed after acceptance, and no edit attempt against it succeeds, regardless of source (XBR-04).
- A Voided proposal is immutable and terminal -- it can never be accepted, edited, resent, or reactivated; it exists solely as evidence.
- Send eligibility (XBR-07) requires the client to have at least one Primary contact; this is re-checked at the moment of send and at the moment of void-and-resend, not only when the client roster was last viewed.
- Accept-vs-void contention: if Owen accepts a version that Nadia voided in the meantime (or is in the process of voiding), the acceptance is refused and Owen is shown the current version -- this rule is jointly enforced, with FEAT-03 owning the acceptance-side refusal and this spec owning the edit-side effect (a successful void makes the prior version's status Voided, which FEAT-03's acceptance check then reads as ineligible). Acceptance is recorded exactly once per proposal.
- Discard permanently removes only Draft-status records; a discarded Draft leaves no trace beyond an Activity Log Entry recording that the discard occurred (FEAT-13), since a never-sent Draft carries no evidentiary content worth retaining.
- Dana (Support Operator) never sees scope_description, price, or request-changes note text on any Proposal, in any status, consistent with the Access Matrix's read-only, content-limited support boundary (ASMP-18, ASMP-23).

## Edge Cases

- **Price entered as exactly zero** -- Fails the positive-amount rule; "Price must be a positive amount" is shown. Zero is not treated as a valid free-of-charge proposal in this product definition.
- **Scope description at exactly 10,000 characters** -- Passes validation. 10,001 characters shows the length error.
- **A Draft is created for a project, then its only active proposal is discarded, then a second Draft is created for the same project** -- Allowed; the one-active-proposal cap only ever compares against the project's current active proposal, and a discarded Draft is not counted once removed.
- **Nadia attempts to send a Draft while the client's Primary contact was removed moments earlier** -- The send-eligibility check re-reads the client's current contact roster at the moment of send, so the block applies even though the Draft itself was created while a Primary contact existed.
- **Nadia attempts to edit a Sent-but-unaccepted proposal in the instant Owen's acceptance is being recorded** -- Whichever transition (the acceptance or the edit's status check) completes first determines the outcome: if acceptance completes first, the edit is refused as already-accepted; if the edit's void completes first, the acceptance is refused as against-a-voided-version. Exactly one of the two prevails; no partial or contradictory state (both Accepted and Voided) is ever produced.
- **A Voided proposal's own copied_from or predecessor content is referenced from the Reuse Proposal Picker (FEAT-02.SPEC-004)** -- Permitted: a Voided proposal's last-sent content remains valid as a copy source even though the proposal itself is immutable and terminal, since copying reads content rather than reactivating the record.

## Acceptance Criteria

**FEAT-02.SPEC-010-AC-01:** Given Nadia leaves scope_description empty on a Draft, when she attempts to save or send, then "Scope description is required" is shown and the operation is blocked.

**FEAT-02.SPEC-010-AC-02:** Given Nadia enters a scope description of exactly 10,000 characters, when she saves, then validation passes; at 10,001 characters, "Scope description must be 10,000 characters or fewer" is shown.

**FEAT-02.SPEC-010-AC-03:** Given Nadia leaves price empty, when she attempts to save or send, then "Price is required" is shown.

**FEAT-02.SPEC-010-AC-04:** Given Nadia enters a price of zero, when she attempts to save or send, then "Price must be a positive amount" is shown.

**FEAT-02.SPEC-010-AC-05:** Given Nadia enters a price of 1.00 in the project's currency, when she saves, then validation passes.

**FEAT-02.SPEC-010-AC-06:** Given a project already has an active (Draft, Sent, or Accepted) proposal, when Nadia attempts to start a new Draft for it via the reuse path, then "This project already has a proposal. Open it from the project instead." is shown and no second Draft is created.

**FEAT-02.SPEC-010-AC-07:** Given the client has no Primary contact, when Nadia attempts to send a valid Draft, then "This client has no Primary contact yet." is shown and the send is blocked.

**FEAT-02.SPEC-010-AC-08:** Given the client has at least one Primary contact and the Draft's fields are valid, when Nadia sends it, then the send proceeds.

**FEAT-02.SPEC-010-AC-09:** Given a proposal's status is Accepted, when Nadia attempts to edit it, then "This proposal has already been accepted and can no longer be edited." is shown and no edit occurs.

**FEAT-02.SPEC-010-AC-10:** Given a proposal's status is Sent and unaccepted, when Nadia edits and saves it, then the edit is allowed through the void-and-resend path (FEAT-02.SPEC-006), not a direct update.

**FEAT-02.SPEC-010-AC-11:** Given a proposal's status is Voided, when any attempt is made to accept it, then the acceptance is refused (owned by FEAT-03) and the current version is shown instead.

**FEAT-02.SPEC-010-AC-12:** Given Nadia (Freelancer) views any of her own proposals, then she sees full content (scope, price, status, history) regardless of status.

**FEAT-02.SPEC-010-AC-13:** Given Owen (Client Primary Contact) attempts to reach a proposal belonging to a different client company, then no route exists and, if an out-of-scope link is followed, the plain explanation defined by XBR-09 is shown.

**FEAT-02.SPEC-010-AC-14:** Given Priya (Client Reviewer Contact) views her portal home, then she sees only the project's stage and never the proposal's scope or price.

**FEAT-02.SPEC-010-AC-15:** Given Dana (Support Operator) views a proposal inside a logged support session, then she sees status and history only, with no scope_description, price, or request-changes note text.

**FEAT-02.SPEC-010-AC-16:** Given a proposal's status is Draft, when Nadia looks for Discard, then it is shown and, on confirmation, the proposal is permanently deleted.

**FEAT-02.SPEC-010-AC-17:** Given a proposal's status is Sent, when Nadia looks for Discard, then it is not shown.

**FEAT-02.SPEC-010-AC-18:** Given Owen, Priya, or Dana looks for Create, Edit, Send, Resend, or Discard controls on any proposal, then none of these controls are ever shown to them.

**FEAT-02.SPEC-010-AC-19:** Given a new blank or copied Draft is created, then its status defaults to Draft and its currency is derived from the owning project, never independently settable.

**FEAT-02.SPEC-010-AC-20:** Given a proposal transitions from Draft to Sent, then sent_at is set to the current date and time and payment_schedule_reference is locked to the project's Payment Schedule at that moment.

**FEAT-02.SPEC-010-AC-21:** Given Owen's acceptance and Nadia's void-and-resend race against the same Sent proposal, when one completes first, then exactly one of "acceptance recorded" or "version voided" prevails and the other is refused -- never both.

**FEAT-02.SPEC-010-AC-22:** Given a proposal was created via FEAT-02.SPEC-008 from a source that is later voided, when Nadia views the new Draft, then copied_from still references the source and the new Draft remains fully editable and independent.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 10 (2 with sub-rules) | 10 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 19 | 19 |
| Defaults/Derivations | 6 | 6 |
| Business Rules | 8 | 8 |
| Edge Cases | 6 | 6 |



# Notification Spec: Proposal Sent/Resent Email

## Overview

**Name:** Proposal Sent/Resent Email
**ID:** FEAT-02.SPEC-011
**Type:** Notification
**Purpose:** Emails the client's Primary Contact the proposal link whenever a proposal is sent, resent, or re-sent after an edit, so the client can review and act on it without a "PDF in email" back-and-forth.
**Parent Feature:** FEAT-02 -- Proposal Creation & Sending

## Scope and Non-Goals

**In Scope:**
- The email delivered on an original send (FEAT-02.SPEC-005), an unchanged resend (FEAT-02.SPEC-007), and a void-and-resend after an edit (FEAT-02.SPEC-006)
- Content variants distinguishing a first send from a resend
- Delivery, retry, and expiry behavior for this transactional record email

**Non-Goals:**
- Deciding when a proposal is sent, resent, or void-and-resent -- owned by FEAT-02.SPEC-005, FEAT-02.SPEC-006, and FEAT-02.SPEC-007; this spec begins where each of their triggers fires.
- The magic-link sign-in flow the email's link leads into -- owned by FEAT-05 (Client Portal Access); this spec defines the email's content and the link's destination, not the sign-in mechanics.
- Confirming acceptance back to Nadia -- owned by FEAT-03 (Proposal Acceptance), which sends its own confirmation once Owen acts; this spec covers only the send/resend email, not the acceptance confirmation.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, on every send, resend, and void-and-resend | Per BRIEF.md, Ecosystem & Integrations, clients will not install an app; email is the sole channel that reaches Owen, and this is a transactional record he cannot opt out of (XBR-30) |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Proposal sent (original) | FEAT-02.SPEC-005 (Proposal Send) | Fires when a Draft successfully transitions to Sent | Project name, client name, scope_description, price, currency, sent_at, Primary Contact name/email, freelancer's Branding Profile |
| Proposal edited and re-sent | FEAT-02.SPEC-006 (Proposal Edit-Before-Acceptance Void & Resend) | Fires when a void-and-resend successfully creates and sends the new version | Same as above, for the new version, plus an indicator that this supersedes a prior version |
| Proposal resent (unchanged) | FEAT-02.SPEC-007 (Proposal Resend) | Fires when a resend of the current Sent version succeeds | Same as the original-send data, for the existing (unchanged) version |

## Audience and Preferences

**Recipients:** Owen -- the client's Primary Contact(s) (Access Matrix: Own-only view, accept, request changes, sign). Priya (Reviewer) never receives this email, since Reviewers have no access to proposal content at all, per the Access Matrix. Nadia does not receive this email herself; she sees the resulting Sent status on FEAT-02.SPEC-003.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| None -- this is a transactional record email | -- | Always on | -- |

This email is a transactional record central to the proposal's status, not an optional notification -- per XBR-30, transactional emails core to the record always send and cannot be switched off.

**Quiet Hours:** N/A -- the product defines no quiet-hours window for this email; it is a time-sensitive, action-required message tied directly to a freelancer-initiated action (send, edit-resend, resend), and holding it would delay the exact moment the client needs to act.

## Content Definition

**Email (original send):**
- **Subject:** New proposal from {freelancer_business_name}: {project_name}
- **Body:**
  Hi {contact_name},

  {freelancer_business_name} has sent you a proposal for {project_name}.

  Scope: {scope_summary}
  Price: {price_formatted}

  Review the full proposal and let {freelancer_business_name} know if you'd like to accept it or request changes.
- **CTA (button):** View Proposal -- deep-links to FEAT-03.SPEC-001 (Proposal Review & Accept) [Owen's own portal proposal view, owned by FEAT-03] for this project's proposal, via magic-link sign-in (FEAT-05)

**Email (edited and re-sent):**
- **Subject:** Updated proposal from {freelancer_business_name}: {project_name}
- **Body:**
  Hi {contact_name},

  {freelancer_business_name} has updated the proposal for {project_name}. The earlier version is no longer valid -- please review the current one below.

  Scope: {scope_summary}
  Price: {price_formatted}

  Review the full proposal and let {freelancer_business_name} know if you'd like to accept it or request changes.
- **CTA (button):** View Updated Proposal -- deep-links to FEAT-03.SPEC-001 (Proposal Review & Accept) [Owen's portal proposal view, FEAT-03] for this project's proposal, via magic-link sign-in (FEAT-05)

**Email (resent, unchanged):**
- **Subject:** Reminder: proposal from {freelancer_business_name} for {project_name}
- **Body:**
  Hi {contact_name},

  Here's the link to the proposal {freelancer_business_name} sent you for {project_name}, in case the earlier email is hard to find.

  Scope: {scope_summary}
  Price: {price_formatted}
- **CTA (button):** View Proposal -- deep-links to FEAT-03.SPEC-001 (Proposal Review & Accept) [Owen's portal proposal view, FEAT-03] for this project's proposal, via magic-link sign-in (FEAT-05)

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {contact_name} | Client Contact -- name | Owen Marsh | Never empty -- contact name is required at contact creation (FEAT-18) |
| {freelancer_business_name} | Freelancer Account -- business_name | Studio Nadia | Falls back to the freelancer's account name if business_name is not yet set |
| {project_name} | Project -- project_name | Autumn Rebrand | Never empty -- required at project creation (FEAT-01) |
| {scope_summary} | Proposal -- scope_description (first plain-text excerpt) | "Full brand identity refresh including logo, colour system, and style guide." | Never empty -- scope_description is required to send (FEAT-02.SPEC-010) |
| {price_formatted} | Proposal -- price and currency | $4,500.00 USD | Never empty -- price is required and positive to send (FEAT-02.SPEC-010) |

The email's visual presentation (header band, logo placement, brand colour) applies the freelancer's Branding Profile (FEAT-19) consistently with FEAT-02.SPEC-002 (Proposal Preview), per XBR-31, falling back to a neutral default when the freelancer has not set one.

## Delivery Rules

**Batching:** None -- each send, resend, or edit-resend is delivered as its own individual email at the moment its trigger fires. A proposal is never batched with any other notification, since it is a singular, time-sensitive action tied to one specific event.
**Deduplication:** At most one email per triggering event. A resend triggered while the rate-limit window (FEAT-02.SPEC-007) is active is blocked at the automation level before this notification is ever triggered, so no duplicate email for the same resend attempt is possible.
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001). After the final failure, the delivery failure is surfaced to Nadia as a warning on the project (XBR-30), since email is the only channel reaching Owen and a silently lost proposal email would leave him unaware a proposal exists.
**Expiry:** This email does not expire in the sense of being withdrawn -- if delivery eventually succeeds after retries, it is still the correct, current content, since the Proposal Detail view it links to always reflects the current version. If all retries are exhausted, the email is not delivered at all and the project-level delivery warning (above) is the surviving signal for Nadia to resend manually (FEAT-02.SPEC-007).

## Edge Cases

- **The proposal is voided (edited and re-sent) before the original send email's retries are exhausted** -- The original send's pending retry is superseded: only the edited version's email is worth delivering, since the original's content is no longer current. The original email is not attempted further; the edited version's own email proceeds through its own delivery and retry cycle.
- **The client's Primary contact email address changes between trigger and delivery** -- The email is addressed to the Primary contact's current email at the moment of actual delivery, not at the moment the automation triggered, so a contact update made in the interim is respected (consistent with FEAT-18 owning contact data as the source of truth).
- **The proposal is discarded -- not applicable to this notification, since Discard (FEAT-02.SPEC-009) only ever applies to a Draft, and a Draft never triggers this email; no disappearing-record scenario exists for this spec's own trigger paths.**
- **Delivery fails on all retries for an original send** -- The delivery warning appears on the project for Nadia (XBR-30); Owen never learns a proposal was intended for him until Nadia notices the warning and uses Resend (FEAT-02.SPEC-007) once the underlying delivery issue (e.g., an invalid address) is corrected via FEAT-18.
- **The client has more than one Primary contact** -- Every current Primary contact receives their own copy of the email, addressed individually, per the Access Matrix's "Own-only" entitlement applying to all Primary contacts of the client company equally.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-005 (Proposal Send) | Triggered by (inbound) | Original send fires this notification |
| FEAT-02.SPEC-006 (Void & Resend) | Triggered by (inbound) | Edit-and-resend fires the edited-version variant |
| FEAT-02.SPEC-007 (Proposal Resend) | Triggered by (inbound) | Unchanged resend fires the resend variant |
| FEAT-03.SPEC-001 (Proposal Review & Accept) | Navigation (outbound) | Every CTA deep-links to Owen's portal proposal view, which surfaces the same status this feature's Detail screen (FEAT-02.SPEC-003) shows Nadia |
| FEAT-03 (Proposal Acceptance) | References (outbound) | The CTA's destination is owned by FEAT-03 as Owen's portal-side proposal view |
| FEAT-05 (Client Portal Access) | References (outbound) | The CTA routes through magic-link sign-in before reaching the proposal view |
| FEAT-18 (Client Contact Management & Roles) | References (inbound) | Recipient identity and current email address sourced from here |
| FEAT-19 (Freelancer Branding) | References (inbound) | Branding Profile applied to the email's visual presentation (XBR-31) |
| FEAT-14.SPEC-001 (Transactional Email Delivery) | References (outbound) | Underlying delivery, retry, and bounce/failure reporting capability this notification is sent through |

## Analytics and Success Signals

- **proposal_email_delivered** (variant: original / edited / resent) -- supports success-metrics.md: "Notification Delivery Reliability"
- **proposal_email_opened** (variant) -- supports success-metrics.md: "Time to Proposal Acceptance" (an opened email is the precondition for the acceptance-speed clock the metric measures)
- **proposal_email_cta_tapped** (variant) -- supports success-metrics.md: "Time to Proposal Acceptance"
- **proposal_email_delivery_failed** (variant, retry_count_exhausted: yes/no) -- supports success-metrics.md: "Notification Delivery Reliability"

## Acceptance Criteria

**FEAT-02.SPEC-011-AC-01:** Given a Draft proposal is successfully sent (FEAT-02.SPEC-005), when this notification fires, then Owen receives an email with the subject "New proposal from {freelancer_business_name}: {project_name}" and a "View Proposal" CTA.

**FEAT-02.SPEC-011-AC-02:** Given a Sent-but-unaccepted proposal is edited and re-sent (FEAT-02.SPEC-006), when this notification fires, then Owen receives an email with the subject "Updated proposal from {freelancer_business_name}: {project_name}" stating the earlier version is no longer valid.

**FEAT-02.SPEC-011-AC-03:** Given Nadia resends an unchanged Sent proposal (FEAT-02.SPEC-007), when this notification fires, then Owen receives an email with the subject "Reminder: proposal from {freelancer_business_name} for {project_name}".

**FEAT-02.SPEC-011-AC-04:** Given the client has two Primary contacts, when a proposal is sent, then both receive their own copy of the email, each addressed individually.

**FEAT-02.SPEC-011-AC-05:** Given Priya (Reviewer) is a contact on the client company, when a proposal is sent, then she does not receive this email.

**FEAT-02.SPEC-011-AC-06:** Given Owen taps the "View Proposal" CTA, then he is routed through magic-link sign-in (FEAT-05) and lands on his portal proposal view (FEAT-03) for this project.

**FEAT-02.SPEC-011-AC-07:** Given delivery of the original-send email fails, when the retry window is exhausted, then a delivery warning appears on the project for Nadia and no further automatic retry occurs.

**FEAT-02.SPEC-011-AC-08:** Given the original send's email delivery is still retrying when the proposal is edited and re-sent, then the original email is not delivered further and only the edited version's email proceeds.

**FEAT-02.SPEC-011-AC-09:** Given the Primary contact's email address is updated between trigger and actual delivery, then the email is delivered to the current address at delivery time.

**FEAT-02.SPEC-011-AC-10:** Given the freelancer has not set a Branding Profile, when this email renders, then it uses the neutral default branding.

**FEAT-02.SPEC-011-AC-11:** Given Nadia has not set any notification preference for this email, when a proposal is sent, then the email always sends -- there is no preference control that can turn it off, per XBR-30.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 3 (original send, edit-resend, resend) | 3 |
| Preference States | 1 (always on) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |

---
document_type: feature-overview
feature_number: FEAT-02
feature_name: Proposal Creation & Sending
feature_slug: proposal-creation-sending
priority_tier: Core
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 11
screen_count: 4
automation_count: 5
logic_rule_count: 1
integration_count: 0
notification_count: 1
---

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

# FEAT-26 — Legally Binding E-Signature for Proposals

This chapter covers Legally Binding E-Signature for Proposals, a Nice-to-Have-tier feature. It contains the feature breakdown brief followed by every specification in full: 5 specifications carrying 81 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-26.SPEC-001 | Signature Signing Step | screen | 22 |
| FEAT-26.SPEC-002 | Signature Recording | automation | 14 |
| FEAT-26.SPEC-003 | E-Signature Opt-In & Signing Access Rules | logic-rule | 21 |
| FEAT-26.SPEC-004 | Signed-Copy Confirmation Notification | notification | 11 |
| FEAT-26.SPEC-005 | Electronic-Signature Attestation Capability | integration | 13 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Legally Binding E-Signature for Proposals

## Summary

**Feature:** Legally Binding E-Signature for Proposals
**ID:** FEAT-26
**Description:** Upgrades the recorded "Accept" click to a legally binding e-signature for freelancers who want stronger contractual weight than a timestamp alone.
**Priority:** Nice-to-Have
**Phase:** v1
**Type:** User-Facing
**Rationale:** BRIEF.md's Open Questions leaves unresolved "do proposals need legally binding e-signatures, or is a recorded, timestamped 'Accept' enough?" — the founder's own framing treats the timestamped Accept as the presumptive default. Nice-to-Have because the MVP default (FEAT-03) already satisfies the brief's evidence requirement; it can be added per proposal without restructuring the acceptance flow. [MODIFIED: phase moved from Later to v1 based on proposals with e-signature being bundled by all 5 profiled competitors (5 sources, HIGH confidence) — freelancers switching from those tools will expect the option soon after launch; tier kept Nice-to-Have because the timestamped Accept remains the brief's presumptive default] [INFERRED: carried from Visionary draft]

**Key Capabilities:**
- Opt-in e-signature — freelancer enables it for a specific proposal
- Signing step — client completes a signature step instead of a plain Accept click

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-26.SPEC-001 | Signature Signing Step | Screen | Owen, Dana | Owen completes a signature step in place of the plain Accept click when e-signature is enabled for the proposal |
| FEAT-26.SPEC-002 | Signature Recording | Automation | Owen, Nadia | Validates and writes the immutable signature record and extends FEAT-03's Acceptance Recording so the signed acceptance still fires the deposit invoice and audit-trail entry |
| FEAT-26.SPEC-003 | E-Signature Opt-In & Signing Access Rules | Logic/Rule | Nadia, Owen, Priya, Dana | Governs per-proposal opt-in scope, who may sign, routing between the plain Accept and the signing step, failed-submission retry, and immutability once signed |
| FEAT-26.SPEC-004 | Signed-Copy Confirmation Notification | Notification | Owen, Nadia | Emails a signed-copy confirmation to both parties in addition to the standard acceptance confirmation |
| FEAT-26.SPEC-005 | Electronic-Signature Attestation Capability | Integration | Owen, Nadia | Product uses an external electronic-signature attestation capability to give the signed record legal weight beyond a self-recorded timestamp (electronic-signature attestation capability) |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Opt-in e-signature | FEAT-26.SPEC-003 | Defines the per-proposal opt-in flag and its scope; the toggle itself is rendered on FEAT-02's send/edit screen, which reads this feature's eligibility rule | Phase 2 (Explicit) |
| Signing step | FEAT-26.SPEC-001, FEAT-26.SPEC-002 | The screen presents the signing step in place of Accept; the automation validates the submission and writes the immutable signature record | Phase 2 (Explicit) |

**Analyst-Discovered Specs** — Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-26.SPEC-003 | E-Signature Opt-In & Signing Access Rules | Phase 5 (Rule Discovery) | The Access field's four differentiated role behaviors (Owen signs, Priya none, Dana view-only), the Validation & Limits field (opt-in per proposal, not account-wide; immutable once signed), the routing decision between FEAT-03's plain Accept and this feature's signing step, and the failure-retry behavior together exceed the 5-rule / shared-across-specs threshold for a standalone Logic/Rule spec |
| FEAT-26.SPEC-004 | Signed-Copy Confirmation Notification | Phase 4 (Notification surfacing) | The Communications field names a signed-copy confirmation email to both parties, distinct from and in addition to FEAT-03's standard acceptance confirmation, with its own audience and delivery behavior — not a same-screen toast |
| FEAT-26.SPEC-005 | Electronic-Signature Attestation Capability | Phase 4 (External Dependencies lens) | The feature's entire reason for existing is to give the acceptance record legal weight a self-recorded timestamp cannot provide on its own; that stronger evidentiary standing depends on an external identity/attestation capability, which the assumptions-constraints.md Dependencies section does not yet name (context package, Section 5) — inventoried here per the context package's own instruction, for the Requirements Architect to add to the External Touchpoints table |

## Entity-Lifecycle Coverage Matrix

**Entity: Proposal** *(this feature only adds the signature record to an already-sent, already-accepting proposal; creation, versioning, and the base acceptance record are owned by FEAT-02 and FEAT-03)*

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A | Owned by FEAT-02 (proposal drafting and sending) | — |
| Read (single) | FEAT-26.SPEC-001 | Signing Step screen loads the proposal's scope, price, and signing-enabled status for the signing contact | Same underlying record FEAT-03.SPEC-001 reads; this feature's screen only renders when signing is enabled |
| Read (list) | N/A | A project has at most one active proposal (dependency map, Proposal Relationships); no list view exists in this feature, same as FEAT-03 | — |
| Update | FEAT-26.SPEC-002 | Signature Recording writes the signature record (signer identity, signature data, timestamp) alongside the `accepted_at` / `accepted_by` fields FEAT-03.SPEC-003 writes | The signature record and the acceptance fields are written together as one signed acceptance (XBR-34) |
| Delete/Archive | N/A | This feature never deletes or archives a Proposal or its signature record. Voiding is owned by FEAT-02 (XBR-06, and a voided proposal cannot be signed — see Side-Effect Inventory); permanent deletion is owned by FEAT-24. No retention/purge decision belongs to this feature. | — |
| State Transition | N/A | The Sent → Accepted transition itself is owned by FEAT-03 (XBR-34: FEAT-03 owns the acceptance record, FEAT-26 extends it); this feature adds the signature record alongside that transition but does not own it | — |

**Referenced Entities (read-only or triggered, not owned by this feature):**

| Entity | Read By | Context |
|--------|---------|---------|
| Client Contact | FEAT-26.SPEC-001, FEAT-26.SPEC-003 | Identifies the signing contact and enforces that only the Primary contact (Owen) may sign, mirroring FEAT-03's access boundary |
| Activity Log Entry | FEAT-26.SPEC-002 (writes, via FEAT-13) | The signed acceptance feeds the append-only audit trail (Interactions field; XBR-04) |
| Notification | FEAT-26.SPEC-004 (writes, via FEAT-14) | The signed-copy confirmation is created as a Notification record and delivered through FEAT-14's transactional email capability |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Nadia enables the e-signature toggle while drafting or editing a proposal (on FEAT-02's screen) | Mark this specific proposal as signing-enabled | Standalone Logic/Rule | FEAT-26.SPEC-003 |
| Owen opens a signing-enabled proposal that has not yet been acted on | Route to the Signing Step screen instead of FEAT-03's plain Accept control | Standalone Logic/Rule, rendered inline in FEAT-03.SPEC-001 | FEAT-26.SPEC-003 |
| Owen opens a proposal that is not signing-enabled | Show FEAT-03's standard Accept flow unchanged | Cross-feature — logged in touchpoints | FEAT-03 responsibility |
| Owen completes the signing step | Validate the submission, call the electronic-signature attestation capability, then write the signature record and trigger FEAT-03's Acceptance Recording | Standalone Automation | FEAT-26.SPEC-002 |
| Signature submission fails (attestation capability error, validation failure, or connectivity drop) | Preserve the entered signature data on screen and allow retry without losing Owen's intent | Standalone Logic/Rule, enforced within FEAT-26.SPEC-001's screen | FEAT-26.SPEC-003 |
| Owen attempts to open the signing step without a connection | Show a plain "signing needs a connection" message; the signing step never pretends to succeed offline | Inline in triggering screen (per the feature's Offline-degraded States field, consistent with ASMP-27) | FEAT-26.SPEC-001 |
| Signature is recorded | Fire the deposit-invoice trigger exactly as a plain acceptance would (XBR-01), using the schedule as it stood at signing | Cross-feature — logged in touchpoints | FEAT-26.SPEC-002 → FEAT-03.SPEC-003 → FEAT-09 |
| Signature is recorded | Write an Activity Log Entry for the signed-acceptance event | Cross-feature — logged in touchpoints | FEAT-26.SPEC-002 → FEAT-13 (XBR-04) |
| Signature is recorded | Email the signed-copy confirmation to Owen and Nadia, in addition to FEAT-03's standard acceptance confirmation | Standalone Notification | FEAT-26.SPEC-004 |
| Owen signs successfully | Show a "Signed on {date}" marker in place of the Accept control, distinct from a plain "Accepted" marker | Inline in triggering screen | FEAT-26.SPEC-001 |
| A proposal is voided (edited and re-sent) before it is signed | The prior signing-enabled version cannot be signed; Owen is shown the current version, same as a plain accept attempt (XBR-06) | Cross-feature — logged in touchpoints | FEAT-03.SPEC-005 responsibility, applied to this feature's screen |
| Priya or an unauthorized contact opens a signing-enabled proposal link | Show no signing content, mirroring FEAT-03's authorization behavior | Standalone Logic/Rule (authorization) | FEAT-26.SPEC-003 |
| Dana opens a signing-enabled proposal inside a support session | Show read-only content (the "Signed on {date}" marker once signed, or the pending signing state before) with no signing control, inside the logged session (FEAT-31) | Standalone Logic/Rule (authorization) | FEAT-26.SPEC-003 |

## Shared Context

**Shared Entities:**
- Proposal — read by FEAT-26.SPEC-001, updated (signature record only, alongside FEAT-03's acceptance fields) by FEAT-26.SPEC-002, governed by FEAT-26.SPEC-003's eligibility and access rules. Fields relevant to this feature: signature record (signer identity, signature data, timestamp), the signing-enabled flag, `status`, `accepted_at`, `accepted_by`.
- Client Contact — read by FEAT-26.SPEC-001 and FEAT-26.SPEC-003 to establish the signing contact's role (Primary vs. Reviewer) and identity, exactly as FEAT-03.SPEC-005 does for plain acceptance.

**Shared UI Patterns:**
- Immutable-record marker — once signed, FEAT-26.SPEC-001 displays a "Signed on {date}" marker in place of the signing control, following the same permanent, non-editable-marker convention FEAT-03.SPEC-001 uses for "Accepted on {date}" (XBR-04), but visually distinct so a client can tell a signed acceptance apart from a plain one (Data Notes field).
- Decision control sizing — the signing control reuses FEAT-03.SPEC-001's large, clearly labeled, keyboard- and screen-reader-usable tap-target convention (ASMP-27), since it occupies the same position in the flow as the Accept control it replaces.

**Shared Validation:**
- FEAT-26.SPEC-003 is the single source of truth for: whether a given proposal is signing-enabled, who may sign it (Owen only), how a failed submission is retried, and how a voided or already-accepted proposal blocks signing. FEAT-26.SPEC-001 and FEAT-26.SPEC-002 both reference FEAT-26.SPEC-003 rather than duplicating these checks, and FEAT-26.SPEC-003 defers to FEAT-03.SPEC-005 for the underlying single-acceptance and voided-proposal enforcement it extends rather than re-deriving it.

## Internal Dependency Map

```
FEAT-02 (Proposal Creation & Sending) -> [Nadia enables the e-signature toggle] -> FEAT-26.SPEC-003 (E-Signature Opt-In & Signing Access Rules) [cross-feature: toggle rendered on FEAT-02's screen]
FEAT-03.SPEC-001 (Proposal Review & Accept) -> [proposal is signing-enabled] -> FEAT-26.SPEC-003 (E-Signature Opt-In & Signing Access Rules) -> [Owen is eligible] -> FEAT-26.SPEC-001 (Signature Signing Step) [cross-feature: replaces FEAT-03's plain Accept control]
FEAT-26.SPEC-001 (Signature Signing Step) -> [Owen submits signature] -> FEAT-26.SPEC-003 (E-Signature Opt-In & Signing Access Rules) -> [submission valid] -> FEAT-26.SPEC-002 (Signature Recording)
FEAT-26.SPEC-001 (Signature Signing Step) -> [submission fails] -> FEAT-26.SPEC-003 (E-Signature Opt-In & Signing Access Rules) -> [preserve entered data] -> FEAT-26.SPEC-001 (retry)
FEAT-26.SPEC-002 (Signature Recording) -> [signature data submitted] -> FEAT-26.SPEC-005 (Electronic-Signature Attestation Capability) -> [attestation confirmed] -> FEAT-26.SPEC-002 (Signature Recording) [inbound event]
FEAT-26.SPEC-002 (Signature Recording) -> [signature record written] -> FEAT-03.SPEC-003 (Acceptance Recording) [cross-feature: same immutable acceptance write, deposit-invoice trigger, and audit-trail entry]
FEAT-26.SPEC-002 (Signature Recording) -> [signature recorded] -> FEAT-26.SPEC-004 (Signed-Copy Confirmation Notification)
```

**Default Entry:** FEAT-26.SPEC-001 (Signature Signing Step) — reached only when Owen opens a signing-enabled proposal; this feature has no independent navigation entry point of its own. FEAT-03.SPEC-001 (Proposal Review & Accept) remains the screen Owen actually lands on from the portal home or the emailed proposal link, and FEAT-26.SPEC-003's routing rule decides whether it shows the plain Accept control or hands off to this feature's signing step.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-26.SPEC-003 | Inbound | FEAT-02 (Proposal Creation & Sending) | The opt-in toggle is exposed on FEAT-02's send/edit screen; this feature owns the eligibility flag and the rules that govern it | Nadia enables e-signature for a proposal before sending |
| FEAT-26.SPEC-001, FEAT-26.SPEC-003 | Inbound | FEAT-03 (Proposal Acceptance) | Replaces FEAT-03.SPEC-001's plain Accept control with the signing step when enabled, with the same Primary-only access and the same immutability (XBR-34) | Owen opens a signing-enabled proposal |
| FEAT-26.SPEC-002 | Outbound | FEAT-03 (Proposal Acceptance) | Signature Recording triggers FEAT-03.SPEC-003's immutable acceptance write, so the signed acceptance still fires the deposit-invoice trigger (XBR-01) and the audit-trail entry | Signing step completed |
| FEAT-26.SPEC-002 | Outbound | FEAT-13 (Immutable Activity & Audit Trail) | Signed acceptance feeds the append-only trail (Interactions field; XBR-04) | Signature recorded |
| FEAT-26.SPEC-004 | Outbound | FEAT-14 (Notifications & Email) | Signed-copy confirmation email relies on FEAT-14's transactional email delivery capability (ASMP-29) for sending and delivery/bounce status | Signature recorded |
| FEAT-26.SPEC-003 | Inbound | FEAT-18 (Client Contact Management) | Role (Primary vs. Reviewer) feeds this feature's signing authorization, mirroring FEAT-03.SPEC-005 (XBR-08) | Contact role assigned or changed |
| FEAT-26.SPEC-001, FEAT-26.SPEC-003 | Inbound | FEAT-31 (Operator Support Access) | Dana's read-only, logged support session sees this feature's screen and marker with no signing control | Support session opened |

## Non-Functional Notes

**Data volumes / growth:** N/A — this feature does not introduce a new data-volume concern beyond the Proposal entity's own growth: at most one signature record per accepted proposal, only for proposals a freelancer opts in (Validation & Limits field), which is already bounded under FEAT-02/FEAT-03.

**Responsiveness:** Client-facing pages become interactive within roughly 2 seconds on a typical mobile connection (ASMP-21); the feature's own States field describes the signing step's Loading state as "N/A — a short signing step," so the signing step itself must complete within that same window rather than introducing a perceptibly slower path than FEAT-03's plain Accept.

**Data sensitivity / privacy:** The signature record (signer identity, signature data, timestamp) is personal data (ASMP-24) and, once written, evidentiary and immutable exactly like the acceptance record it extends (ASMP-25; dependency map, Proposal Data Sensitivity). The "Signed on {date}" marker is deliberately distinct from a plain "Accepted" marker so the stronger evidentiary status is visible to both parties (Data Notes field). Strict client isolation applies as in FEAT-03 (ASMP-23; XBR-09): only Owen's own company's proposal is ever reachable through this feature.

**Compliance flags:** GDPR-class handling applies to the signer's identity (ASMP-24); if that contact is later erased (FEAT-18), the signature record remains on the record under their name as evidence, exactly as FEAT-03's acceptance record does (XBR-27, ASMP-20). This feature's purpose is to give the acceptance record legal weight beyond a self-recorded timestamp; the specific regulatory standard that weight must satisfy (e.g., which jurisdictions' electronic-signature law the attestation capability must meet) is a product decision for Stage 4's selection of the electronic-signature attestation capability (FEAT-26.SPEC-005), not a determination this Brief makes.

## Non-Goals

- **Forcing e-signature account-wide** — Excluded per this feature's own Validation & Limits field: e-signature is available per proposal, not forced account-wide; a freelancer who never opts in never sees any change to FEAT-03's standard Accept flow.
- **Signing while offline** — Excluded per this feature's own States field ("Offline-degraded: signing requires connectivity, consistent with its legal-record purpose") and ASMP-27's principle that actions creating evidentiary records never pretend to succeed offline.
- **A formal "decline" state on the signing step** — Excluded per FEAT-03's own Non-Goals, which this feature inherits: there is no in-product decline; Request Changes (FEAT-03.SPEC-002) remains the product's only structured "not yet" path, unchanged by whether e-signature is enabled.
- **Multi-party or witnessed signing** — Excluded per the Access field: only the Primary contact (Owen) signs, the same access boundary as standard acceptance (FEAT-03); the product defines no witness, co-signer, or notarization role, and scope-boundaries.md's SC-01 excludes any multi-seat or team-of-many model that a witness role would imply.
- **A library of legal contract templates** — Adjacent capability excluded per scope-boundaries.md (SC-13): the product gives freelancers stronger evidentiary weight on their own scope text, not lawyer-vetted contract templates; providing legal documents across jurisdictions is outside a solo founder's capacity.
- **Retention/purge policy for the signature record** — Not applicable to this feature: the signature record is never deleted or archived independently of the Proposal it belongs to; retention and eventual deletion remain owned entirely by FEAT-24 (account deletion, subject to legal retention), so no separate lifecycle gap exists here to resolve.
</content>



# Screen Spec: Signature Signing Step

## Overview

**Name:** Signature Signing Step
**ID:** FEAT-26.SPEC-001
**Type:** Screen
**Purpose:** Owen, the client's Primary Contact, completes a signature step in place of the plain Accept click when e-signature is enabled for the proposal, producing a legally binding signed record instead of a timestamped click.
**Parent Feature:** FEAT-26 -- Legally Binding E-Signature for Proposals

## Scope and Non-Goals

**In Scope:**
- Rendering the same read-only proposal summary (scope, price, payment schedule) FEAT-03.SPEC-001 shows, followed by the signature capture step
- The signature capture step: a full-legal-name field, an attestation statement, and the "Sign" control that replaces the plain "Accept" control
- The "Request Changes" control, carried over unchanged from FEAT-03.SPEC-001
- The disclosure notice about what signature data is shared with the attestation capability, and the "How your signature is verified" link that reopens it (content owned by FEAT-26.SPEC-005)
- The slow-attestation "Still working" note on the Sign button (behavior owned by FEAT-26.SPEC-005), the proposal-content load-error state, and the "not authorized" denied state shown when a write-time authorization check fails (FEAT-26.SPEC-002)
- Displaying the permanent "Signed on {date}" marker once the signature is recorded, visually distinct from FEAT-03.SPEC-001's "Accepted on {date}" marker
- Redirecting to the current proposal version when the opened version has been voided
- Showing Priya's stage-only view and Dana's read-only support view of this same signing step

**Non-Goals:**
- A formal "decline" state -- excluded per the Feature Breakdown Brief's Non-Goals, inherited from FEAT-03: there is no in-product decline; Request Changes remains the product's only structured "not yet" path, unchanged by whether e-signature is enabled.
- Signing while offline -- excluded per the Feature Breakdown Brief's Non-Goals and ASMP-27: actions that create evidentiary records never pretend to succeed offline, so this screen never queues a signature for later submission.
- Multi-party or witnessed signing -- excluded per the Feature Breakdown Brief's Non-Goals: only the Primary contact (Owen) signs, the same access boundary as standard acceptance (FEAT-03); no witness, co-signer, or notarization step is rendered.
- Determining whether this proposal is signing-enabled or whether Owen is eligible to sign it -- owned entirely by FEAT-26.SPEC-003 (E-Signature Opt-In & Signing Access Rules); this screen only renders once that routing decision has already sent Owen here from FEAT-03.SPEC-001.
- Editing or voiding the proposal -- owned entirely by FEAT-02 (Proposal Creation & Sending); this screen only reads and reacts to the proposal's current state.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-03.SPEC-001 (Proposal Review & Accept) | Owen opens a proposal that FEAT-26.SPEC-003's routing rule finds signing-enabled and Owen eligible to sign | The proposal reference, its scope/price/payment-schedule content, and the signing contact's identity (name, from Client Contact) |
| This screen (retry after a failed signature submission) | Owen retries after FEAT-26.SPEC-002 (Signature Recording) reports a failed submission | The full legal name and attestation state Owen had already entered, preserved for retry |
| FEAT-03.SPEC-002 (Request Changes) | Owen taps "Back" or completes sending a change-request note | Confirmation that the note was sent; proposal state re-loaded, routing re-evaluated by FEAT-26.SPEC-003 |
| FEAT-26.SPEC-004 (Signed-Copy Confirmation Notification) | The recipient taps the email's "View signed proposal" CTA | The signed proposal reference; the screen shows "Signed on {accepted_at_date}" |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | None -- this screen is not part of Nadia's own workspace; she sees the resulting "Signed on {date}" status on the project view (FEAT-01), not this screen | No | Attempting to open this screen's link is treated as an out-of-scope link: plain explanation and a fresh-link option (XBR-09), never proposal or signature content |
| Owen (Client Primary Contact) | Full screen -- proposal summary, signature capture step, and current signing status of his own company's proposal (Own-only) | Sign the proposal; open Request Changes (FEAT-03.SPEC-002) | -- |
| Priya (Client Reviewer Contact) | No proposal or signature content -- portal home shows only the project's stage label (e.g., "Proposal signed"), per the Access Matrix (Proposals & Acceptance: None for Reviewers) | No | Attempting to reach this screen's link shows the project's stage label only, with no scope, price, signature capture, or Request Changes controls |
| Dana (Support Operator) | Full screen content, read-only, inside a logged support session (FEAT-31) | No actions -- the signature capture step and Request Changes control are not rendered | If Dana attempts an action outside the support session's read-only bounds, the action is not available on screen; there is no control to attempt it with |
| Unauthenticated | No | No | Redirected to the client portal's magic-link sign-in (FEAT-05); after signing in as a recognized contact, the user lands on this screen if they are Owen and eligible to sign, or on the stage-only view if Priya |
| Expired session | No | No | Magic link is single-use and time-limited (XBR-28); an expired link shows a plain explanation and a "request a fresh link" option (FEAT-05); no proposal or signature content is shown in the meantime |

## Layout and Content

**Header:** Project name and client company name at the top, with the proposal's status label ("Sent -- signature required" or "Signed on {date}") shown beside it.

**Body:** A single-column, read-only-then-actionable presentation, in this order:
- Scope description -- full text, identical to what FEAT-03.SPEC-001 shows, rendered before any control below it activates
- Price -- the proposal's price in the project's set currency
- Payment schedule summary -- the same short, plain-language statement of the payment structure FEAT-03.SPEC-001 shows
- A "Signature" section, grouped below the price and clearly separated from the read-only content above it:
  - An attestation statement: "By signing below, I agree to the scope and price of this proposal and intend this to be my legally binding signature."
  - The disclosure notice, placed directly beneath the attestation statement and above the "Full legal name" field, so it is read before the signature is captured: "Signing shares your full legal name and the time you sign with an external electronic-signature service, which attests to your signature. Your proposal's scope, price, and payment terms are never shared." It is shown expanded the first time Owen opens the signing step in a signing session and collapsed to the link below on later opens within that session (text owned by FEAT-26.SPEC-005, Consent and Disclosure). It has no Continue/Cancel gate.
  - A "How your signature is verified" text link, always visible directly beneath the "Full legal name" field; activating it expands (reopens) the same disclosure notice in place, and activating it again collapses it
  - A "Full legal name" text field, pre-filled with the signing Client Contact's name on file and editable, so the signature captures the name as Owen intends it to read
  - The "Sign" button -- large, clearly labeled, replacing the position FEAT-03.SPEC-001's "Accept" button occupies. While signing is in progress, the button shows a loading state; after 10 seconds without completion, a note appears directly beneath it: "Still working -- this is taking longer than usual."
- The "Request Changes" button, positioned beside "Sign" exactly as it sits beside "Accept" on FEAT-03.SPEC-001 (Shared UI Patterns: Decision control sizing)
- Once signed: the "Signature" section and "Request Changes" button are replaced in place by a single "Signed on {date}" marker (Shared UI Patterns: Immutable-record marker), visually distinct from a plain "Accepted on {date}" marker so a client can tell a signed acceptance apart from a plain one (Data Notes field); the price and scope remain visible below it, now in a visually settled, non-editable presentation

**Footer:** None -- both controls sit in the body, directly below the price and signature section.

### Responsive Behavior

- **Compact breakpoint:** Single-column body as described, full width; the "Full legal name" field spans the full content width; "Sign" and "Request Changes" stack vertically, Sign above Request Changes, each spanning the full content width for an easy mobile tap target.
- **Medium size class and above:** Body content is capped at a consistent platform-wide reading width and horizontally centered; "Sign" and "Request Changes" sit side by side, Sign on the left.
- **Payment schedule summary and attestation statement:** Uniform scaling, no structural change across breakpoints.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Scope description, price, payment schedule summary | Screen loads | Reads the current Proposal record's `scope_description`, `price`, `currency`, `payment_schedule_reference` | Content renders fully; signature controls remain inactive until render completes | Loading indicator while content renders, then full content appears |
| Full legal name field | Type | Captures the entered name, overriding the pre-filled default | Field shows the entered text | Standard input focus state |
| Full legal name field | Blur (empty) | Triggers field validation via FEAT-26.SPEC-003 | Error state on field | "Enter your full legal name to sign." below the field |
| Sign button | Tap | 1. Checks eligibility and field validity via FEAT-26.SPEC-003. 2. If valid, triggers FEAT-26.SPEC-002 (Signature Recording), which submits the signature data for attestation via FEAT-26.SPEC-005. | Button shows a loading state during the check, attestation, and write | Success: the Signature section and Request Changes are both replaced by the "Signed on {date}" marker. Failure: see Edge Cases (already signed, voided, attestation failure, connectivity). |
| "How your signature is verified" link | Tap, or Enter/Space when focused | Toggles the disclosure notice open or closed in place (same text as the first-open notice, per FEAT-26.SPEC-005) | Disclosure notice expanded or collapsed; link's expanded/collapsed state updates | The notice appears or disappears beneath the attestation statement; focus stays on the link |
| Disclosure notice | Screen loads for the first time in a signing session | Renders expanded above the "Full legal name" field | Notice expanded; no acknowledgement required | Notice is visible with the signature capture step; Sign is not gated on it |
| Sign button (slow state) | 10 seconds elapse after Sign was tapped with no success or failure yet | Shows the note beneath the button | Note visible; Sign stays in its loading state and other controls stay unchanged | "Still working -- this is taking longer than usual."; the note clears when the attempt succeeds or fails |
| Sign button (not authorized) | FEAT-26.SPEC-002 returns the "denied -- not authorized" outcome (role no longer Primary, or status no longer Active, at the moment of Sign) | Replaces the screen body with the denied message and a "Go to portal home" link | Not-authorized state entered; no signature recorded; entered name discarded | "You no longer have permission to sign this proposal. If you think this is a mistake, ask the person who sent it." |
| "Try again" control (load error) | Tap | Re-requests the proposal content | Loading state re-entered; on success, Ready-to-sign (or Signed) state loads | Loading indicator, then content or the same load-error message again |
| Request Changes button | Tap | Navigates to FEAT-03.SPEC-002 (Request Changes) | Screen transitions to the Request Changes form | Standard navigation transition |
| "Signed on {date}" marker | None -- display-only | None | None | Non-interactive; communicates the permanent record |
| Payment schedule summary | None -- display-only | None | None | Non-interactive; provides context ahead of the decision |

### Accessibility Notes

- **Focus order:** Header status label -> scope description -> price -> payment schedule summary -> attestation statement -> disclosure notice (when expanded) -> full legal name field -> "How your signature is verified" link -> Sign button -> "Still working" note (when shown) -> Request Changes button (or, once signed, the "Signed on {date}" marker in their place).
- **Disclosure link and notice:** The link is a standard keyboard-operable control (Enter and Space toggle it) exposing its expanded/collapsed state to assistive technology; toggling never moves focus off the link. When the notice first renders expanded, it is part of the reading order before the full legal name field, so a screen-reader user hears it before entering a name.
- **Slow-state announcement:** The "Still working -- this is taking longer than usual." note is announced as a polite live-region update when it appears, so a screen-reader user is not left waiting silently.
- **Error announcements:** The load-error message, the sign-failed message, and the not-authorized message are each announced as live-region updates when they appear; on load error, focus moves to the "Try again" control, and on not-authorized, focus moves to the "Go to portal home" link.
- **Content-ready announcement:** When the scope and price finish rendering and the signature controls become active, this transition is announced to assistive technology so a screen-reader user is not left waiting on a silently inert control.
- **Signature announcement:** When signing succeeds, the "Signed on {date}" marker replacing the controls is announced as a live-region update, distinct in its announced text from FEAT-03.SPEC-001's "Accepted on {date}" announcement.
- **Keyboard alternatives:** The full legal name field, Sign, and Request Changes are all standard activatable controls reachable and operable by keyboard; there are no pointer-only gestures on this screen.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Empty | N/A -- this screen exists only once FEAT-26.SPEC-003's routing rule has sent Owen here for a signing-enabled proposal; there is no zero-data variant of this screen to render | Never entered | Never entered |
| Loading | Scope, price, and payment schedule summary show a loading indicator; the signature section is not yet rendered | Screen first opens, or Owen taps "Try again" from the Load error state | Proposal content finishes loading (Ready to sign or Signed) or fails (Load error) |
| Ready to sign (default) | Full scope, price, and payment schedule summary shown; full legal name field pre-filled and editable; Sign and Request Changes are both active | Content load completes and the proposal's status is Sent with signing enabled | Owen taps Sign (success) or navigates to Request Changes |
| Signed | Scope and price remain visible; the Signature section and Request Changes are replaced by the "Signed on {date}" marker | The signature is successfully recorded (this session or a prior one) | Never exits -- this is a permanent, terminal state for this proposal version |
| Load error | Message in place of the proposal content: "We couldn't load this proposal. Check your connection and try again." with a "Try again" control; the Signature section, Sign, and Request Changes are not rendered; no stale or partial proposal content is shown | The proposal content request fails server-side or times out on first load or on a Try-again attempt | Owen taps "Try again" and the content loads (Ready-to-sign or Signed), or he navigates away |
| Signing in progress (slow) | Sign button in loading state with the note "Still working -- this is taking longer than usual." beneath it; scope, price, and full legal name field remain visible | 10 seconds pass after Sign was tapped without a success or failure result | The attempt succeeds (Signed) or fails (Error -- sign failed, Voided -- redirected, or Not authorized) |
| Not authorized | Screen body replaced by "You no longer have permission to sign this proposal. If you think this is a mistake, ask the person who sent it." and a "Go to portal home" link; no proposal content, signature capture step, or Request Changes control is shown | FEAT-26.SPEC-002 returns "denied -- not authorized" at Sign time (the contact's role is no longer Primary, or status is no longer Active) | Owen follows "Go to portal home"; subsequent access follows the Access and Visibility table for his current role or, if Removed, the expired-link explanation |
| Error (sign failed) | Inline message below the controls: "We couldn't record your signature. Check your connection and try again." Full legal name field retains its entered value; controls remain active for retry. | The signature submission fails mid-flight (attestation-capability error, validation failure, or connectivity drop) | Owen retries successfully, or navigates away |
| Voided -- redirected | Brief message "This proposal has been updated" before the current version loads | Owen opens a proposal version that was edited and re-sent after this link was generated | Redirect completes and the current version's Ready-to-sign (or Signed) state loads |
| Offline/Degraded | Scope and price remain visible from the last successful load; the signature section is visibly disabled with the message "Reconnect to sign" beneath it | Connectivity is lost while this screen is open | Connectivity returns -- controls re-activate automatically, no false success is ever shown |

## Validation Rules

Validation and eligibility governed by FEAT-26.SPEC-003 (E-Signature Opt-In & Signing Access Rules). See that spec for the full-legal-name field rule, signing-eligibility (signing-enabled, not-yet-signed, not-voided), and role-based access rules. This screen calls that eligibility check at the moment Sign is tapped, not before.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|-------------------------------------|
| Request Changes button tap | FEAT-03.SPEC-002 (Request Changes) | FEAT-03 |
| Successful signature | Stays on this screen, now in the Signed state | -- |
| Voided-version redirect | This same screen, reloaded against the current proposal version | -- |
| Return navigation from FEAT-03.SPEC-002 | This screen, reloaded (routing re-evaluated by FEAT-26.SPEC-003) | -- |

## Data Model

**Creates:** None.
**Reads:** Proposal -- `scope_description`, `price`, `currency`, `payment_schedule_reference`, `status`, `sent_at`, signing-enabled flag, signature record fields (signer identity, signature data, timestamp), `accepted_at`, `accepted_by`. Client Contact -- the signed-in contact's `name`, `role`, and `status`, to pre-fill the full legal name field and confirm the viewer is a Primary contact in Active status.
**Updates:** None directly -- Sign triggers FEAT-26.SPEC-002, which performs the Proposal update (signature record and the `accepted_at`/`accepted_by` fields together, per FEAT-03.SPEC-003's write).
**Deletes:** None.

## Business Rules

- XBR-34: When enabled for a proposal, a legally binding e-signature replaces the plain Accept click with the same Primary-only access and the same immutability; this screen is that replacement, rendered only when FEAT-26.SPEC-003's routing rule finds the proposal signing-enabled.
- Eligibility to sign (signing-enabled, not-yet-signed, voided-cannot-sign) is governed by FEAT-26.SPEC-003 -- this screen does not duplicate that logic, it calls the check.
- XBR-06: A voided proposal cannot be signed; this screen redirects Owen to the current version rather than showing the stale one as actionable.
- XBR-08, XBR-09: Only a Primary contact at the owning client may view or sign, and only within his own company's proposal; Priya (Reviewer) never reaches this screen's content, and an out-of-scope link shows a plain explanation and a fresh-link option.
- The scope and price render fully before the signature controls activate, preventing an accidental early tap, consistent with FEAT-03.SPEC-001's same rule.
- The full legal name field is pre-filled from the Client Contact's name on file but remains editable, so the captured signature reflects the name Owen intends to sign with.
- The disclosure notice and the "How your signature is verified" link are part of the signature capture step and are never rendered for Dana (read-only support view) or once the proposal is Signed; the notice text is owned by FEAT-26.SPEC-005 and is not restated or altered here.
- A signer whose role or status no longer qualifies at the moment of Sign is denied by FEAT-26.SPEC-002's authoritative write-time check (governed by FEAT-26.SPEC-003) and sees the Not authorized state; this screen never records a signature for that contact.

## Edge Cases

- **Owen taps Sign a second time, or against a version voided in the meantime** -- Reject-with-refresh per FEAT-26.SPEC-003: the screen shows "This proposal has already been signed" (if already signed) or redirects to the current version (if voided), without recording a duplicate signature.
- **Connectivity drops immediately after Owen taps Sign** -- The action is retried without recording a duplicate signature or losing the entered full legal name (Feature Breakdown Brief, Side-Effect Inventory); the screen shows the Error state with a retry option rather than a false success, and the full legal name field keeps its entered value.
- **The attestation capability (FEAT-26.SPEC-005) rejects or times out on the signature submission** -- Same Error state as a connectivity failure: "We couldn't record your signature. Check your connection and try again." The entered full legal name is preserved; the signature is not recorded until a retry succeeds.
- **Owen navigates away and returns before signing** -- The screen re-fetches the proposal's current state; the full legal name field re-fills from the Client Contact's name on file, since no local draft exists to preserve across navigations.
- **A proposal that was signing-enabled is voided and re-sent without signing enabled on the new version** -- The redirect lands Owen on FEAT-03.SPEC-001's plain Accept flow for the current version, per FEAT-26.SPEC-003's routing rule re-evaluating the current version's signing-enabled flag, not the version this screen was opened for.
- **Priya follows a proposal link meant for Owen** -- No proposal or signature content is shown; her portal home shows only the project's stage label, per the Access Matrix.
- **Dana opens this screen inside a support session** -- Full content renders read-only; the signature capture step and Request Changes control are not rendered at all, so there is no control for Dana to attempt.
- **Owen's role is changed to Reviewer, or his status to Removed, while he is on this screen, and he then taps Sign** -- FEAT-26.SPEC-002's write-time check denies the action; the screen shows the Not authorized state with the message "You no longer have permission to sign this proposal. If you think this is a mistake, ask the person who sent it.", no signature is recorded, and the entered name is discarded.
- **The proposal content fails to load (server-side failure or timeout)** -- The screen shows the Load error state with "We couldn't load this proposal. Check your connection and try again." and a "Try again" control; no signature controls are rendered, so Owen cannot sign against content he has not seen.
- **Attestation takes longer than 10 seconds** -- The "Still working -- this is taking longer than usual." note appears beneath the loading Sign button; Sign stays disabled while in flight, and the note clears when the attempt finishes either way.
- **Owen collapses the disclosure notice, then re-opens it with the keyboard** -- Enter or Space on the "How your signature is verified" link toggles the same notice; focus stays on the link.
- **Owen attempts to open the signing step without a connection** -- The screen shows "Reconnect to sign" and the signature controls are disabled; the signing step never pretends to succeed offline, consistent with ASMP-27 and the feature's Offline-degraded field.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|-------------------|--------------|
| FEAT-03.SPEC-001 (Proposal Review & Accept) | Navigation (inbound) | Owen arrives here when FEAT-26.SPEC-003's routing rule finds the proposal signing-enabled and Owen eligible; otherwise FEAT-03.SPEC-001 shows the plain Accept flow |
| FEAT-03.SPEC-002 (Request Changes) | Navigation (outbound) | Request Changes button navigates here, carried over unchanged from FEAT-03 |
| FEAT-26.SPEC-002 (Signature Recording) | Triggers (outbound) | Sign button, once eligible, triggers the signature validation, attestation, and immutable write; its "denied -- not authorized" outcome drives the Not authorized state |
| FEAT-26.SPEC-005 (Electronic-Signature Attestation Capability) | References (outbound) | Owns the disclosure notice text, the "How your signature is verified" link behavior, and the 10-second "Still working" note surfaced on this screen |
| FEAT-26.SPEC-003 (E-Signature Opt-In & Signing Access Rules) | References (outbound) | Eligibility, field validation, access, and concurrent-action rules for the Sign action |
| FEAT-05 (Client Portal Access) | Navigation (inbound) | Owen ultimately arrives from his portal home, by way of FEAT-03.SPEC-001's routing |
| FEAT-31 (Operator Support Access) | Navigation (inbound) | Dana views this screen read-only inside a logged support session |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|---------------|-------------------|
| esignature_signing_step_viewed | proposal reference, contact role | Screen finishes loading and the signature capture step is shown to Owen | N/A -- no Stage 2 metric in success-metrics.md is connected to FEAT-26 (its Connected Feature slice is empty; "Time to Proposal Acceptance" is connected to FEAT-03, not to this feature's signing behavior specifically); retained as the product-defined signal named in product-features.md's Signals field (esignature_enabled, proposal_signed) |
| proposal_signed | proposal reference, time elapsed since sent_at | FEAT-26.SPEC-002 confirms the signature was written | N/A -- same reason as above; product-features.md names this exact signal (proposal_signed) for the feature without a connected Stage 2 metric |
| esignature_sign_failed | failure reason (connectivity, attestation-capability error, already signed, voided, not authorized) | The Sign action does not complete successfully | N/A -- no connected success metric measures signing failures; recorded here as diagnostic-only exhaust from the sign flow, mirroring FEAT-03.SPEC-001's proposal_accept_failed event |

## Acceptance Criteria

**FEAT-26.SPEC-001-AC-01:** Given Owen opens a signing-enabled proposal routed to him by FEAT-26.SPEC-003, when the screen loads, then the scope description, price, and signature capture step render fully before the Sign and Request Changes controls become active.

**FEAT-26.SPEC-001-AC-02:** Given Owen is on this screen with the proposal in Sent status and signing enabled, when he enters his full legal name and taps Sign, then the signature is recorded and the Signature section and Request Changes are replaced by a "Signed on {date}" marker.

**FEAT-26.SPEC-001-AC-03:** Given Owen is on this screen, when he taps Request Changes, then he is navigated to FEAT-03.SPEC-002 (Request Changes).

**FEAT-26.SPEC-001-AC-04:** Given Owen is on a proposal already showing "Signed on {date}", when he views the screen, then no signature capture step or Request Changes control is shown.

**FEAT-26.SPEC-001-AC-05:** Given Owen leaves the full legal name field empty and moves focus away, when field validation runs, then the field shows the error "Enter your full legal name to sign."

**FEAT-26.SPEC-001-AC-06:** Given Owen taps Sign on a proposal that was already signed moments earlier, when the eligibility check runs, then the screen shows "This proposal has already been signed" and no duplicate signature is recorded.

**FEAT-26.SPEC-001-AC-07:** Given Owen opens a signing-enabled proposal link for a version that Nadia has since edited and re-sent, when the screen loads, then he is redirected to the current version of the proposal.

**FEAT-26.SPEC-001-AC-08:** Given Owen taps Sign and loses connectivity mid-flight, when connectivity drops, then the screen shows an inline error message with a retry option, the entered full legal name is preserved, and no signature is recorded until a retry succeeds.

**FEAT-26.SPEC-001-AC-09:** Given the attestation capability rejects Owen's signature submission, when the rejection is returned, then the screen shows the same inline error and retry option as a connectivity failure, with the entered full legal name preserved.

**FEAT-26.SPEC-001-AC-10:** Given Owen loses connectivity while viewing the screen, when he attempts to interact with the signature controls, then they are visibly disabled with the message "Reconnect to sign."

**FEAT-26.SPEC-001-AC-11:** Given Priya (Reviewer) follows a signing-enabled proposal link meant for Owen, when her portal home loads, then she sees only the project's stage label and no scope, price, signature capture, or Request Changes control.

**FEAT-26.SPEC-001-AC-12:** Given Dana (Support Operator) opens this screen inside a logged support session, when the screen loads, then the full proposal and signature content renders but no signature capture step or Request Changes control is shown.

**FEAT-26.SPEC-001-AC-13:** Given an unauthenticated visitor opens a signing-enabled proposal link, when the screen would otherwise load, then they are redirected to the client portal's magic-link sign-in.

**FEAT-26.SPEC-001-AC-14:** Given Owen's sign-in session has expired, when he opens the signing-enabled proposal link, then he sees a plain explanation and a "request a fresh link" option, with no proposal or signature content shown.

**FEAT-26.SPEC-001-AC-15:** Given Owen is on the screen in the Ready-to-sign state, when he views the layout at a compact breakpoint, then the Sign and Request Changes controls are stacked vertically, Sign above Request Changes.

**FEAT-26.SPEC-001-AC-16:** Given Owen successfully signs the proposal, when the signature is recorded, then a proposal_signed analytics event is emitted with the proposal reference and time elapsed since sent_at.

**FEAT-26.SPEC-001-AC-17:** Given Owen opens a signing-enabled proposal's signing step for the first time in a signing session, when the screen loads, then the disclosure notice "Signing shares your full legal name and the time you sign with an external electronic-signature service, which attests to your signature." is shown expanded beneath the attestation statement and above the "Full legal name" field, and Sign is not gated on any acknowledgement.

**FEAT-26.SPEC-001-AC-18:** Given the disclosure notice is collapsed, when Owen focuses the "How your signature is verified" link and presses Enter or Space (or taps it), then the same disclosure notice expands in place, focus remains on the link, and activating the link again collapses it.

**FEAT-26.SPEC-001-AC-19:** Given Owen has tapped Sign and the attempt has not completed, when 10 seconds elapse, then the note "Still working -- this is taking longer than usual." appears beneath the loading Sign button, is announced to assistive technology, and clears when the attempt succeeds or fails.

**FEAT-26.SPEC-001-AC-20:** Given Owen's role is changed to Reviewer or his status to Removed while he is on the screen, when he taps Sign and FEAT-26.SPEC-002 returns "denied -- not authorized", then the screen body is replaced by "You no longer have permission to sign this proposal. If you think this is a mistake, ask the person who sent it." with a "Go to portal home" link, and no signature is recorded.

**FEAT-26.SPEC-001-AC-21:** Given the proposal content fails to load, when the load fails, then the screen shows "We couldn't load this proposal. Check your connection and try again." with a "Try again" control, and the Signature section, Sign, and Request Changes are not rendered.

**FEAT-26.SPEC-001-AC-22:** Given the screen is in the Load error state, when Owen taps "Try again" and the content then loads, then the screen leaves the Load error state and shows the Ready-to-sign state (or the Signed state if a signature already exists).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 12 | 12 |
| States | 10 (empty, loading, ready to sign, signed, load error, signing in progress (slow), not authorized, error (sign failed), voided-redirected, offline) | 10 |
| Business Rules | 8 | 8 |
| Edge Cases | 12 | 12 |



# Automation Spec: Signature Recording

## Overview

**Name:** Signature Recording
**ID:** FEAT-26.SPEC-002
**Type:** Automation
**Purpose:** Validates Owen's signature submission, submits it for electronic-signature attestation, and writes the immutable signature record alongside FEAT-03's acceptance fields so the signed acceptance still fires the deposit invoice and audit-trail entry.
**Parent Feature:** FEAT-26 -- Legally Binding E-Signature for Proposals

## Scope and Non-Goals

**In Scope:**
- Re-checking signing eligibility at write time (signing-enabled, not-yet-signed, not-voided) and the actor's authorization at write time (signing contact's role still Primary and status still Active)
- Submitting the signature data (full legal name, timestamp) to the electronic-signature attestation capability (FEAT-26.SPEC-005) and receiving its attestation confirmation
- Writing the Proposal's signature record (signer identity, signature data, timestamp) and, in the same atomic step, the `status`, `accepted_at`, and `accepted_by` fields FEAT-03.SPEC-003 owns
- Firing FEAT-03.SPEC-003's downstream effects for the signed acceptance: the deposit-invoice trigger to FEAT-09 (XBR-01) and the audit-trail entry to FEAT-13 (XBR-05)
- Firing FEAT-26.SPEC-004 (Signed-Copy Confirmation Notification) in addition to FEAT-03.SPEC-006's standard acceptance confirmation

**Non-Goals:**
- Generating or sending the deposit invoice itself -- owned entirely by FEAT-09 (Invoice Generation & Sending); this automation only fires the trigger per XBR-01, exactly as FEAT-03.SPEC-003 does for a plain acceptance.
- Composing or sending either confirmation email's content -- FEAT-03.SPEC-006 and FEAT-26.SPEC-004 own their own content; this automation only triggers both.
- Determining whether the proposal is signing-enabled or who may sign it -- owned by FEAT-26.SPEC-003 (E-Signature Opt-In & Signing Access Rules), which this automation re-checks and enforces at write time rather than re-deriving.
- Choosing or configuring the electronic-signature attestation vendor -- vendor selection is a Stage 4 architecture decision (FEAT-26.SPEC-005); this automation only defines the functional exchange with whatever capability is selected.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|-------------------|
| Owen taps Sign | FEAT-26.SPEC-001 (Signature Signing Step) | Fires after FEAT-26.SPEC-003's eligibility check passes on that screen (proposal is Sent, signing-enabled, not yet signed, not Voided, and the actor is a Primary contact at the owning client) | Proposal reference, signing Client Contact's identity, entered full legal name, current timestamp |

## Processing Logic

1. Receive the proposal reference, the signing Client Contact's identity, and the entered full legal name from FEAT-26.SPEC-001, after FEAT-26.SPEC-003's eligibility check has passed on that screen.
2. Re-check eligibility and authorization at write time against current state: (a) the Proposal's `status` must still be `Sent`, its signing-enabled flag must still be set, and no signature record must yet exist; and (b) the signing Client Contact's current `role` must still be Primary and `status` must still be Active, at the client that owns the proposal (FEAT-26.SPEC-003 Authorization Rules; XBR-08, XBR-27). If check (b) fails, stop immediately and return the "denied -- not authorized" outcome: no write is performed and nothing is submitted to attestation. Check (b) is evaluated before the outcomes in steps 3 and 4 so an unauthorized actor learns nothing about the proposal's signed or voided state. This re-check is the authoritative one -- the screen-time check in step 1 only gates the user's initial tap.
3. If the re-check fails because a signature record already exists, stop and return the "already signed" outcome (no write performed).
4. If the re-check fails because the proposal is `Voided`, stop and return the "voided -- redirect" outcome (no write performed).
5. If the re-check passes, submit the signature data (the entered full legal name, the signing Client Contact's reference, and the current timestamp) to the electronic-signature attestation capability (FEAT-26.SPEC-005) for attestation.
6. If the attestation capability confirms the submission, proceed to step 7. If it rejects the submission or the request fails, stop and return the "attestation failed" outcome (no write performed).
7. Write the signature and acceptance in a single, atomic step: set the signature record's signer identity, signature data (the entered full legal name), and timestamp; and set the Proposal's `status` to `Accepted`, `accepted_at` to the current timestamp, and `accepted_by` to the signing Client Contact's reference. This write is exactly-once: the atomic step itself is what prevents two concurrent passes of steps 2-7 from both succeeding.
8. Read the Project's Payment Schedule as it stood at this exact moment (dependency map, Payment Schedule Contention: "a trigger uses the schedule as it stood at the moment of the triggering action").
9. If the Payment Schedule's structure includes a deposit, fire the deposit-invoice trigger to FEAT-09 (XBR-01), passing the Payment Schedule's deposit terms as read in step 8, exactly as FEAT-03.SPEC-003 does for a plain acceptance.
10. If the Payment Schedule's structure does not include a deposit, take no invoicing action.
11. Fire the audit-trail entry to FEAT-13 (XBR-05) with event type "proposal signed," the signing contact as actor, the timestamp from step 7, and the Proposal as the affected record.
12. Fire FEAT-03.SPEC-006 (Acceptance Confirmation Notification) and FEAT-26.SPEC-004 (Signed-Copy Confirmation Notification) to notify Owen and Nadia.
13. Return the success outcome to FEAT-26.SPEC-001, which displays the "Signed on {date}" marker.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|-----------------|-------------------|
| Signature recorded, no deposit due | Attestation confirmed; write succeeds; Payment Schedule has no deposit trigger | Proposal signature record written; `status` set to Accepted, `accepted_at` and `accepted_by` written | FEAT-26.SPEC-001 shows the "Signed on {date}" marker | FEAT-26.SPEC-001, FEAT-03.SPEC-006, FEAT-26.SPEC-004, FEAT-13 |
| Signature recorded, deposit invoice triggered | Attestation confirmed; write succeeds; Payment Schedule includes a deposit | Proposal signature record written; `status` set to Accepted, `accepted_at` and `accepted_by` written; deposit-invoice trigger fired to FEAT-09 | FEAT-26.SPEC-001 shows the "Signed on {date}" marker; the deposit invoice appears shortly after via FEAT-09's own notification | FEAT-26.SPEC-001, FEAT-03.SPEC-006, FEAT-26.SPEC-004, FEAT-09, FEAT-13 |
| Denied -- not authorized | Write-time check (step 2b) finds the signing contact's role is no longer Primary (e.g., changed to Reviewer) or status is no longer Active (e.g., Removed) | None -- no attestation submission, no write | FEAT-26.SPEC-001 shows the Not authorized state: "You no longer have permission to sign this proposal. If you think this is a mistake, ask the person who sent it." with a "Go to portal home" link; no proposal content is shown | FEAT-26.SPEC-001 |
| Already signed | Write-time re-check finds a signature record already exists (a concurrent signature won the race, or the client re-submitted) | None -- no duplicate write | FEAT-26.SPEC-001 shows "This proposal has already been signed" | FEAT-26.SPEC-001 |
| Voided -- redirect | Write-time re-check finds `status` is `Voided` (Nadia edited and re-sent between screen load and this write) | None | FEAT-26.SPEC-001 redirects Owen to the current proposal version | FEAT-26.SPEC-001 |
| Attestation failed | The electronic-signature attestation capability rejects the submission or the request itself fails | None -- no partial signature record is written | FEAT-26.SPEC-001 shows "We couldn't record your signature. Check your connection and try again." with the entered full legal name preserved | FEAT-26.SPEC-001 |
| Write failure (connectivity or processing error, after attestation is confirmed) | The write itself does not complete after the attestation capability has already confirmed | None -- the write either fully completes or leaves no partial signature record | FEAT-26.SPEC-001 shows the same inline error with a retry option; retrying re-runs this automation from step 2, re-submitting to attestation only if step 5 has not already succeeded for a record that now exists | FEAT-26.SPEC-001 |

## Data Model

**Reads:** Proposal -- `status`, `sent_at`, signing-enabled flag, existing signature record (to confirm eligibility). Payment Schedule -- `structure`, `deposit_amount` (read as it stands at the exact moment of signing). Client Contact -- the signing contact's identity, `role`, and `status` (re-read at write time for the authorization check).
**Creates:** None directly -- the deposit-invoice trigger causes FEAT-09 to create an Invoice; the audit-trail trigger causes FEAT-13 to create an Activity Log Entry. Neither record is created by this automation itself.
**Updates:** Proposal -- signature record (signer identity, signature data, timestamp), `status` (Sent to Accepted), `accepted_at`, `accepted_by`. Written exactly once; never altered afterward (XBR-04).
**Deletes:** None.

## Business Rules

- XBR-34: When enabled for a proposal, a legally binding e-signature replaces the plain Accept click with the same Primary-only access and the same immutability. This automation is the write that carries out that replacement.
- XBR-01: Signing a proposal immediately generates and sends a deposit invoice when the Payment Schedule includes a deposit, using the schedule as it stood at signing -- identical to FEAT-03.SPEC-003's rule for a plain acceptance.
- XBR-04: The signature and acceptance record are never silently altered once written -- the signature record, `accepted_at`, and `accepted_by` are set exactly once and are never updated by any later process.
- XBR-05: Signing writes an append-only Activity Log Entry with actor and timestamp.
- The Proposal Contention resolution (dependency map): signing is recorded exactly once; a second signing attempt after the first succeeds is denied with "This proposal has already been signed" -- enforced by this automation's atomic write in Processing Logic step 7, not by the triggering screen.
- The write-time eligibility re-check (step 2) is authoritative over the screen-time check performed by FEAT-26.SPEC-003 on FEAT-26.SPEC-001 -- the screen check only prevents an obviously stale tap; this automation's own re-check is what actually guarantees exactly-once signing.
- XBR-08, XBR-27: Only a Primary contact in Active status may sign. This automation re-checks the actor's current role and status at write time (step 2b) -- a contact whose role changed or who was Removed after the screen loaded is denied with the "denied -- not authorized" outcome, and no signature or acceptance is written for them.
- The signature is not written until the electronic-signature attestation capability confirms it (step 6) -- a rejected or failed attestation never leaves a partial signature record.

## Edge Cases

- **Owen taps Sign, and a second signing attempt for the same proposal is submitted before the first completes (e.g., a slow connection and a retried tap)** -- Concurrent trigger firing: both invocations reach step 2 near-simultaneously, but the atomic write in step 7 admits only one. The first to complete the atomic write succeeds; the second's re-check (step 2) then finds a signature record already exists and returns the "already signed" outcome. Neither invocation blocks the other; there is no queuing.
- **Owen taps Sign, this automation begins, and the screen submits a retry before the first run finishes** -- Trigger fires while a previous run is in flight: FEAT-26.SPEC-001 disables the Sign control while the automation is running (screen-level debounce), so a second automation run for the same proposal from the same session cannot start until the first completes. If it did reach this automation regardless, step 2's re-check on the second run would find the first run's write already applied (once it commits) and return "already signed," or would race the still-in-flight first run under the same exactly-once write guarantee as the concurrent-trigger case.
- **Owen's role is changed to Reviewer, or his status to Removed, after the signing step loaded but before this automation's write-time check** -- Step 2b finds the role or status no longer qualifies and returns "denied -- not authorized"; no attestation submission and no write occur, and FEAT-26.SPEC-001 shows the Not authorized state. If the change lands after step 2b has passed, the atomic write in step 7 still proceeds for that in-flight action, since authorization was valid at the authoritative check.
- **Nadia edits and re-sends the proposal between Owen's tap and this automation's write** -- The write-time re-check (step 2) is the authority here, not the screen-time check: if the void completes before this automation's re-check runs, the re-check finds `status: Voided` and returns "voided -- redirect" with no signature recorded.
- **The electronic-signature attestation capability confirms the submission but the atomic write then fails (e.g., a connectivity drop immediately after step 6)** -- No partial signature record is left; the acceptance record and signature remain unwritten until a retry re-submits and succeeds. Owen sees the same retry-capable error as an outright attestation rejection.
- **The Payment Schedule is being adjusted by Nadia at the exact moment of signing** -- Per the dependency map's Payment Schedule Contention, this automation reads the schedule as it stood at the moment of signing (step 8); a schedule edit saved afterward is dated and applies to later triggers only, never retroactively to this signature's deposit determination.
- **The deposit-invoice trigger to FEAT-09 fails to fire after the signature write has already succeeded** -- The signature and acceptance record are not rolled back (they are already the evidentiary, immutable record per XBR-04); the deposit-invoice trigger is retried by FEAT-09's own retry handling. Owen still sees "Signed on {date}" -- the deposit invoice's own appearance is FEAT-09's concern, not this automation's failure path.
- **The audit-trail trigger to FEAT-13 fails to fire** -- Non-blocking: the signature record and any deposit-invoice trigger already fired are unaffected; the audit-trail write is retried by FEAT-13's own handling.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|-------------------|--------------|
| FEAT-26.SPEC-001 (Signature Signing Step) | Triggered by (inbound) | Sign button tap, after eligibility passes |
| FEAT-26.SPEC-003 (E-Signature Opt-In & Signing Access Rules) | References (inbound) | Eligibility, sign-once, and voided-proposal rules this automation re-checks and enforces at write time |
| FEAT-26.SPEC-005 (Electronic-Signature Attestation Capability) | Triggers (outbound) | Submits the signature data for attestation before the record is written |
| FEAT-03.SPEC-003 (Acceptance Recording) | References (outbound) | This automation performs the same acceptance write FEAT-03.SPEC-003 owns, extended with the signature record, per XBR-34 |
| FEAT-03.SPEC-006 (Acceptance Confirmation Notification) | Triggers (outbound) | Fires the standard confirmation email to Owen and Nadia once the signature is written |
| FEAT-26.SPEC-004 (Signed-Copy Confirmation Notification) | Triggers (outbound) | Fires the signed-copy confirmation email to Owen and Nadia in addition to the standard confirmation |
| FEAT-09 (Invoice Generation & Sending) | Triggers (outbound) | Fires the deposit-invoice trigger (XBR-01) when the schedule includes a deposit |
| FEAT-13 (Immutable Activity & Audit Trail) | Triggers (outbound) | Fires the append-only trail entry for the signed-acceptance event (XBR-05) |

## Analytics and Success Signals

- **proposal_signed** (proposal reference, deposit invoice triggered: yes/no, time elapsed since sent_at) -- N/A -- no Stage 2 metric in success-metrics.md is connected to FEAT-26; this event is the product-defined signal named in product-features.md's Signals field (proposal_signed), retained so signing volume is observable even without a connected metric
- **esignature_attestation_confirmed** (proposal reference) -- N/A -- no connected success metric measures attestation outcomes specifically; retained as the trigger-side signal for the electronic-signature capability's own reliability, mirroring FEAT-03.SPEC-003's deposit_invoice_auto_generated event
- **signature_write_failed** (failure reason: attestation-rejected / attestation-timeout / connectivity / concurrent-write-lost / not-authorized) -- N/A -- no connected success metric measures write failures; recorded as diagnostic-only exhaust from this automation's write path, mirroring FEAT-03.SPEC-003's proposal_accept_write_failed event

## Acceptance Criteria

**FEAT-26.SPEC-002-AC-01:** Given Owen submits his signature on a proposal in Sent status with a Payment Schedule that includes no deposit, when the attestation capability confirms the submission and this automation runs, then the signature record is written, the Proposal's `status` becomes Accepted with `accepted_at` and `accepted_by` set, and no invoice trigger fires.

**FEAT-26.SPEC-002-AC-02:** Given Owen submits his signature on a proposal in Sent status with a Payment Schedule that includes a deposit, when this automation runs, then the signature and acceptance are recorded and the deposit-invoice trigger fires to FEAT-09 (XBR-01), using the schedule as it stood at that moment.

**FEAT-26.SPEC-002-AC-03:** Given the signature write succeeds, when this automation completes, then it fires FEAT-03.SPEC-006 and FEAT-26.SPEC-004 to notify Owen and Nadia, and fires the FEAT-13 audit-trail entry for the signed-acceptance event.

**FEAT-26.SPEC-002-AC-04:** Given two signing attempts for the same proposal reach this automation at effectively the same moment, when the write-time re-check runs for each, then exactly one signature is recorded and the other invocation returns "already signed."

**FEAT-26.SPEC-002-AC-05:** Given Owen submits his signature on a proposal that Nadia voids by editing and re-sending before this automation's write-time re-check runs, when the re-check finds `status: Voided`, then no signature is recorded and FEAT-26.SPEC-001 redirects Owen to the current version.

**FEAT-26.SPEC-002-AC-06:** Given Owen submits his signature, when the electronic-signature attestation capability rejects the submission, then no signature record is written and FEAT-26.SPEC-001 shows a retry-capable error with the entered full legal name preserved.

**FEAT-26.SPEC-002-AC-07:** Given the attestation capability confirms Owen's submission but connectivity drops before the atomic write completes, when the failure occurs, then no partial signature record is left and FEAT-26.SPEC-001 shows a retry option.

**FEAT-26.SPEC-002-AC-08:** Given Owen retries signing after a failed write, when this automation re-runs, then it re-checks eligibility from the current Proposal state rather than assuming the prior attempt's context still holds.

**FEAT-26.SPEC-002-AC-09:** Given Nadia adjusts the Payment Schedule at the exact moment Owen's signature is being written, when this automation reads the schedule in step 8, then it uses the schedule as it stood at the moment of signing, and Nadia's adjustment applies only to later triggers.

**FEAT-26.SPEC-002-AC-10:** Given the signature write succeeds but the deposit-invoice trigger to FEAT-09 fails to fire, when this automation completes, then the signature and acceptance record remain valid and unaffected, and the invoice trigger is retried by FEAT-09's own handling.

**FEAT-26.SPEC-002-AC-11:** Given the audit-trail trigger to FEAT-13 fails to fire after the signature write succeeds, when this automation completes, then the signature record is unaffected and the trail write is retried by FEAT-13's own handling.

**FEAT-26.SPEC-002-AC-12:** Given the signature is recorded, when the analytics signal is emitted, then a proposal_signed event carries the proposal reference, whether a deposit invoice was triggered, and the time elapsed since the proposal was sent.

**FEAT-26.SPEC-002-AC-13:** Given Owen's role has been changed to Reviewer after the signing step loaded, when he taps Sign and this automation's write-time check runs, then it returns "denied -- not authorized", no attestation submission and no signature or acceptance write occurs, and FEAT-26.SPEC-001 shows "You no longer have permission to sign this proposal. If you think this is a mistake, ask the person who sent it."

**FEAT-26.SPEC-002-AC-14:** Given Owen's status has been set to Removed after the signing step loaded, when he taps Sign, then this automation returns "denied -- not authorized" before evaluating signed or voided state, the Proposal's `status`, signature record, `accepted_at`, and `accepted_by` are unchanged, and no invoice trigger, audit-trail entry for a signature, or confirmation notification fires.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 7 (no deposit, deposit triggered, denied-not-authorized, already signed, voided-redirect, attestation failed, write failure) | 7 |
| Business Rules | 8 | 8 |
| Edge Cases | 8 (concurrent tap, run-in-flight, unauthorized-actor, void-race, attestation-confirmed-then-write-fails, schedule-race, invoice-trigger-failure, audit-trigger-failure) | 8 |



# Logic/Rule Spec: E-Signature Opt-In & Signing Access Rules

## Overview

**Name:** E-Signature Opt-In & Signing Access Rules
**ID:** FEAT-26.SPEC-003
**Type:** Logic/Rule
**Purpose:** Governs the per-proposal e-signature opt-in scope, who may sign, the routing decision between FEAT-03's plain Accept and this feature's signing step, the full-legal-name field rule, failed-submission retry, and immutability once signed.
**Parent Feature:** FEAT-26 -- Legally Binding E-Signature for Proposals
**Governed Entity:** Proposal (signing-enabled flag and signature-record fields); the routing decision between FEAT-03.SPEC-001 and FEAT-26.SPEC-001 is governed here as this spec's second concern, since the Feature Breakdown Brief groups it with the same Validation & Limits field.

## Scope and Non-Goals

**In Scope:**
- The per-proposal signing-enabled flag: who may set it and its scope (one proposal, not account-wide)
- Eligibility rules for signing (signing-enabled, sign-once, voided-cannot-sign)
- The routing rule that decides, each time a Primary contact opens a proposal, whether FEAT-03.SPEC-001's plain Accept or FEAT-26.SPEC-001's signing step is shown
- The full-legal-name field's validation rule
- Authorization rules for view and sign actions on the signing step, per role
- Failed-submission retry behavior (preserving entered data)
- Concurrent-action resolution for the signing-enabled flag and the signature record (reject-with-refresh), as it applies to this feature's actions

**Non-Goals:**
- Proposal creation, editing, and voiding rules -- owned entirely by FEAT-02 (Proposal Creation & Sending); this spec only reads the Proposal's current `status` and signing-enabled flag to determine eligibility for this feature's actions.
- Rendering the opt-in toggle control itself -- owned by FEAT-02's send/edit screen; this spec owns only the eligibility flag and the rules that govern it, per the Feature Breakdown Brief's Capability Coverage Map.
- The underlying single-acceptance and voided-proposal enforcement FEAT-03.SPEC-005 already owns for a plain Accept -- this spec defers to FEAT-03.SPEC-005 for that base enforcement rather than re-deriving it, and adds only the signing-specific rules that extend it (XBR-34).
- Selecting or configuring the electronic-signature attestation vendor -- vendor selection is a Stage 4 decision (FEAT-26.SPEC-005); this spec's eligibility rules apply regardless of which capability performs the attestation.

## Governed Entity

**Entity:** Proposal
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|--------------|
| status | enum (Draft, Sent, Voided, Accepted) | The proposal's current lifecycle state; this spec reads it to gate signing eligibility, deferring to FEAT-03.SPEC-005 for the base enforcement of this field |
| sent_at | date/time | When the proposal was sent; no validation rule beyond data type in this spec |
| accepted_at | date/time | Written once, by FEAT-26.SPEC-002 for a signed acceptance or by FEAT-03.SPEC-003 for a plain one, never altered; this spec defines the additional condition (signing-enabled) under which FEAT-26.SPEC-002 may write it |
| accepted_by | reference (Client Contact) | The accepting/signing contact's identity; this spec defines who may cause this field to be set through the signing path specifically |
| payment_schedule_reference | reference (Payment Schedule) | Link to the project's Payment Schedule; no validation rule beyond data type in this spec |
| signing-enabled flag | boolean | Set by Nadia while drafting or editing the proposal on FEAT-02's screen (before sending); this spec is the single source of truth for who may set it and its scope |
| signature record (signer identity, signature data, timestamp) | composite | Written once by FEAT-26.SPEC-002 when signing succeeds; this spec defines the eligibility condition (signing-enabled, sign-once, not-voided) that gates the write, and its immutability once written |

**Secondary governed field (routing decision, not a Proposal field):**

| Field | Data Type | Description |
|-------|-----------|--------------|
| routing outcome | derived | Which screen (FEAT-03.SPEC-001 or FEAT-26.SPEC-001) a Primary contact sees when opening a proposal that has not yet been acted on; derived from the signing-enabled flag, governed here per the Feature Breakdown Brief's Side-Effect Inventory |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|----------------------|
| FEAT-02 | Proposal Creation & Sending | Screen-level opt-in toggle reads and writes the signing-enabled flag governed here (rendered on FEAT-02's own screen, per the Feature Breakdown Brief's Capability Coverage Map) |
| FEAT-03.SPEC-001 | Proposal Review & Accept | Applies the routing rule on screen entry -- shows the plain Accept flow unchanged when the proposal is not signing-enabled, or hands off to FEAT-26.SPEC-001 when it is |
| FEAT-26.SPEC-001 | Signature Signing Step | Access and Visibility on screen entry; full-legal-name field validation on Sign tap and on change; failed-submission retry behavior (screen-level, non-authoritative) |
| FEAT-26.SPEC-002 | Signature Recording | Authoritative write-time re-check of signing eligibility (signing-enabled, sign-once, voided-cannot-sign) and of the actor's authorization (role still Primary, status still Active; returns the "denied -- not authorized" outcome) before any attestation submission or signature write |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| status | No validation beyond data type -- this spec reads it, FEAT-02 owns writing it (except the signed transition, governed below) | Always | -- | -- | -- |
| sent_at | No validation beyond data type | Always | -- | -- | -- |
| accepted_at | Must be set exactly once, only by FEAT-26.SPEC-002's write when signing-enabled, or FEAT-03.SPEC-003's write when not; never altered afterward | Proposal transitions Sent -> Accepted (via either path) | On write (FEAT-26.SPEC-002 or FEAT-03.SPEC-003) | N/A -- system-set field, no user-facing error; a second attempted write is intercepted by the sign-once rule below, not a field-level message | Yes |
| accepted_by | Must reference a Client Contact with role Primary and status Active at the client owning the proposal | Proposal transitions Sent -> Accepted (via either path) | On write (FEAT-26.SPEC-002 or FEAT-03.SPEC-003) | N/A -- system-set field; the gating condition surfaces as the authorization denial below, not a field-level message | Yes |
| payment_schedule_reference | No validation beyond data type | Always | -- | -- | -- |
| signing-enabled flag | May be set only while the proposal is in Draft (before sending); no validation beyond data type once set | Always | On write (FEAT-02) | N/A -- system-set field; FEAT-02 owns its own screen-level rules for when the toggle is shown | -- |
| signature record (signer identity, signature data, timestamp) | Must be set exactly once, only by FEAT-26.SPEC-002's write, and only when the signing-enabled flag is set; never altered afterward | Proposal transitions Sent -> Accepted via the signing path | On write (FEAT-26.SPEC-002) | N/A -- system-set field; the gating condition surfaces as the eligibility and authorization denials below | Yes |
| full legal name (entered on FEAT-26.SPEC-001, not a stored Proposal field) | Required, non-empty | Always, when signing | On blur (screen) and on write (FEAT-26.SPEC-002, authoritative) | "Enter your full legal name to sign." | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Sign-once | signature record, status | Signing may only proceed when no signature record yet exists and `status` is `Sent`; once a signature record exists, it is fixed and no further sign attempt may change it | "This proposal has already been signed." |
| Voided-cannot-sign | status | Signing may only proceed when `status` is `Sent`, never when `status` is `Voided` | N/A -- Owen is redirected to the current proposal version rather than shown an inline error (FEAT-26.SPEC-001) |
| Signing-enabled gates the signing path | signing-enabled flag, signature record | A signature record may be written only when the signing-enabled flag is set on that proposal version; when it is not set, the plain Accept path (FEAT-03.SPEC-003) is the only route to `accepted_at`/`accepted_by` | N/A -- the routing rule below determines which screen Owen reaches; there is no direct error path for this condition |
| Signing-enabled is per proposal, not account-wide | signing-enabled flag | The flag applies only to the specific proposal version it was set on; enabling it for one proposal never changes any other proposal's flag, current or future | N/A -- no error condition; this is a scoping rule, not a validation failure |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Set the signing-enabled flag for a proposal | Nadia (Freelancer) | Own-only, and only while the proposal is in Draft (before sending), on FEAT-02's screen | -- |
| Set the signing-enabled flag for a proposal | Owen (Client Primary Contact) | Never | The toggle is not rendered on any screen Owen can reach; he never sets his own signing requirement |
| Set the signing-enabled flag for a proposal | Priya (Client Reviewer Contact) | Never | The toggle is not rendered on any screen Priya can reach |
| Set the signing-enabled flag for a proposal | Dana (Support Operator) | Never | The toggle is not rendered inside a support session (FEAT-31); it is read-only there |
| View the signing step (scope, price, signature capture) | Owen (Client Primary Contact) | Own-only -- only his own company's signing-enabled proposal, and only when `status` is `Sent` or `Accepted` (XBR-08, XBR-09) | -- |
| View the signing step | Priya (Client Reviewer Contact) | Never | Full screen is not shown; portal home shows only the project's stage label, per the Access Matrix (Proposals & Acceptance: None for Reviewers) |
| View the signing step | Nadia (Freelancer) | Never through this feature's screens -- she sees the resulting "Signed on {date}" status on the project view (FEAT-01), not FEAT-26.SPEC-001 | Attempting to open FEAT-26.SPEC-001's link is treated as an out-of-scope link (XBR-09): plain explanation and a fresh-link option |
| View the signing step | Dana (Support Operator) | Always, read-only, inside a logged support session (FEAT-31) | -- |
| Sign the proposal | Owen (Client Primary Contact) | Own-only, and only when `status` is `Sent`, the signing-enabled flag is set, and no signature record yet exists (sign-once and voided-cannot-sign, above) | If a signature record already exists: "This proposal has already been signed." If `status` is `Voided`: redirected to the current version, no error text shown |
| Sign the proposal | Owen (Client Primary Contact) whose role has since changed to Reviewer or whose status has since become Removed | Never -- the Primary and Active condition is re-checked at the moment of the Sign action by FEAT-26.SPEC-002 (authoritative), not only at screen load | FEAT-26.SPEC-001 shows the Not authorized state: "You no longer have permission to sign this proposal. If you think this is a mistake, ask the person who sent it." with a "Go to portal home" link; no signature is recorded and no proposal content is shown |
| Sign the proposal | Priya (Client Reviewer Contact) | Never | The signature capture step is not shown -- she never reaches signing content |
| Sign the proposal | Nadia (Freelancer) | Never | No signature capture control exists on any screen Nadia can reach; signing is exclusively a client-side action, same as plain acceptance |
| Sign the proposal | Dana (Support Operator) | Never | The signature capture step is not rendered inside a support session (FEAT-31) |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|---------------|----------------------|
| signing-enabled flag | Defaults to unset (not signing-enabled) for every new proposal | On proposal creation (FEAT-02) | Yes -- Nadia may set it while the proposal is in Draft, before sending (see Authorization Rules) |
| routing outcome | Derived as: show FEAT-26.SPEC-001 when the current proposal version's signing-enabled flag is set and the proposal is `Sent` or `Accepted` via the signing path; otherwise show FEAT-03.SPEC-001's plain Accept flow | Every time a Primary contact opens a proposal that has not yet been acted on, and on the redirect after a void-and-resend | No -- the routing outcome is fully derived from the current version's signing-enabled flag; neither Owen nor Nadia can override it once the version is sent |
| accepted_at (via the signing path) | Derived as the current timestamp at the moment FEAT-26.SPEC-002's write succeeds | On the Sent -> Accepted transition via signing, once | No |
| accepted_by (via the signing path) | Derived as the Client Contact reference of the eligible Primary contact whose sign attempt wins the exactly-once write (FEAT-26.SPEC-002) | On the Sent -> Accepted transition via signing, once | No |

## Business Rules

- XBR-34: When enabled for a proposal, a legally binding e-signature replaces the plain Accept click with the same Primary-only access and the same immutability; otherwise the timestamped Accept is the default. This spec is the single source of truth for the routing decision and eligibility that carries out that replacement.
- XBR-06: A voided proposal cannot be signed; a project has at most one active proposal at a time, and a voided proposal directs the client to the current one.
- XBR-08: Only Primary contacts sign proposals; Reviewer contacts never see proposal or signature content.
- XBR-09: Client isolation -- a contact reaches only their own company's proposal; an out-of-scope or expired link shows a plain explanation and a fresh-link option, never another company's data.
- Signing is available per proposal, not forced account-wide: a freelancer who never opts in never sees any change to FEAT-03's standard Accept flow, per the Feature Breakdown Brief's Validation & Limits field.
- Once signed, the signature record is immutable like any other acceptance (Feature Breakdown Brief, Validation & Limits): FEAT-26.SPEC-002 writes it exactly once, and no later process may alter it (XBR-04).
- If a signature submission fails mid-flight (attestation-capability error, validation failure, or connectivity drop), the entered full legal name is preserved on screen and Owen may retry without losing his intent, per the Feature Breakdown Brief's Side-Effect Inventory -- enforced within FEAT-26.SPEC-002's atomic write, per this spec's sign-once rule.
- The Proposal Contention resolution for a race between Nadia's edit-and-void (FEAT-02) and Owen's sign attempt (this feature) is reject-with-refresh: a sign attempt against a version voided in the meantime is refused and the client is shown the current version.
- Dana's support session sees this feature's screens read-only, per XBR-29: no signature capture control is ever rendered for her, regardless of the proposal's state.

## Edge Cases

- **Owen signs a proposal at the exact instant its `status` field is Sent, the signing-enabled flag is set, and no signature record yet exists** -- Standard eligible-sign path; no boundary issue since this is the normal signing-enabled Sent state.
- **Owen enters exactly one character as his full legal name** -- Passes validation; the rule requires only non-empty, no minimum length beyond that.
- **Owen leaves the full legal name field with only whitespace** -- Treated as empty for this rule's purpose and fails validation with "Enter your full legal name to sign.", mirroring the whitespace-only treatment FEAT-03.SPEC-005 applies to the change-request note.
- **Nadia enables signing on a proposal, then edits it again before sending, disabling and re-enabling the flag** -- No error condition; the flag simply reflects its state at the moment the proposal is finally sent, since it may only be set while the proposal is in Draft.
- **Two attempts to sign the same proposal arrive within the same instant** -- The sign-once rule is enforced atomically by FEAT-26.SPEC-002's write, not by this spec directly; this spec defines that exactly one may succeed and the other must see "This proposal has already been signed."
- **A Primary contact's role is changed to Reviewer, or their status is set to Removed, while they are mid-session on FEAT-26.SPEC-001** -- The authorization condition (Own-only, Primary, Active) is re-checked at the moment of the Sign action, not only at screen load; if the role or status has changed, FEAT-26.SPEC-002's write-time check returns "denied -- not authorized" and FEAT-26.SPEC-001 shows: "You no longer have permission to sign this proposal. If you think this is a mistake, ask the person who sent it." with a "Go to portal home" link. No attestation submission or signature write occurs, no proposal content remains on screen, and the contact's next access follows their current role (Reviewer: stage-only view; Removed: expired-link explanation with fresh-link option), consistent with XBR-27.
- **A signing-enabled proposal is voided and re-sent as a new version with the signing-enabled flag left unset on the new version** -- The routing rule re-evaluates against the current version each time: the redirect after the void lands Owen on FEAT-03.SPEC-001's plain Accept flow, not FEAT-26.SPEC-001, because the current version's own flag now governs the decision.
- **The proposal transitions from Sent to Voided between FEAT-26.SPEC-001's screen-level eligibility check and Owen's Sign tap reaching FEAT-26.SPEC-002** -- The screen-level check is non-authoritative; FEAT-26.SPEC-002's write-time re-check is what actually catches this and returns the voided-redirect outcome.
- **Owen attempts to sign a proposal for a project belonging to a different client than the one his contact record is scoped to** -- Denied per XBR-09: this is treated as an out-of-scope link, showing a plain explanation and a fresh-link option, never another company's proposal.

## Acceptance Criteria

**FEAT-26.SPEC-003-AC-01:** Given Nadia is drafting a proposal that has not yet been sent, when she enables the e-signature toggle on FEAT-02's screen, then the signing-enabled flag is set on that specific proposal only.

**FEAT-26.SPEC-003-AC-02:** Given a proposal has its signing-enabled flag set and status Sent, when Owen (a Primary contact) opens it, then the routing rule sends him to FEAT-26.SPEC-001 (Signature Signing Step) rather than FEAT-03.SPEC-001's plain Accept flow.

**FEAT-26.SPEC-003-AC-03:** Given a proposal does not have its signing-enabled flag set, when Owen opens it, then the routing rule shows FEAT-03.SPEC-001's plain Accept flow unchanged.

**FEAT-26.SPEC-003-AC-04:** Given Owen is on FEAT-26.SPEC-001 with the proposal signing-enabled and Sent, when he leaves the full legal name field empty and moves focus away, then the field shows "Enter your full legal name to sign."

**FEAT-26.SPEC-003-AC-05:** Given Owen enters a single-character full legal name, when the field rule is checked, then the name passes validation.

**FEAT-26.SPEC-003-AC-06:** Given Owen enters only whitespace as his full legal name, when the field rule is checked, then it is denied with "Enter your full legal name to sign."

**FEAT-26.SPEC-003-AC-07:** Given a proposal already has a signature record, when Owen (or any Primary contact) attempts to sign it again, then the attempt is denied with "This proposal has already been signed."

**FEAT-26.SPEC-003-AC-08:** Given a proposal has `status: Voided`, when Owen attempts to sign it, then the attempt is denied and he is redirected to the current proposal version, with no error text shown.

**FEAT-26.SPEC-003-AC-09:** Given Owen's signature submission fails mid-flight, when the failure occurs, then the entered full legal name is preserved on FEAT-26.SPEC-001 and he may retry without re-entering it.

**FEAT-26.SPEC-003-AC-10:** Given Owen (Client Primary Contact) at the owning client, when he attempts to view a signing-enabled proposal, then he can view the full signing step.

**FEAT-26.SPEC-003-AC-11:** Given Priya (Client Reviewer Contact), when she attempts to view a signing-enabled proposal, then no signing content is shown and her portal home shows only the project's stage label.

**FEAT-26.SPEC-003-AC-12:** Given Nadia (Freelancer), when she attempts to open FEAT-26.SPEC-001's link directly, then the link is treated as out-of-scope and she sees a plain explanation with a fresh-link option, never signing content through this feature's own screen.

**FEAT-26.SPEC-003-AC-13:** Given Dana (Support Operator) inside a logged support session, when she views a signing-enabled proposal, then she sees the full content read-only, with no signature capture control rendered.

**FEAT-26.SPEC-003-AC-14:** Given Nadia (Freelancer), when she attempts to set the signing-enabled flag herself outside FEAT-02's own screen, then no such control exists for her to attempt this with -- the flag is set only through FEAT-02's opt-in toggle.

**FEAT-26.SPEC-003-AC-15:** Given two Primary contacts at the same client both attempt to sign at effectively the same moment, when the write-time eligibility check runs for each, then exactly one succeeds and the other is denied with "This proposal has already been signed."

**FEAT-26.SPEC-003-AC-16:** Given a Primary contact's status is changed to Removed while they are mid-session on the signing step, when they attempt to sign, then FEAT-26.SPEC-002 denies the action and FEAT-26.SPEC-001 shows "You no longer have permission to sign this proposal. If you think this is a mistake, ask the person who sent it." with a "Go to portal home" link, and no signature is recorded.

**FEAT-26.SPEC-003-AC-21:** Given a Primary contact's role is changed to Reviewer while they are mid-session on the signing step, when they attempt to sign, then FEAT-26.SPEC-002's write-time check denies the action before any attestation submission, FEAT-26.SPEC-001 shows the same "You no longer have permission to sign this proposal." message, and the Proposal's signature record and `status` are unchanged.

**FEAT-26.SPEC-003-AC-17:** Given a proposal transitions from Sent to Voided between the screen's own eligibility check and the write-time re-check, when FEAT-26.SPEC-002 re-checks eligibility, then the voided-cannot-sign rule denies the write and the client is redirected to the current version.

**FEAT-26.SPEC-003-AC-18:** Given a contact attempts to reach a signing-enabled proposal belonging to a different client than their own, when the access check runs, then the attempt is denied as out-of-scope with a plain explanation and a fresh-link option, never another company's data.

**FEAT-26.SPEC-003-AC-19:** Given the signature write succeeds for one Primary contact's attempt, when `accepted_at` and `accepted_by` are derived, then they are set exactly once from that attempt's timestamp and contact reference, with no user override available.

**FEAT-26.SPEC-003-AC-20:** Given a signing-enabled proposal is voided and re-sent as a new version with the flag left unset, when Owen opens the current version, then the routing rule shows FEAT-03.SPEC-001's plain Accept flow, based on the current version's own signing-enabled flag.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 8 | 8 |
| Cross-Field Rules | 4 | 4 |
| Authorization Rules | 13 | 13 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 9 | 9 |
| Edge Cases | 9 | 9 |



# Notification Spec: Signed-Copy Confirmation Notification

## Overview

**Name:** Signed-Copy Confirmation Notification
**ID:** FEAT-26.SPEC-004
**Type:** Notification
**Purpose:** Emails Owen and Nadia a signed-copy confirmation the instant a proposal's signature is recorded, in addition to FEAT-03's standard acceptance confirmation, so both parties have a distinct, durable record that this acceptance carries the stronger evidentiary weight of a signature.
**Parent Feature:** FEAT-26 -- Legally Binding E-Signature for Proposals

## Scope and Non-Goals

**In Scope:**
- The signed-copy confirmation email sent to Owen (the signing contact) and to Nadia when a signature is recorded
- Content, delivery rules, and edge cases for this single notification, distinct from FEAT-03.SPEC-006's standard acceptance confirmation

**Non-Goals:**
- The standard acceptance confirmation itself -- owned entirely by FEAT-03.SPEC-006 (Acceptance Confirmation Notification), which FEAT-26.SPEC-002 also fires for a signed acceptance per the Feature Breakdown Brief's Communications field ("in addition to the standard acceptance confirmation"); this spec covers only the signed-copy confirmation, never the standard one's content.
- The deposit invoice's own notification -- owned by FEAT-09 (Invoice Generation & Sending); this spec's content states only whether a deposit invoice was triggered, exactly as FEAT-03.SPEC-006 does.
- An in-app notification channel -- product-features.md phases the In-App Notification Center (FEAT-29) as Later, outside MVP and outside this feature's v1 phase; at this feature's priority tier the only notification channel the product defines is transactional email (ASMP-29), so this spec uses no other channel.
- A preference to turn this notification off -- excluded per XBR-30: transactional emails core to the record (this is the evidentiary confirmation of a signed acceptance) always send and cannot be disabled; only optional notifications carry an on/off preference.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, to both Owen and Nadia, the instant the signature is recorded | Owen's sessions are short and triggered by a specific email, not habitual browsing (user-persona.md, Client Primary Contact, Behavioral Context); Nadia is not necessarily inside the product at the moment a client signs, and needs a durable record that the stronger, signed form of acceptance now exists. Email is the product's sole notification channel at this feature's phase (ASMP-29); no in-app channel exists yet (FEAT-29 is Later). |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|-------------------|
| Signature recorded | FEAT-26.SPEC-002 (Signature Recording) | Fires immediately once the signature write succeeds, alongside (and in addition to) FEAT-03.SPEC-006's own trigger from the same write | Proposal reference, `accepted_at`, `accepted_by` (Owen's Client Contact reference), the signer's full legal name, project name, client company name, whether a deposit invoice was triggered |

## Audience and Preferences

**Recipients:** Owen (Client Primary Contact, the signing contact) and Nadia (Freelancer, the project owner) -- both are entitled to the signed-acceptance event per the Access Matrix (Proposals & Acceptance: Full for Nadia, Own-only sign/view for Owen) and per the Feature Breakdown Brief's Communications field ("A signed-copy confirmation email to both parties, in addition to the standard acceptance confirmation").

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|----------------------|
| N/A | N/A | Always on -- transactional | N/A -- per XBR-30, this is a transactional email core to the evidentiary record and cannot be disabled by either recipient |

**Quiet Hours:** N/A -- the product defines no quiet-hours window for transactional confirmation emails; this notification is the timestamped record of a moment that just happened for both parties, and holding it would misrepresent when the signature was actually confirmed.

## Content Definition

**Email (to Owen):**
- **Subject:** Your signed copy of the proposal for {project_name}
- **Body:**
  Hi {owen_first_name},

  You signed the proposal for {project_name} on {accepted_at_date} as {signer_full_legal_name}. This is your signed copy, carrying stronger evidentiary weight than a recorded acceptance alone.

  {deposit_invoice_line}
- **CTA (button):** View signed proposal -- deep-links to FEAT-26.SPEC-001 (Signature Signing Step) for this proposal, now showing "Signed on {accepted_at_date}"

**Email (to Nadia):**
- **Subject:** {client_company_name} signed the proposal for {project_name}
- **Body:**
  Hi {nadia_first_name},

  {owen_full_name} at {client_company_name} signed the proposal for {project_name} on {accepted_at_date} as {signer_full_legal_name}. This signed record carries stronger evidentiary weight than a plain timestamped acceptance.

  {deposit_invoice_line_nadia}
- **CTA (button):** View project -- deep-links to FEAT-01 (Client & Project Management), project view, for this project

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|--------------------------|-----------------|--------------------------|
| {project_name} | Project -- project_name | Brand Refresh Q1 | Never empty -- required at project creation (FEAT-01) |
| {owen_first_name} | Client Contact -- name (first token) | Owen | Greeting renders as "Hi," |
| {owen_full_name} | Client Contact -- name | Owen Carter | "the client's Primary contact" |
| {client_company_name} | Client -- client_name | Carter & Co | Never empty -- required at client creation (FEAT-01) |
| {accepted_at_date} | Proposal -- accepted_at (formatted in the recipient's own time zone, FEAT-15) | March 4, 2026 | Never empty -- `accepted_at` is written by FEAT-26.SPEC-002 before this notification fires |
| {signer_full_legal_name} | Proposal -- signature record, signature data (the full legal name entered on FEAT-26.SPEC-001) | Owen Carter | Never empty -- FEAT-26.SPEC-003's field validation rule requires a non-empty full legal name before FEAT-26.SPEC-002 can write the signature record |
| {nadia_first_name} | Freelancer Account -- name (first token) | Nadia | Greeting renders as "Hi," |
| {deposit_invoice_line} | Derived -- whether FEAT-26.SPEC-002 triggered a deposit invoice | "A deposit invoice has been sent to you separately." | "No deposit is due under this project's payment schedule." |
| {deposit_invoice_line_nadia} | Derived -- whether FEAT-26.SPEC-002 triggered a deposit invoice | "A deposit invoice has been generated and sent automatically." | "This project's payment schedule has no deposit, so no invoice was generated yet." |

## Delivery Rules

**Batching:** None -- each signature is a single, discrete evidentiary event; it is never combined with any other notification, including FEAT-03.SPEC-006's standard confirmation, which is delivered as its own separate email even though both fire from the same write.
**Deduplication:** At most one signed-copy confirmation email per recipient per signature. Because FEAT-26.SPEC-002 writes the signature exactly once (sign-once, enforced atomically), this notification's trigger fires exactly once per Proposal; a retried Sign attempt that returns "already signed" never re-fires this notification.
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window` per FEAT-14's transactional email delivery capability (ASMP-29). After the final failure, the failure is surfaced to Nadia as a delivery warning on the affected project (XBR-30); the "Signed on {date}" marker on FEAT-26.SPEC-001 stands as the enduring in-product record regardless of email delivery outcome.
**Expiry:** This notification never expires undelivered in the sense of being withdrawn -- it is retried per the rule above, and if all retries fail, the delivery warning (XBR-30) is the surviving signal rather than a silently dropped message, since the underlying signature record itself never disappears.

## Edge Cases

- **Owen's email address bounces** -- The failure is retried per the Retry on failure rule; after the final failure, Nadia sees a delivery warning on the project (XBR-30) and is advised to correct Owen's contact email (FEAT-18). The signature record itself is unaffected -- it does not depend on this email's delivery.
- **FEAT-03.SPEC-006's standard confirmation delivers but this signed-copy confirmation fails** -- The two notifications are independent deliveries from the same trigger; a failure of one does not affect the other's delivery or retry schedule. Owen and Nadia may see one email before the other, but both are eventually delivered or, on final failure, surfaced as a delivery warning.
- **The deposit invoice trigger fails after the signature write succeeds (FEAT-26.SPEC-002's own failure path)** -- This notification still fires with the "no deposit due" fallback line only if the Payment Schedule genuinely has no deposit; if a deposit was due but the invoice trigger failed, the {deposit_invoice_line} placeholder still reflects that a deposit invoice was triggered -- FEAT-09 owns communicating its own delivery outcome.
- **Owen's Client Contact record is removed (access revoked) between signing and this notification's delivery** -- The notification still delivers to the email address captured in `accepted_by` at the moment of signing, since that identity is preserved as evidence even after removal (XBR-27); the notification's content is a record of what happened, not a live view of current access.
- **Nadia's account has no `business_name` set yet** -- Not applicable to this notification's content, since neither email template references the freelancer's business name; both templates identify the freelancer only by first name in the greeting.
- **Two signature confirmation attempts race due to a transient duplicate trigger fire** -- Deduplication holds: since FEAT-26.SPEC-002 only ever writes the signature once and only fires this notification's trigger once per successful write, no second instance of this notification is ever queued for the same signature.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|-------------------|----------------|
| FEAT-26.SPEC-002 (Signature Recording) | Triggered by (inbound) | Fires this notification immediately once the signature write succeeds, alongside FEAT-03.SPEC-006 |
| FEAT-03.SPEC-006 (Acceptance Confirmation Notification) | References (inbound) | Both notifications fire from the same write; this spec covers only the additional signed-copy content |
| FEAT-26.SPEC-001 (Signature Signing Step) | Navigation (outbound) | Owen's CTA deep-links back to the now-Signed proposal screen |
| FEAT-01 (Client & Project Management) | Navigation (outbound) | Nadia's CTA deep-links to the project view |
| FEAT-14 (Notifications (Email)) | References (outbound) | Owns the transactional email delivery capability and delivery/bounce status reporting this notification relies on |
| FEAT-09 (Invoice Generation & Sending) | References (outbound) | The {deposit_invoice_line} placeholder reflects whether FEAT-26.SPEC-002 triggered FEAT-09's deposit invoice; FEAT-09 sends its own separate notification |

## Analytics and Success Signals

- **signed_copy_confirmation_sent** (recipient: owen / nadia, deposit invoice triggered: yes / no) -- N/A -- no Stage 2 metric in success-metrics.md is connected to FEAT-26 (its Connected Feature slice is empty); retained as the notification-side delivery signal for the product-defined proposal_signed behavior
- **signed_copy_confirmation_delivery_failed** (recipient: owen / nadia, retries exhausted: yes / no) -- N/A -- no Stage 2 metric measures delivery failures for this feature; retained so a silently undelivered evidentiary confirmation is observable via the project's delivery warning (XBR-30) rather than invisible

## Acceptance Criteria

**FEAT-26.SPEC-004-AC-01:** Given Owen signs a proposal with no deposit due, when FEAT-26.SPEC-002 records the signature, then Owen and Nadia each receive a signed-copy confirmation email, and both bodies include "No deposit is due under this project's payment schedule" (Owen's variant) / "no invoice was generated yet" (Nadia's variant).

**FEAT-26.SPEC-004-AC-02:** Given Owen signs a proposal whose Payment Schedule includes a deposit, when FEAT-26.SPEC-002 records the signature and triggers the deposit invoice, then Owen's confirmation email states "A deposit invoice has been sent to you separately" and Nadia's states "A deposit invoice has been generated and sent automatically."

**FEAT-26.SPEC-004-AC-03:** Given Owen signs the proposal, when this notification and FEAT-03.SPEC-006's standard confirmation both fire from the same write, then Owen and Nadia each receive two separate emails -- the standard acceptance confirmation and this signed-copy confirmation -- never combined into one.

**FEAT-26.SPEC-004-AC-04:** Given Owen receives the signed-copy confirmation email, when he taps "View signed proposal", then he lands on FEAT-26.SPEC-001 showing "Signed on {accepted_at_date}".

**FEAT-26.SPEC-004-AC-05:** Given Nadia receives the signed-copy confirmation email, when she taps "View project", then she lands on the project view in FEAT-01.

**FEAT-26.SPEC-004-AC-06:** Given the signature is recorded, when this notification's trigger fires, then no on/off preference is available to either recipient to suppress it -- it always sends.

**FEAT-26.SPEC-004-AC-07:** Given Owen's email address bounces on the first delivery attempt, when the delivery capability retries, then up to platform parameter: `transactional-email-retry-count` retries occur over platform parameter: `transactional-email-retry-window` before Nadia sees a delivery warning on the project.

**FEAT-26.SPEC-004-AC-08:** Given all retries for Owen's email are exhausted, when the final failure occurs, then Nadia sees a delivery warning on the affected project and the "Signed on {date}" marker remains the enduring in-product record regardless.

**FEAT-26.SPEC-004-AC-09:** Given the signature is recorded exactly once (per FEAT-26.SPEC-002's sign-once guarantee), when a second, redundant Sign attempt returns "already signed", then this notification's trigger does not fire a second time.

**FEAT-26.SPEC-004-AC-10:** Given Owen's Client Contact record is later removed, when this notification is still pending delivery, then it still delivers to the email address captured in `accepted_by` at the moment of signing.

**FEAT-26.SPEC-004-AC-11:** Given the signed-copy confirmation is successfully delivered to both recipients, when the analytics signal is emitted, then a signed_copy_confirmation_sent event is recorded once per recipient with the deposit-invoice-triggered flag.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on -- transactional) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 6 | 6 |



# Integration Spec: Electronic-Signature Attestation Capability

## Overview

**Name:** Electronic-Signature Attestation Capability
**ID:** FEAT-26.SPEC-005
**Type:** Integration
**Purpose:** The product submits the signature data captured on the signing step to an external electronic-signature attestation capability and uses its confirmation to give the signed acceptance legal weight beyond a self-recorded timestamp.
**Parent Feature:** FEAT-26 -- Legally Binding E-Signature for Proposals

## Scope and Non-Goals

**In Scope:**
- Submitting the signature data FEAT-26.SPEC-002 captures for attestation, and receiving the attestation capability's confirmation
- Receiving and reacting to a rejection or failure of that submission
- User-facing behavior when the capability is slow, unavailable, or rejects a submission, for the one screen this affects (FEAT-26.SPEC-001)
- Disclosure to Owen about what signature data is shared with the attestation capability

**Non-Goals:**
- Choosing the electronic-signature attestation vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate, so this spec stays vendor-neutral throughout.
- Determining which jurisdictions' electronic-signature law the attestation must satisfy -- excluded per the Feature Breakdown Brief's Compliance flags: the specific regulatory standard is a product decision for Stage 4's selection of the capability, not a determination this spec makes.
- Writing the signature record itself or firing the deposit-invoice and audit-trail effects -- owned entirely by FEAT-26.SPEC-002 (Signature Recording); this spec defines only the exchange with the external capability that FEAT-26.SPEC-002 consumes before it writes.
- The signing step's own screen mechanics (the full legal name field, the Sign control, the layout) -- owned by FEAT-26.SPEC-001 (Signature Signing Step); this spec defines only the attestation-capability behavior that screen surfaces.

## Capability Category

**Category:** Electronic-signature attestation
**Dependency Source:** Not present in assumptions-constraints.md's Dependencies section (ASMP-28 to ASMP-32); added from the validated FEAT-26 Feature Breakdown Brief per Step 7.5 rule 4, since the Brief's Analyst-Discovered Specs table identified this dependency ahead of the Requirements Architect's dependency-map pass.
**External Touchpoint:** "Electronic-signature attestation for legally binding proposal signing -- v1 phase" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-26)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Owen's signature carries legal weight beyond a self-recorded timestamp, because an external capability attests to it | Signing step -- client completes a signature step instead of a plain Accept click | FEAT-26.SPEC-001 (Signature Signing Step), FEAT-26.SPEC-002 (Signature Recording) |
| The signature record written on the Proposal reflects a confirmed attestation, not just an in-product write | Opt-in e-signature -- freelancer enables it for a specific proposal | FEAT-26.SPEC-002 (Signature Recording) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Signer's full legal name | Proposal -- signature record, signature data (the name Owen entered on FEAT-26.SPEC-001) | Owen taps Sign and FEAT-26.SPEC-002's eligibility re-check passes | The attestation capability must know whose signature it is attesting to |
| Signature timestamp | Proposal -- signature record, timestamp | Owen taps Sign | The attestation capability records the moment of signing as part of what it attests to |
| Proposal reference | Proposal -- an identifying reference (not the scope or price content) | Owen taps Sign | Ties the attestation outcome back to the right proposal's signature record |

Scope description, price, payment schedule terms, and every other Proposal field never leave the product through this capability -- the attestation capability attests to the signature event itself, not to the commercial content the signature applies to.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Attestation confirmation | The capability confirms the signature submission | Consumed directly by FEAT-26.SPEC-002's processing logic as the gate before it writes the Proposal's signature record; no separate field is stored beyond the signature record itself, since the confirmed write is the evidence of attestation |
| Rejection or failure reason (plain-language category) | The capability rejects the submission or the request itself fails | Consumed directly by FEAT-26.SPEC-002, which returns the "attestation failed" outcome to FEAT-26.SPEC-001; not persisted on the Proposal, since no signature record exists to attach it to |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Attestation confirmed | The capability confirms Owen's signature submission | None directly -- the confirmation is the gate FEAT-26.SPEC-002 checks in its own processing (step 6) before it writes the signature record | None directly on this event -- Owen sees the "Signed on {date}" marker once FEAT-26.SPEC-002's write completes, per that spec's own outcome | FEAT-26.SPEC-002 (Signature Recording) |
| Attestation rejected or request failed | The capability rejects the submission, or the request itself does not complete (timeout, connectivity) | None -- no partial signature record is ever written | FEAT-26.SPEC-001 shows "We couldn't record your signature. Check your connection and try again." with the entered full legal name preserved | FEAT-26.SPEC-001 (Signature Signing Step), FEAT-26.SPEC-002 (Signature Recording) |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-26.SPEC-001 (Signature Signing Step) | The Sign button shows a loading state during the check, attestation, and write; after 10 seconds a note appears beneath it: "Still working -- this is taking longer than usual." The rest of the screen (scope, price, full legal name field) remains fully visible; nothing else is blocked. | The Sign button returns the inline error "We couldn't record your signature. Check your connection and try again." with the entered full legal name preserved. Owen can still view the proposal's scope and price, and can still navigate to Request Changes. | The same inline error and retry option as when the capability is down: "We couldn't record your signature. Check your connection and try again." The proposal is unchanged -- no signature record is written, and the entered full legal name is preserved for the retry. |

## Consent and Disclosure

- **First signing disclosure** -- The first time Owen opens a signing-enabled proposal's signing step, a notice appears alongside the signature capture step: "Signing shares your full legal name and the time you sign with an external electronic-signature service, which attests to your signature." This is stated once per signing session (it does not require a separate "Continue"/"Cancel" gate, since the attestation submission is inherent to the Sign action Owen has already chosen to take by opening the signing step); an ongoing "How your signature is verified" link on FEAT-26.SPEC-001 reopens the same notice.
- **What is never shared** -- The proposal's scope description, price, payment schedule terms, and every field beyond the signer's full legal name, the signature timestamp, and a proposal reference stay inside the product. This boundary is stated in the disclosure notice.

## Edge Cases

- **The attestation capability confirms the submission, but the confirmation event arrives twice (a transient duplicate delivery)** -- The second delivery changes nothing: FEAT-26.SPEC-002 only proceeds to its write once per submission, and once the signature record exists, a repeated confirmation for the same submission has no further effect -- there is no signature record left to write a second time.
- **An attestation confirmation arrives for a proposal that was voided between the submission and the confirmation's arrival** -- FEAT-26.SPEC-002's write-time re-check (its own step 2) still runs before any write; it finds `status: Voided` and returns the "voided -- redirect" outcome, discarding the confirmation without writing a signature record.
- **A rejection and a confirmation both arrive for the same submission, out of order** -- Only the first outcome FEAT-26.SPEC-002 receives is acted on: if attestation was already confirmed and the write already succeeded, a later, stray rejection message for the same submission has no signature record left to affect, since XBR-04 makes the write immutable once it succeeds. If the rejection is processed first, no write occurs and a later stray confirmation would attempt to attest to a request FEAT-26.SPEC-002 has already abandoned; because FEAT-26.SPEC-002 only starts a new submission from a fresh Sign tap, no unsolicited late confirmation is ever consumed for a submission Owen has already retried or abandoned.
- **The capability goes down mid-submission, after the signature data has left the product but before any confirmation or rejection arrives** -- FEAT-26.SPEC-002 treats an unanswered request the same as a failure after its own timeout: no signature record is written, and FEAT-26.SPEC-001 shows the retry-capable error. No half-written signature state is ever left on the Proposal.
- **Owen retries signing after an attestation rejection, entering the same full legal name** -- Treated as a fresh submission; the prior rejection has no bearing on the retry's outcome, and a successful attestation on retry proceeds to the write exactly as a first-attempt success would.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-26.SPEC-002 (Signature Recording) | Triggered by (inbound) | FEAT-26.SPEC-002 submits the signature data to this capability as its own processing step, before writing the signature record |
| FEAT-26.SPEC-002 (Signature Recording) | Affects (outbound) | This capability's confirmation or rejection is the gate FEAT-26.SPEC-002's write depends on |
| FEAT-26.SPEC-001 (Signature Signing Step) | Affects (outbound) | Degradation states and the disclosure notice surface here |

## Analytics and Success Signals

- **esignature_attestation_confirmed** (proposal reference) -- N/A -- no Stage 2 metric in success-metrics.md is connected to FEAT-26; retained as the capability-side reliability signal for the electronic-signature attestation dependency
- **esignature_attestation_rejected** (proposal reference, rejection reason category) -- N/A -- no connected success metric measures attestation rejections; retained so the reliability of this external dependency is observable rather than invisible
- **esignature_degradation_shown** (condition: slow / down / rejected; screen: FEAT-26.SPEC-001) -- N/A -- no Stage 2 metric measures degradation frequency for this feature; retained so the product's tolerance for capability trouble is observable, mirroring the payment-collection Integration spec's convention

## Acceptance Criteria

**FEAT-26.SPEC-005-AC-01:** Given Owen is on FEAT-26.SPEC-001 with a valid full legal name entered, when he taps Sign and the attestation capability confirms the submission, then FEAT-26.SPEC-002 proceeds to write the signature record.

**FEAT-26.SPEC-005-AC-02:** Given Owen taps Sign, when the attestation capability rejects the submission, then no signature record is written and FEAT-26.SPEC-001 shows "We couldn't record your signature. Check your connection and try again." with the entered full legal name preserved.

**FEAT-26.SPEC-005-AC-03:** Given Owen taps Sign, when the request to the attestation capability times out without a response, then FEAT-26.SPEC-001 shows the same retry-capable error as a rejection, and no signature record is written.

**FEAT-26.SPEC-005-AC-04:** Given Owen taps Sign, when the attestation capability is slow to respond, then the Sign button shows a loading state, and after 10 seconds a note appears: "Still working -- this is taking longer than usual."

**FEAT-26.SPEC-005-AC-05:** Given Owen taps Sign while the attestation capability is unavailable, when the request cannot be completed, then FEAT-26.SPEC-001 shows the retry-capable error and Owen can still view the proposal's scope and price or navigate to Request Changes.

**FEAT-26.SPEC-005-AC-06:** Given Owen opens a signing-enabled proposal's signing step for the first time, when the screen loads, then the disclosure notice states that signing shares his full legal name and the signing timestamp with an external electronic-signature service.

**FEAT-26.SPEC-005-AC-07:** Given the disclosure notice has already been shown to Owen, when he looks for it again, then a "How your signature is verified" link on FEAT-26.SPEC-001 reopens the same notice.

**FEAT-26.SPEC-005-AC-08:** Given Owen signs a proposal, when the signature data is submitted for attestation, then the proposal's scope description, price, and payment schedule terms are not included in what is sent.

**FEAT-26.SPEC-005-AC-09:** Given an attestation confirmation for Owen's submission is delivered twice due to a transient duplicate delivery, when the second delivery arrives, then no second signature record is written and no duplicate effect occurs.

**FEAT-26.SPEC-005-AC-10:** Given Owen's proposal is voided by Nadia between his Sign tap and the attestation confirmation's arrival, when FEAT-26.SPEC-002 re-checks eligibility, then the confirmation is discarded without writing a signature record, and Owen is redirected to the current version.

**FEAT-26.SPEC-005-AC-11:** Given the attestation capability goes down after the signature data has left the product but before any response arrives, when FEAT-26.SPEC-002's own timeout elapses, then no signature record is written and FEAT-26.SPEC-001 shows the retry-capable error.

**FEAT-26.SPEC-005-AC-12:** Given Owen retries signing after an attestation rejection, when he re-submits with the same full legal name, then the retry is treated as a fresh submission independent of the prior rejection's outcome.

**FEAT-26.SPEC-005-AC-13:** Given the attestation capability confirms a submission, when the analytics signal is emitted, then an esignature_attestation_confirmed event is recorded with the proposal reference.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 2 | 2 |
| Inbound Events | 2 | 2 |
| Degradation Paths | 3 (1 screen x slow / down / rejects) | 3 |
| Consent and Disclosure | 2 | 2 |
| Edge Cases | 5 | 5 |

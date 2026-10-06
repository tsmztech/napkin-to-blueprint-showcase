---
document_type: feature-overview
feature_number: FEAT-26
feature_name: Legally Binding E-Signature for Proposals
feature_slug: legally-binding-e-signature-for-proposals
priority_tier: Nice-to-Have
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-27
spec_count: 5
screen_count: 1
automation_count: 1
logic_rule_count: 1
integration_count: 1
notification_count: 1
---

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

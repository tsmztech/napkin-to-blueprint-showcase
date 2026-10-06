---
document_type: spec
spec_type: integration
spec_id: FEAT-26.SPEC-005
spec_name: Electronic-Signature Attestation Capability
spec_slug: electronic-signature-attestation-capability
parent_feature: FEAT-26
parent_feature_name: Legally Binding E-Signature for Proposals
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

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

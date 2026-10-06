---
document_type: spec
spec_type: integration
spec_id: FEAT-27.SPEC-002
spec_name: Domain Verification & Secure Serving
spec_slug: domain-verification-secure-serving
parent_feature: FEAT-27
parent_feature_name: Custom Domain per Freelancer
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 15
---

# Integration Spec: Domain Verification & Secure Serving

## Overview

**Name:** Domain Verification & Secure Serving
**ID:** FEAT-27.SPEC-002
**Type:** Integration
**Purpose:** Verifies that Nadia controls the domain she added and, once verified, serves her portal securely at it, reporting verification, failure, and re-check results back to the product.
**Parent Feature:** FEAT-27 -- Custom Domain per Freelancer

## Scope and Non-Goals

**In Scope:**
- Submitting a domain for control verification when Nadia adds or replaces one, or requests a re-check
- Serving the portal securely at a domain once it is verified
- Accepting a submission and issuing the verification instructions (the specific steps Nadia must complete), which sets the Verifying state
- Reporting verification succeeded, verification failed (with a specific reason), and re-check results back to the product
- User-facing behavior when the capability is slow, unavailable, or rejects a request
- Disclosure to Nadia about what is shared with the capability

**Non-Goals:**
- The Custom Domain Settings screen's own mechanics (input, buttons, confirmation dialogs) -- owned by FEAT-27.SPEC-001; this spec defines only the verification-and-serving behavior that screen surfaces
- The one-domain-per-account limit, domain-format validation, and the fallback-to-default guarantee -- owned by FEAT-27.SPEC-003; this spec relies on that rule to know there is at most one record to verify per freelancer and never re-derives the limit itself
- Ongoing monitoring or alerting for a domain that later breaks after going live -- excluded per product-features.md's States field, which frames this feature as "a one-time configuration step, not an ongoing runtime dependency for either party's session"; this spec verifies once and serves once verified, with no continuous health-check loop
- Choosing the domain-verification vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate for this capability

## Capability Category

**Category:** Domain verification and secure serving
**Dependency Source:** ASMP-32 -- "Domain-verification capability (Later phase)" (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Domain verification and secure serving at a freelancer's own domain -- Later phase (ASMP-32)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-27, FEAT-05)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Nadia's added or replaced domain is checked for her control of it | Verify and go live | FEAT-27.SPEC-001 (Custom Domain Settings) |
| Nadia is shown the specific steps she must complete to prove control, and the record moves to Verifying only once the capability has issued them | Verify and go live | FEAT-27.SPEC-001 (Verifying presentation) |
| Once verified, the portal and every client-facing link serve securely at the custom domain, with the shared default address still reachable (XBR-35) | Verify and go live | FEAT-27.SPEC-001 (status display); FEAT-05 (Client Portal Access) resolves the served address |
| A failed verification names the specific problem to Nadia and offers a re-check, while the shared default domain keeps serving the portal | Verify and go live | FEAT-27.SPEC-001 |
| Nadia is emailed once her domain is verified and live | Verify and go live | FEAT-27.SPEC-004 (Custom Domain Verified Confirmation) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Domain name | Custom Domain Record -- domain_name | Nadia adds a domain, replaces it, or requests a re-check | The capability must know exactly which domain to check control of and, once verified, which domain to serve the portal at |

Nothing else leaves the product for this capability: no client data, no freelancer personal data, no proposal, deliverable, invoice, or payment content ever crosses this boundary, since the Custom Domain Record carries no personal data (feature-dependency-map.md, Data Sensitivity: "None -- a domain name is public information").

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Submission accepted, with verification instructions (the specific steps Nadia must complete, in plain language) | The capability accepts the submitted domain_name from an Add, Replace-save, or Re-check and issues the steps for it | Custom Domain Record -- verification_state set to Verifying, with the verification instructions stored as system-written detail on the record (alongside the failure reason detail), replacing any earlier instructions |
| Verification succeeded | The capability confirms Nadia controls the submitted domain and secure serving is ready | Custom Domain Record -- verification_state set to Verified |
| Verification failed, with a specific reason | The capability cannot confirm control, or secure serving cannot be established, for the submitted domain | Custom Domain Record -- verification_state set to Verification Failed, with the specific failure reason recorded |
| Re-check result (succeeded or failed, same shape as above) | Nadia requests a re-check on a previously failed domain | Custom Domain Record -- verification_state updated to Verified or Verification Failed accordingly |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Verification instructions issued | The capability accepts a submitted domain_name (from Add, Replace-save, or Re-check) and issues the steps for proving control | Custom Domain Record: verification_state set to Verifying (this integration is the only writer of Verifying); verification instructions stored; on a Re-check, the prior failure reason is cleared | FEAT-27.SPEC-001 switches the badge to "Verifying", shows the verification steps and the propagation note, and (on a Re-check) shows the toast "Re-checking your domain." and stops showing "Re-check" | FEAT-27.SPEC-001 |
| Verification succeeded | The capability confirms control of the submitted domain_name and secure serving is ready | Custom Domain Record: verification_state set to Verified | FEAT-27.SPEC-001 shows a "Verified · Live" badge and confirms the shared default address remains reachable too; FEAT-27.SPEC-004 sends Nadia the confirmation email | FEAT-27.SPEC-001, FEAT-27.SPEC-004, FEAT-05 (resolves the served address per XBR-35) |
| Verification failed | The capability cannot confirm control of the submitted domain_name, or cannot establish secure serving for it | Custom Domain Record: verification_state set to Verification Failed, with the specific failure reason recorded | FEAT-27.SPEC-001 names the specific reason and offers "Re-check"; the shared default domain keeps serving the portal throughout, per XBR-35 | FEAT-27.SPEC-001 |
| Re-check result | Nadia's "Re-check" request (FEAT-27.SPEC-001), after the instructions-issued event above put the record in Verifying, resolves | Custom Domain Record: verification_state set to Verified or Verification Failed with a fresh reason | Same feedback as the corresponding succeeded/failed event above | FEAT-27.SPEC-001, and FEAT-27.SPEC-004 if the re-check succeeds |

No event in this integration involves multi-step branching beyond the direct verification_state update and the resulting screen/notification feedback, so none of these events route to a standalone Automation spec.

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-27.SPEC-001 (Custom Domain Settings) | The "Verifying" badge persists past the usual duration; a note appears: "This is taking longer than usual -- domain records can take time to propagate. Check back later; if verification does not succeed you will be able to re-check." No "Re-check" button is shown while Verifying (FEAT-27.SPEC-003 makes Re-check available only from Verification Failed); this integration always resolves the record to Verified or Verification Failed. The rest of the screen, including Remove, remains fully usable, and the shared default address keeps serving the portal throughout. | "Add domain", "Replace domain" (save), and "Re-check" are disabled with the message: "Domain verification is temporarily unavailable right now. Try again in a few minutes -- your portal keeps working at the shared default address the whole time." Viewing the current domain and its last known state, and removing the domain, remain available. | The submitted request is declined with the plain-language reason attached: "Verification could not start: {reason}. Check the domain and try again." The typed domain_name stays in the input exactly as entered. No half-created record results: a rejected first Add leaves no Custom Domain Record, and a rejected Replace-save or Re-check leaves the prior domain and its verification_state (Verified, Verifying, or Verification Failed) unchanged. |

## Consent and Disclosure

- **First domain-submission disclosure** -- The first time Nadia's account submits a domain (add, replace-save, or re-check) on FEAT-27.SPEC-001, and after FEAT-27.SPEC-003's format validation has passed, a notice appears before the request is sent: "To verify you control this domain and serve your portal securely at it, the domain name you enter is shared with the domain-verification capability. Nothing else about your account, clients, or data is shared." Options: "Continue" and "Cancel". "Continue" records Nadia's acknowledgment on her account and lets the request proceed; "Cancel" sends nothing, creates or changes no record, and records no acknowledgment, so the notice appears again on her next attempt. Shown once per account; afterwards a "How this is shared" link on FEAT-27.SPEC-001 reopens the same notice text in view-only form (a single "Close" button, nothing sent).
- **What is never shared** -- Every field on every other entity in the product -- client data, proposals, deliverables, invoices, payments, and the freelancer's own personal details -- stays inside the product. Only the submitted domain_name ever crosses this boundary. This boundary is stated explicitly in the disclosure notice above.

## Edge Cases

- **A verification event arrives for a domain Nadia has since removed** -- The event is discarded silently: no Custom Domain Record exists to update, and no user feedback fires, since there is nothing left on FEAT-27.SPEC-001 to reflect it.
- **The same verification-succeeded event is delivered twice** -- The second delivery changes nothing: a Custom Domain Record already Verified stays Verified, and FEAT-27.SPEC-004's confirmation email is not sent a second time (per that spec's own deduplication rule).
- **Events arrive out of order (a failure result for an earlier submission arrives after a later re-check's success)** -- The record reflects the most recent event by the event's own time, not its arrival time; a stale failure arriving after a newer success does not overwrite the Verified state.
- **The capability goes down mid-verification-request** -- If the request was not confirmed sent, the typed domain_name stays in the input and nothing is recorded: a first Add leaves no record, and a Replace-save or Re-check leaves the prior domain and verification_state as they were; no half-submitted state results. FEAT-27.SPEC-001 shows its submission-failure or capability-unavailable message accordingly.
- **The capability accepts a submission but never issues instructions or a result** -- The record stays Added (no instructions yet); FEAT-27.SPEC-001 keeps its "Added" presentation, Nadia can still Replace or Remove, and the shared default address keeps serving throughout.
- **Nadia replaces her domain while a verification for the previous domain is still in flight** -- The in-flight verification for the old domain_name is disregarded once it resolves (there is no longer a record for it to update, per FEAT-27.SPEC-003's replace behavior); only the new domain_name's verification result is applied.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-27.SPEC-001 (Custom Domain Settings) | Triggered by (inbound) | Add, Replace-save, and Re-check submit verification requests to this integration |
| FEAT-27.SPEC-001 (Custom Domain Settings) | Affects (outbound) | verification_state, the specific failure reason, and degradation messages surface here |
| FEAT-27.SPEC-003 (Custom Domain Validation & Fallback Rule) | References (inbound) | This integration relies on the one-domain-per-account limit to know there is at most one record to verify per freelancer, and on the fallback guarantee to keep the default address serving throughout verification and failure |
| FEAT-27.SPEC-004 (Custom Domain Verified Confirmation) | Triggers (outbound) | The verification-succeeded event fires this notification |
| FEAT-05 (Client Portal Access (Magic-Link Login)) | Affects (outbound) | Resolves, per XBR-35, which address (custom or shared default) the portal and client-facing links are served at |
| FEAT-31 (Operator Support Access) | References (inbound) | Dana's read-only render of FEAT-27.SPEC-001 during a logged support session displays this integration's reported verification_state, with no control over it |

## Analytics and Success Signals

- **custom_domain_verification_succeeded** (none beyond the event itself) -- N/A -- no success-metrics.md metric is connected to Custom Domain per Freelancer (FEAT-27); retained per product-features.md's own Signals field so verification outcomes remain observable
- **custom_domain_verification_failed** (failure_reason_category) -- N/A -- no success-metrics.md metric is connected to FEAT-27; retained for the same reason, so failure patterns remain observable to the product even without a Stage 2 metric measuring them
- **custom_domain_degradation_shown** (condition: slow / down / rejected; screen: FEAT-27.SPEC-001) -- N/A -- no success-metrics.md metric measures degradation frequency for this feature; retained so the product's tolerance for capability trouble on a Nice-to-Have, Later-phase feature stays observable rather than invisible

## Acceptance Criteria

**FEAT-27.SPEC-002-AC-01:** Given Nadia adds a domain on FEAT-27.SPEC-001, when the capability confirms she controls it, then the Custom Domain Record's verification_state is set to Verified, the screen shows "Verified · Live", and Nadia receives the FEAT-27.SPEC-004 confirmation email.

**FEAT-27.SPEC-002-AC-02:** Given Nadia adds a domain, when the capability cannot confirm control, then verification_state is set to Verification Failed with the specific reason recorded, and FEAT-27.SPEC-001 names that reason and offers "Re-check".

**FEAT-27.SPEC-002-AC-03:** Given Nadia's domain is in Verification Failed, when she taps "Re-check" and the capability now confirms control, then verification_state updates to Verified and the confirmation email is sent.

**FEAT-27.SPEC-002-AC-04:** Given Nadia's domain is Verifying, when the capability's response takes longer than usual, then FEAT-27.SPEC-001 shows "This is taking longer than usual -- domain records can take time to propagate. Check back later; if verification does not succeed you will be able to re-check.", shows no "Re-check" button, and the shared default address keeps serving the portal.

**FEAT-27.SPEC-002-AC-05:** Given the capability is unavailable, when Nadia taps "Add domain", "Replace domain" (save), or "Re-check", then that control is disabled with "Domain verification is temporarily unavailable right now. Try again in a few minutes -- your portal keeps working at the shared default address the whole time." and no Custom Domain Record is left half-created.

**FEAT-27.SPEC-002-AC-06:** Given the capability rejects a submitted domain, when the rejection is reported, then FEAT-27.SPEC-001 shows "Verification could not start: {reason}. Check the domain and try again.", the typed domain_name stays in the input, and no record is created or altered by the rejected request.

**FEAT-27.SPEC-002-AC-07:** Given Nadia's account has never submitted a domain before, when she taps "Add domain" with a validly formatted domain for the first time, then the data-sharing notice appears with "Continue" and "Cancel", and no domain data leaves the product until she chooses "Continue"; if she chooses "Cancel", nothing is sent and the notice appears again on her next attempt.

**FEAT-27.SPEC-002-AC-08:** Given a Custom Domain Record is already Verified, when the same verification-succeeded event is delivered again, then nothing changes and no duplicate confirmation email is sent.

**FEAT-27.SPEC-002-AC-09:** Given Nadia removes her domain, when a verification event for that removed domain later arrives, then it is discarded silently with no user feedback and no record updated.

**FEAT-27.SPEC-002-AC-10:** Given a stale failure event for an earlier submission arrives after a newer verification-succeeded event, when both have been received, then the record reflects the more recent event by event time and stays Verified.

**FEAT-27.SPEC-002-AC-11:** Given Nadia replaces her domain while the previous domain's verification is still in flight, when the previous verification later resolves, then it is disregarded and only the new domain's verification result is applied.

**FEAT-27.SPEC-002-AC-12:** Given a verification-succeeded event arrives for any freelancer's domain, when it is processed, then FEAT-05's portal and client-facing links resolve to that domain per XBR-35, while the shared default address remains reachable.

**FEAT-27.SPEC-002-AC-13:** Given Dana is viewing FEAT-27.SPEC-001 inside a logged support session (FEAT-31), when this integration's reported verification_state changes, then she sees the updated state with no control over it.

**FEAT-27.SPEC-002-AC-14:** Given the capability accepts the domain Nadia submitted via Add, Replace-save, or Re-check, when it issues the verification instructions, then the Custom Domain Record's verification_state is set to Verifying by this integration, the instructions are stored on the record, and FEAT-27.SPEC-001 shows the "Verifying" badge with those steps (on a Re-check, the prior failure reason is cleared).

**FEAT-27.SPEC-002-AC-15:** Given Nadia's record is Verifying, Verified, or Added, when she views FEAT-27.SPEC-001, then no "Re-check" button is shown; it is available only after this integration reports Verification Failed.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 5 | 5 |
| Inbound Events | 4 | 4 |
| Degradation Paths | 3 (1 screen x 3 conditions) | 3 |
| Consent and Disclosure | 2 | 2 |
| Edge Cases | 6 | 6 |

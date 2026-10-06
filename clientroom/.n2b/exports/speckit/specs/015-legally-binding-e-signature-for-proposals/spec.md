# Feature Specification: Legally Binding E-Signature for Proposals

**Blueprint feature:** FEAT-26
**Priority tier:** Nice-to-Have
**Build order:** 015 of 33
**Depends on:** FEAT-03
**Blueprint source:** `docs/blueprint/specifications/FEAT-26-legally-binding-e-signature-for-proposals/`

## User Scenarios & Testing (mandatory)

### User Story 1 - Signature Signing Step (Priority: P3)

Owen, the client's Primary Contact, completes a signature step in place of the plain Accept click when e-signature is enabled for the proposal, producing a legally binding signed record instead of a timestamped click.

**Acceptance Scenarios:**

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

### User Story 2 - Signature Recording (Priority: P3)

Validates Owen's signature submission, submits it for electronic-signature attestation, and writes the immutable signature record alongside FEAT-03's acceptance fields so the signed acceptance still fires the deposit invoice and audit-trail entry.

**Acceptance Scenarios:**

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

### User Story 3 - E-Signature Opt-In & Signing Access Rules (Priority: P3)

Governs the per-proposal e-signature opt-in scope, who may sign, the routing decision between FEAT-03's plain Accept and this feature's signing step, the full-legal-name field rule, failed-submission retry, and immutability once signed.

**Acceptance Scenarios:**

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

### User Story 4 - Signed-Copy Confirmation Notification (Priority: P3)

Emails Owen and Nadia a signed-copy confirmation the instant a proposal's signature is recorded, in addition to FEAT-03's standard acceptance confirmation, so both parties have a distinct, durable record that this acceptance carries the stronger evidentiary weight of a signature.

**Acceptance Scenarios:**

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

### User Story 5 - Electronic-Signature Attestation Capability (Priority: P3)

The product submits the signature data captured on the signing step to an external electronic-signature attestation capability and uses its confirmation to give the signed acceptance legal weight beyond a self-recorded timestamp.

**Acceptance Scenarios:**

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

### Edge Cases

- **FEAT-26.SPEC-001 (Signature Signing Step):** A repeat Sign tap or a signing attempt against a voided version is rejected-with-refresh (already-signed message or redirect to the current version), and connectivity or attestation failures show an error while retrying without duplicating the signature or losing the entered name. Navigating away and returning re-fetches the proposal and re-fills the name from the contact on file. Source: `docs/blueprint/specifications/FEAT-26-legally-binding-e-signature-for-proposals/FEAT-26.SPEC-001-signature-signing-step.md` (section: Edge Cases)
- **FEAT-26.SPEC-002 (Signature Recording):** Concurrent or retried signing submissions are admitted once by the atomic write, with the Sign control disabled while a run is in flight. The write-time re-check is authoritative, so a role demoted to Reviewer, a contact removed, or a proposal voided and re-sent after the screen loaded all return a denial or void outcome. Source: `docs/blueprint/specifications/FEAT-26-legally-binding-e-signature-for-proposals/FEAT-26.SPEC-002-signature-recording.md` (section: Edge Cases)
- **FEAT-26.SPEC-003 (E-Signature Opt-In & Signing Access Rules):** A full legal name of exactly one character passes while whitespace-only input fails with the enter-your-full-legal-name message. The signing-enabled flag simply reflects its state when the proposal is finally sent, so toggling it before sending raises no error. Source: `docs/blueprint/specifications/FEAT-26-legally-binding-e-signature-for-proposals/FEAT-26.SPEC-003-e-signature-opt-in-signing-access-rules.md` (section: Edge Cases)
- **FEAT-26.SPEC-004 (Signed-Copy Confirmation Notification):** A bounced address is retried then surfaced as a delivery warning to the freelancer (XBR-30), and the signed-copy confirmation is delivered independently of the standard acceptance confirmation (FEAT-03.SPEC-006). It uses a no-deposit-due line only when the schedule has no deposit, and still goes to the address captured in accepted_by if the contact is removed. Source: `docs/blueprint/specifications/FEAT-26-legally-binding-e-signature-for-proposals/FEAT-26.SPEC-004-signed-copy-confirmation-notification.md` (section: Edge Cases)
- **FEAT-26.SPEC-005 (Electronic-Signature Attestation Capability):** A duplicate attestation confirmation changes nothing, a confirmation for a proposal voided in the meantime still passes through the write-time re-check, and when a rejection and confirmation arrive out of order only the first outcome received is acted on. If the capability goes down mid-submission, an unanswered request is treated as a failure after timeout. Source: `docs/blueprint/specifications/FEAT-26-legally-binding-e-signature-for-proposals/FEAT-26.SPEC-005-electronic-signature-attestation-capability.md` (section: Edge Cases)

## Requirements (mandatory)

### Functional Requirements

- **FR-001**: The system MUST implement **FEAT-26.SPEC-001** (Signature Signing Step) as specified: Owen, the client's Primary Contact, completes a signature step in place of the plain Accept click when e-signature is enabled for the proposal, producing a legally binding signed record instead of a timestamped click. Full spec: `docs/blueprint/specifications/FEAT-26-legally-binding-e-signature-for-proposals/FEAT-26.SPEC-001-signature-signing-step.md`
- **FR-002**: The system MUST implement **FEAT-26.SPEC-002** (Signature Recording) as specified: Validates Owen's signature submission, submits it for electronic-signature attestation, and writes the immutable signature record alongside FEAT-03's acceptance fields so the signed acceptance still fires the deposit invoice and audit-trail entry. Full spec: `docs/blueprint/specifications/FEAT-26-legally-binding-e-signature-for-proposals/FEAT-26.SPEC-002-signature-recording.md`
- **FR-003**: The system MUST implement **FEAT-26.SPEC-003** (E-Signature Opt-In & Signing Access Rules) as specified: Governs the per-proposal e-signature opt-in scope, who may sign, the routing decision between FEAT-03's plain Accept and this feature's signing step, the full-legal-name field rule, failed-submission retry, and immutability once signed. Full spec: `docs/blueprint/specifications/FEAT-26-legally-binding-e-signature-for-proposals/FEAT-26.SPEC-003-e-signature-opt-in-signing-access-rules.md`
- **FR-004**: The system MUST implement **FEAT-26.SPEC-004** (Signed-Copy Confirmation Notification) as specified: Emails Owen and Nadia a signed-copy confirmation the instant a proposal's signature is recorded, in addition to FEAT-03's standard acceptance confirmation, so both parties have a distinct, durable record that this acceptance carries the stronger evidentiary weight of a signature. Full spec: `docs/blueprint/specifications/FEAT-26-legally-binding-e-signature-for-proposals/FEAT-26.SPEC-004-signed-copy-confirmation-notification.md`
- **FR-005**: The system MUST implement **FEAT-26.SPEC-005** (Electronic-Signature Attestation Capability) as specified: The product submits the signature data captured on the signing step to an external electronic-signature attestation capability and uses its confirmation to give the signed acceptance legal weight beyond a self-recorded timestamp. Full spec: `docs/blueprint/specifications/FEAT-26-legally-binding-e-signature-for-proposals/FEAT-26.SPEC-005-electronic-signature-attestation-capability.md`

### Key Entities

- Proposal (update: signature record)

## Success Criteria (mandatory)

### Measurable Outcomes

- **SC-001**: Enabling e-signature on a proposal and a client signing it are each observable as distinct signals (esignature_enabled, proposal_signed); no metric in the success-metrics register connects to this feature, so the outcome is grounded in its Signals alone. Source: `docs/blueprint/features/product-features.md`

## Assumptions

- **ASMP-11**: A plain timestamped Accept is assumed enough for most scope disputes, with Legally Binding E-Signature for Proposals (FEAT-26) added for those who need more. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-15**: Records are append-only and immutable once created, including signature records. Full register: `docs/blueprint/features/assumptions-constraints.md`

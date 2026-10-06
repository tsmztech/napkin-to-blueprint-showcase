---
document_type: spec
spec_type: screen
spec_id: FEAT-26.SPEC-001
spec_name: Signature Signing Step
spec_slug: signature-signing-step
parent_feature: FEAT-26
parent_feature_name: Legally Binding E-Signature for Proposals
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 22
---

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

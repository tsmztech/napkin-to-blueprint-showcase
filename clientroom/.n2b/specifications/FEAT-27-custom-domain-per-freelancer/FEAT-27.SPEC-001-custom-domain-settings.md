---
document_type: spec
spec_type: screen
spec_id: FEAT-27.SPEC-001
spec_name: Custom Domain Settings
spec_slug: custom-domain-settings
parent_feature: FEAT-27
parent_feature_name: Custom Domain per Freelancer
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 29
---

# Screen Spec: Custom Domain Settings

## Overview

**Name:** Custom Domain Settings
**ID:** FEAT-27.SPEC-001
**Type:** Screen
**Purpose:** Nadia adds, views, replaces, retries, and removes her custom domain from a single settings surface that shows its current verification state.
**Parent Feature:** FEAT-27 -- Custom Domain per Freelancer

## Scope and Non-Goals

**In Scope:**
- Adding a domain when none is configured
- Displaying the current domain and its verification state (Added, Verifying, Verified, Verification Failed with its specific reason)
- Replacing the configured domain with a different one, and cancelling that replacement
- Requesting a re-check after a failed verification
- Removing the configured domain and confirming the reversion to the shared default portal address
- The data-sharing disclosure notice shown before the first submission, and the "How this is shared" link that reopens it (content owned by FEAT-27.SPEC-002)
- Surfacing capability trouble (slow, unavailable, rejected) and the confirmation-email delivery-failure warning (owned by FEAT-27.SPEC-002 and FEAT-27.SPEC-004 respectively)
- Dana's read-only render of this same screen inside a logged support session (FEAT-31)
- Presenting the custom domain alongside the freelancer's existing branding (read-only reference to the Branding Profile)

**Non-Goals:**
- Verifying domain control and serving the portal securely at a verified domain -- owned by FEAT-27.SPEC-002 (Domain Verification & Secure Serving); this screen only submits requests to it and displays its reported state
- The one-domain-per-account limit, domain-format validation, and the fallback-to-default guarantee -- owned by FEAT-27.SPEC-003 (Custom Domain Validation & Fallback Rule); this screen enforces those rules by reference, it does not define them
- Editing the freelancer's logo or brand colour -- owned by FEAT-19.SPEC-001 (Branding Settings); this screen reads the Branding Profile for display only
- Ongoing monitoring or alerting for a domain that later breaks after going live -- excluded per product-features.md's States field: this is "a one-time configuration step, not an ongoing runtime dependency for either party's session," so this screen has no post-verification health-check surface

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-19.SPEC-001 (Branding Settings) | Nadia taps "Go further with your own domain" | None -- screen loads whatever Custom Domain Record currently exists, or the Empty state if none |
| FEAT-31 (Operator Support Access) | Dana opens a logged support session on a freelancer account and navigates here | Read-only render; no action controls are exposed |
| Direct return (default entry) | Nadia navigates back into custom domain setup after a previous visit | Screen reloads the current Custom Domain Record and its verification_state |
| FEAT-27.SPEC-004 (Custom Domain Verified Confirmation) | Nadia taps "View custom domain settings" in the confirmation email | Screen loads the current Custom Domain Record, normally showing the Verified state |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | All actions: add, replace, cancel replace, request re-check (from Verification Failed only), remove, open the "How this is shared" notice | -- |
| Owen (Client Primary Contact) | No | No | This screen is not part of the client portal -- Owen has no route to it; he only experiences whichever domain FEAT-27.SPEC-003's fallback rule resolves as active, with no visibility into this configuration surface |
| Priya (Client Reviewer Contact) | No | No | Same as Owen -- no route to this screen; she experiences whichever domain is active with no visibility into its configuration |
| Dana (Support Operator) | Full screen, read-only render (domain, verification state, failure reason or verification steps, delivery-failure warning, and Branding Profile preview visible) | None -- Add, Replace, Re-check, Remove, "How this is shared", and warning-dismiss controls are omitted entirely, not merely disabled | Support sessions are strictly read-only and logged (XBR-29, ASMP-18); Dana can reach this screen only inside an active, logged support session opened through FEAT-31 -- outside such a session she has no route to it at all |
| Unauthenticated | No | No | Redirected to the sign-in screen for Nadia's own account; there is no client-facing or public route to this screen |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." Any domain text typed but not yet submitted is discarded -- this screen holds no draft worth preserving across a session boundary since Add/Replace submit immediately on confirmation |

## Layout and Content

**Header:** Screen title "Custom Domain" with a back arrow (returns to FEAT-19.SPEC-001, Branding Settings).

**Delivery-warning banner (conditional):** Appears at the top of the body only when FEAT-27.SPEC-004 reports that the domain-verified confirmation email failed after all retries: "We couldn't email you the confirmation for {domain_name}. Your domain is still verified and live." with a "Dismiss" control. Not shown otherwise.

**Body:** A single content region that shows exactly one of the following presentations, depending on the initial fetch outcome, the current Custom Domain Record, and its verification_state (per the Shared UI Pattern: one settings surface carries every state of the domain lifecycle rather than splitting add/verify/remove into separate screens):

- **Loading:** While the Custom Domain Record and Branding Profile are being fetched on open, placeholder blocks stand in for the domain area and the branding preview; no action controls are shown.
- **Load error:** If the Custom Domain Record cannot be fetched, the message "We couldn't load your custom domain settings." with a "Try again" button. The back arrow stays available.
- **No domain configured:** A short explanation that the portal is currently reachable only at the shared default portal address, a "Your domain" text input, and an "Add domain" button below it.
- **Replace form (Nadia tapped "Replace domain"):** The current domain name shown as a reference, the "Your domain" input (empty), a "Save" button, and a "Cancel" control.
- **Domain configured (Added, Verifying, Verified, or Verification Failed):** The configured domain name displayed as text, a status badge showing the current verification_state, and, below it:
  - When Added: a note that verification is being requested. This is a brief transitional presentation between a successful submission and FEAT-27.SPEC-002 reporting that verification instructions were issued.
  - When Verifying: the verification steps (the verification instructions FEAT-27.SPEC-002 issued and the product stored on the Custom Domain Record), plus a note that domain records can take time to propagate. No "Re-check" button is shown in this state. If FEAT-27.SPEC-002 reports the verification is slow, the note "This is taking longer than usual -- domain records can take time to propagate. Check back later; if verification does not succeed you will be able to re-check." is added.
  - When Verified: a confirmation line that the portal is now reachable at this domain, alongside the shared default address, which always remains reachable too.
  - When Verification Failed: the specific failure reason FEAT-27.SPEC-002 reports, plus a "Re-check" button (the only state in which Re-check appears, per FEAT-27.SPEC-003).
  - In every configured state: a "Replace domain" link and a "Remove domain" button.
- **Capability-unavailable message:** When FEAT-27.SPEC-002 reports the capability is down, the message "Domain verification is temporarily unavailable right now. Try again in a few minutes -- your portal keeps working at the shared default address the whole time." appears above the domain area, and Add, Save, and Re-check are shown disabled.

**"How this is shared" link:** A text link under the domain area, shown for Nadia in every presentation except Loading and Load error. It opens the data-sharing notice in view-only form.

**Data-sharing notice (dialog):** Text owned by FEAT-27.SPEC-002 (Consent and Disclosure): "To verify you control this domain and serve your portal securely at it, the domain name you enter is shared with the domain-verification capability. Nothing else about your account, clients, or data is shared." Shown with "Continue" and "Cancel" when it interrupts a first submission, or with a single "Close" button when opened from the "How this is shared" link.

**Branding preview region:** Below the domain area, a read-only preview showing the freelancer's current logo and brand colour (read from the Branding Profile, FEAT-19), with the note that a verified custom domain is typically adopted alongside this branding to complete the white-label experience. If only the Branding Profile fetch fails, this region shows "Branding preview unavailable right now." and the domain area is unaffected.

**Footer:** None -- all actions live in the body.

For Dana's read-only render, the same regions appear (domain name, status badge, verification steps or failure reason, delivery-failure warning, branding preview) with the "Add domain" button, "Your domain" input, "Replace domain" link, "Re-check" button, "Remove domain" button, "How this is shared" link, and "Dismiss" control omitted entirely -- she sees values only, consistent with FEAT-19.SPEC-001's equivalent Dana render.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described above, full width; domain status badge wraps below the domain name if space is constrained.
- **Medium size class and above:** Content remains single-column, capped at a consistent platform-wide form width (exact value is the design layer's decision) and horizontally centered; no structural change beyond width capping.
- **Branding preview region:** Uniform scaling, no structural change -- it renders identically to its equivalent region on FEAT-19.SPEC-001 across breakpoints.

## Interactions

Who writes verification_state: this screen and FEAT-27.SPEC-003 set it to Added when a record is created or its domain is replaced; FEAT-27.SPEC-002 alone sets Verifying (when it accepts a submission and issues verification instructions), Verified, and Verification Failed. This screen displays those reports and never writes them.

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Screen open | Load | Fetches the Custom Domain Record and the Branding Profile | Loading presentation until both settle; then the presentation matching the record (or No domain configured) | Placeholder blocks while loading |
| "Try again" button (Load error) | Tap | Repeats the fetch of the Custom Domain Record | Returns to Loading; then to the matching presentation or, if it fails again, Load error | Placeholder blocks while loading |
| "Your domain" input | Type | Captures the entered domain text | Field shows entered text | Standard input focus state |
| "Your domain" input | Blur | No inline validation at this point -- format is checked on submit (per FEAT-27.SPEC-003) | None | None |
| "Add domain" button | Tap | 1. Validate the entered domain via FEAT-27.SPEC-003. 2. If invalid, show the inline error and stop. 3. If valid and Nadia has not yet acknowledged the data-sharing notice, open the notice with "Continue" and "Cancel" and wait. 4. If valid and already acknowledged (or after "Continue"), create the Custom Domain Record (verification_state Added) and submit the domain to FEAT-27.SPEC-002 as one operation. | Button shows a loading state during the create-and-submit step. On acceptance, badge shows "Added" and then "Verifying" once FEAT-27.SPEC-002 reports verification instructions issued | Success: screen switches to the configured presentation showing the verification steps. Failure (invalid format): inline error message below the input per FEAT-27.SPEC-003's exact text. Failure (rejected or not confirmed sent): see the Rejects and Error (submission) rows below |
| "Add domain" button (while loading) | Tap | No action -- ignored while the create is already in progress | Button remains in its loading state | No additional feedback |
| Data-sharing notice "Continue" (first-submission form) | Tap | Records Nadia's acknowledgment against her account (so the notice is not interrupting again), closes the notice, and proceeds with the pending Add, Replace-save, or Re-check request | Acknowledgment flag set; acting button shows its loading state | Notice closes |
| Data-sharing notice "Cancel" (first-submission form) | Tap | Closes the notice and sends nothing to FEAT-27.SPEC-002; no Custom Domain Record is created or changed and no acknowledgment is recorded | Screen returns to the presentation it was in before the tap, with the typed domain text preserved in the input; the notice will appear again on the next submission attempt | Notice closes; no toast |
| "How this is shared" link | Tap | Opens the same notice text in view-only form (single "Close" button); never sends anything | None | Dialog opens; focus moves into it |
| Data-sharing notice "Close" (link form) | Tap | Closes the notice | None | Dialog closes; focus returns to the link |
| "Replace domain" link | Tap | Reopens the "Your domain" input, pre-empty, in place of the current domain display | Screen shows the Replace form, with the current domain name shown as a reference above it | Standard transition, no page navigation |
| "Save" button (Replace form) | Tap | 1. Validate the new domain via FEAT-27.SPEC-003. 2. If valid, apply the data-sharing notice rule as for "Add domain". 3. Replace domain_name on the existing Custom Domain Record, reset verification_state to Added, and submit the new domain to FEAT-27.SPEC-002 as one operation. | Button shows a loading state. On acceptance, badge shows "Added" and then "Verifying" once verification instructions are issued | Success: screen returns to the configured presentation showing the new domain and its verification steps. Failure (format): inline error message per FEAT-27.SPEC-003. Failure (rejected or not confirmed sent): the prior domain and state stay as they were; see the Rejects and Error (submission) rows |
| "Save" button (while loading) | Tap | No action -- ignored while the replace is already in progress | Button remains in its loading state | No additional feedback |
| "Cancel" control (Replace form) | Tap | Discards any typed text and returns to the configured presentation of the existing domain; no request is made and the record is unchanged | Screen returns to the configured presentation showing the prior domain and state | Standard transition, no toast |
| "Re-check" button | Tap | Visible only when verification_state is Verification Failed. Applies the data-sharing notice rule, then re-submits the same domain_name to FEAT-27.SPEC-002 | Button shows a loading state and the failure reason stays visible until FEAT-27.SPEC-002 accepts the request; on acceptance the badge switches to "Verifying" (written by FEAT-27.SPEC-002), the failure reason is cleared, and the "Re-check" button is no longer shown | Toast on acceptance: "Re-checking your domain." |
| "Re-check" button (while loading) | Tap | No action -- ignored while the re-submission is in flight, between the tap and FEAT-27.SPEC-002's acceptance | Button remains in its loading state | No additional feedback |
| Rejected submission | FEAT-27.SPEC-002 rejects an Add, Save, or Re-check | Shows the reason; nothing else changes. A rejected first Add leaves no Custom Domain Record; a rejected Replace-save or Re-check leaves the prior domain and its state (Verified, Verifying, or Verification Failed) untouched. The typed domain text stays in the input | Acting button leaves its loading state | Inline message: "Verification could not start: {reason}. Check the domain and try again." |
| Submission failure (request not confirmed sent) | The Add, Save, or Re-check request fails before FEAT-27.SPEC-002 confirms receipt (for example, Nadia's connection drops) | Nothing is recorded: no record is created or changed | Acting button leaves its loading state; typed text stays in the input | Inline message: "Something went wrong and your domain wasn't saved. Check your connection and try again." |
| Capability slow | FEAT-27.SPEC-002 reports verification is taking longer than usual while Verifying | Adds the slow note to the Verifying presentation; Remove and Replace stay usable | Badge stays "Verifying" | Note: "This is taking longer than usual -- domain records can take time to propagate. Check back later; if verification does not succeed you will be able to re-check." |
| Capability down | FEAT-27.SPEC-002 reports the capability is unavailable | Disables Add, Save, and Re-check; viewing the current domain and state, Replace (opening the form), Cancel, and Remove remain available | The capability-unavailable message appears; disabled controls stay disabled until the capability is reported available on a later load or retry of the screen | Message: "Domain verification is temporarily unavailable right now. Try again in a few minutes -- your portal keeps working at the shared default address the whole time." |
| Delivery-warning "Dismiss" control | Tap | Hides the delivery-failure banner for this failure; it does not resend the email | Banner removed | None |
| "Remove domain" button | Tap | Opens a confirmation dialog before any change is made | None yet -- confirmation required first | Dialog: "Remove {domain_name}? Your portal will immediately go back to the shared default address. This can't be undone -- re-adding this domain later starts verification from scratch." with "Remove" and "Keep domain" options |
| Confirmation dialog "Remove" | Tap | Deletes the Custom Domain Record via FEAT-27.SPEC-003 | Screen returns to the "No domain configured" presentation | Toast: "Domain removed. Your portal is reachable at the shared default address again." |
| Confirmation dialog "Keep domain" | Tap | Closes the dialog, no change made | None | Dialog closes |
| Back arrow | Tap | Navigate to FEAT-19.SPEC-001 (Branding Settings); in the Replace form or with unsubmitted text in the "Your domain" input, the typed text is discarded without a prompt | Screen closes | Standard transition |
| Branding preview region | -- | Display-only -- reflects the current Branding Profile; not interactive on this screen | None | None |

### Accessibility Notes

- **Focus order:** Back arrow -> delivery-warning banner (when shown) -> domain area (input or domain display, depending on state) -> Replace/Save/Cancel/Re-check/Remove controls, in the order they appear -> "How this is shared" link -> branding preview (non-interactive, skipped in tab order).
- **Validation announcements:** When the domain input enters an error state after Add/Replace, or an inline rejection or submission-failure message appears, the message is announced to assistive technology and programmatically associated with the input.
- **Status announcements:** A verification_state change (Added -> Verifying, Verifying -> Verified, Verifying -> Verification Failed), the slow note, and the capability-unavailable message are announced to assistive technology as they happen, since the status badge is the primary content of this screen once a domain is configured.
- **Dialogs:** Focus moves into the "Remove {domain_name}?" dialog and into the data-sharing notice on open, and returns to the control that opened it on close (either option).
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading (initial fetch) | Placeholder blocks for the domain area and branding preview; no controls | Screen opens or "Try again" is tapped | The Custom Domain Record fetch succeeds (matching presentation) or fails (Load error) |
| Load error | "We couldn't load your custom domain settings." with "Try again"; back arrow available | The Custom Domain Record fetch fails | "Try again" is tapped |
| No domain configured (default) | "Your domain" input and "Add domain" button, with the shared-default-address explanation | Screen opens and no Custom Domain Record exists, or Nadia removes the domain | Nadia successfully adds a domain |
| Replace form | Current domain shown as reference, empty "Your domain" input, "Save" and "Cancel" | Nadia taps "Replace domain" | Save succeeds, Cancel is tapped, or Nadia leaves the screen |
| Added | Domain name with an "Added" badge and a note that verification is being requested | Add or Replace-save is accepted, before FEAT-27.SPEC-002 reports instructions issued | FEAT-27.SPEC-002 reports verification instructions issued (Verifying) |
| Verifying | Domain name with a "Verifying" badge, the verification steps, and the propagation note; no Re-check button | FEAT-27.SPEC-002 reports instructions issued (for an Add, Replace-save, or accepted Re-check) | FEAT-27.SPEC-002 reports the domain verified or failed |
| Verifying (slow) | Verifying presentation plus the slow note | FEAT-27.SPEC-002 reports verification is taking longer than usual | The capability reports the domain verified or failed |
| Verified / Live | Domain name with a "Verified · Live" badge and the confirmation that the portal is reachable there | FEAT-27.SPEC-002 reports verification succeeded | Nadia replaces or removes the domain (returns to Added, or to empty) |
| Verification Failed | Domain name with a "Verification Failed" badge, the specific failure reason, and a "Re-check" button | FEAT-27.SPEC-002 reports verification failed | Re-check is accepted (returns to Verifying), or the domain is replaced or removed |
| Loading (action, transient) | The acting button (Add, Save, or Re-check) shows a loading state; the rest of the screen remains as it was | Add, Save, or Re-check is tapped and any required notice is continued | The action completes (accepted, rejected, or failed to send) |
| Notice open | The data-sharing notice over the screen | First submission needs acknowledgment, or "How this is shared" is tapped | Continue, Cancel, or Close |
| Error (format) | Inline error message below the domain input, per FEAT-27.SPEC-003's exact text | Domain-format validation fails on Add or Save | Nadia corrects the input and resubmits |
| Error (rejected) | Inline "Verification could not start: {reason}. Check the domain and try again." message; prior domain and state unchanged | FEAT-27.SPEC-002 rejects the submission | Nadia edits the input and resubmits, or leaves |
| Error (submission failure) | Inline "Something went wrong and your domain wasn't saved. Check your connection and try again." message; nothing changed | The request fails before FEAT-27.SPEC-002 confirms receipt | Nadia retries the action |
| Degraded (capability down) | Capability-unavailable message; Add, Save, and Re-check disabled; viewing, Replace (opening the form), Cancel, and Remove still work | FEAT-27.SPEC-002 reports the capability unavailable | The capability is reported available again on a later load or retry |
| Delivery warning shown | Banner "We couldn't email you the confirmation for {domain_name}. Your domain is still verified and live." above the body | FEAT-27.SPEC-004 reports delivery failed after all retries | Nadia taps "Dismiss", or the domain is replaced or removed |
| Offline | The fetch and submissions fail per Load error and Error (submission failure); no separate offline presentation exists | Nadia's connection is lost | Connection returns and Nadia retries |

## Validation Rules

Validation governed by FEAT-27.SPEC-003 (Custom Domain Validation & Fallback Rule). See that spec for the domain-format rule, the one-domain-per-account cardinality rule, and the fallback guarantee. This screen applies validation on Add and Save submit, before the data-sharing notice can appear; there is no on-blur validation for the domain input.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-19.SPEC-001 (Branding Settings) | FEAT-19 (Freelancer Branding) |
| Successful add or replace | Remains on this screen, now showing the configured presentation | -- |
| Confirmed remove | Remains on this screen, now showing the "No domain configured" presentation | -- |
| Replace "Cancel" | Remains on this screen, showing the existing domain | -- |

## Data Model

**Creates:** Custom Domain Record -- domain_name set from the "Your domain" input; verification_state set to Added. Validated by FEAT-27.SPEC-003 before creation, and created only in the same operation that FEAT-27.SPEC-002 accepts.
**Reads:** Custom Domain Record -- domain_name, verification_state, the specific failure reason, and the verification instructions, for display. Branding Profile (FEAT-19) -- logo and brand_colour, for the read-only preview region. Nadia's data-sharing acknowledgment flag on her account, to decide whether the notice interrupts a submission.
**Updates:** Custom Domain Record -- domain_name (on Replace) and verification_state (on Replace, resetting to Added). verification_state values Verifying, Verified, and Verification Failed, the failure reason, and the verification instructions are written by FEAT-27.SPEC-002's reports, which this screen reflects but does not itself write. Nadia's data-sharing acknowledgment flag -- set on "Continue".
**Deletes:** Custom Domain Record -- on a confirmed Remove. Hard delete, no restore path.

## Business Rules

- FEAT-27.SPEC-003 governs domain format, the one-domain-per-account limit, and the fallback guarantee -- this screen enforces those rules by reference and never duplicates them.
- XBR-35: with a verified custom domain, the portal and client-facing links use it, and the shared default address always remains available as a fallback -- this screen's copy in every state reflects that guarantee explicitly (e.g., the Verified state names the default address as still reachable; the Remove confirmation names the immediate reversion to it; the capability-unavailable message names it).
- Replacing a domain (FEAT-27.SPEC-003) resets verification_state to Added and re-submits verification via FEAT-27.SPEC-002 -- Nadia cannot keep a prior Verified state against a new domain name.
- Removing a domain is a hard delete with no cascade: nothing else in the product references the Custom Domain Record directly (dependency map, Relationships), so removal has no downstream cleanup beyond the fallback reversion itself.
- Data-sharing notice: the first time Nadia's account attempts an Add, Replace-save, or Re-check, the notice interrupts after format validation passes and before anything is sent. After "Continue" is chosen once for the account, later submissions send directly and only the "How this is shared" link shows the notice again. Nothing leaves the product while the notice is open or after "Cancel" (FEAT-27.SPEC-002, Consent and Disclosure).
- Re-check availability: "Re-check" is shown only in Verification Failed (FEAT-27.SPEC-003 Authorization Rules). While Verifying, including the slow case, Nadia waits, replaces, or removes; FEAT-27.SPEC-002 resolves every Verifying record to Verified or Verification Failed, after which Re-check is available if needed.

## Edge Cases

- **Nadia taps "Add domain" twice rapidly** -- The second tap is ignored while the first create-and-submit request is in progress (button in loading state).
- **Nadia navigates away while Add or Replace is loading** -- The request completes in the background; the next time she opens this screen it reflects whatever the request resolved to (accepted and Verifying, or unchanged if it was rejected or never confirmed sent).
- **Nadia types a domain in the Replace form (or the Add input) and navigates away before submitting** -- The back arrow leaves immediately with no prompt; the typed text is discarded, nothing is sent, and the existing record is unchanged. Returning shows the configured presentation (or the empty Add input).
- **Nadia taps Cancel in the data-sharing notice** -- Nothing is sent, no record is created or changed, no acknowledgment is recorded, and her typed text stays in the input; the notice reappears on her next submission attempt.
- **Nadia's own two sessions both act on the domain (e.g., one tab open on this screen, another used to remove the domain)** -- There is at most one Custom Domain Record and no other human writer (dependency map, Contention: "None -- only Nadia configures it"), so the second session's view simply reflects whichever action completed last; there is no true conflict to reject, only a stale display that refreshes to the current record the next time the screen loads or an action is attempted against it.
- **Verification is still pending when Nadia navigates away and returns** -- The screen re-fetches the Custom Domain Record on load and shows its current verification_state, whatever FEAT-27.SPEC-002 has reported since her last visit.
- **Nadia taps Remove while verification is still Verifying (not yet Verified or Failed)** -- The same confirmation dialog applies; confirming removes the record and reverts to the shared default address immediately, discarding the in-progress verification.
- **Nadia re-adds the exact domain she just removed** -- Treated as a brand-new Custom Domain Record per FEAT-27.SPEC-003: verification starts from Added, with no memory of the prior verification outcome.
- **The capability goes down after Nadia opens the Replace form** -- Save becomes disabled with the capability-unavailable message; Cancel and the typed text are unaffected.
- **The delivery-failure warning is showing and Nadia replaces or removes the domain** -- The banner is removed, because it described the earlier domain's confirmation email.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-27.SPEC-002 (Domain Verification & Secure Serving) | Triggers (outbound) | Add, Replace-save, and Re-check submit requests to this integration; this screen displays its reported verification_state, verification instructions, failure reasons, and degradation messages, and shows its data-sharing notice |
| FEAT-27.SPEC-003 (Custom Domain Validation & Fallback Rule) | References (inbound) | Domain-format validation, the one-domain-per-account limit, Re-check availability, and the fallback guarantee govern Add, Replace, Re-check, and Remove |
| FEAT-27.SPEC-004 (Custom Domain Verified Confirmation) | Navigation (inbound) and Affects (inbound) | The email's CTA lands here; a delivery failure after all retries surfaces here as the delivery-warning banner |
| FEAT-19.SPEC-001 (Branding Settings) | Navigation (inbound) | Entry point via "Go further with your own domain"; also read for the Branding Profile preview |
| FEAT-31 (Operator Support Access) | Navigation (inbound) | Dana's read-only entry point during a logged support session |
| FEAT-24 (Data Export & Account Deletion) | References (informational) | The Custom Domain Record this screen manages is also deleted as part of a full account deletion cascade, never by this screen directly |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| custom_domain_added | none beyond the event itself (domain_name is not included, per data-minimization discipline) | Add is accepted and the Custom Domain Record is created | N/A -- no success-metrics.md metric is connected to Custom Domain per Freelancer (FEAT-27); retained per product-features.md's own Signals field so custom-domain adoption remains observable |
| custom_domain_replaced | none | Replace is accepted | N/A -- no success-metrics.md metric is connected to FEAT-27; retained for the same reason as custom_domain_added |
| custom_domain_removed | none | Remove is confirmed | N/A -- no success-metrics.md metric is connected to FEAT-27; retained so removal (and reversion to the fallback) remains observable |
| custom_domain_recheck_requested | none | Re-check is accepted by FEAT-27.SPEC-002 | N/A -- no success-metrics.md metric is connected to FEAT-27; retained so re-check usage after a failure remains observable |
| custom_domain_disclosure_responded | response: continue / cancel / closed | Nadia chooses an option in the data-sharing notice | N/A -- no success-metrics.md metric is connected to FEAT-27; retained so drop-off at the disclosure remains observable |

## Acceptance Criteria

**FEAT-27.SPEC-001-AC-01:** Given Nadia is on the Custom Domain Settings screen with no domain configured and has already acknowledged the data-sharing notice, when she enters a validly formatted domain and taps "Add domain", then the Custom Domain Record is created with verification_state Added, the domain is submitted to FEAT-27.SPEC-002, and once FEAT-27.SPEC-002 reports verification instructions issued the screen shows the domain with a "Verifying" badge and those steps.

**FEAT-27.SPEC-001-AC-02:** Given Nadia enters an invalidly formatted domain, when she taps "Add domain", then an inline error appears below the input per FEAT-27.SPEC-003's exact text, no Custom Domain Record is created, and the data-sharing notice does not appear.

**FEAT-27.SPEC-001-AC-03:** Given Nadia has a domain in the Verification Failed state with a specific reason shown, when she taps "Re-check" and FEAT-27.SPEC-002 accepts the request, then the badge switches to "Verifying", the failure reason is cleared, the "Re-check" button is no longer shown, and the toast "Re-checking your domain." appears.

**FEAT-27.SPEC-001-AC-04:** Given Nadia has a Verified domain, when she taps "Replace domain" and submits a new, validly formatted domain, then domain_name is updated, verification_state resets to Added, the new domain is submitted to FEAT-27.SPEC-002, and the badge moves to "Verifying" once verification instructions are issued for the new domain.

**FEAT-27.SPEC-001-AC-05:** Given Nadia taps "Remove domain", when she confirms "Remove" in the dialog, then the Custom Domain Record is deleted, the screen returns to the "No domain configured" presentation, and a toast confirms the portal is reachable at the shared default address again.

**FEAT-27.SPEC-001-AC-06:** Given Nadia taps "Remove domain", when she chooses "Keep domain" in the dialog, then the dialog closes and the Custom Domain Record is unchanged.

**FEAT-27.SPEC-001-AC-07:** Given Nadia's domain reaches the Verified state, when the screen displays it, then the copy explicitly states that the shared default address remains reachable too, per XBR-35.

**FEAT-27.SPEC-001-AC-08:** Given Dana opens a logged support session on a freelancer account (FEAT-31) and navigates to this screen, when it renders, then she sees the domain name, verification state, and branding preview with every Add, Replace, Re-check, Remove, "How this is shared", and Dismiss control omitted.

**FEAT-27.SPEC-001-AC-09:** Given Owen or Priya, when they look for any route to this screen from the client portal, then none exists -- they experience only whichever domain FEAT-27.SPEC-003 resolves as active.

**FEAT-27.SPEC-001-AC-10:** Given an unauthenticated visitor requests this screen's address directly, when the request is made, then they are redirected to Nadia's sign-in screen with no client-facing route ever exposed.

**FEAT-27.SPEC-001-AC-11:** Given Nadia's session expires while she is on this screen, when she attempts an action, then the dialog "Your session has expired. Sign in to continue." appears and any unsubmitted domain text is discarded.

**FEAT-27.SPEC-001-AC-12:** Given Nadia taps "Add domain" twice rapidly, when the first request is still in progress, then the second tap has no effect and the button remains in its loading state.

**FEAT-27.SPEC-001-AC-13:** Given Nadia has one browser tab open on this screen while removing the domain from a second tab, when she returns to the first tab and takes any action, then it reflects the domain's current state (removed) rather than the stale state it loaded with.

**FEAT-27.SPEC-001-AC-14:** Given Nadia re-adds the exact domain she previously removed, when Add succeeds, then verification starts fresh from Added and reaches Verifying only when FEAT-27.SPEC-002 issues new instructions, with no memory of the prior verification outcome.

**FEAT-27.SPEC-001-AC-15:** Given this screen is loading on a narrow (compact) viewport, when the domain area renders, then the status badge wraps below the domain name if space is constrained, with no structural change to the single-column layout.

**FEAT-27.SPEC-001-AC-16:** Given Nadia's account has never acknowledged the data-sharing notice and she enters a validly formatted domain, when she taps "Add domain", then the notice appears with "Continue" and "Cancel", and no Custom Domain Record is created and nothing is sent to FEAT-27.SPEC-002 until she taps "Continue".

**FEAT-27.SPEC-001-AC-17:** Given the data-sharing notice is open on a first submission, when Nadia taps "Cancel", then the notice closes, no record is created or changed, nothing is sent, the typed domain text remains in the input, and the notice appears again on her next submission attempt.

**FEAT-27.SPEC-001-AC-18:** Given Nadia has previously tapped "Continue" on the notice, when she later submits a Replace or Re-check, then no notice interrupts, and when she taps "How this is shared" then the same notice text opens with a single "Close" button and sends nothing.

**FEAT-27.SPEC-001-AC-19:** Given Nadia's domain is Verifying and FEAT-27.SPEC-002 reports verification is slow, when the screen displays it, then the note "This is taking longer than usual -- domain records can take time to propagate. Check back later; if verification does not succeed you will be able to re-check." appears, no "Re-check" button is shown, and Replace and Remove remain usable.

**FEAT-27.SPEC-001-AC-20:** Given FEAT-27.SPEC-002 reports the capability is unavailable, when Nadia views the screen, then "Domain verification is temporarily unavailable right now. Try again in a few minutes -- your portal keeps working at the shared default address the whole time." appears, Add, Save, and Re-check are disabled, and viewing the current domain and Remove remain available.

**FEAT-27.SPEC-001-AC-21:** Given Nadia submits a domain that FEAT-27.SPEC-002 rejects, when the rejection is reported, then "Verification could not start: {reason}. Check the domain and try again." appears inline, her typed text stays in the input, a first Add leaves no Custom Domain Record, and a Replace-save or Re-check leaves the prior domain and its state unchanged.

**FEAT-27.SPEC-001-AC-22:** Given Nadia taps "Add domain" and her connection drops before FEAT-27.SPEC-002 confirms receipt, when the request fails, then "Something went wrong and your domain wasn't saved. Check your connection and try again." appears, no record is created or changed, and her typed text stays in the input.

**FEAT-27.SPEC-001-AC-23:** Given the confirmation email for a verified domain fails delivery after all retries (FEAT-27.SPEC-004), when Nadia opens this screen, then the banner "We couldn't email you the confirmation for {domain_name}. Your domain is still verified and live." appears with a "Dismiss" control, and tapping "Dismiss" hides it without resending the email; Dana sees the banner without a Dismiss control.

**FEAT-27.SPEC-001-AC-24:** Given Nadia opens this screen, when the Custom Domain Record and Branding Profile are still being fetched, then placeholder blocks appear in place of the domain area and branding preview and no action controls are shown.

**FEAT-27.SPEC-001-AC-25:** Given the Custom Domain Record fetch fails on open, when the screen settles, then "We couldn't load your custom domain settings." appears with a "Try again" button that repeats the fetch, and the back arrow still returns to FEAT-19.SPEC-001.

**FEAT-27.SPEC-001-AC-26:** Given Nadia has tapped "Replace domain" and typed a new domain, when she taps "Cancel", then the typed text is discarded, no request is made, and the screen shows her existing domain and state unchanged.

**FEAT-27.SPEC-001-AC-27:** Given Nadia has typed text in the Replace form and has not submitted it, when she taps the back arrow, then she leaves to FEAT-19.SPEC-001 with no prompt, the text is discarded, and her existing record is unchanged when she returns.

**FEAT-27.SPEC-001-AC-28:** Given Nadia's domain is in Added, Verifying, or Verified, when she views the screen, then no "Re-check" button is shown; it appears only in Verification Failed.

**FEAT-27.SPEC-001-AC-29:** Given FEAT-27.SPEC-002 has issued verification instructions for Nadia's domain, when the screen shows the Verifying state, then the verification steps are displayed with the propagation note, and the badge value "Verifying" was set by FEAT-27.SPEC-002's report rather than by this screen.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 26 | 26 |
| States | 17 (loading fetch, load error, no domain, replace form, added, verifying, verifying slow, verified, verification failed, loading action, notice open, error format, error rejected, error submission failure, degraded down, delivery warning, offline) | 17 |
| Business Rules | 6 | 6 |
| Edge Cases | 10 | 10 |

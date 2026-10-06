# Feature Specification: Custom Domain per Freelancer

**Blueprint feature:** FEAT-27
**Priority tier:** Nice-to-Have
**Build order:** 016 of 33
**Depends on:** FEAT-05, FEAT-19
**Blueprint source:** `docs/blueprint/specifications/FEAT-27-custom-domain-per-freelancer/`

## User Scenarios & Testing (mandatory)

### User Story 1 - Custom Domain Settings (Priority: P3)

Nadia adds, views, replaces, retries, and removes her custom domain from a single settings surface that shows its current verification state.

**Acceptance Scenarios:**

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

### User Story 2 - Domain Verification & Secure Serving (Priority: P3)

Verifies that Nadia controls the domain she added and, once verified, serves her portal securely at it, reporting verification, failure, and re-check results back to the product.

**Acceptance Scenarios:**

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

### User Story 3 - Custom Domain Validation & Fallback Rule (Priority: P3)

Enforces the domain-format rule and the one-domain-per-account limit on the Custom Domain Record, and guarantees the shared default portal address always remains reachable as a fallback.

**Acceptance Scenarios:**

**FEAT-27.SPEC-003-AC-01:** Given Nadia has no Custom Domain Record, when she submits "myportal.com" via Add domain, then the domain-format rule passes and a Custom Domain Record is created with verification_state Added.

**FEAT-27.SPEC-003-AC-02:** Given Nadia submits an empty domain field, when she taps Add domain, then she sees "Enter your domain to continue." and no record is created.

**FEAT-27.SPEC-003-AC-03:** Given Nadia submits "http://myportal.com", when she taps Add domain, then she sees "Enter a valid domain name (e.g., myportal.com) -- without \"http://\" or any extra text." and no record is created.

**FEAT-27.SPEC-003-AC-04:** Given Nadia already has a Custom Domain Record for "myportal.com", when she submits "newdomain.com" via Replace domain, then domain_name updates to "newdomain.com" and verification_state resets to Added, re-triggering verification.

**FEAT-27.SPEC-003-AC-05:** Given Nadia's Custom Domain Record is in Verification Failed, when she taps Re-check, then the request is allowed and verification is re-triggered for the same domain_name.

**FEAT-27.SPEC-003-AC-06:** Given Nadia's Custom Domain Record is Verified or Verifying (including a slow verification), when she looks for a Re-check control, then none is shown -- Re-check is available only from Verification Failed.

**FEAT-27.SPEC-003-AC-07:** Given Nadia (Freelancer), when she attempts to add, replace, request a re-check on, or remove her domain, then every action is allowed without further condition beyond the record-existence and verification-state conditions stated above.

**FEAT-27.SPEC-003-AC-08:** Given Owen or Priya, when either looks for any control to add, replace, re-check, or remove a custom domain, then no such control or route exists anywhere in the client portal.

**FEAT-27.SPEC-003-AC-09:** Given Dana is inside an active, logged support session opened via FEAT-31, when she views FEAT-27.SPEC-001, then she can see the domain and its verification state but has no Add, Replace, Re-check, or Remove control available to her.

**FEAT-27.SPEC-003-AC-10:** Given Dana's support session has closed, when she attempts to view the Custom Domain Record again, then she has no route to it at all.

**FEAT-27.SPEC-003-AC-11:** Given a domain is verified and reachable, when a client contact opens a portal link, then it resolves to the verified custom domain while the shared default address remains reachable too, per XBR-35.

**FEAT-27.SPEC-003-AC-12:** Given Nadia's domain is in Verification Failed, when a client contact opens a portal link during that time, then the portal is still fully reachable at the shared default address, per the fallback guarantee.

**FEAT-27.SPEC-003-AC-13:** Given Nadia removes her verified domain, when the removal is confirmed, then verification_state and domain_name are cleared (the record is deleted), and portal links immediately resolve to the shared default address again.

**FEAT-27.SPEC-003-AC-14:** Given Nadia re-adds the exact domain she just removed, when the new record is created, then verification_state starts at Added with no memory of the prior verification outcome.

### User Story 4 - Custom Domain Verified Confirmation (Priority: P3)

Emails Nadia once her custom domain is verified and live, so she knows without checking the settings screen that her portal now serves at her own domain.

**Acceptance Scenarios:**

**FEAT-27.SPEC-004-AC-01:** Given Nadia's domain reaches verification_state Verified, when this notification fires, then she receives an email with the subject "Your custom domain is verified and live" naming her verified domain.

**FEAT-27.SPEC-004-AC-02:** Given Nadia receives this confirmation email, when she taps "View custom domain settings", then she lands on FEAT-27.SPEC-001 showing her domain's Verified state.

**FEAT-27.SPEC-004-AC-03:** Given a verification-succeeded event is delivered twice for the same already-Verified record, when the second delivery is processed, then no second confirmation email is sent.

**FEAT-27.SPEC-004-AC-04:** Given there is no notification preference for this email, when Nadia has every optional notification turned off on FEAT-21.SPEC-002, then this confirmation is still sent whenever her domain verifies.

**FEAT-27.SPEC-004-AC-05:** Given delivery of this email fails once for a transient reason, when the delivery capability retries within platform parameter: `transactional-email-retry-window`, then up to platform parameter: `transactional-email-retry-count` retries occur before any failure is surfaced to Nadia.

**FEAT-27.SPEC-004-AC-06:** Given delivery of this email fails after all retries are exhausted, when the final failure is processed, then a delivery-failure warning appears on FEAT-27.SPEC-001.

**FEAT-27.SPEC-004-AC-07:** Given Nadia removes her domain shortly after verification but before this email is delivered, when delivery proceeds, then the email is still sent describing the domain that was verified.

**FEAT-27.SPEC-004-AC-08:** Given Nadia's domain fails verification (or a re-check still fails), then this notification never fires for that event -- only a successful transition into Verified triggers it.

**FEAT-27.SPEC-004-AC-09:** Given Owen, Priya, or Dana, then none of them ever receives this notification, since only Nadia is entitled to it.

**FEAT-27.SPEC-004-AC-10:** Given Nadia replaces her domain again before this email for the first domain is delivered, when delivery proceeds, then the email is delivered describing the first, already-verified domain, unaffected by the later replacement.

### Edge Cases

- **FEAT-27.SPEC-001 (Custom Domain Settings):** Double taps on Add domain are ignored, an Add or Replace in flight completes in the background and the screen reflects the result on next open, and leaving the form with typed text discards it with no prompt. Cancel in the data-sharing notice sends nothing and keeps the typed text, with the notice reappearing on the next attempt. Source: `docs/blueprint/specifications/FEAT-27-custom-domain-per-freelancer/FEAT-27.SPEC-001-custom-domain-settings.md` (section: Edge Cases)
- **FEAT-27.SPEC-002 (Domain Verification & Secure Serving):** A verification event for a removed domain is discarded silently and a duplicate verified event changes nothing (no second confirmation email). Events are applied by event time so a stale failure after a later success is ignored, and if the capability goes down before a request is confirmed sent nothing is recorded and the typed domain stays. Source: `docs/blueprint/specifications/FEAT-27-custom-domain-per-freelancer/FEAT-27.SPEC-002-domain-verification-secure-serving.md` (section: Edge Cases)
- **FEAT-27.SPEC-003 (Custom Domain Validation & Fallback Rule):** A domain with an http:// or https:// prefix is rejected by the format rule, a whitespace-only value is treated as empty, and the shortest valid form (two labels around one dot, such as ab.co) passes. Submitting the already-configured domain via Replace is a normal Replace that resets verification to Added and re-triggers it. Source: `docs/blueprint/specifications/FEAT-27-custom-domain-per-freelancer/FEAT-27.SPEC-003-custom-domain-validation-fallback-rule.md` (section: Edge Cases)
- **FEAT-27.SPEC-004 (Custom Domain Verified Confirmation):** The email is still delivered if the domain record is removed or replaced after verification, since it reports a completed historical event. A duplicate verified event sends only one notification, and the notification has no preference control or quiet-hours window to collide with. Source: `docs/blueprint/specifications/FEAT-27-custom-domain-per-freelancer/FEAT-27.SPEC-004-custom-domain-verified-confirmation.md` (section: Edge Cases)

## Requirements (mandatory)

### Functional Requirements

- **FR-001**: The system MUST implement **FEAT-27.SPEC-001** (Custom Domain Settings) as specified: Nadia adds, views, replaces, retries, and removes her custom domain from a single settings surface that shows its current verification state. Full spec: `docs/blueprint/specifications/FEAT-27-custom-domain-per-freelancer/FEAT-27.SPEC-001-custom-domain-settings.md`
- **FR-002**: The system MUST implement **FEAT-27.SPEC-002** (Domain Verification & Secure Serving) as specified: Verifies that Nadia controls the domain she added and, once verified, serves her portal securely at it, reporting verification, failure, and re-check results back to the product. Full spec: `docs/blueprint/specifications/FEAT-27-custom-domain-per-freelancer/FEAT-27.SPEC-002-domain-verification-secure-serving.md`
- **FR-003**: The system MUST implement **FEAT-27.SPEC-003** (Custom Domain Validation & Fallback Rule) as specified: Enforces the domain-format rule and the one-domain-per-account limit on the Custom Domain Record, and guarantees the shared default portal address always remains reachable as a fallback. Full spec: `docs/blueprint/specifications/FEAT-27-custom-domain-per-freelancer/FEAT-27.SPEC-003-custom-domain-validation-fallback-rule.md`
- **FR-004**: The system MUST implement **FEAT-27.SPEC-004** (Custom Domain Verified Confirmation) as specified: Emails Nadia once her custom domain is verified and live, so she knows without checking the settings screen that her portal now serves at her own domain. Full spec: `docs/blueprint/specifications/FEAT-27-custom-domain-per-freelancer/FEAT-27.SPEC-004-custom-domain-verified-confirmation.md`

### Key Entities

- Custom Domain Record (create, update)

## Success Criteria (mandatory)

### Measurable Outcomes

- **SC-001**: Custom domain additions, successful verifications and failed verifications are each observable as distinct signals (custom_domain_added, custom_domain_verified, custom_domain_verification_failed); no metric in the success-metrics register connects to this feature, so the outcome is grounded in its Signals alone. Source: `docs/blueprint/features/product-features.md`

## Assumptions

- **ASMP-32**: The domain-verification capability is a Later-phase dependency; without it only the shared default portal address is available and nothing in the MVP depends on it. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-24**: Personal data worldwide is treated as GDPR-class, which bears on the data-sharing notice shown when adding a domain. Full register: `docs/blueprint/features/assumptions-constraints.md`

# Feature Specification: Onboarding / First-Run Setup

**Blueprint feature:** FEAT-20
**Priority tier:** Important
**Build order:** 031 of 33
**Depends on:** FEAT-01, FEAT-02, FEAT-19, FEAT-32, FEAT-33
**Blueprint source:** `docs/blueprint/specifications/FEAT-20-onboarding-first-run-setup/`

## User Scenarios & Testing (mandatory)

### User Story 1 - Sign-Up & Account Creation (Priority: P2)

A new freelancer creates her Freelancer Account, which is created and immediately transitioned to Active state, starting the guided onboarding sequence.

**Acceptance Scenarios:**

**FEAT-20.SPEC-001-AC-01:** Given a new visitor is on the Sign-Up screen, when she fills in her name, a unique sign-in email, matching passwords, accepts the Terms, and taps "Create my account", then her Freelancer Account is created in Active state, `onboarding_started` is emitted, and she is taken directly into FEAT-20.SPEC-002 (Onboarding Guided Sequence).

**FEAT-20.SPEC-001-AC-02:** Given a visitor is on the Sign-Up screen, when she taps "Create my account" with the name field empty, then the name field shows "Your name is required" and creation does not proceed.

**FEAT-20.SPEC-001-AC-03:** Given a visitor enters an email that already belongs to an existing Freelancer Account, when she blurs the email field, then she sees "An account already exists for this email. Sign in instead." with a link to sign-in.

**FEAT-20.SPEC-001-AC-04:** Given a visitor enters mismatched values in Password and Confirm Password, when she blurs the Confirm Password field, then she sees "Passwords don't match".

**FEAT-20.SPEC-001-AC-05:** Given a visitor fills the form correctly but leaves the Terms checkbox unchecked, when she taps "Create my account", then she sees "You must accept the Terms of Service and Privacy Policy to continue" and no account is created.

**FEAT-20.SPEC-001-AC-06:** Given Nadia (already signed in) navigates directly to the Sign-Up screen's URL, when the screen would otherwise render, then she is redirected to her current onboarding step or dashboard instead, with no form shown.

**FEAT-20.SPEC-001-AC-07:** Given a client contact who is also a freelancer follows the "Made with Clientroom" mark from a portal she was viewing (FEAT-33), when she completes this form, then a new Freelancer Account is created with the referring-portal reference carried forward, unrelated to her existing client-contact access.

**FEAT-20.SPEC-001-AC-08:** Given a visitor loses connectivity while filling the form, when she attempts to submit, then the banner "This step needs a connection. Your details are saved here and will be submitted once you're back online." appears and her typed values remain in the form.

**FEAT-20.SPEC-001-AC-09:** Given two visitors submit the same email at effectively the same time, when the second submission's account creation is committed, then it is rejected with "An account already exists for this email. Sign in instead." even though it may have passed the earlier field-level check.

**FEAT-20.SPEC-001-AC-10:** Given a visitor taps "Create my account" while a previous submission from the same tap is still processing, when she taps a second time, then the second tap has no effect and the button remains in its loading state.

**FEAT-20.SPEC-001-AC-11:** Given Dana's operator identity has no product sign-up path, when anyone attempts to provision operator access through this screen, then the attempt is undefined here -- it would only ever create an ordinary Freelancer Account, never operator access, since FEAT-31 owns that provisioning separately.

**FEAT-20.SPEC-001-AC-12:** Given a visitor arrived via a "Made with Clientroom" mark and completes the form, when the Freelancer Account is created, then `onboarding_referring_portal_ref` holds that reference on the account, the onboarding-progress state holds the FEAT-20.SPEC-005 defaults, and if Nadia closes the browser before reaching the "How did you hear" question the reference is still present on the account at her next sign-in.

**FEAT-20.SPEC-001-AC-13:** Given a visitor has typed values into the form, when she taps the "Terms of Service" or "Privacy Policy" link in the checkbox label, then that document opens in a new browser tab and, on returning to the form, every entered value and the checkbox state are unchanged.

**FEAT-20.SPEC-001-AC-14:** Given the Error banner is showing after a transient creation failure, when she taps its "Retry" button, then the same validated values (and the same referring-portal reference) are re-submitted, and on success she is taken into FEAT-20.SPEC-002 with `onboarding_started` emitted exactly once.

**FEAT-20.SPEC-001-AC-15:** Given a visitor is on the Sign-Up screen, when she taps "Already have an account? Sign in" in the footer, then she is taken to the sign-in screen, exactly as with the header "Sign in" link.

### User Story 2 - Onboarding Guided Sequence (Priority: P2)

The step-by-step shell that welcomes Nadia, asks the optional "how did you hear" question, hosts navigation into each guided step, shows progress, shows the Ready state (with a "Go to your dashboard" button) once onboarding's exit criteria are met, shows the welcome-email delivery warning if that email could not be delivered; Dana views the same progress read-only inside a logged support session.

**Acceptance Scenarios:**

**FEAT-20.SPEC-002-AC-01:** Given Nadia has just created her account, when the guided sequence first opens, then she sees the Welcome message and the "How did you hear about us?" question.

**FEAT-20.SPEC-002-AC-02:** Given Nadia is on the Welcome state, when she types an answer and taps Continue, then FEAT-20.SPEC-004 is triggered with her answer (onboarding_how_did_you_hear_resolved is set true by FEAT-20.SPEC-004, not by this screen) and she advances to the "Add first client and project" step.

**FEAT-20.SPEC-002-AC-03:** Given Nadia is on the Welcome state, when she taps Skip instead of answering, then FEAT-20.SPEC-004 is triggered with "unknown" as the answer and she advances to the next step.

**FEAT-20.SPEC-002-AC-04:** Given Nadia is on the "Add first client and project" step, when she taps the button, then she is taken to FEAT-01.SPEC-001 (Add Client), and no "Skip for now" link is shown for this mandatory step.

**FEAT-20.SPEC-002-AC-05:** Given Nadia completes adding a client and project and returns to this shell, when the return is processed, then `onboarding_step_completed` is emitted, FEAT-20.SPEC-003 re-checks exit criteria, and she advances to the "Set your branding" step.

**FEAT-20.SPEC-002-AC-06:** Given Nadia is on the "Set your branding" step, when she taps "Skip for now", then `onboarding_step_skipped` is emitted, the confirmation "You can set this up anytime from Settings." appears, and she advances to "Connect payments" with no penalty.

**FEAT-20.SPEC-002-AC-07:** Given Nadia is on the "Connect payments" step, when she taps "Skip for now", then `onboarding_step_skipped` is emitted and she advances to "Draft your first proposal".

**FEAT-20.SPEC-002-AC-08:** Given Nadia has added a first client and project, when she reaches the "Draft the first proposal" step and taps its button, then she is taken to FEAT-02.SPEC-001 for the new project.

**FEAT-20.SPEC-002-AC-09:** Given Nadia has a first client, project, and drafted proposal, when FEAT-20.SPEC-003 reports the exit criteria met, then this shell shows the Ready state with a "Go to your dashboard" button, does not route automatically, and renders no step controls.

**FEAT-20.SPEC-002-AC-10:** Given Nadia is on the Ready state, when she taps "Go to your dashboard", then onboarding_ready_acknowledged becomes true and she is routed to FEAT-12 (Freelancer Financial Dashboard).

**FEAT-20.SPEC-002-AC-11:** Given Nadia's branding logo upload fails inside the "Set branding" step, when she returns to this shell, then progression continues to the next step regardless, the step is marked "Finish later from Settings," and `onboarding_step_failed_continued` is emitted, per FEAT-20.SPEC-005 Rule R-04 for optional steps.

**FEAT-20.SPEC-002-AC-12:** Given Dana opens a read-only support session on Nadia's account, when she views this shell, then she sees the identical progress indicator and current step with no step-entry buttons, "Skip for now," or "continue anyway" controls rendered.

**FEAT-20.SPEC-002-AC-13:** Given Owen (Client Primary Contact) or Priya (Client Reviewer Contact) in a portal session attempts to reach this location, when the attempt is made, then he or she sees the page "This page isn't available" with "That page isn't part of your portal." and a "Back to your portal" button leading to their portal home, and no onboarding data is loaded.

**FEAT-20.SPEC-002-AC-14:** Given Nadia closes the browser mid-step and signs back in later, when this shell reopens, then it resumes on the exact step reflected by her onboarding-progress state.

**FEAT-20.SPEC-002-AC-15:** Given Nadia loses connectivity while this shell is open, when the loss occurs, then the banner "This step needs a connection." appears in the body panel; if she then opens a step that creates a record, that step's own screen shows its own offline behavior.

**FEAT-20.SPEC-002-AC-16:** Given Nadia navigates directly to the proposal-draft screen's URL before adding any client, when the screen loads, then it redirects her back to this shell's current step rather than opening in an invalid state.

**FEAT-20.SPEC-002-AC-17:** Given Nadia completes her first client, project, and proposal draft by navigating directly to each feature's screen rather than tapping this shell's own buttons, when she next opens this shell, then FEAT-20.SPEC-003 has already recorded completion (triggered by the record saves) and the Ready state is shown, provided she has not yet tapped "Go to your dashboard".

**FEAT-20.SPEC-002-AC-18:** Given an unauthenticated visitor attempts to reach this shell's URL, when the attempt is made, then she is redirected to the sign-in screen.

**FEAT-20.SPEC-002-AC-19:** Given the read of onboarding-progress state fails when this shell opens, when the failure occurs, then the banner "We couldn't load your setup progress." appears with a Retry button and no step panel is shown.

**FEAT-20.SPEC-002-AC-20:** Given Nadia saw the "How did you hear" question, closed the browser without answering, and signs in again, when the shell opens, then onboarding_how_did_you_hear_resolved is still false and the Welcome state is shown again; after she answers or skips, later opens never show it.

**FEAT-20.SPEC-002-AC-21:** Given onboarding is Complete and Nadia has already tapped "Go to your dashboard", when she opens this location by sign-in, bookmark, or the welcome email's "Continue setup" link, then she is redirected to FEAT-12 with no message and the Ready state is not shown.

**FEAT-20.SPEC-002-AC-22:** Given Nadia's project creation fails inside the "Add first client and project" step, when she returns to this shell, then the same step's panel shows "Not finished" with a "Try again" button and the message "That didn't finish saving. Try again when you're ready.", with no "Skip for now" or "continue anyway" control.

**FEAT-20.SPEC-002-AC-23:** Given Nadia saves her proposal draft (the final mandatory step) and FEAT-20.SPEC-003 cannot finish its check, when she next sees this shell, then the Completion-check notice "We couldn't confirm your setup just now." with a "Check again" button is shown, and tapping "Check again" (or the next shell open) re-runs the check and shows the Ready state once it succeeds.

**FEAT-20.SPEC-002-AC-24:** Given the welcome email's retries were exhausted without delivery and Nadia has not dismissed the warning, when she opens this shell in the In Progress, Welcome, or Ready state, then the warning "We couldn't deliver your welcome email to {sign_in_email}. If that address is wrong, you can correct it in Settings." is shown with "Go to Settings" and "Dismiss".

**FEAT-20.SPEC-002-AC-25:** Given the welcome-email warning is shown, when Nadia taps "Go to Settings" she is taken to FEAT-21.SPEC-005, and when she taps "Dismiss" onboarding_welcome_warning_dismissed becomes true and the warning is never shown again.

**FEAT-20.SPEC-002-AC-26:** Given Dana has no open support session on Nadia's account, when she reaches this location, then she sees "This page isn't available" with "Open a support session on a freelancer's account to view their setup progress." and a "Go to support access" button, and no freelancer data is loaded.

**FEAT-20.SPEC-002-AC-27:** Given onboarding is In Progress, when Nadia opens the dashboard or Settings directly, then it opens normally with no gate or redirect back to this shell, and her next sign-in lands on this shell at her current step.

### User Story 3 - Onboarding Completion Detection (Priority: P2)

Detects when a first client, project, and drafted proposal all exist for the Freelancer Account, marks onboarding complete, and signals FEAT-20.SPEC-002 to render the Ready state. This automation only signals; FEAT-20.SPEC-002 owns the Ready state and the route into the dashboard.

**Acceptance Scenarios:**

**FEAT-20.SPEC-003-AC-01:** Given Nadia has added a first client and project but has not yet drafted a proposal, when this automation evaluates the exit criteria, then onboarding is not marked complete and FEAT-20.SPEC-002 remains on its current step.

**FEAT-20.SPEC-003-AC-02:** Given Nadia has a first client, project, and drafted proposal, when this automation evaluates the exit criteria after the proposal draft is saved, then onboarding is marked complete, `onboarding_completed` is emitted, and FEAT-20.SPEC-002 shows the Ready state.

**FEAT-20.SPEC-003-AC-03:** Given onboarding is already marked complete, when this automation is triggered again (e.g., by an unrelated later action), then no action is taken and no duplicate `onboarding_completed` is emitted.

**FEAT-20.SPEC-003-AC-04:** Given Nadia skipped branding and payment connection entirely, when she completes the client, project, and proposal draft, then onboarding is marked complete regardless of the skipped steps.

**FEAT-20.SPEC-003-AC-05:** Given Nadia creates her first project by navigating directly to FEAT-01.SPEC-002 rather than through FEAT-20.SPEC-002's own button, when the project is created, then this automation still re-evaluates the exit criteria from that trigger.

**FEAT-20.SPEC-003-AC-06:** Given Nadia's drafted proposal is later voided and re-sent, when this automation is asked to re-evaluate at any later point, then onboarding remains complete -- the earlier draft already satisfied the criterion and completion is never reversed.

**FEAT-20.SPEC-003-AC-07:** Given Nadia completes her client/project step in one browser tab and drafts her proposal in another at effectively the same moment, when both triggers fire, then onboarding is marked complete exactly once, with no duplicate `onboarding_completed` event.

**FEAT-20.SPEC-003-AC-08:** Given a prior evaluation for Nadia's account is still updating onboarding-progress state, when a new trigger fires for the same account before that update finishes, then the new evaluation reads the just-updated state and correctly takes no further action.

**FEAT-20.SPEC-003-AC-09:** Given Nadia's Freelancer Account is deleted between a step completing and this automation's evaluation, when the automation attempts to read the account, then it takes no action and nothing is signaled to any shell.

**FEAT-20.SPEC-003-AC-10:** Given Nadia has a first client and project, when a Proposal for that project exists in any status of Draft or later, then the "drafted proposal" exit criterion is satisfied.

**FEAT-20.SPEC-003-AC-11:** Given the check for Nadia's final mandatory step could not read the proposal, when the failure occurs, then FEAT-20.SPEC-002 shows "We couldn't confirm your setup just now." with "Check again", and tapping it re-runs this automation and, once it succeeds, onboarding is marked complete and the Ready state is shown.

**FEAT-20.SPEC-003-AC-12:** Given a previous check failed and Nadia closes and reopens the shell, when the shell opens while onboarding is In Progress, then this automation re-runs on open and marks onboarding complete if all three records exist.

**FEAT-20.SPEC-003-AC-13:** Given onboarding completes, when the outcome is signalled, then FEAT-20.SPEC-002 renders the Ready state and this automation performs no navigation of its own.

**FEAT-20.SPEC-003-AC-14:** Given onboarding completes while onboarding_how_did_you_hear_resolved is false, when completion is recorded, then FEAT-20.SPEC-004 is triggered once with "unknown" and this spec does not write the field (FEAT-20.SPEC-004 sets it true after its hand-off attempt).

### User Story 4 - Referral Attribution Capture Hand-off (Priority: P2)

Captures Nadia's optional "How did you hear about us?" answer and the referring-portal reference, then hands both to Portal Referral Attribution (FEAT-33) for recording.

**Acceptance Scenarios:**

**FEAT-20.SPEC-004-AC-01:** Given Nadia types "A friend recommended it" and taps Continue on the "How did you hear" question, when this automation fires, then it hands off that answer to FEAT-33 for the Referral Attribution record.

**FEAT-20.SPEC-004-AC-02:** Given Nadia taps Skip without entering an answer, when this automation fires, then it hands off "unknown" as the self-reported source to FEAT-33.

**FEAT-20.SPEC-004-AC-03:** Given Nadia arrived by following a "Made with Clientroom" mark, when this automation fires (even in a later session than sign-up), then the referring-portal reference read from her account's onboarding_referring_portal_ref is included unchanged in the hand-off to FEAT-33.

**FEAT-20.SPEC-004-AC-04:** Given Nadia arrived without following any referral mark, when this automation fires, then the hand-off records the referring-portal reference as absent rather than substituting any inferred value.

**FEAT-20.SPEC-004-AC-05:** Given the hand-off to FEAT-33 fails, when the failure occurs, then Nadia's onboarding progression in FEAT-20.SPEC-002 continues unaffected and she is never re-asked the question.

**FEAT-20.SPEC-004-AC-06:** Given Nadia types only whitespace into the answer field before tapping Continue, when this automation fires, then the self-reported source is recorded as "unknown," identical to an explicit Skip.

**FEAT-20.SPEC-004-AC-07:** Given onboarding has already fired this hand-off once for Nadia's account, when any later action occurs, then this automation never fires a second time for that account.

**FEAT-20.SPEC-004-AC-08:** Given the hand-off succeeds, when it completes, then Nadia sees no confirmation of her own -- the hand-off is invisible, since her onboarding step already advanced in FEAT-20.SPEC-002.

**FEAT-20.SPEC-004-AC-09:** Given the hand-off fails for an account with a referring-portal reference present, when the failure occurs, then `referral_attribution_handoff_failed` is emitted noting a referring-portal reference was present, so the dropped attribution is observable.

**FEAT-20.SPEC-004-AC-10:** Given Nadia saw the "How did you hear" question, closed the browser without answering, and signed in again the next day, when she then answers, then this automation fires once and includes the referring-portal reference persisted at sign-up.

**FEAT-20.SPEC-004-AC-11:** Given onboarding completes while the question was never answered or skipped, when completion is recorded, then this automation fires once with "unknown" and the persisted reference.

**FEAT-20.SPEC-004-AC-12:** Given Nadia answers in one tab and skips in another at effectively the same moment, when both run, then FEAT-33 ends with exactly one Referral Attribution record for her account, only one run's compare-and-set changes onboarding_how_did_you_hear_resolved from false to true, and the other run does nothing further.

**FEAT-20.SPEC-004-AC-13:** Given Nadia answers or skips the question, when FEAT-20.SPEC-002 triggers this automation, then onboarding_how_did_you_hear_resolved is still false when the trigger arrives and becomes true only after this automation's hand-off attempt finishes, written by this automation and not by FEAT-20.SPEC-002.

**FEAT-20.SPEC-004-AC-14:** Given the hand-off to FEAT-33 fails, when this automation finishes, then onboarding_how_did_you_hear_resolved is true, the question is never shown again, and no retry of the hand-off occurs.

### User Story 5 - Onboarding Step Sequencing & Exit-Criteria Rules (Priority: P2)

Governs which onboarding steps are mandatory versus optional, the exit criteria for leaving onboarding, skip/resume behavior, and failure tolerance -- the single source of truth every other spec in this feature reads rather than duplicates.

**Acceptance Scenarios:**

**FEAT-20.SPEC-005-AC-01:** Given Nadia is on the "Add first client and project" step, when FEAT-20.SPEC-002 renders it, then no "Skip for now" control is shown, per Rule R-01.

**FEAT-20.SPEC-005-AC-02:** Given Nadia is on the "Set branding" step, when FEAT-20.SPEC-002 renders it, then a "Skip for now" control is shown, per Rule R-01.

**FEAT-20.SPEC-005-AC-03:** Given Nadia has a first Client, Project, and drafted Proposal, when FEAT-20.SPEC-003 evaluates the exit criteria, then onboarding_status transitions to Complete regardless of branding or payment-connection state, per Rule R-02.

**FEAT-20.SPEC-005-AC-04:** Given Nadia skips "Connect payments," when she later opens Settings & Account Management (FEAT-21) and connects her payment account there, then no penalty or re-entry into onboarding is required, per Rule R-03.

**FEAT-20.SPEC-005-AC-05:** Given Nadia's logo upload fails inside the "Set branding" step, when she returns to FEAT-20.SPEC-002, then progression continues to the next step and the branding step is recorded in onboarding_failed_incomplete_steps as completable later from Settings, per Rule R-04.

**FEAT-20.SPEC-005-AC-06:** Given onboarding_status has already transitioned to Complete, when Nadia's project is later archived, then onboarding_status remains Complete, per Rule R-05.

**FEAT-20.SPEC-005-AC-07:** Given Nadia has answered or skipped the "How did you hear" question, when any later session opens FEAT-20.SPEC-002, then the question is never shown again, per Rule R-06.

**FEAT-20.SPEC-005-AC-08:** Given Nadia (Freelancer) is viewing her own onboarding progress, when she attempts any step action while onboarding_status is In Progress, then the action is allowed.

**FEAT-20.SPEC-005-AC-09:** Given onboarding_status has already transitioned to Complete, when Nadia opens FEAT-20.SPEC-002's URL, then she sees the Ready state if onboarding_ready_acknowledged is false, or is redirected to the dashboard (FEAT-12) with no message if it is true; in neither case is any step control rendered.

**FEAT-20.SPEC-005-AC-10:** Given Dana opens a logged support session on Nadia's account, when she views onboarding progress, then she can see it but no step-entry, skip, or continue-past-failure control is available to her, per the Authorization Rules.

**FEAT-20.SPEC-005-AC-11:** Given Dana's support session on this account is not open, when she attempts to reach the freelancer-side onboarding location, then she sees the page "This page isn't available" with "Open a support session on a freelancer's account to view their setup progress." and a "Go to support access" button, and no freelancer data is loaded.

**FEAT-20.SPEC-005-AC-12:** Given Owen (Client Primary Contact) or Priya (Client Reviewer Contact) in a portal session opens the freelancer-side onboarding location, then the page "This page isn't available" with "That page isn't part of your portal." and a "Back to your portal" button appears, and no onboarding data is loaded.

**FEAT-20.SPEC-005-AC-13:** Given a new Freelancer Account is created, when its onboarding-progress state is initialized, then onboarding_current_step defaults to "How did you hear," onboarding_status defaults to In Progress, and every set field defaults empty, with FEAT-20.SPEC-001 performing the initialisation in the same step that creates the account, per the Defaults and Derivations table.

**FEAT-20.SPEC-005-AC-14:** Given onboarding_status transitions to Complete, when the transition occurs, then onboarding_completed_at is set once to that moment and is never subsequently changed.

**FEAT-20.SPEC-005-AC-15:** Given Nadia's project creation fails inside the "Add first client and project" step, when she returns to FEAT-20.SPEC-002, then the step stays current marked "Not finished" with a "Try again" action, no "continue anyway" is offered, nothing is added to onboarding_failed_incomplete_steps, and onboarding_status stays In Progress, per Rule R-04.

**FEAT-20.SPEC-005-AC-16:** Given Nadia opens the "How did you hear" question and closes the browser without answering, when she signs in later, then onboarding_how_did_you_hear_resolved is still false and the question is shown again, per Rule R-06.

**FEAT-20.SPEC-005-AC-17:** Given Nadia answers or skips the question, when the answer or skip is saved, then FEAT-20.SPEC-002 only triggers FEAT-20.SPEC-004, and FEAT-20.SPEC-004 alone sets onboarding_how_did_you_hear_resolved true after its hand-off attempt (true even if the hand-off fails) and not before, per Rule R-06.

**FEAT-20.SPEC-005-AC-18:** Given a new Freelancer Account arrived via a FEAT-33 mark, when it is created, then onboarding_referring_portal_ref holds that reference and never changes afterward.

**FEAT-20.SPEC-005-AC-19:** Given onboarding is In Progress, when Nadia opens the dashboard or any other feature directly, then it opens normally with no gate, redirect, or hidden content, and her next sign-in lands on FEAT-20.SPEC-002 at her current step, per Rule R-08.

**FEAT-20.SPEC-005-AC-20:** Given onboarding_status becomes Complete while onboarding_how_did_you_hear_resolved is false, when the transition occurs, then the account is auto-resolved as "unknown", the FEAT-20.SPEC-004 hand-off fires once with the persisted referring-portal reference, and the question is never shown, per Rule R-09.

**FEAT-20.SPEC-005-AC-21:** Given onboarding_status is Complete and Nadia has tapped "Go to your dashboard", when she later opens FEAT-20.SPEC-002's URL or signs in, then she lands on the dashboard (FEAT-12) and the Ready state is not shown again, per Rule R-08.

### User Story 6 - Welcome Email (Priority: P2)

Confirms to Nadia that her Freelancer Account was created, using the transactional email delivery capability, so she has a record of her sign-up separate from the in-product session.

**Acceptance Scenarios:**

**FEAT-20.SPEC-006-AC-01:** Given Nadia successfully creates her Freelancer Account, when the account is created, then the welcome email is sent once to her sign-in email address with the subject "Welcome to Clientroom, {freelancer_name}."

**FEAT-20.SPEC-006-AC-02:** Given Nadia receives the welcome email, when she taps "Continue setup," then she lands on FEAT-20.SPEC-002 (Onboarding Guided Sequence) at her current onboarding step.

**FEAT-20.SPEC-006-AC-03:** Given Nadia's sign-in email address does not exist, when delivery is attempted, then it is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window` before a delivery warning appears to her inside the guided sequence.

**FEAT-20.SPEC-006-AC-04:** Given Nadia's Freelancer Account is deleted before this email is delivered, when the deletion completes, then delivery of this email is cancelled.

**FEAT-20.SPEC-006-AC-05:** Given Nadia already completed onboarding by the time this email is delivered, when she taps "Continue setup," then she lands on the Ready state if she has not yet tapped "Go to your dashboard", or is redirected to her dashboard (FEAT-12) if she has, rather than mid-sequence, per FEAT-20.SPEC-002.

**FEAT-20.SPEC-006-AC-06:** Given this feature defines no preference control for this email, when Nadia looks in her notification preferences (FEAT-21), then no toggle for this email exists there -- it always sends.

**FEAT-20.SPEC-006-AC-07:** Given Nadia's account creation attempt fails validation and no account is created, then no welcome email is sent, since the trigger fires only on a successful, committed account creation.

**FEAT-20.SPEC-006-AC-08:** Given delivery succeeds on the first attempt, when it is delivered, then `welcome_email_delivered` is emitted with outcome "delivered."

**FEAT-20.SPEC-006-AC-09:** Given all retries are exhausted with no successful delivery, when the final retry fails, then the next time Nadia opens FEAT-20.SPEC-002 the warning is shown in its notice area with "Go to Settings" and "Dismiss", and `welcome_email_delivery_warning_shown` is emitted at that display.

### Edge Cases

- **FEAT-20.SPEC-001 (Sign-Up & Account Creation):** Two visitors submitting the same email at once are resolved by an authoritative uniqueness re-check at commit (first wins, the second is rejected), navigating away needs no confirmation since no record exists, and double taps are ignored. If creation succeeds but navigation is interrupted, the next sign-in resumes at the current onboarding step. Source: `docs/blueprint/specifications/FEAT-20-onboarding-first-run-setup/FEAT-20.SPEC-001-sign-up-account-creation.md` (section: Edge Cases)
- **FEAT-20.SPEC-002 (Onboarding Guided Sequence):** Closing the browser resumes exactly where she left off with completed steps not re-shown as incomplete, and jumping to a step out of order redirects back to the shell when its prerequisite is missing. An unanswered How did you hear question shows Welcome again with the persisted referral reference unaffected, and a skipped branding step is not revisited even if set later from Settings. Source: `docs/blueprint/specifications/FEAT-20-onboarding-first-run-setup/FEAT-20.SPEC-002-onboarding-guided-sequence.md` (section: Edge Cases)
- **FEAT-20.SPEC-003 (Onboarding Completion Detection):** Exit criteria evaluate historical existence, so archiving a project or later voiding and re-sending a proposal never un-completes onboarding. A client without a project leaves onboarding in progress, and concurrent step completions in two tabs each read current state independently. Source: `docs/blueprint/specifications/FEAT-20-onboarding-first-run-setup/FEAT-20.SPEC-003-onboarding-completion-detection.md` (section: Edge Cases)
- **FEAT-20.SPEC-004 (Referral Attribution Capture Hand-off):** A failed hand-off to FEAT-33 is not retried and onboarding continues with the source never recorded for that account. Empty free text after trimming is treated as an explicit Skip (unknown), a later deletion of the referring account is FEAT-33's concern, and an unanswered question leaves the reference intact until she answers or skips. Source: `docs/blueprint/specifications/FEAT-20-onboarding-first-run-setup/FEAT-20.SPEC-004-referral-attribution-capture-hand-off.md` (section: Edge Cases)
- **FEAT-20.SPEC-005 (Onboarding Step Sequencing & Exit-Criteria Rules):** A step skipped then completed later from Settings keeps its recorded skip, a skipped step cannot also be failed, and a failed step never gates exit criteria so completion proceeds normally. The mandatory Add first client and project step has no skip control, so skipping it cannot arise. Source: `docs/blueprint/specifications/FEAT-20-onboarding-first-run-setup/FEAT-20.SPEC-005-onboarding-step-sequencing-exit-criteria-rules.md` (section: Edge Cases)
- **FEAT-20.SPEC-006 (Welcome Email):** A mistyped or nonexistent address bounces, retries to the standard count and window, then surfaces a delivery warning in the guided sequence. Deletion before delivery cancels the email, and the email carries no preference or quiet-hours behavior. Source: `docs/blueprint/specifications/FEAT-20-onboarding-first-run-setup/FEAT-20.SPEC-006-welcome-email.md` (section: Edge Cases)

## Requirements (mandatory)

### Functional Requirements

- **FR-001**: The system MUST implement **FEAT-20.SPEC-001** (Sign-Up & Account Creation) as specified: A new freelancer creates her Freelancer Account, which is created and immediately transitioned to Active state, starting the guided onboarding sequence. Full spec: `docs/blueprint/specifications/FEAT-20-onboarding-first-run-setup/FEAT-20.SPEC-001-sign-up-account-creation.md`
- **FR-002**: The system MUST implement **FEAT-20.SPEC-002** (Onboarding Guided Sequence) as specified: The step-by-step shell that welcomes Nadia, asks the optional "how did you hear" question, hosts navigation into each guided step, shows progress, shows the Ready state (with a "Go to your dashboard" button) once onboarding's exit criteria are met, shows the welcome-email delivery warning if that email could not be delivered; Dana views the same progress read-only inside a logged support session. Full spec: `docs/blueprint/specifications/FEAT-20-onboarding-first-run-setup/FEAT-20.SPEC-002-onboarding-guided-sequence.md`
- **FR-003**: The system MUST implement **FEAT-20.SPEC-003** (Onboarding Completion Detection) as specified: Detects when a first client, project, and drafted proposal all exist for the Freelancer Account, marks onboarding complete, and signals FEAT-20.SPEC-002 to render the Ready state. This automation only signals; FEAT-20.SPEC-002 owns the Ready state and the route into the dashboard. Full spec: `docs/blueprint/specifications/FEAT-20-onboarding-first-run-setup/FEAT-20.SPEC-003-onboarding-completion-detection.md`
- **FR-004**: The system MUST implement **FEAT-20.SPEC-004** (Referral Attribution Capture Hand-off) as specified: Captures Nadia's optional "How did you hear about us?" answer and the referring-portal reference, then hands both to Portal Referral Attribution (FEAT-33) for recording. Full spec: `docs/blueprint/specifications/FEAT-20-onboarding-first-run-setup/FEAT-20.SPEC-004-referral-attribution-capture-hand-off.md`
- **FR-005**: The system MUST implement **FEAT-20.SPEC-005** (Onboarding Step Sequencing & Exit-Criteria Rules) as specified: Governs which onboarding steps are mandatory versus optional, the exit criteria for leaving onboarding, skip/resume behavior, and failure tolerance -- the single source of truth every other spec in this feature reads rather than duplicates. Full spec: `docs/blueprint/specifications/FEAT-20-onboarding-first-run-setup/FEAT-20.SPEC-005-onboarding-step-sequencing-exit-criteria-rules.md`
- **FR-006**: The system MUST implement **FEAT-20.SPEC-006** (Welcome Email) as specified: Confirms to Nadia that her Freelancer Account was created, using the transactional email delivery capability, so she has a record of her sign-up separate from the in-product session. Full spec: `docs/blueprint/specifications/FEAT-20-onboarding-first-run-setup/FEAT-20.SPEC-006-welcome-email.md`

### Key Entities

- Freelancer Account (create) [AUDIT-ADDED: 3 -- inverse check: sign-up creates the account]; otherwise onboarding orchestrates existing entities rather than owning one of its own.

## Success Criteria (mandatory)

### Measurable Outcomes

- **SC-001**: At least 70% of new freelancers have a first client, project, and drafted proposal within their first session, and the median time from sign-up to that point is under 15 minutes (metric: First-Session Activation). Source: `docs/blueprint/features/success-metrics.md`
- **SC-002**: Onboarding starts, completed steps, completions and skipped steps are each observable as distinct signals (onboarding_started, onboarding_step_completed, onboarding_completed, onboarding_step_skipped). Source: `docs/blueprint/features/product-features.md`

## Assumptions

- **ASMP-08**: Freelancers are assumed to prefer fixed, sensible behavior over configuring their own workflows, matching the short guided sequence. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-01**: Freelancers work primarily from a laptop or desktop with reliable internet access. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-29**: Transactional email delivery capability is required for the welcome email. Full register: `docs/blueprint/features/assumptions-constraints.md`

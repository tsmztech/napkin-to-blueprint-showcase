---
document_type: spec
spec_type: screen
spec_id: FEAT-20.SPEC-001
spec_name: Sign-Up & Account Creation
spec_slug: sign-up-account-creation
parent_feature: FEAT-20
parent_feature_name: Onboarding / First-Run Setup
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 15
---

# Screen Spec: Sign-Up & Account Creation

## Overview

**Name:** Sign-Up & Account Creation
**ID:** FEAT-20.SPEC-001
**Type:** Screen
**Purpose:** A new freelancer creates her Freelancer Account, which is created and immediately transitioned to Active state, starting the guided onboarding sequence.
**Parent Feature:** FEAT-20 -- Onboarding / First-Run Setup

## Scope and Non-Goals

**In Scope:**
- Capturing the minimum data needed to create a Freelancer Account: name, sign-in email, and a sign-in credential
- Carrying forward the referring-portal reference when the visitor arrived via a "Made with Clientroom" mark (FEAT-33), and persisting it on the new Freelancer Account's onboarding-progress state at creation so it survives across sessions
- Initialising the Freelancer Account's onboarding-progress state to the defaults defined in FEAT-20.SPEC-005 (Defaults and Derivations) at the moment of creation
- Creating the Freelancer Account and transitioning it to Active state on successful submission
- Emitting `onboarding_started` and handing off into the guided sequence (FEAT-20.SPEC-002)
- Preserving entered data and offering retry on a failed save, including while offline

**Non-Goals:**
- Capturing business details, payment terms, or a time zone -- those fields belong to the Freelancer Account's later completion in Settings & Account Management (FEAT-21), not to sign-up; onboarding creates the record with only its two required fields (name, sign-in email) and reads it thereafter
- Asking "How did you hear about us?" -- that optional question is asked inside the guided sequence (FEAT-20.SPEC-002), not on this screen, per the Brief's Side-Effect Inventory
- Client-contact sign-in -- Owen and Priya reach their portals through a magic link (FEAT-05.SPEC-002), never through this form; this screen exists only to create a new Freelancer Account
- A separate email-verification gate before the account becomes Active -- per the Entity-Lifecycle Coverage Matrix, "Created → Active happens immediately on successful sign-up, with no separate verification gate for initial creation"; only a later sign-in email *change* (FEAT-21) requires re-verification

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-33 (Portal Referral Attribution) "Made with Clientroom" mark | Visitor follows the mark on a client-facing portal page or email and chooses to sign up | The referring Freelancer Account reference, carried silently through the form to account creation, where it is persisted on the account's onboarding-progress state and later handed off by FEAT-20.SPEC-004 |
| External marketing entry (organic, direct, search) | Visitor reaches the product's public sign-up entry with no referral context | None -- referring-portal reference is absent; FEAT-20.SPEC-004 records the source as unknown unless answered later |

## Access and Visibility

This screen has no signed-in state of its own -- it is the entry point that creates a session, so "Can Act" describes who may complete it rather than a permission grant on an existing record.

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|--------------------------|
| Unauthenticated visitor | Yes | Yes -- may create a new Freelancer Account | -- |
| Nadia (Freelancer, already signed in) | Yes, briefly | No -- an already-authenticated freelancer is redirected before the form renders | Redirected to her post-sign-in landing destination (FEAT-20.SPEC-005 Rule R-08: her current onboarding step in FEAT-20.SPEC-002 while onboarding is In Progress, otherwise the dashboard, FEAT-12) with no message needed, since nothing was denied -- she already has an account |
| Owen (Client Primary Contact, signed in to a portal session) | Yes | Yes -- may create a separate, unrelated Freelancer Account of her own (the referral growth path in FEAT-33's happy path: "a client contact who is also a freelancer... follows the mark -> signs up"); doing so never affects his existing portal access | -- |
| Priya (Client Reviewer Contact, signed in to a portal session) | Yes | Yes -- identical to Owen's case above | -- |
| Dana (Support Operator) | Yes | No -- the operator identity is provisioned separately from any product sign-up path, per BRIEF.md's "not a product role" | This screen has no awareness of the operator identity; completing it would simply create an ordinary Freelancer Account, which is never how Dana's access is granted (FEAT-31 owns operator provisioning) |
| Expired session | Yes | Yes -- this screen requires no existing session, so an expired session has no effect on reaching or completing it | -- |

## Layout and Content

**Header:** Product wordmark, and a "Sign in" link (for an existing freelancer who reached this page by mistake) at the top right.

**Body:** A single-column form with the following fields, in order:
- Full name (text input, required)
- Sign-in email (text input, required)
- Password (masked text input, required)
- Confirm password (masked text input, required)
- Terms of Service and Privacy acknowledgment (checkbox, required; the label contains two inline links, "Terms of Service" and "Privacy Policy", each opening its document)
- "Create my account" button, below the form fields, full width

All fields use one consistent input treatment platform-wide. Required-field indicators are shown on every field, since every field here is required.

**Footer:** A single line: "Already have an account? Sign in" linking to the sign-in screen.

### Responsive Behavior

- **Compact breakpoint:** Single-column form, full width, as described above; the "Create my account" button remains directly below the form fields.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|---------------|----------|
| Full name input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Full name input | Blur (empty) | Triggers field validation | Error state on field | "Your name is required" below field |
| Sign-in email input | Blur | Validates email format and uniqueness | Error state if invalid or already registered | "Enter a valid email address" or "An account already exists for this email. Sign in instead." with a link to sign-in |
| Password input | Type | Captures input, evaluates minimum strength | Field shows entered characters (masked) | Inline strength/requirement hint below the field |
| Confirm password input | Blur | Compares against Password value | Error state if mismatched | "Passwords don't match" |
| Terms checkbox | Tap | Toggles acknowledgment | Checkbox shows checked/unchecked | -- |
| "Sign in" link (header) | Tap | Navigate to the sign-in screen | Screen closes; entered values are discarded, with no confirmation (no record exists yet) | Standard navigation transition |
| "Sign in" link (footer, "Already have an account? Sign in") | Tap | Navigate to the sign-in screen (identical to the header link) | Screen closes; entered values are discarded, with no confirmation | Standard navigation transition |
| "Terms of Service" link (inside the checkbox label) | Tap | Opens the Terms of Service document in a new browser tab, leaving this form untouched | No change to any field or to the checkbox state | The document opens in the new tab; closing it returns Nadia to the form with every entered value intact |
| "Privacy Policy" link (inside the checkbox label) | Tap | Opens the Privacy Policy document in a new browser tab, leaving this form untouched | No change to any field or to the checkbox state | Same as the Terms of Service link |
| Error banner "Retry" button | Tap | Re-submits the same validated field values through steps 2-4 of the "Create my account" action (create the Freelancer Account, emit `onboarding_started`, navigate to FEAT-20.SPEC-002); the referring-portal reference is re-sent unchanged | Banner is replaced by the Creating state (button loading, fields disabled) | Success: navigates into FEAT-20.SPEC-002. Failure: the banner returns with the failure message and the Retry button |
| "Create my account" button | Tap | 1. Validate all fields. 2. If valid, create the Freelancer Account and transition it to Active (which fires FEAT-20.SPEC-006, Welcome Email, independently of this screen's navigation). 3. Emit `onboarding_started`. 4. Navigate to FEAT-20.SPEC-002. | Button shows loading state during creation | Success: navigates directly into FEAT-20.SPEC-002 (Onboarding Guided Sequence), which opens on its Welcome state. Failure: inline error messages; button returns to normal state. |
| "Create my account" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Full name -> Sign-in email -> Password -> Confirm password -> Terms checkbox -> Terms of Service link -> Privacy Policy link -> Create my account -> Sign in (footer link). When the Error banner is showing, focus moves to the banner and its Retry button precedes the first field.
- **Validation announcements:** When a field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Creation feedback:** On failure, an error banner is announced and focus moves to the first field in error (or to the banner itself for a non-field error such as a connectivity failure).
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-------------------|------------------|
| Empty (default) | All form fields empty, "Create my account" enabled | Screen first opens | User begins typing in any field |
| Filling | Form fields contain user input | User types in any field | User taps "Create my account" or navigates away |
| Validation Error | Failed fields highlighted with error messages below them | Client-side validation fails on blur or submit | User corrects the field and re-triggers validation |
| Creating | Button shows loading state, form fields disabled | Validation passes and account creation begins | Creation completes or fails |
| Error | Error banner at top of form with a retry option | Account creation fails (e.g., a transient failure after validation passed) | User taps Retry or corrects the field named in the error and resubmits |
| Offline/Degraded | Banner "This step needs a connection. Your details are saved here and will be submitted once you're back online." at the top; form remains editable and entered values are preserved | Connectivity is lost while the screen is open or while creation is in progress | Connectivity is restored -- the user taps "Create my account" again to submit (no silent background retry, since account creation must never appear to succeed without the freelancer's own confirming action) |

## Validation Rules

| Field | Condition | When Checked | Error Message |
|-------|-----------|-----------------|------------------|
| Full name | Required, non-empty, max 100 characters | On blur | "Your name is required" / "Name must be 100 characters or fewer" |
| Sign-in email | Required; valid email format; must not already belong to a Freelancer Account | On blur and on submit | "Enter a valid email address" / "An account already exists for this email. Sign in instead." |
| Password | Required; minimum length and a mix of character types | On blur | "Choose a stronger password" (with an inline hint of the specific unmet requirement) |
| Confirm password | Must exactly match Password | On blur and on submit | "Passwords don't match" |
| Terms acknowledgment | Must be checked | On submit | "You must accept the Terms of Service and Privacy Policy to continue" |

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|---------------------------------------|
| "Sign in" link (header or footer) tap | Sign-in screen | -- (outside this feature's scope; owned by the product's baseline authentication surface) |
| "Terms of Service" link tap | Terms of Service document, in a new browser tab | -- (the product's legal-document surface, outside this feature's scope) |
| "Privacy Policy" link tap | Privacy Policy document, in a new browser tab | -- (the product's legal-document surface, outside this feature's scope) |
| Error banner "Retry" success | FEAT-20.SPEC-002 (Onboarding Guided Sequence) | -- |
| Successful account creation | FEAT-20.SPEC-002 (Onboarding Guided Sequence) | -- |

## Data Model

**Creates:** Freelancer Account -- `name` and sign-in `email` set from form input; a sign-in credential is set (never displayed back to the freelancer or visible to the operator); `status` set to Active immediately, with no intermediate unverified state. In the same atomic step, the account's onboarding-progress state is initialised to the FEAT-20.SPEC-005 defaults (current step "How did you hear", status In Progress, empty completed/skipped/failed sets, question unresolved, Ready screen unacknowledged, welcome-email warning not dismissed) and `onboarding_referring_portal_ref` is set to the referring-portal reference carried from a FEAT-33 entry, or to absent when there was none. All other Freelancer Account fields (business details, payment terms, time zone, notification preferences) remain unset at this point and are completed later through Settings & Account Management (FEAT-21).
**Reads:** None -- this is a creation screen.
**Updates:** None.
**Deletes:** None.

## Business Rules

- Account creation and the Active state transition happen as one atomic step -- there is no intermediate "pending" Freelancer Account visible anywhere in the product.
- The referring-portal reference carried from a FEAT-33 entry is retained in the form's context through any creation retry, and is persisted on the new Freelancer Account's onboarding-progress state (`onboarding_referring_portal_ref`) as part of the same atomic creation. From that point it lives on the account, not in the browser session, so closing the browser before the "How did you hear" moment never loses it; FEAT-20.SPEC-004 reads it from the account when Nadia answers or skips (or when onboarding completes first).
- This screen is the sole initialiser of the onboarding-progress defaults: the defaults in FEAT-20.SPEC-005 (Defaults and Derivations) are applied here, at creation, and nowhere else.
- `onboarding_started` is emitted exactly once, at the moment the Freelancer Account is created -- never re-emitted on a later visit to onboarding.
- On successful creation, the welcome email (FEAT-20.SPEC-006) is triggered independently of this screen's own navigation -- a slow or failed email send never blocks or delays entry into FEAT-20.SPEC-002.

## Edge Cases

- **Two visitors submit the same email at effectively the same time** -- The email-uniqueness check is re-verified authoritatively at the moment of commit; the first submission to commit succeeds, and the second is rejected with "An account already exists for this email. Sign in instead." even if both passed the earlier blur-time check.
- **User navigates away mid-form** -- No confirmation dialog is shown (no record exists yet to lose); returning to the sign-up entry point starts with an empty form.
- **User taps "Create my account" twice rapidly** -- Second tap is ignored while the first creation attempt is in progress (button in loading state).
- **Creation succeeds but the subsequent navigation is interrupted (e.g., the browser closes)** -- The Freelancer Account already exists and is Active; the freelancer's next sign-in resumes directly at her current onboarding step (FEAT-20.SPEC-002), which reads onboarding-progress state from the account rather than assuming this screen's navigation completed.
- **User arrives here already signed in** -- Handled by the Access and Visibility redirect above; no form is shown.
- **Connectivity is lost after the user has typed but before submitting** -- The Offline/Degraded state applies; typed values are preserved and the user submits once reconnected, with no automatic background submission.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|--------------------|------------------|
| FEAT-20.SPEC-002 (Onboarding Guided Sequence) | Navigation (outbound) | Successful account creation navigates directly into the guided sequence's Welcome state |
| FEAT-20.SPEC-006 (Welcome Email) | Triggers (outbound) | Account creation triggers the welcome email confirming account creation |
| FEAT-20.SPEC-004 (Referral Attribution Capture Hand-off) | References (outbound) | The referring-portal reference carried from a FEAT-33 entry is passed through this screen for later hand-off |
| FEAT-33 (Portal Referral Attribution) | Navigation (inbound) | A visitor following the "Made with Clientroom" mark arrives here with the referring portal known |
| FEAT-24 (Data Export & Account Deletion) | References (inbound) | The Freelancer Account created here is the same record FEAT-24 later deletes, ending any onboarding-progress state with it |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|-------------------|----------------------|
| onboarding_started | referring-portal reference present (yes/no) | Freelancer Account is created and transitioned to Active | supports success-metrics.md: "First-Session Activation" |
| account_creation_failed | failure reason (validation / duplicate-email / connectivity) | Creation attempt does not result in an Active Freelancer Account | N/A -- no Stage 2 metric measures failed sign-up attempts directly; retained so first-session friction is observable rather than invisible |

## Acceptance Criteria

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

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|----------------|-------|
| Interactions | 13 | 13 |
| States | 6 (empty, filling, validation error, creating, error, offline) | 6 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

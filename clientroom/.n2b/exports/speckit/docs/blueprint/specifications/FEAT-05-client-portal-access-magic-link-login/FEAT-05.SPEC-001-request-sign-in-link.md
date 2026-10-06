---
document_type: spec
spec_type: screen
spec_id: FEAT-05.SPEC-001
spec_name: Request Sign-In Link
spec_slug: request-sign-in-link
parent_feature: FEAT-05
parent_feature_name: Client Portal Access (Magic-Link Login)
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Screen Spec: Request Sign-In Link

## Overview

**Name:** Request Sign-In Link
**ID:** FEAT-05.SPEC-001
**Type:** Screen
**Purpose:** A client contact enters their email and requests a one-time sign-in link, without ever seeing an account-creation step.
**Parent Feature:** FEAT-05 -- Client Portal Access (Magic-Link Login)

## Scope and Non-Goals

**In Scope:**
- Collecting the contact's email address
- Submitting the request and showing a same-screen confirmation
- Re-entry point for a fresh request after an expired or invalid link (FEAT-05.SPEC-002)
- Neutral confirmation messaging regardless of whether the email matched a recognized contact, so an unrecognized email is never distinguishable from a recognized one

**Non-Goals:**
- Account creation, password entry, or any credential set-up -- excluded per BRIEF.md, Target Users & Roles: contacts "never hit an account-creation wall"; magic-link email is this feature's only sign-in path.
- Determining whether the submitted email is recognized, generating the link, and enforcing single-use/time-limit rules -- owned by FEAT-05.SPEC-004 (Magic Link Issuance) and FEAT-05.SPEC-006 (Link Validity & Recognition Rules); this screen only collects the email and displays the confirmation.
- Client-side roles beyond Primary and Reviewer -- excluded per scope-boundaries.md SC-02: the product models exactly Owen (Primary) and Priya (Reviewer); no further client-side tier exists to select on this screen.
- Native mobile app entry -- excluded per scope-boundaries.md SC-06: this is a web screen made excellent on mobile browsers, not an app-based sign-in flow.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| External (FEAT-33, public product page) | A returning contact with no live session reaches the portal from the "Made with Clientroom" referral mark or the public product page | None -- form starts empty |
| Direct arrival (no active session) | Contact opens the shared portal address directly, or a session has lapsed | None -- form starts empty |
| FEAT-05.SPEC-002 (Link Verification Landing) | Contact taps "Request a new link" on an expired, invalid, or out-of-scope link | Email pre-filled when it was known from the expired link's context; otherwise empty |

**Default Entry:** This is the feature's Default Entry screen -- the screen shown to any contact who arrives with no active session, per the Feature Breakdown Brief's Internal Dependency Map.

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Owen (Client Primary Contact) | Full screen | Submit a sign-in request for their own email | -- |
| Priya (Client Reviewer Contact) | Full screen | Submit a sign-in request for their own email | -- |
| Nadia (Freelancer) | Full screen (this is a public-facing entry point, not a freelancer surface) | No portal-specific action -- Nadia signs in through her own freelancer account, not through this screen | This screen never authenticates Nadia as a freelancer; submitting her own email here is treated the same as any email (an unrecognized-as-client-contact address), and she sees the same neutral confirmation with no link ever issued |
| Dana (Support Operator) | Full screen (public entry point) | None -- Dana never signs in as a client contact (SC-04) | Same neutral confirmation as any submission; no link is ever issued to Dana because she holds no Client Contact record |
| Unauthenticated | Yes -- this screen requires no authentication by definition | Yes -- submitting the request is the unauthenticated action this screen exists for | -- |
| Expired session | Yes -- a contact whose session lapsed is routed here | Yes | The contact is returned here automatically from any portal page once their session lapses (per the Default Entry note in the Feature Breakdown Brief); no error dialog appears, since arriving here is the expected recovery path |

## Layout and Content

**Header:** The owning freelancer's Branding Profile logo (or the neutral default when unset) at the top, centered, per XBR-31. Below it, the screen title "Sign in to your client portal."

**Body:** A single-column form containing:
- Email (text input, required, email-format keyboard on mobile)
- "Send sign-in link" (primary action button, full width)
- A single line of explanatory text below the title: "We'll email you a link. No password needed."

**Footer:** The discreet "Made with Clientroom" referral mark (FEAT-33), positioned below the form, never overriding the freelancer's branding (XBR-31, XBR-32).

### Responsive Behavior

- **Compact breakpoint (phone width):** Single-column form, full width with the platform's standard side gutter; button spans the form width; logo scales to a size that keeps the header short enough that the email field is visible without scrolling.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Email input | Type | Captures the entered email | Field shows entered text | Standard input focus state |
| Email input | Blur (empty or malformed) | Validates the email format per FEAT-05.SPEC-006 | Field shows error state if invalid | "Enter a valid email address" below the field |
| "Send sign-in link" button | Tap | Submits the email to FEAT-05.SPEC-004 (Magic Link Issuance) | Button shows loading state during submission | On completion, the screen replaces the form with the confirmation state (see States) |
| "Send sign-in link" button (while loading) | Tap | No action -- debounced against double-submit | None | Button remains in loading state |
| Confirmation state's "Try a different email" link | Tap | Returns the form to its empty, editable state | Confirmation replaced by the empty form | Form reappears with focus on the email field |
| "Made with Clientroom" referral mark | Tap | Navigates externally to the public product page (FEAT-33); does not affect this screen's own state | This screen's state is unchanged | Standard external navigation |

### Accessibility Notes

- **Focus order:** Logo (skippable, not focusable) -> title -> email input -> "Send sign-in link" button -> referral mark link.
- **Validation announcements:** The email-format error is announced to assistive technology and programmatically associated with the field when it appears on blur.
- **Confirmation announcement:** When the confirmation state replaces the form, its heading is announced so a screen-reader user is told the request was received.
- **Keyboard alternatives:** Every action on this screen (submit, "Try a different email") is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Empty (default) | Empty email field, "Send sign-in link" enabled once the field has content | Screen first opens | User submits a request |
| Filling | Email field contains user input | User types in the field | User submits or navigates away |
| Submitting | Button shows loading state, field disabled | User taps "Send sign-in link" with a validly formatted email | Submission completes |
| Confirmation | Form replaced by: "Check your email. If {email} matches an account, we've sent a sign-in link to it." with a "Try a different email" link | Submission completes, regardless of whether the email matched a recognized contact | User taps "Try a different email," or navigates away |
| Validation Error | Email field shows error state and message | Blur or submit with an invalid email format | User corrects the field |
| Offline/Degraded | Banner "You're offline. Connect to the internet to request a sign-in link." above the form; the "Send sign-in link" button is disabled while offline | Connectivity is lost while the screen is open, or the screen loads without connectivity | Connectivity is restored -- banner clears and the button re-enables; no request is queued, since a stale request would defeat the single-use/time-limited link model (FEAT-05.SPEC-006) |

## Validation Rules

Validation governed by FEAT-05.SPEC-006 (Link Validity & Recognition Rules). See that spec for the email-format rule and for why recognition itself is never disclosed on this screen. This screen applies format validation on field blur and on form submit.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Contact clicks the emailed link (external, not a navigation from this screen) | FEAT-05.SPEC-002 (Link Verification Landing) | -- |

<!-- This screen has no in-screen navigation action -- the only path forward is through the emailed link, which the contact opens from their email client, not from a control on this screen. -->

## Data Model

**Creates:** None directly -- the submitted email is handed to FEAT-05.SPEC-004 (Magic Link Issuance), which performs the recognition lookup and any resulting write.
**Reads:** Branding Profile -- logo and brand_colour, to render the header per the owning freelancer (resolved from the portal address the contact arrived at).
**Updates:** None.
**Deletes:** None.

## Business Rules

- The confirmation message is identical whether or not the submitted email matches a recognized Client Contact (FEAT-05.SPEC-006) -- this screen never reveals which emails are recognized, so it cannot be used to enumerate a freelancer's clients.
- XBR-28: only contacts added through Client Contact Management & Roles (FEAT-18) are ever issued a link; this screen accepts any email as input but the issuance decision belongs entirely to FEAT-05.SPEC-004 and FEAT-05.SPEC-006.
- Double-submission is prevented while a request is in flight (button loading state); a second request for the same email after the first completes is a legitimate re-request and is not blocked by this screen (FEAT-05.SPEC-006 governs invalidation of the prior link).

## Edge Cases

- **User submits the same email twice in quick succession, each completing separately** -- Each submission is treated as an independent re-request; FEAT-05.SPEC-006's invalidate-prior-link rule ensures only the most recently issued link remains usable. The screen shows the same confirmation both times.
- **User navigates away during "Submitting" and returns** -- The screen reloads to its empty, default state; whether the in-flight request completed does not change what this screen shows, since it never discloses recognition status.
- **User pastes an email with leading/trailing whitespace** -- Whitespace is trimmed before format validation and submission.
- **Screen loaded from a shared or bookmarked link with no session** -- No pre-fill occurs; the form starts empty as it would for any direct arrival.
- **This screen never loads or updates a shared entity, so no concurrent-edit conflict applies here** -- N/A: the only write this feature performs (Client Contact's `last_sign_in`) happens in FEAT-05.SPEC-005 after verification, not on this screen.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-05.SPEC-004 (Magic Link Issuance) | Triggers (outbound) | Submitting the form triggers issuance processing for the entered email |
| FEAT-05.SPEC-006 (Link Validity & Recognition Rules) | References (inbound) | Email-format validation and the no-enumeration confirmation rule are governed there |
| FEAT-05.SPEC-002 (Link Verification Landing) | Navigation (inbound) | The "Request a new link" control on an expired/invalid link routes here |
| FEAT-33 (Portal Referral Attribution) | Navigation (inbound) | A returning contact with no live session reaches this screen from the public product page or referral mark |
| FEAT-19 (Freelancer Branding) | References (inbound) | Branding Profile supplies the logo and colour shown in the header |
| FEAT-33 (Portal Referral Attribution) | Navigation (outbound) | The footer referral mark links out to the public product page |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|------------------|
| magic_link_requested | request_type (initial / re-request, inferred from Entry Point) | The contact submits a validly formatted email and the request reaches FEAT-05.SPEC-004 | supports success-metrics.md: "Client Portal Login Success" |
| magic_link_request_validation_failed | reason (invalid_format) | Form submit is blocked by field-level validation | N/A -- no Stage 2 metric measures client-side input error frequency; retained so a rise in malformed submissions is visible to the freelancer's product team, not silently dropped |

## Acceptance Criteria

**FEAT-05.SPEC-001-AC-01:** Given Owen is on the Request Sign-In Link screen, when he enters his email and taps "Send sign-in link," then the button shows a loading state and, on completion, the screen shows "Check your email. If {his email} matches an account, we've sent a sign-in link to it."

**FEAT-05.SPEC-001-AC-02:** Given Priya is on the Request Sign-In Link screen, when she enters an email that has never been added as a client contact, then she sees the identical confirmation message as a recognized contact, and no link is ever issued to that address.

**FEAT-05.SPEC-001-AC-03:** Given Owen is on the Request Sign-In Link screen, when he leaves the email field blank or malformed and moves focus away, then the field shows the error "Enter a valid email address" and the button remains disabled for an empty field.

**FEAT-05.SPEC-001-AC-04:** Given Owen is on the confirmation state, when he taps "Try a different email," then the form returns to its empty, editable state with focus on the email field.

**FEAT-05.SPEC-001-AC-05:** Given Owen taps "Send sign-in link" and taps it again before the first request completes, then the second tap has no effect and the button remains in its loading state.

**FEAT-05.SPEC-001-AC-06:** Given Priya loses connectivity while viewing this screen, when the screen detects the loss, then the banner "You're offline. Connect to the internet to request a sign-in link." appears and the "Send sign-in link" button is disabled.

**FEAT-05.SPEC-001-AC-07:** Given Priya's connectivity is restored after the offline banner appeared, when connectivity returns, then the banner clears, the button re-enables, and no request was queued or auto-submitted while she was offline.

**FEAT-05.SPEC-001-AC-08:** Given a contact arrives at this screen from FEAT-05.SPEC-002's "Request a new link" control with a known email, when the screen loads, then the email field is pre-filled with that address.

**FEAT-05.SPEC-001-AC-09:** Given Owen reaches this screen through his freelancer's Branding Profile, when the screen renders, then the header shows that freelancer's logo and brand colour, or the neutral default if none is set.

**FEAT-05.SPEC-001-AC-10:** Given a contact's session lapses while viewing any portal page, when the lapse is detected, then the contact is returned to this screen with no error dialog, since this is the feature's expected recovery path.

**FEAT-05.SPEC-001-AC-11:** Given Priya is on the Request Sign-In Link screen, when she taps the "Made with Clientroom" referral mark, then she is navigated externally to the public product page (FEAT-33) and this screen's own state is unaffected.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 6 (empty, filling, submitting, confirmation, validation error, offline) | 6 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |

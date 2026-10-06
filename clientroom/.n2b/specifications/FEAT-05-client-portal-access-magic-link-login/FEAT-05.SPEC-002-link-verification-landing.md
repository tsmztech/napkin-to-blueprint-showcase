---
document_type: spec
spec_type: screen
spec_id: FEAT-05.SPEC-002
spec_name: Link Verification Landing
spec_slug: link-verification-landing
parent_feature: FEAT-05
parent_feature_name: Client Portal Access (Magic-Link Login)
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Screen Spec: Link Verification Landing

## Overview

**Name:** Link Verification Landing
**ID:** FEAT-05.SPEC-002
**Type:** Screen
**Purpose:** The contact sees their emailed link being verified, then either enters the portal or sees a plain explanation with a one-tap way to request a fresh link.
**Parent Feature:** FEAT-05 -- Client Portal Access (Magic-Link Login)

## Scope and Non-Goals

**In Scope:**
- The in-progress verification appearance while FEAT-05.SPEC-005 validates the clicked token
- The successful-verification transition into the portal
- The expired, already-used, and out-of-scope error explanations, each with a one-tap re-request
- The offline/degraded state when the link cannot be verified without connectivity

**Non-Goals:**
- Validating the token, creating the scoped session, and recording the sign-in -- owned by FEAT-05.SPEC-005 (Magic Link Verification); this screen only reflects that automation's outcome.
- Determining what "expired," "already used," and "out-of-scope" mean and how invalidation works -- owned by FEAT-05.SPEC-006 (Link Validity & Recognition Rules) and FEAT-05.SPEC-007 (Portal Access & Isolation Rules); this screen shows the same plain explanation for every disqualifying reason rather than distinguishing them.
- Collecting a new email to request a fresh link -- the one-tap re-request either resubmits the already-known email directly or routes to FEAT-05.SPEC-001 (Request Sign-In Link) when no email is known from context; either way, the email-entry work belongs to that screen.
- Custom-domain serving of this landing page -- deferred per scope-boundaries.md's Deferral note and ASMP-32: in MVP this screen is served only at the shared default address; FEAT-27 (Later phase) is the only source of a custom-domain path.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| External (email client) | Contact clicks the sign-in link in the Magic Link Sign-In Email (FEAT-05.SPEC-008) | The token embedded in the link |
| External (email client) | Contact clicks a proposal-sent, deliverable-ready, or invitation email's embedded link from FEAT-02 (FEAT-02.SPEC-011), FEAT-06 (FEAT-06.SPEC-006), or FEAT-18 (FEAT-18.SPEC-010, New Contact Invitation Email) | The token embedded in the link, plus the specific record the originating email pointed to |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Owen (Client Primary Contact) | Full screen, for a link addressed to Owen's own recognized contact record | Request a fresh link if the clicked one fails | -- |
| Priya (Client Reviewer Contact) | Full screen, for a link addressed to Priya's own recognized contact record | Request a fresh link if the clicked one fails | -- |
| Nadia (Freelancer) | Full screen if she opens a client-addressed link (this is a public entry point, not role-gated by URL alone) | None -- Nadia holds no Client Contact record, so the token addressed to a contact never verifies for her | Same plain expired/invalid explanation and re-request option shown to any contact whose token does not verify; never a distinct "not a client" message, which would leak that the address is freelancer-owned |
| Dana (Support Operator) | Full screen if she opens a client-addressed link | None -- Dana never signs in as a client contact (SC-04) | Same plain expired/invalid explanation as above |
| Unauthenticated | Yes -- verifying a link is itself the unauthenticated step | Yes -- the one-tap re-request is available without prior authentication | -- |
| Expired session | N/A -- this screen is reached only from a fresh link click, never from an existing session; a contact with an expired session lands on FEAT-05.SPEC-001 instead, per that spec's Access and Visibility table | N/A | N/A |

## Layout and Content

**Header:** The owning freelancer's Branding Profile logo (or neutral default), centered, per XBR-31.

**Body (Verifying state):** A centered progress indicator with the text "Signing you in..."

**Body (Error state):** A plain, non-alarming heading -- "This link isn't valid anymore" -- with one line of explanation ("Sign-in links are single-use and time-limited, and yours has expired, already been used, or was requested again since.") and a single "Send me a new link" button.

**Body (Offline/Degraded state):** A connectivity notice heading -- "You're offline" -- with the text "We can't verify your sign-in link without a connection. Reconnect and try the link again." and a "Retry" button.

**Footer:** The discreet "Made with Clientroom" referral mark (FEAT-33), consistent with FEAT-05.SPEC-001 and FEAT-05.SPEC-003.

### Responsive Behavior

- **Compact breakpoint (phone width):** Content vertically centered in the viewport, full width with the platform's standard side gutter; the "Send me a new link" / "Retry" button spans the content width.
- **Medium size class and above:** Content remains centered and capped at a consistent platform-wide width; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| (automatic) | Screen loads with a token | Triggers FEAT-05.SPEC-005 (Magic Link Verification) | Screen enters Verifying state | Progress indicator with "Signing you in..." |
| "Send me a new link" button | Tap | If the contact's email is known from the clicked link's context, resubmits it directly to FEAT-05.SPEC-004 (Magic Link Issuance); otherwise navigates to FEAT-05.SPEC-001 (Request Sign-In Link) | Button shows loading state, or screen navigates | On direct resubmission: transitions to the same "Check your email" confirmation described in FEAT-05.SPEC-001. On navigation: FEAT-05.SPEC-001 loads with the email field empty |
| "Retry" button (offline state) | Tap | Re-attempts verification of the same token | Screen returns to Verifying state | Progress indicator reappears |
| "Made with Clientroom" referral mark | Tap | Navigates externally to the public product page (FEAT-33); does not affect this screen's own state | This screen's state is unchanged | Standard external navigation |

### Accessibility Notes

- **Focus order:** Logo (skippable) -> heading -> body text -> primary action button (when present).
- **Verifying announcement:** The "Signing you in..." status is announced to assistive technology as the screen enters the Verifying state, so a screen-reader user is not left in silence during the wait.
- **Error/offline announcement:** The heading of the Error or Offline/Degraded state is announced immediately when that state is entered.
- **Keyboard alternatives:** "Send me a new link" and "Retry" are reachable and activatable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Verifying (default) | Progress indicator, "Signing you in..." | Screen first opens with a token | FEAT-05.SPEC-005 returns an outcome |
| Success (transitional) | Same as Verifying, momentarily, before navigation | FEAT-05.SPEC-005 confirms the token is valid and the session is created | Immediate navigation to FEAT-05.SPEC-003 (Portal Home) |
| Error (expired, used, or out-of-scope) | Plain explanation heading, body text, and "Send me a new link" button | FEAT-05.SPEC-005 reports the token as invalid, expired, already used, or out of the contact's scope (FEAT-05.SPEC-006, FEAT-05.SPEC-007) | Contact taps "Send me a new link" |
| Offline/Degraded | Connectivity notice with "Retry" button; no automatic queued retry | Verification cannot reach FEAT-05.SPEC-005 due to lost connectivity, at load or mid-verification | Connectivity is restored and the contact taps "Retry" -- verification is not attempted automatically, since a delayed automatic retry against a single-use, time-limited link could itself land on an inconsistent state |

## Validation Rules

Validation governed by FEAT-05.SPEC-006 (Link Validity & Recognition Rules) for token validity, and FEAT-05.SPEC-007 (Portal Access & Isolation Rules) for scope. See those specs for the full set of conditions that produce the Error state on this screen; this screen applies no rules of its own beyond routing the outcome to Success, Error, or Offline/Degraded.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Verification succeeds | FEAT-05.SPEC-003 (Portal Home) | -- |
| "Send me a new link" (email unknown from context) | FEAT-05.SPEC-001 (Request Sign-In Link) | -- |

## Data Model

**Creates:** None directly -- session creation and the resulting Activity Log Entry are written by FEAT-05.SPEC-005.
**Reads:** Branding Profile -- logo and brand_colour, resolved from the token's associated freelancer once verification identifies it (or the neutral default while the freelancer is not yet known, during the Verifying state).
**Updates:** None directly -- FEAT-05.SPEC-005 updates the Client Contact's `last_sign_in` on success.
**Deletes:** None.

## Business Rules

- The same plain explanation is shown for every disqualifying reason -- expired, already used, or out-of-scope (FEAT-05.SPEC-007) -- so this screen never distinguishes "wrong company" from "expired," per XBR-09's requirement that an out-of-scope link never reveal another company's data.
- Automatic retry is never attempted on this screen: a failed or offline verification always requires an explicit tap ("Retry" or "Send me a new link"), so a stale automatic re-check can never silently consume a link's single use.
- The one-tap re-request (FEAT-05.SPEC-006) issues a new link under the invalidate-prior-link rule -- tapping it does not attempt to revalidate the link that failed.

## Edge Cases

- **Contact double-taps "Send me a new link"** -- The second tap is ignored while the first request is in flight (button loading state), mirroring FEAT-05.SPEC-001's double-submit prevention.
- **Token is well-formed but was never issued (tampered or guessed link)** -- FEAT-05.SPEC-005 reports it as invalid; this screen shows the same Error state as an expired link, with no distinct message that would hint the token format was recognized.
- **Contact clicks an old email's link after already signing in via a newer one** -- The older token verifies as already-used per FEAT-05.SPEC-006's invalidate-on-re-request rule; the Error state and re-request option are shown exactly as for any used link.
- **Connectivity drops mid-verification (after tap, before an outcome returns)** -- The screen transitions from Verifying to Offline/Degraded rather than hanging indefinitely; tapping "Retry" re-attempts verification of the same token, which remains valid to retry since a network failure never consumes the link's single use.
- **This screen never loads or updates a shared entity, so no concurrent-edit conflict applies here** -- N/A: verification's write to Client Contact's `last_sign_in` happens inside FEAT-05.SPEC-005, and that write is a single-field overwrite by a single automation, not a screen-mediated edit.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-05.SPEC-005 (Magic Link Verification) | Triggers (outbound) | Screen load triggers token verification; the automation's outcome drives this screen's state |
| FEAT-05.SPEC-006 (Link Validity & Recognition Rules) | References (inbound) | Defines what makes a token expired, used, or otherwise invalid |
| FEAT-05.SPEC-007 (Portal Access & Isolation Rules) | References (inbound) | Defines the out-of-scope condition shown identically to an expired link |
| FEAT-05.SPEC-004 (Magic Link Issuance) | Triggers (outbound) | The re-request control resubmits the known email to issuance |
| FEAT-05.SPEC-001 (Request Sign-In Link) | Navigation (outbound) | The re-request control routes here when no email is known from context |
| FEAT-05.SPEC-003 (Portal Home) | Navigation (outbound) | Successful verification navigates here |
| FEAT-02 (Proposal Creation & Sending), FEAT-06 (Deliverable Upload & Sharing), FEAT-18 (Client Contact Management & Roles) | Navigation (inbound) | Their emailed links are the entry points that land a contact here |
| FEAT-33 (Portal Referral Attribution) | Navigation (outbound) | The footer referral mark links out to the public product page |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|------------------|
| magic_link_verification_error_shown | reason (invalid_expired_or_used / out_of_scope, without further detail per the no-distinction rule above) | The screen enters the Error state | supports success-metrics.md: "Client Portal Login Success" |
| magic_link_offline_shown | -- | The screen enters the Offline/Degraded state | N/A -- no Stage 2 metric measures connectivity-caused verification interruptions; retained so this failure mode is distinguishable from an expired-link failure when reviewing login success |

## Acceptance Criteria

**FEAT-05.SPEC-002-AC-01:** Given Owen has just clicked his sign-in link, when the screen loads, then it shows the Verifying state with "Signing you in..." while FEAT-05.SPEC-005 validates the token.

**FEAT-05.SPEC-002-AC-02:** Given Owen's token verifies successfully, when verification completes, then he is navigated to FEAT-05.SPEC-003 (Portal Home) without further action.

**FEAT-05.SPEC-002-AC-03:** Given Priya clicks a link that has already been used, when verification completes, then she sees "This link isn't valid anymore" with the explanation text and a "Send me a new link" button.

**FEAT-05.SPEC-002-AC-04:** Given Owen clicks a link addressed to a different freelancer's client company than his own, when verification completes, then he sees the identical "This link isn't valid anymore" explanation as an expired link -- never another company's data.

**FEAT-05.SPEC-002-AC-05:** Given Priya is shown the Error state and her email is known from the clicked link's context, when she taps "Send me a new link," then a new link request is submitted directly and she sees the "Check your email" confirmation without re-entering her address.

**FEAT-05.SPEC-002-AC-06:** Given Owen is shown the Error state and no email is known from context, when he taps "Send me a new link," then he is navigated to FEAT-05.SPEC-001 (Request Sign-In Link) with an empty email field.

**FEAT-05.SPEC-002-AC-07:** Given Priya opens her sign-in link with no connectivity, when the screen attempts verification, then she sees "You're offline" with a "Retry" button and no verification attempt is made until she taps it.

**FEAT-05.SPEC-002-AC-08:** Given Owen taps "Retry" on the Offline/Degraded state after connectivity is restored, when the retry fires, then the screen returns to the Verifying state and re-attempts validation of the same token.

**FEAT-05.SPEC-002-AC-09:** Given Priya taps "Send me a new link" and taps it again before the first request completes, then the second tap has no effect and the button remains in its loading state.

**FEAT-05.SPEC-002-AC-10:** Given Owen clicks a well-formed but never-issued token, when verification completes, then he sees the same "This link isn't valid anymore" Error state as any other invalid link.

**FEAT-05.SPEC-002-AC-11:** Given the screen is showing any state, when it renders its header, then it shows the owning freelancer's Branding Profile logo and colour, or the neutral default when the freelancer is not yet resolved.

**FEAT-05.SPEC-002-AC-12:** Given Owen is on this screen in any state, when he taps the "Made with Clientroom" referral mark, then he is navigated externally to the public product page (FEAT-33) and this screen's own state is unaffected.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 4 | 4 |
| States | 4 (verifying, success, error, offline) | 4 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |

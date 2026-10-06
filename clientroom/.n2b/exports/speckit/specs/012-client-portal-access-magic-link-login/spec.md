# Feature Specification: Client Portal Access (Magic-Link Login)

**Blueprint feature:** FEAT-05
**Priority tier:** Core
**Build order:** 012 of 33
**Depends on:** FEAT-18
**Blueprint source:** `docs/blueprint/specifications/FEAT-05-client-portal-access-magic-link-login/`

## User Scenarios & Testing (mandatory)

### User Story 1 - Request Sign-In Link (Priority: P1)

A client contact enters their email and requests a one-time sign-in link, without ever seeing an account-creation step.

**Acceptance Scenarios:**

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

### User Story 2 - Link Verification Landing (Priority: P1)

The contact sees their emailed link being verified, then either enters the portal or sees a plain explanation with a one-tap way to request a fresh link.

**Acceptance Scenarios:**

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

### User Story 3 - Portal Home (Priority: P1)

The contact sees their company's projects, each project's current stage and milestones, and exactly what is waiting on them, scoped to what their role allows.

**Acceptance Scenarios:**

**FEAT-05.SPEC-003-AC-01:** Given Owen signs in and lands on Portal Home, when the screen loads, then he sees his client company's projects, each with its stage, milestone summary, and any items waiting on him.

**FEAT-05.SPEC-003-AC-02:** Given Priya signs in and lands on Portal Home, when she views her "Waiting on you" region, then it lists only deliverables waiting for her review and never a proposal or invoice item.

**FEAT-05.SPEC-003-AC-03:** Given Owen has a project with a proposal waiting on him, when he taps that waiting item, then he is navigated to FEAT-03 (Proposal Acceptance) for that specific proposal.

**FEAT-05.SPEC-003-AC-04:** Given Owen is on Portal Home, when he taps "Invite a colleague," then he is navigated into FEAT-18's invite-a-Reviewer flow.

**FEAT-05.SPEC-003-AC-05:** Given Priya is on Portal Home, when she looks for an "Invite a colleague" action, then it is not shown, since only Owen (Primary contact) can invite.

**FEAT-05.SPEC-003-AC-06:** Given a contact with no active projects lands on Portal Home, when the screen loads, then it shows "Nothing here yet -- {freelancer name} hasn't sent you anything to review." rather than an error.

**FEAT-05.SPEC-003-AC-07:** Given Owen's project data fails to load, when the load fails, then he sees "We couldn't load your projects. Try again." with a Retry button.

**FEAT-05.SPEC-003-AC-08:** Given Priya loses connectivity while viewing a previously loaded Portal Home, when connectivity drops, then the banner "You're offline -- showing the last loaded view." appears and the last-loaded project list remains visible.

**FEAT-05.SPEC-003-AC-09:** Given Priya's connectivity returns after the offline banner appeared, when connectivity is restored, then the banner clears and the screen refreshes automatically.

**FEAT-05.SPEC-003-AC-10:** Given Owen is a client contact for two different freelancers, when he signs into each freelancer's portal separately, then each Portal Home shows only that freelancer's projects under that freelancer's own branding, never both together.

**FEAT-05.SPEC-003-AC-11:** Given Owen taps a project card with no items currently waiting on him, when the card is shown, then it displays the project's stage and milestone summary with no "Waiting on you" region.

**FEAT-05.SPEC-003-AC-12:** Given Portal Home renders for either Owen or Priya, when the page becomes interactive, then it emits `portal_home_viewed` with the load time, supporting the mobile responsiveness target.

**FEAT-05.SPEC-003-AC-13:** Given Owen is on Portal Home, when he taps the "Made with Clientroom" referral mark, then he is navigated externally to the public product page (FEAT-33) and this screen's own state is unaffected.

### User Story 4 - Magic Link Issuance (Priority: P1)

Generates a single-use, time-limited sign-in link for a recognized contact and invalidates any prior unused link for that contact.

**Acceptance Scenarios:**

**FEAT-05.SPEC-004-AC-01:** Given Owen submits his own recognized, Active email on FEAT-05.SPEC-001, when the automation runs, then a new single-use token is generated and handed to FEAT-05.SPEC-008 for delivery.

**FEAT-05.SPEC-004-AC-02:** Given Priya submits an email that matches no Client Contact, when the automation runs, then no token is created and the triggering screen shows the identical confirmation as a successful issuance.

**FEAT-05.SPEC-004-AC-03:** Given Owen has an unused, unexpired token from a prior request, when he submits a new request, then the prior token is invalidated before the new token is generated.

**FEAT-05.SPEC-004-AC-04:** Given a person's email matches Active Client Contact records under two different freelancers, when the automation runs, then a separate token is issued for each freelancer, each independently scoped and each triggering its own FEAT-05.SPEC-008 email.

**FEAT-05.SPEC-004-AC-05:** Given a Client Contact whose `status` is Removed submits their email, when the automation runs, then the outcome is identical to No Match -- no token is issued.

**FEAT-05.SPEC-004-AC-06:** Given Priya submits her email with extra whitespace and different letter casing than stored, when the automation runs, then it still matches her Client Contact record and issues a token.

**FEAT-05.SPEC-004-AC-07:** Given Owen submits two requests within moments of each other from two open tabs, when both complete, then exactly one valid token remains for Owen's contact record.

**FEAT-05.SPEC-004-AC-08:** Given the automation encounters an internal processing error while checking recognition, when it fails, then the triggering screen still shows the same neutral confirmation as a successful request, and no email is sent.

**FEAT-05.SPEC-004-AC-09:** Given Priya taps "Send me a new link" on FEAT-05.SPEC-002 with her email known from context, when the automation runs, then it processes identically to a fresh submission from FEAT-05.SPEC-001, including invalidating her prior token.

### User Story 5 - Magic Link Verification (Priority: P1)

Validates a clicked link, creates the scoped session, and records the sign-in.

**Acceptance Scenarios:**

**FEAT-05.SPEC-005-AC-01:** Given Owen clicks a valid, unused, unexpired link addressed to his Active Client Contact record, when verification runs, then the token is marked used, his `last_sign_in` is updated, a sign-in Activity Log Entry is written, and he is delivered to FEAT-05.SPEC-003 (Portal Home).

**FEAT-05.SPEC-005-AC-02:** Given Priya clicks a link that has already been used, when verification runs, then the outcome is Invalid -- already used, and FEAT-05.SPEC-002 shows the plain explanation.

**FEAT-05.SPEC-005-AC-03:** Given Owen clicks a link after platform parameter: `magic-link-expiry-window` has elapsed since issuance, when verification runs, then the outcome is Invalid -- expired.

**FEAT-05.SPEC-005-AC-04:** Given Owen requests a new link while an old one is still unused, then clicks the old link, when verification runs, then the outcome is Invalid -- invalidated by re-request.

**FEAT-05.SPEC-005-AC-05:** Given Priya's Client Contact record was removed by FEAT-18 after her link was issued, when she clicks the link, then the outcome is Invalid -- contact no longer Active, and no session is created.

**FEAT-05.SPEC-005-AC-06:** Given a fabricated or guessed token that was never issued, when verification runs, then the outcome is Invalid -- unknown token, shown identically to any other invalid outcome.

**FEAT-05.SPEC-005-AC-07:** Given a valid token is opened on two devices at effectively the same moment, when both verifications run, then exactly one succeeds and the other returns Invalid -- already used.

**FEAT-05.SPEC-005-AC-08:** Given an email client pre-fetches Owen's sign-in link as a preview without his own tap, when the pre-fetch occurs, then the token is not consumed, and Owen's own subsequent tap still verifies successfully.

**FEAT-05.SPEC-005-AC-09:** Given verification succeeds but the Activity Log Entry hand-off to FEAT-13 fails, when this occurs, then Owen still reaches Portal Home normally, and the trail entry is retried by FEAT-13 rather than blocking his sign-in.

**FEAT-05.SPEC-005-AC-10:** Given a processing error occurs before the token is marked used, when the error occurs, then the token remains valid and unused, and the contact can retry the same link.

### User Story 6 - Link Validity & Recognition Rules (Priority: P1)

Governs single-use and time-limited link enforcement, invalidation on re-request, and that only recognized, Active contacts can obtain or use a link.

**Acceptance Scenarios:**

**FEAT-05.SPEC-006-AC-01:** Given Owen submits a validly formatted, recognized, Active email, when the request is processed, then a new token is issued with status Issued and an expiry of platform parameter: `magic-link-expiry-window` from now.

**FEAT-05.SPEC-006-AC-02:** Given Priya submits an email matching no Client Contact, when the request is processed, then no token is created and she sees the identical confirmation as a recognized submission.

**FEAT-05.SPEC-006-AC-03:** Given Dana (Support Operator) submits her own email, when the request is processed, then no token is issued, since she holds no Client Contact record, and she sees the same neutral confirmation.

**FEAT-05.SPEC-006-AC-04:** Given Owen has an Issued, unused token, when he requests a new link, then the prior token transitions to Invalidated before the new token is issued.

**FEAT-05.SPEC-006-AC-05:** Given Owen's token's expires_at has passed and it was never used, when he clicks it, then it is treated as Expired and he sees "This link isn't valid anymore."

**FEAT-05.SPEC-006-AC-06:** Given Priya clicks a token exactly at its expiry instant, when verification checks it, then it is treated as expired (the boundary itself does not verify).

**FEAT-05.SPEC-006-AC-07:** Given Owen's Client Contact status changes to Removed after his token was issued but before he clicks it, when he clicks the still-unused, unexpired token, then it fails per the Active-contact condition and he sees the plain explanation.

**FEAT-05.SPEC-006-AC-08:** Given a person is a Client Contact for two different freelancers with the same email, when they request a link, then each freelancer issues and governs its own token independently.

**FEAT-05.SPEC-006-AC-09:** Given Owen submits his email with different letter casing and surrounding whitespace than stored, when the request is processed, then it still matches his Client Contact record.

**FEAT-05.SPEC-006-AC-10:** Given Priya's token has already transitioned to Used, when she clicks the same link again, then it is treated as invalid and she sees the plain explanation, never a distinct "already signed in" message that would confirm the token had once been valid.

**FEAT-05.SPEC-006-AC-11:** Given Owen requests a link and lets it expire without ever re-requesting, when he later requests a fresh link, then the fresh token is Issued with a full new expiry window, independent of the expired one.

**FEAT-05.SPEC-006-AC-12:** Given Priya's token is Invalidated by a later re-request, when she clicks the invalidated (not the new) link, then she sees the same plain explanation as an expired or used link.

**FEAT-05.SPEC-006-AC-13:** Given Nadia (Freelancer) submits her own sign-in email on the client portal's request screen, when the request is processed, then no token is issued to her, since she holds no Client Contact record, and she sees the same neutral confirmation as any other submission.

### User Story 7 - Portal Access & Isolation Rules (Priority: P1)

Governs client isolation, role-scoped portal display, and multi-freelancer separation for a contact who serves several freelancers.

**Acceptance Scenarios:**

**FEAT-05.SPEC-007-AC-01:** Given Owen verifies a link addressed to his Client Contact record, when the session is created, then it is scoped to exactly his own client company and its owning freelancer.

**FEAT-05.SPEC-007-AC-02:** Given Priya's session is scoped to her client company, when she views Portal Home, then her "Waiting on you" region never lists a proposal or invoice item.

**FEAT-05.SPEC-007-AC-03:** Given Owen's session is scoped to his client company, when he views Portal Home, then he sees proposal, deliverable, milestone-approval, and invoice waiting items for that company.

**FEAT-05.SPEC-007-AC-04:** Given Owen is a Client Contact for two different freelancers, when he signs into each portal separately, then each session shows only that freelancer's data, under that freelancer's own branding, with no path from one into the other.

**FEAT-05.SPEC-007-AC-05:** Given a verified session resolves to a client company different from the one a deep link expected (a scope mismatch), when the mismatch is detected, then the contact sees the same "not valid anymore" explanation as an expired token, never the other company's data.

**FEAT-05.SPEC-007-AC-06:** Given Priya is on Portal Home, when she looks for an "Invite a colleague" control, then it is not shown, since inviting is Primary-only.

**FEAT-05.SPEC-007-AC-07:** Given Owen is on Portal Home, when he looks for an "Invite a colleague" control, then it is shown and opens FEAT-18's invite flow for his own client company.

**FEAT-05.SPEC-007-AC-08:** Given Nadia (Freelancer) attempts to reach the client portal shell, when she does so, then there is no entry path that authenticates her into it, since she holds no Client Contact record.

**FEAT-05.SPEC-007-AC-09:** Given Dana (Support Operator) has an open, valid support session on a freelancer's account, when she attempts to reach that freelancer's client-facing portal, then she has no valid client token and cannot enter it (SC-04).

**FEAT-05.SPEC-007-AC-10:** Given Nadia changes Priya's role from Reviewer to Primary while Priya's portal session is already open, when Priya continues browsing that open session, then it continues to reflect Reviewer-level display until she signs in again with a fresh session.

**FEAT-05.SPEC-007-AC-11:** Given a contact's client company is archived while their portal session is open, when they reload Portal Home, then the archived project's state is shown normally rather than being treated as an isolation violation.

**FEAT-05.SPEC-007-AC-12:** Given Owen holds separate Client Contact records for two freelancers, when Nadia (Freelancer A) views her own client roster, then she has no visibility into Owen's relationship with Freelancer B, and neither freelancer's session can ever expose the other's client data.

### User Story 8 - Magic Link Sign-In Email (Priority: P1)

Emails the one-time sign-in link to the requesting contact whenever a link is issued, so they can enter their scoped portal without a password.

**Acceptance Scenarios:**

**FEAT-05.SPEC-008-AC-01:** Given Owen's email is recognized and a token is issued, when FEAT-05.SPEC-004 completes, then Owen receives an email with subject "Sign in to your {his freelancer's business name} client portal" and a "Sign in" button linking to his issued token.

**FEAT-05.SPEC-008-AC-02:** Given Priya's freelancer's `business_name` is not yet set, when her sign-in email is composed, then the subject renders as "Sign in to your client portal" and the greeting still uses her name.

**FEAT-05.SPEC-008-AC-03:** Given Owen taps the "Sign in" button in the email, when the link opens, then he lands on FEAT-05.SPEC-002 (Link Verification Landing) with his token.

**FEAT-05.SPEC-008-AC-04:** Given Owen is a Client Contact for two freelancers and requests a link once, when both issuances complete, then he receives two separate emails, each with that freelancer's own branding and its own token.

**FEAT-05.SPEC-008-AC-05:** Given delivery of Priya's sign-in email fails on the first attempt, when the delivery capability retries, then it retries up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window` before giving up.

**FEAT-05.SPEC-008-AC-06:** Given all retries for Owen's sign-in email are exhausted, when the final attempt fails, then Nadia sees a delivery warning on the client's contact record, since Owen has no other way to be reached.

**FEAT-05.SPEC-008-AC-07:** Given Priya requests a fresh link while a prior email is still in transit, when the prior email eventually delivers, then clicking its link shows "This link isn't valid anymore," since FEAT-05.SPEC-006 invalidated the prior token independently of this email's delivery.

**FEAT-05.SPEC-008-AC-08:** Given this notification has no preference control for the recipient, when a Client Contact requests a link, then the email always sends -- there is no opt-out surface to check.

**FEAT-05.SPEC-008-AC-09:** Given a link is requested at any hour, when the email is triggered, then it sends immediately with no quiet-hours hold, since the underlying credential is time-limited.

**FEAT-05.SPEC-008-AC-10:** Given the freelancer's Branding Profile has a logo and brand colour set, when this email renders, then it displays that branding alongside the "Made with Clientroom" referral mark, with the mark never overriding the branding.

### User Story 9 - Portal Record First-View Capture (Priority: P1)

Detects a client contact's first view of a proposal, deliverable, or invoice reached through the portal and hands the timestamped event to FEAT-13's audit trail.

**Acceptance Scenarios:**

**FEAT-05.SPEC-009-AC-01:** Given Owen opens a proposal waiting on him for the first time through his portal session, when the screen opens, then a first-view event is captured and handed to FEAT-13 with his identity, the timestamp, and the proposal reference.

**FEAT-05.SPEC-009-AC-02:** Given Priya has already had a first-view event captured for a specific deliverable, when she opens that same deliverable again, then no new event is captured.

**FEAT-05.SPEC-009-AC-03:** Given Owen opens the same invoice twice in quick succession from two open tabs, when both opens are processed, then exactly one first-view event exists for that invoice and Owen's contact record.

**FEAT-05.SPEC-009-AC-04:** Given both Owen and Priya open the same deliverable for the first time, each from their own portal session, when both views occur, then two separate first-view events are captured, one per contact.

**FEAT-05.SPEC-009-AC-05:** Given the hand-off to FEAT-13 fails after Owen's first view of an invoice is captured, when the failure occurs, then Owen's invoice screen still renders normally, and FEAT-13's own retry eventually completes the trail write.

**FEAT-05.SPEC-009-AC-06:** Given a deliverable is later superseded by a new version after Priya's first view of the original was captured, when she opens the superseded version again, then no new first-view event is captured, since her first-view event for that deliverable already exists.

**FEAT-05.SPEC-009-AC-07:** Given Owen opens a proposal through the portal for the first time, when the first-view event is captured, then the timestamp recorded is the moment the screen opened, not any later moment such as scrolling or closing the screen.

**FEAT-05.SPEC-009-AC-08:** Given this automation fires from FEAT-03, FEAT-06, FEAT-07, FEAT-09, or FEAT-10's own screens, when any of those triggers fire, then the same detection and idempotency logic applies uniformly regardless of which feature's screen triggered it.

**FEAT-05.SPEC-009-AC-09:** Given Owen's first-view event for a proposal is already recorded, when he later re-opens that proposal after it has been voided and re-sent as a new Proposal record (FEAT-02's void-and-resend), then the new Proposal record is a distinct record from the voided one, so his open of the new record is evaluated as its own first-view candidate, independent of the voided proposal's already-captured event.

### Edge Cases

- **FEAT-05.SPEC-001 (Request Sign-In Link):** Duplicate submissions are independent re-requests where only the most recent link stays valid (FEAT-05.SPEC-006), whitespace around the email is trimmed, and returning mid-submit reloads the empty default state since recognition status is never disclosed. A shared or bookmarked link starts with an empty form. Source: `docs/blueprint/specifications/FEAT-05-client-portal-access-magic-link-login/FEAT-05.SPEC-001-request-sign-in-link.md` (section: Edge Cases)
- **FEAT-05.SPEC-002 (Link Verification Landing):** Double taps on Send me a new link are ignored, and a tampered, guessed or superseded token shows the same Error state as an expired link without hinting at token format. A connectivity drop mid-verification moves to an Offline/Degraded state with a Retry that re-attempts the same token. Source: `docs/blueprint/specifications/FEAT-05-client-portal-access-magic-link-login/FEAT-05.SPEC-002-link-verification-landing.md` (section: Edge Cases)
- **FEAT-05.SPEC-003 (Portal Home):** A contact whose projects have nothing waiting sees project cards with no Waiting on you region (not the empty state), and the snapshot does not live-update when another contact resolves an item. Large project counts scroll without pagination, and a client company archived before verification completes shows the Empty state rather than an error. Source: `docs/blueprint/specifications/FEAT-05-client-portal-access-magic-link-login/FEAT-05.SPEC-003-portal-home.md` (section: Edge Cases)
- **FEAT-05.SPEC-004 (Magic Link Issuance):** Email matching is case-insensitive and whitespace-trimmed, the contact's current status at request time governs (a Removed contact yields No Match, a re-activated one is issued a link), and a new request invalidates the prior unused token. Concurrent requests run independently with the last invalidation winning. Source: `docs/blueprint/specifications/FEAT-05-client-portal-access-magic-link-login/FEAT-05.SPEC-004-magic-link-issuance.md` (section: Edge Cases)
- **FEAT-05.SPEC-005 (Magic Link Verification):** Whichever verification reaches the used-marking step first wins, including pre-fetching email clients and the same token opened on two devices, and the loser sees Invalid already used. The used-state check and write are one atomic step, and a contact removed after issuance fails verification before any session is created. Source: `docs/blueprint/specifications/FEAT-05-client-portal-access-magic-link-login/FEAT-05.SPEC-005-magic-link-verification.md` (section: Edge Cases)
- **FEAT-05.SPEC-006 (Link Validity & Recognition Rules):** An expired token stays Expired when the contact simply requests again, and the exact expiry instant counts as expired (strictly before expires_at is valid). A contact's status is re-evaluated at click time, so a Removed contact's unused token fails. Source: `docs/blueprint/specifications/FEAT-05-client-portal-access-magic-link-login/FEAT-05.SPEC-006-link-validity-recognition-rules.md` (section: Edge Cases)
- **FEAT-05.SPEC-007 (Portal Access & Isolation Rules):** A person who is a contact for two freelancers has two fully independent records and sessions, deep links are re-evaluated against the current session's scope at open time, and a role change applies to future actions only (an open session is not retroactively altered). An archived client company leaves the session scoped to it and shows archived state normally. Source: `docs/blueprint/specifications/FEAT-05-client-portal-access-magic-link-login/FEAT-05.SPEC-007-portal-access-isolation-rules.md` (section: Edge Cases)
- **FEAT-05.SPEC-008 (Magic Link Sign-In Email):** A bounced address is retried and then surfaced to the freelancer as a delivery warning on the contact record. A prior link is invalidated independently of its email's delivery state, a token expiring before delivery shows not-valid on click, and a contact for two freelancers gets a separate email per matched issuance. Source: `docs/blueprint/specifications/FEAT-05-client-portal-access-magic-link-login/FEAT-05.SPEC-008-magic-link-sign-in-email.md` (section: Edge Cases)
- **FEAT-05.SPEC-009 (Portal Record First-View Capture):** The first-view event is recorded once per record and contact pair, so a second open or concurrent opens from two devices take No-Action, with the existence check and creation treated as one atomic step. Two different contacts opening the same deliverable are each tracked independently. Source: `docs/blueprint/specifications/FEAT-05-client-portal-access-magic-link-login/FEAT-05.SPEC-009-portal-record-first-view-capture.md` (section: Edge Cases)

## Requirements (mandatory)

### Functional Requirements

- **FR-001**: The system MUST implement **FEAT-05.SPEC-001** (Request Sign-In Link) as specified: A client contact enters their email and requests a one-time sign-in link, without ever seeing an account-creation step. Full spec: `docs/blueprint/specifications/FEAT-05-client-portal-access-magic-link-login/FEAT-05.SPEC-001-request-sign-in-link.md`
- **FR-002**: The system MUST implement **FEAT-05.SPEC-002** (Link Verification Landing) as specified: The contact sees their emailed link being verified, then either enters the portal or sees a plain explanation with a one-tap way to request a fresh link. Full spec: `docs/blueprint/specifications/FEAT-05-client-portal-access-magic-link-login/FEAT-05.SPEC-002-link-verification-landing.md`
- **FR-003**: The system MUST implement **FEAT-05.SPEC-003** (Portal Home) as specified: The contact sees their company's projects, each project's current stage and milestones, and exactly what is waiting on them, scoped to what their role allows. Full spec: `docs/blueprint/specifications/FEAT-05-client-portal-access-magic-link-login/FEAT-05.SPEC-003-portal-home.md`
- **FR-004**: The system MUST implement **FEAT-05.SPEC-004** (Magic Link Issuance) as specified: Generates a single-use, time-limited sign-in link for a recognized contact and invalidates any prior unused link for that contact. Full spec: `docs/blueprint/specifications/FEAT-05-client-portal-access-magic-link-login/FEAT-05.SPEC-004-magic-link-issuance.md`
- **FR-005**: The system MUST implement **FEAT-05.SPEC-005** (Magic Link Verification) as specified: Validates a clicked link, creates the scoped session, and records the sign-in. Full spec: `docs/blueprint/specifications/FEAT-05-client-portal-access-magic-link-login/FEAT-05.SPEC-005-magic-link-verification.md`
- **FR-006**: The system MUST implement **FEAT-05.SPEC-006** (Link Validity & Recognition Rules) as specified: Governs single-use and time-limited link enforcement, invalidation on re-request, and that only recognized, Active contacts can obtain or use a link. Full spec: `docs/blueprint/specifications/FEAT-05-client-portal-access-magic-link-login/FEAT-05.SPEC-006-link-validity-recognition-rules.md`
- **FR-007**: The system MUST implement **FEAT-05.SPEC-007** (Portal Access & Isolation Rules) as specified: Governs client isolation, role-scoped portal display, and multi-freelancer separation for a contact who serves several freelancers. Full spec: `docs/blueprint/specifications/FEAT-05-client-portal-access-magic-link-login/FEAT-05.SPEC-007-portal-access-isolation-rules.md`
- **FR-008**: The system MUST implement **FEAT-05.SPEC-008** (Magic Link Sign-In Email) as specified: Emails the one-time sign-in link to the requesting contact whenever a link is issued, so they can enter their scoped portal without a password. Full spec: `docs/blueprint/specifications/FEAT-05-client-portal-access-magic-link-login/FEAT-05.SPEC-008-magic-link-sign-in-email.md`
- **FR-009**: The system MUST implement **FEAT-05.SPEC-009** (Portal Record First-View Capture) as specified: Detects a client contact's first view of a proposal, deliverable, or invoice reached through the portal and hands the timestamped event to FEAT-13's audit trail. Full spec: `docs/blueprint/specifications/FEAT-05-client-portal-access-magic-link-login/FEAT-05.SPEC-009-portal-record-first-view-capture.md`

### Key Entities

- Client Contact (authenticate)

## Success Criteria (mandatory)

### Measurable Outcomes

- **SC-001**: At least 95% of magic-link sign-in attempts succeed on the first try, and a failed or expired attempt can be recovered with one additional request (metric: Client Portal Login Success). Source: `docs/blueprint/features/success-metrics.md`
- **SC-002**: Client-facing pages (deliverable review, approval, invoice payment) load and become interactive within 2 seconds on a typical mobile connection (metric: Client Portal Mobile Responsiveness). Source: `docs/blueprint/features/success-metrics.md`
- **SC-003**: Magic-link requests, uses, expiries and portal home views are each observable as distinct signals (magic_link_requested, magic_link_used, magic_link_expired, portal_home_viewed). Source: `docs/blueprint/features/product-features.md`

## Assumptions

- **ASMP-02**: Client contacts review and act primarily from mobile browsers. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-03**: Email is the sole client-facing channel and contacts monitor it. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-23**: Strict data isolation between clients, enforced by the scoped portal session. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-21**: Client-facing pages become interactive within roughly 2 seconds on a typical mobile connection. Full register: `docs/blueprint/features/assumptions-constraints.md`

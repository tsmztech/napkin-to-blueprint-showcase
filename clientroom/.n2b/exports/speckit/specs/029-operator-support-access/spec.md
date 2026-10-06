# Feature Specification: Operator Support Access

**Blueprint feature:** FEAT-31
**Priority tier:** Important
**Build order:** 029 of 33
**Depends on:** FEAT-13, FEAT-14
**Blueprint source:** `docs/blueprint/specifications/FEAT-31-operator-support-access/`

## User Scenarios & Testing (mandatory)

### User Story 1 - Contact Support Screen (Priority: P2)

Nadia describes a problem she is having and sends a support request from inside the product, and receives an on-screen and emailed confirmation that it was received.

**Acceptance Scenarios:**

**FEAT-31.SPEC-001-AC-01:** Given Nadia is on the Contact Support screen, when she types a description of her problem and taps Send, then the Support Access Session record is created with her account and description, an on-screen confirmation "We received your request -- Dana will get back to you by email." appears, and the confirmation email (FEAT-31.SPEC-006) is triggered.

**FEAT-31.SPEC-001-AC-02:** Given Nadia is on the Contact Support screen, when she taps Send with the problem description empty, then the field shows the error "Describe your problem before sending." and the send does not proceed.

**FEAT-31.SPEC-001-AC-03:** Given Nadia has typed an unsent description, when she taps the back arrow, then a confirmation dialog appears asking "You have an unsent message. Discard?" with "Discard" and "Keep Editing" options.

**FEAT-31.SPEC-001-AC-04:** Given Nadia loses connectivity while filling the form, when she taps Send, then the error banner "Could not send your request. Check your connection and try again." appears and her entered text is preserved.

**FEAT-31.SPEC-001-AC-05:** Given Owen or Priya is signed in to their portal, when they look for a way to contact support, then no such entry exists anywhere in their view.

**FEAT-31.SPEC-001-AC-06:** Given Nadia already has one unopened support request pending, when she submits a second, different request, then both appear as separate entries in Dana's queue (FEAT-31.SPEC-002).

**FEAT-31.SPEC-001-AC-07:** Given Nadia's session has expired while she was typing a problem description, when the expiry is detected, then the dialog "Your session has expired. Sign in to continue." appears and her typed text is restored after she signs back in.

**FEAT-31.SPEC-001-AC-08:** Given Nadia taps Send twice in rapid succession, when the first send is still in progress, then the second tap has no effect and the button remains in its loading state.

**FEAT-31.SPEC-001-AC-09:** Given Nadia's send request fails on the server, when the failure is returned, then the error banner "Could not send your request. Check your connection and try again." appears with a Retry option and her entered text remains in the field.

### User Story 2 - Operator Support Session Console (Priority: P2)

Dana sees the queue of open support requests, opens a read-only session on one named freelancer's account, works through that freelancer's own screens under a permanent read-only banner, and steps away when done -- the session then ends on its own after inactivity.

**Acceptance Scenarios:**

**FEAT-31.SPEC-002-AC-01:** Given Dana opens the Support Session Console with two pending requests, when the queue loads, then both appear oldest first, each with the freelancer's account name, a preview of the request text, and the submission time.

**FEAT-31.SPEC-002-AC-02:** Given Dana has no session open, when she taps "Open Session" on a queued request, then FEAT-31.SPEC-003 opens the session and the mirrored view appears under the "Read-only support session -- {freelancer_account name}" banner.

**FEAT-31.SPEC-002-AC-03:** Given Dana is inside an open session, when she looks at any edit, send, approve, or pay control on a mirrored screen, then it is disabled or not shown, and a direct attempt shows "Not available in a support session."

**FEAT-31.SPEC-002-AC-04:** Given Dana is inside an open session, when she attempts to download a deliverable file, then the control is not offered and a direct attempt shows "Downloads are not available in a support session."

**FEAT-31.SPEC-002-AC-05:** Given Dana already has a session open on one freelancer account, when she taps "Open Session" on a different queued request, then the attempt is refused with "Your support session on {freelancer_account name} is still open and ends automatically after inactivity. Tap Resume to return to it.", the "Resume" notice remains shown, and no second session opens.

**FEAT-31.SPEC-002-AC-06:** Given Dana is inside an open session, when she taps "Return to queue," then the queue view appears with a persistent "Session open on {freelancer_account name} -- Resume" notice, and the session itself remains open.

**FEAT-31.SPEC-002-AC-07:** Given Dana's open session has just closed automatically after inactivity (FEAT-31.SPEC-004) while she was viewing a mirrored screen, then she is returned to the queue view immediately with a notice naming the freelancer and stating the session closed after inactivity.

**FEAT-31.SPEC-002-AC-08:** Given a queued request's account data cannot be loaded read-only, when Dana taps "Open Session" on it, then nothing about the account changes and she sees the specific reason inline on that row.

**FEAT-31.SPEC-002-AC-09:** Given Dana taps "Open Session" twice in rapid succession on the same request, when the first attempt is still in progress, then the second tap has no effect.

**FEAT-31.SPEC-002-AC-10:** Given no support requests are pending, when Dana opens the console, then it shows "No open support requests."

**FEAT-31.SPEC-002-AC-11:** Given Dana is inside a session viewing payment settings, when she looks for the payment connection detail, then she sees connection status only, never the processor account reference or credentials.

**FEAT-31.SPEC-002-AC-12:** Given Dana opens the console, when the queue fetch is in progress, then the body shows "Loading support requests..." with no rows and no "Open Session" controls.

**FEAT-31.SPEC-002-AC-13:** Given the queue fetch fails, when the console finishes loading, then the body shows "Support requests could not be loaded right now." with a "Retry" button; when Dana taps "Retry" and the fetch succeeds, then the queue (or "No open support requests.") appears.

**FEAT-31.SPEC-002-AC-14:** Given a session could not open and the row shows its inline reason with a "Retry" button, when Dana taps "Retry," then the row returns to its loading state and the open action runs again for that same request.

**FEAT-31.SPEC-002-AC-15:** Given Dana has a session open and has returned to the queue, when she taps "Open Session" on a different request, then she sees no manual close instruction, only the message naming the open account, stating it ends automatically after inactivity, and pointing to Resume, and tapping "Resume" returns her to the open session.

### User Story 3 - Support Session Open & Read-Only Enforcement (Priority: P2)

Opens the Support Access Session record the instant Dana starts a session, and enforces that every edit, send, approve, pay, file-download, and export-generation control is unavailable for the session's duration.

**Acceptance Scenarios:**

**FEAT-31.SPEC-003-AC-01:** Given Dana selects a queued request with no other session currently open, when she taps "Open Session," then `operator` and `opened_at` are set on the record, the mirrored read-only view appears with the permanent banner, the opened notice (FEAT-31.SPEC-007) is triggered, and the trail entry is written.

**FEAT-31.SPEC-003-AC-02:** Given Dana already has a session open on one freelancer account, when she attempts to open a second, then authorization fails, no record changes, and she sees "Your support session on {freelancer_account name} is still open and ends automatically after inactivity. Tap Resume to return to it."

**FEAT-31.SPEC-003-AC-03:** Given the selected request's freelancer account cannot be loaded read-only, when this automation attempts to open the session, then `operator` and `opened_at` remain unset and Dana sees the specific reason inline.

**FEAT-31.SPEC-003-AC-04:** Given a session has opened successfully, when Dana views any mirrored screen, then every edit, send, approve, pay, file-download, and export-generation control is disabled or not shown.

**FEAT-31.SPEC-003-AC-05:** Given Dana attempts to open two different requests from two browser tabs at effectively the same time, when both "Open Session" taps fire, then only the first to complete its authorization check succeeds and the second is refused with "Your support session on {freelancer_account name} is still open and ends automatically after inactivity. Tap Resume to return to it."

**FEAT-31.SPEC-003-AC-06:** Given this automation is still processing a prior "Open Session" tap for a request, when Dana taps the same request's "Open Session" control again, then no second run starts because the control is disabled while the first is in flight.

**FEAT-31.SPEC-003-AC-07:** Given the account loads read-only and `operator`/`opened_at` are set but the FEAT-13.SPEC-003 session-opened trail entry cannot be written, when this automation handles the failure, then `operator` and `opened_at` are unset together, no notice is triggered, the mirrored view never appears, and Dana sees "The session could not be started right now. Nothing was opened." with a "Retry" button while the request stays in the queue.

**FEAT-31.SPEC-003-AC-08:** Given the trail entry was written but FEAT-31.SPEC-007 does not accept the notice trigger, when this automation handles the failure, then `operator` and `opened_at` are unset together, a "Support session did not start" trail entry follows the earlier entry, the mirrored view never appears, and Dana sees the same message with "Retry."

**FEAT-31.SPEC-003-AC-09:** Given a prior attempt was rolled back by AC-07 or AC-08, when Dana taps "Retry" and the trail entry and notice trigger both succeed, then the session opens with the permanent banner and Nadia's trail shows the session-opened entry.

### User Story 4 - Support Session Auto-Close on Inactivity (Priority: P2)

Closes an open Support Access Session automatically after a period of inactivity, recording the close time and returning Dana to the queue.

**Acceptance Scenarios:**

**FEAT-31.SPEC-004-AC-01:** Given Dana's open session has had no action for platform parameter: `support-session-inactivity-timeout-minutes`, when this automation evaluates it, then `closed_at` is set, Dana is returned to the queue with a notice naming the freelancer and stating the session closed after inactivity, and the trail entry is written with closure reason "inactivity."

**FEAT-31.SPEC-004-AC-02:** Given Dana performs an action one second before the inactivity threshold would be reached, when this automation next evaluates the session, then the last-action timestamp has reset and the session remains open.

**FEAT-31.SPEC-004-AC-03:** Given a session has already been closed by an earlier evaluation cycle, when a second evaluation cycle reaches the same session, then no further change is made and no duplicate notice or trail entry is produced.

**FEAT-31.SPEC-004-AC-04:** Given two inactivity-evaluation cycles reach the same open session at effectively the same time, when both attempt to close it, then only the first sets `closed_at` and the second finds it already closed.

**FEAT-31.SPEC-004-AC-05:** Given the freelancer account behind an open session is deleted (FEAT-24) while the session is still open, when the deletion completes, then this automation takes no further action on that now-removed record.

**FEAT-31.SPEC-004-AC-06:** Given Dana has just closed her previous session automatically, when she returns to the queue, then no other session for her remains open, so no second close-in-flight scenario can occur for her.

**FEAT-31.SPEC-004-AC-07:** Given the inactivity threshold is reached and the `closed_at` write fails, when this automation handles the failure, then `closed_at` stays unset, no trail entry is written, the session remains open and read-only with the banner still shown, and Dana is not returned to the queue; on the next evaluation cycle the close is re-attempted if she has still taken no action.

**FEAT-31.SPEC-004-AC-08:** Given `closed_at` has been set but the FEAT-13.SPEC-003 session-closed trail entry cannot be written, when this automation handles the failure, then the session stays closed, Dana is returned to the queue with the normal notice, and the trail entry is re-attempted on each later evaluation cycle until accepted, producing exactly one session-closed entry with closure reason "inactivity."

**FEAT-31.SPEC-004-AC-09:** Given a `closed_at` write failed and Dana performs an action before the next evaluation cycle, when this automation next evaluates the session, then the last-action timestamp has reset and the session stays open.

### User Story 5 - Support Access Authorization & Read-Only Rules (Priority: P2)

Governs who may open, view, or never see a support session, the one-account-at-a-time and unconditional read-only constraints, the excluded actions (file downloads, data/accounting export generation, signing in as a client contact), and Nadia's standing right to see every session on her account.

**Acceptance Scenarios:**

**FEAT-31.SPEC-005-AC-01:** Given Nadia is submitting a support request, when she leaves the problem description empty and attempts to send, then she sees "Describe your problem before sending." and the send does not proceed.

**FEAT-31.SPEC-005-AC-02:** Given Nadia enters a problem description of exactly 2000 characters, when she sends it, then it is accepted; at 2001 characters, she sees "Please shorten your message to 2000 characters or fewer."

**FEAT-31.SPEC-005-AC-03:** Given a session is opened by FEAT-31.SPEC-003, when the record is written, then `operator` and `opened_at` are both set together -- neither is ever set without the other.

**FEAT-31.SPEC-005-AC-04:** Given a session's `opened_at` is unset, then FEAT-31.SPEC-004's auto-close evaluation never considers it, since `closed_at` can only be set on a record where `opened_at` is already set.

**FEAT-31.SPEC-005-AC-05:** Given Nadia is signed in to her own account, when she opens her activity trail, then she sees every support session on her account, past and present.

**FEAT-31.SPEC-005-AC-06:** Given Owen or Priya is signed in to their client portal, when they look anywhere in their product view, then no support session content or entry is ever shown to them.

**FEAT-31.SPEC-005-AC-07:** Given Dana is signed in as the operator, when she opens the Support Session Console, then she sees the queue of pending requests.

**FEAT-31.SPEC-005-AC-08:** Given Nadia, Owen, or Priya attempts to reach the Support Session Console, then it does not exist anywhere in their product.

**FEAT-31.SPEC-005-AC-09:** Given Dana has no session currently open, when she opens one on a queued request, then the open succeeds.

**FEAT-31.SPEC-005-AC-10:** Given Dana already has a session open, when she attempts to open a second, then she sees "Your support session on {freelancer_account name} is still open and ends automatically after inactivity. Tap Resume to return to it." with no instruction to close anything, the open session stays open, and the second session does not open.

**FEAT-31.SPEC-005-AC-11:** Given Dana is inside an open session, when she views the freelancer's account data, then she sees it exactly as the freelancer would, for the duration of the session only.

**FEAT-31.SPEC-005-AC-12:** Given Dana is inside an open session, when she attempts any edit, send, approve, or pay action, then the control is not shown as available, and a direct attempt shows "Not available in a support session."

**FEAT-31.SPEC-005-AC-13:** Given Dana is inside an open session, when she attempts to download a deliverable file, then she sees "Downloads are not available in a support session."

**FEAT-31.SPEC-005-AC-14:** Given Dana is inside an open session, when she attempts to generate a data or accounting export, then she sees "Exports are not available in a support session."

**FEAT-31.SPEC-005-AC-15:** Given Dana is diagnosing a client-side sign-in problem, when she looks for a way to sign in as the client contact, then no such control exists anywhere in the product.

**FEAT-31.SPEC-005-AC-16:** Given Dana is inside an open session viewing account settings, when she looks for Nadia's sign-in credentials, then they are never displayed.

**FEAT-31.SPEC-005-AC-17:** Given Dana is inside an open session viewing payment settings, when she looks for the processor account reference, then only the connection status is shown, never the reference itself.

**FEAT-31.SPEC-005-AC-18:** Given Dana is inside an open session, when she looks for a way to end it manually, then no such control exists anywhere -- the session remains open until it closes automatically on inactivity.

**FEAT-31.SPEC-005-AC-19:** Given Dana attempts to construct a direct link to a file bypassing the rendered console, when the request reaches the authorization boundary, then it is refused with the same exact denied message as the rendered control.

**FEAT-31.SPEC-005-AC-20:** Given a queued request's freelancer account begins deletion (FEAT-24) before Dana opens it, when she attempts to open it, then the request has already been removed from the queue as part of that deletion.

**FEAT-31.SPEC-005-AC-21:** Given Nadia enters only whitespace in the problem description, when she attempts to send, then she sees "Describe your problem before sending." exactly as if the field were empty.

**FEAT-31.SPEC-005-AC-22:** Given no card data of any kind exists in the product, when Dana views any screen inside a session, then no card data is ever shown, because none is ever held.

**FEAT-31.SPEC-005-AC-23:** Given Dana has one session open in one browser tab, when she attempts to open a different request in a second tab, then the second attempt is refused with "Your support session on {freelancer_account name} is still open and ends automatically after inactivity. Tap Resume to return to it." even though the first tab's screen shows no visible change.

**FEAT-31.SPEC-005-AC-24:** Given a session has already closed (`closed_at` set), when anyone looks for a way to re-open it from the queue, then it is not offered there -- it was already removed from the pending queue the moment it opened, and closing does not return it.

### User Story 6 - Support Request Confirmation (Priority: P2)

Sends Nadia a confirmation email the moment her support request is received, so she knows it reached Dana even after she leaves the Contact Support screen.

**Acceptance Scenarios:**

**FEAT-31.SPEC-006-AC-01:** Given Nadia submits a support request describing a payment connection problem, when the Support Access Session record is created, then she receives an email with the subject "We received your support request" quoting her exact description back to her.

**FEAT-31.SPEC-006-AC-02:** Given Nadia submits two separate support requests on the same day, when both are created, then she receives two separate confirmation emails, each quoting its own request, never combined into one.

**FEAT-31.SPEC-006-AC-03:** Given this email fails to deliver on its first attempt, when the retry window runs, then it is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window` before being surfaced as a delivery warning on Nadia's account.

**FEAT-31.SPEC-006-AC-04:** Given this email exhausts all retries without delivering, when the final attempt fails, then no further attempt is made and Dana still sees the request in her queue regardless.

**FEAT-31.SPEC-006-AC-05:** Given Nadia's sign-in email is mid-change (pending re-verification) when this notification is triggered, then it is sent to whichever address is current and verified at the moment of the send attempt.

**FEAT-31.SPEC-006-AC-06:** Given Nadia's account is deleted before this email's send attempt completes, when the deletion finishes first, then this confirmation is never sent.

**FEAT-31.SPEC-006-AC-07:** Given Dana opens a session on Nadia's account moments after the request is submitted, when this confirmation email is delivered, then its content still describes only the original request, not the session opening.

**FEAT-31.SPEC-006-AC-08:** Given Nadia has no notification preferences that could disable this email, when a request is submitted, then the confirmation always sends -- there is no control anywhere that turns it off.

### User Story 7 - Support Session Opened Notice (Priority: P2)

Sends Nadia an email notice whenever a support session opens on her account, so the operator's access is always visibly announced to her, never silent.

**Acceptance Scenarios:**

**FEAT-31.SPEC-007-AC-01:** Given Dana opens a session on Nadia's account at 2:14 PM (her local time), when FEAT-31.SPEC-003 sets `operator` and `opened_at`, then Nadia receives an email with the subject "A support session was opened on your account" naming that exact time.

**FEAT-31.SPEC-007-AC-02:** Given this notice is delivered, when Nadia taps "View activity trail," then she lands on her Activity Trail (FEAT-13.SPEC-001) and sees the session entry.

**FEAT-31.SPEC-007-AC-03:** Given the session Dana opened closes automatically within a minute of opening, when this notice's delivery attempt runs, then it still sends normally.

**FEAT-31.SPEC-007-AC-04:** Given Nadia has three pending, unopened support requests and Dana opens a session on only one, when this notice is delivered, then it describes only the session that opened, not the other pending requests.

**FEAT-31.SPEC-007-AC-05:** Given this email fails to deliver on its first attempt, when the retry window runs, then it is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window` before being surfaced as a delivery warning on Nadia's account.

**FEAT-31.SPEC-007-AC-06:** Given this email exhausts all retries without delivering, when the final attempt fails, then Nadia's activity trail entry for the session remains visible regardless.

**FEAT-31.SPEC-007-AC-07:** Given Nadia's account is deleted before this email's send attempt completes, when the deletion finishes first, then this notice is never sent.

**FEAT-31.SPEC-007-AC-08:** Given Nadia has no notification preferences that could disable this email, when a session opens on her account, then the notice always sends -- there is no control anywhere that turns it off.

### Edge Cases

- **FEAT-31.SPEC-001 (Contact Support Screen):** Unsent text triggers a Discard / Keep Editing confirmation, double taps on Send are ignored, and a network failure preserves the text with a retry banner. A second request while an earlier one is unopened creates a separate Support Access Session record, both appearing independently in the operator's queue. Source: `docs/blueprint/specifications/FEAT-31-operator-support-access/FEAT-31.SPEC-001-contact-support-screen.md` (section: Edge Cases)
- **FEAT-31.SPEC-002 (Operator Support Session Console):** A session that cannot open (account data not loadable read-only) changes nothing and shows the reason inline on the queue row, double taps are ignored, and opening a second session while one is open is refused with a resume prompt. A failed or slow queue fetch shows Loading then a Load-error state with Retry without affecting an open session. Source: `docs/blueprint/specifications/FEAT-31-operator-support-access/FEAT-31.SPEC-002-operator-support-session-console.md` (section: Edge Cases)
- **FEAT-31.SPEC-003 (Support Session Open & Read-Only Enforcement):** An account deleted before the session loads gives a load failure with the request removed from the queue, and an authorization pass followed by a load failure leaves operator and opened_at unset. Concurrent open attempts are each checked against the one-account-at-a-time rule, and a failed trail entry or notice rolls back so the request returns to the pending queue. Source: `docs/blueprint/specifications/FEAT-31-operator-support-access/FEAT-31.SPEC-003-support-session-open-read-only-enforcement.md` (section: Edge Cases)
- **FEAT-31.SPEC-004 (Support Session Auto-Close on Inactivity):** An action at the exact moment of the inactivity threshold resets the timestamp and cancels that cycle's close, and an account deleted while a session is open removes the session as part of deletion. Evaluation is idempotent across overlapping cycles, and since one session per operator is open at a time no duplicate run exists. Source: `docs/blueprint/specifications/FEAT-31-operator-support-access/FEAT-31.SPEC-004-support-session-auto-close-on-inactivity.md` (section: Edge Cases)
- **FEAT-31.SPEC-005 (Support Access Authorization & Read-Only Rules):** A directly constructed link to a file or export is refused with the same denial message as the UI, and authorization is checked at the Open Session selection rather than retained from an earlier load (a second open elsewhere is refused). A request for an account that has begun deletion cannot be opened, and request text of exactly 2,000 characters passes while 2,001 fails. Source: `docs/blueprint/specifications/FEAT-31-operator-support-access/FEAT-31.SPEC-005-support-access-authorization-read-only-rules.md` (section: Edge Cases)
- **FEAT-31.SPEC-006 (Support Request Confirmation):** The confirmation always describes exactly what was sent since requests cannot be edited or withdrawn, a session opened before delivery produces its own separate notice, and the email goes to the currently verified sign-in address. An account deleted before sending cancels the pending notification. Source: `docs/blueprint/specifications/FEAT-31-operator-support-access/FEAT-31.SPEC-006-support-request-confirmation.md` (section: Edge Cases)
- **FEAT-31.SPEC-007 (Support Session Opened Notice):** The notice still sends even if the session auto-closes immediately, it describes only the session that actually opened (never other pending requests), and deletion before send cancels delivery. If every retry fails the activity trail entry (FEAT-13) remains the authoritative record, satisfying ASMP-23. Source: `docs/blueprint/specifications/FEAT-31-operator-support-access/FEAT-31.SPEC-007-support-session-opened-notice.md` (section: Edge Cases)

## Requirements (mandatory)

### Functional Requirements

- **FR-001**: The system MUST implement **FEAT-31.SPEC-001** (Contact Support Screen) as specified: Nadia describes a problem she is having and sends a support request from inside the product, and receives an on-screen and emailed confirmation that it was received. Full spec: `docs/blueprint/specifications/FEAT-31-operator-support-access/FEAT-31.SPEC-001-contact-support-screen.md`
- **FR-002**: The system MUST implement **FEAT-31.SPEC-002** (Operator Support Session Console) as specified: Dana sees the queue of open support requests, opens a read-only session on one named freelancer's account, works through that freelancer's own screens under a permanent read-only banner, and steps away when done -- the session then ends on its own after inactivity. Full spec: `docs/blueprint/specifications/FEAT-31-operator-support-access/FEAT-31.SPEC-002-operator-support-session-console.md`
- **FR-003**: The system MUST implement **FEAT-31.SPEC-003** (Support Session Open & Read-Only Enforcement) as specified: Opens the Support Access Session record the instant Dana starts a session, and enforces that every edit, send, approve, pay, file-download, and export-generation control is unavailable for the session's duration. Full spec: `docs/blueprint/specifications/FEAT-31-operator-support-access/FEAT-31.SPEC-003-support-session-open-read-only-enforcement.md`
- **FR-004**: The system MUST implement **FEAT-31.SPEC-004** (Support Session Auto-Close on Inactivity) as specified: Closes an open Support Access Session automatically after a period of inactivity, recording the close time and returning Dana to the queue. Full spec: `docs/blueprint/specifications/FEAT-31-operator-support-access/FEAT-31.SPEC-004-support-session-auto-close-on-inactivity.md`
- **FR-005**: The system MUST implement **FEAT-31.SPEC-005** (Support Access Authorization & Read-Only Rules) as specified: Governs who may open, view, or never see a support session, the one-account-at-a-time and unconditional read-only constraints, the excluded actions (file downloads, data/accounting export generation, signing in as a client contact), and Nadia's standing right to see every session on her account. Full spec: `docs/blueprint/specifications/FEAT-31-operator-support-access/FEAT-31.SPEC-005-support-access-authorization-read-only-rules.md`
- **FR-006**: The system MUST implement **FEAT-31.SPEC-006** (Support Request Confirmation) as specified: Sends Nadia a confirmation email the moment her support request is received, so she knows it reached Dana even after she leaves the Contact Support screen. Full spec: `docs/blueprint/specifications/FEAT-31-operator-support-access/FEAT-31.SPEC-006-support-request-confirmation.md`
- **FR-007**: The system MUST implement **FEAT-31.SPEC-007** (Support Session Opened Notice) as specified: Sends Nadia an email notice whenever a support session opens on her account, so the operator's access is always visibly announced to her, never silent. Full spec: `docs/blueprint/specifications/FEAT-31-operator-support-access/FEAT-31.SPEC-007-support-session-opened-notice.md`

### Key Entities

- Support Access Session (create, read)
- Activity Log Entry (create)
- Freelancer Account (read)

## Success Criteria (mandatory)

### Measurable Outcomes

- **SC-001**: Support requests sent, support sessions opened and support sessions closed are each observable as distinct signals (support_request_sent, support_session_opened, support_session_closed); no metric in the success-metrics register connects to this feature, so the outcome is grounded in its Signals alone. Source: `docs/blueprint/features/product-features.md`

## Assumptions

- **ASMP-18**: Support access is read-only and always visible to the freelancer, announced and kept in her activity trail. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-23**: Any operator access is read-only and visible to the freelancer. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-13**: Solo-freelancer-only platform with the Support Operator as a distinct, bounded role. Full register: `docs/blueprint/features/assumptions-constraints.md`

# Feature Specification: Settings & Account Management

**Blueprint feature:** FEAT-21
**Priority tier:** Important
**Build order:** 027 of 33
**Depends on:** FEAT-14
**Blueprint source:** `docs/blueprint/specifications/FEAT-21-settings-account-management/`

## User Scenarios & Testing (mandatory)

### User Story 1 - Account Profile (Priority: P2)

Nadia views and edits her name and account profile, and reaches every other Settings section (Notification Preferences, Login & Security, Business Details & Payment Terms, Branding, and account closure) from here; Dana views her name read-only inside a logged support session (never her sign-in email).

**Acceptance Scenarios:**

**FEAT-21.SPEC-001-AC-01:** Given Nadia is on the Account Profile screen, when she clears the name field and taps Save, then the name field shows an error state with the message "Name is required" and the save does not proceed.

**FEAT-21.SPEC-001-AC-02:** Given Nadia enters "Nadia Voss" as her name and taps Save, then the profile saves, a "Profile updated" toast appears, and the field shows "Nadia Voss".

**FEAT-21.SPEC-001-AC-03:** Given Nadia is on the Account Profile screen, when she taps "Change" next to her sign-in email, then she is navigated to FEAT-21.SPEC-003 (Login & Security).

**FEAT-21.SPEC-001-AC-04:** Given Nadia taps "Close account", then she is navigated to FEAT-24 (Data Export & Account Deletion) and no closure or deletion happens inside this screen.

**FEAT-21.SPEC-001-AC-05:** Given Nadia taps "Branding" from the navigation shell, then she is navigated to FEAT-19.SPEC-001 (Branding Settings).

**FEAT-21.SPEC-001-AC-06:** Given Dana (Support Operator) opens this screen inside a logged FEAT-31 support session, when she views the profile, then she sees the current name only -- no sign-in email row, no Save button, no "Change" link, no delivery-warning banner, and neither a "Login & Security" nor a "Close account" item in the navigation shell.

**FEAT-21.SPEC-001-AC-07:** Given Dana (Support Operator) is viewing the profile read-only, when she attempts to submit a change through any means, then the attempt is refused with "Support sessions are read-only."

**FEAT-21.SPEC-001-AC-08:** Given Owen (Client Primary Contact) is signed in, when he looks for a Settings entry in navigation, then none is shown.

**FEAT-21.SPEC-001-AC-09:** Given Nadia's profile save fails server-side, when the failure occurs, then the error banner "Could not save your profile. Check your connection and try again." appears with a Retry button, and her entered name remains in the field.

**FEAT-21.SPEC-001-AC-10:** Given Nadia has an unsaved name edit, when she navigates away, then a confirmation dialog appears: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.

**FEAT-21.SPEC-001-AC-11:** Given Nadia saves her name from a second open session while this screen is also open in a first session with a different unsaved name, when the first session's Save is tapped, then the first session's value overwrites the second session's saved value (last-write-wins), with no merge dialog shown.

**FEAT-21.SPEC-001-AC-12:** Given Nadia loses connectivity while on this screen, then the Save button is disabled and a banner reads "You're offline. Reconnect to save changes."

**FEAT-21.SPEC-001-AC-13:** Given Nadia's expired session is detected while she has an unsaved name edit, when the "Your session has expired" dialog is dismissed by signing in again, then her unsaved name edit is restored on this screen.

**FEAT-21.SPEC-001-AC-14:** Given FEAT-21.SPEC-011 has reported a confirmation-email delivery failure after final retry, when Nadia opens this screen, then the banner "We couldn't deliver the confirmation email for your recent sign-in email change. If you didn't make this change, open Login & Security and sign out other devices." appears above the form with "Go to Login & Security" and "Dismiss".

**FEAT-21.SPEC-001-AC-15:** Given the delivery-warning banner is showing, when Nadia taps "Go to Login & Security", then she is navigated to FEAT-21.SPEC-003; when she taps "Dismiss", then the banner is removed and does not reappear on later visits for that failure.

**FEAT-21.SPEC-001-AC-16:** Given a delivery failure is unacknowledged, when Dana (Support Operator) opens this screen inside a logged support session, then no delivery-warning banner is rendered for her.

**FEAT-21.SPEC-001-AC-17:** Given Nadia is on the Account Profile screen, when she taps "Payment account" in the navigation shell, then she is navigated to FEAT-32.SPEC-001 (Payment Account connection screen), and when she taps that screen's back arrow she returns to this screen; and given Dana (Support Operator) is inside a logged support session, then no "Payment account" item is rendered in her navigation shell.

### User Story 2 - Notification Preferences (Priority: P2)

Nadia turns optional notifications on or off, with transactional record emails always shown locked-on; Dana views the same preferences read-only inside a logged support session.

**Acceptance Scenarios:**

**FEAT-21.SPEC-002-AC-01:** Given Nadia is on the Notification Preferences screen, when she views a transactional notification type, then it shows locked-on with an "Always sent" label and no toggle interaction is available.

**FEAT-21.SPEC-002-AC-02:** Given Nadia is on the Notification Preferences screen, when she turns an optional notification type off, then the toggle switches off, briefly shows "Saved", and the preference is saved.

**FEAT-21.SPEC-002-AC-03:** Given Nadia turns an optional notification type back on, then the toggle switches on and is saved.

**FEAT-21.SPEC-002-AC-04:** Given Nadia's toggle save fails server-side, when the failure occurs, then the toggle reverts to its prior position and shows "Could not save. Try again."

**FEAT-21.SPEC-002-AC-05:** Given Dana (Support Operator) opens this screen inside a logged FEAT-31 support session, when she views the preferences, then every toggle -- transactional and optional -- is shown as a static, disabled indicator.

**FEAT-21.SPEC-002-AC-06:** Given Dana (Support Operator) is viewing preferences read-only, when she attempts to change a toggle through any means, then the attempt is refused with "Support sessions are read-only."

**FEAT-21.SPEC-002-AC-07:** Given Owen (Client Primary Contact) is signed in, when he looks for a Settings entry in navigation, then none is shown.

**FEAT-21.SPEC-002-AC-08:** Given Nadia taps the same optional toggle twice rapidly, when the first save is still in progress, then the second tap is ignored.

**FEAT-21.SPEC-002-AC-09:** Given Nadia loses connectivity on this screen, then all toggles are disabled and a banner reads "You're offline. Reconnect to change preferences."

**FEAT-21.SPEC-002-AC-10:** Given a new optional notification type is added by FEAT-14 while Nadia's screen is already open, when she reopens or refreshes the screen, then the new type appears with its default preference applied.

**FEAT-21.SPEC-002-AC-11:** Given Nadia toggles a preference in one open session, when a second open session of hers with the same screen is refreshed, then it shows the newly saved state for that preference.

### User Story 3 - Login & Security (Priority: P2)

Nadia starts a sign-in email or login method change, views and manages her signed-in devices, and signs out other sessions; never surfaced to Dana or any client contact.

**Acceptance Scenarios:**

**FEAT-21.SPEC-003-AC-01:** Given Nadia is on the Login & Security screen, when she taps "Change sign-in email" and enters a new valid email, then FEAT-21.SPEC-005 starts the change and the screen shows "Verification pending for {new email}".

**FEAT-21.SPEC-003-AC-02:** Given Nadia enters her currently active sign-in email as the "new" email, when she submits, then she sees "This is already your sign-in email." and no pending change is created.

**FEAT-21.SPEC-003-AC-03:** Given Nadia has a pending sign-in email change, when she taps "Resend link", then a new verification link is sent and she sees "A new verification link has been sent."

**FEAT-21.SPEC-003-AC-04:** Given Nadia has a pending sign-in email change, when she taps "Cancel", then the pending banner clears, she sees "Sign-in email change cancelled.", and her prior sign-in email remains active.

**FEAT-21.SPEC-003-AC-05:** Given Nadia has a pending change, when she taps "Use a different email" on the pending banner and submits a second, different valid email, then the pending banner updates to "Verification pending for {newest email}" and the "Change sign-in email" button remains hidden.

**FEAT-21.SPEC-003-AC-06:** Given Nadia's pending change expires while she is on this screen, then the pending banner clears automatically and the "Change sign-in email" button reappears.

**FEAT-21.SPEC-003-AC-07:** Given Nadia is signed in on two other devices in addition to her current one, when she opens this screen, then the device list shows all three sessions with the current one labeled "This device".

**FEAT-21.SPEC-003-AC-08:** Given Nadia has one other active session, when she taps "Sign out other devices" and confirms, then FEAT-21.SPEC-006 runs and the device list refreshes to show only "This device", with the toast "All other sessions signed out."

**FEAT-21.SPEC-003-AC-09:** Given Nadia has no other active sessions, when she views this screen, then "Sign out other devices" is not shown.

**FEAT-21.SPEC-003-AC-10:** Given Dana (Support Operator) is inside a logged FEAT-31 support session, when she views the Settings navigation shell, then "Login & Security" is not listed and this screen is not reachable.

**FEAT-21.SPEC-003-AC-11:** Given Owen (Client Primary Contact) is signed in, when he looks for a Settings entry in navigation, then none is shown.

**FEAT-21.SPEC-003-AC-12:** Given Nadia's email-change submission fails server-side, when the failure occurs, then the error banner "Could not complete this action. Check your connection and try again." appears with a Retry button.

**FEAT-21.SPEC-003-AC-13:** Given Nadia loses connectivity on this screen, then all actions are disabled and a banner reads "You're offline. Reconnect to manage login and security."

**FEAT-21.SPEC-003-AC-14:** Given Nadia's expired session is detected while she has an unsubmitted new-email entry in the open change form, when she signs in again, then the entry is restored on this screen.

### User Story 4 - Business Details & Payment Terms (Priority: P2)

Nadia sets her business name, address, tax ID, and default payment terms that print on every invoice; Dana views the same values read-only inside a logged support session.

**Acceptance Scenarios:**

**FEAT-21.SPEC-004-AC-01:** Given Nadia is on the Business Details & Payment Terms screen with business name empty, when she taps Save, then the business name field shows the advisory "Business name is required before invoicing", the save still proceeds for the other fields with the toast "Business details updated", and the completeness indicator reads "Business details are incomplete -- required before your first invoice can be sent."

**FEAT-21.SPEC-004-AC-02:** Given Nadia fills in business name, business address, and default payment terms, when she taps Save, then the details save, a "Business details updated" toast appears, and the completeness indicator reads "Business details are complete."

**FEAT-21.SPEC-004-AC-03:** Given Nadia leaves the tax ID field empty, when she saves with the other required fields complete, then the save succeeds and the completeness indicator reads "Business details are complete." (tax ID is optional).

**FEAT-21.SPEC-004-AC-04:** Given Nadia arrives from FEAT-21.SPEC-009's incomplete-details prompt, when the screen loads, then the banner "Complete your business details before sending your first invoice." appears above the form.

**FEAT-21.SPEC-004-AC-05:** Given Dana (Support Operator) opens this screen inside a logged FEAT-31 support session, when she views the fields, then she sees the current values with no Save button.

**FEAT-21.SPEC-004-AC-06:** Given Dana (Support Operator) is viewing business details read-only, when she attempts to submit a change through any means, then the attempt is refused with "Support sessions are read-only."

**FEAT-21.SPEC-004-AC-07:** Given Owen (Client Primary Contact) is signed in, when he looks for a Settings entry in navigation, then none is shown.

**FEAT-21.SPEC-004-AC-08:** Given Nadia's business details save fails server-side, when the failure occurs, then the error banner "Could not save your business details. Check your connection and try again." appears with a Retry button, and her entered fields remain populated.

**FEAT-21.SPEC-004-AC-09:** Given Nadia has an unsaved edit, when she navigates away, then a confirmation dialog appears: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.

**FEAT-21.SPEC-004-AC-10:** Given Nadia clears a previously filled business address, leaving it empty, when she saves, then the save succeeds with the toast "Business details updated" and the completeness indicator switches to "Business details are incomplete -- required before your first invoice can be sent."

**FEAT-21.SPEC-004-AC-11:** Given Nadia loses connectivity on this screen, then the Save button is disabled and a banner reads "You're offline. Reconnect to save changes."

**FEAT-21.SPEC-004-AC-12:** Given Nadia's expired session is detected while she has unsaved field edits, when she signs in again, then her unsaved edits are restored on this screen.

**FEAT-21.SPEC-004-AC-13:** Given Nadia fills only the tax ID and leaves business name, business address, and default payment terms empty, when she taps Save, then the tax ID is saved, the toast "Business details updated" appears, each empty required field shows its advisory message, and the indicator reads "Business details are incomplete -- required before your first invoice can be sent."

### User Story 5 - Sign-In Email & Login Method Change (Priority: P2)

Processes a pending sign-in email or login method change through re-verification before it takes effect, and expires or lets Nadia resend an unconfirmed change.

**Acceptance Scenarios:**

**FEAT-21.SPEC-005-AC-01:** Given Nadia submits a new, valid sign-in email on FEAT-21.SPEC-003, when this automation processes the submission, then a pending change is created, a re-verification link is sent to the new email, and FEAT-21.SPEC-003 shows "Verification pending for {new email}".

**FEAT-21.SPEC-005-AC-02:** Given Nadia has a pending change, when she taps "Resend link", then a fresh token is issued, a new link is sent, and the pending change's expiry resets to a full platform parameter: `email-change-reverification-window`.

**FEAT-21.SPEC-005-AC-03:** Given Nadia follows a valid, non-expired re-verification link, when this automation processes it, then her sign-in email is updated to the target email, the pending change clears, and FEAT-21.SPEC-011 sends the account-critical change confirmation email.

**FEAT-21.SPEC-005-AC-04:** Given Nadia has a pending change, when she taps "Cancel", then the pending change clears and her prior sign-in email remains active with no confirmation email sent.

**FEAT-21.SPEC-005-AC-05:** Given Nadia's pending change reaches platform parameter: `email-change-reverification-window` with no completed re-verification, when this automation's expiry check runs, then the pending change clears and her prior sign-in email remains active.

**FEAT-21.SPEC-005-AC-06:** Given Nadia has an already-confirmed change, when the same re-verification link is followed a second time, then she sees "This verification link has already been used or has expired." and no further data changes.

**FEAT-21.SPEC-005-AC-07:** Given Nadia taps "Use a different email" on FEAT-21.SPEC-003's pending banner and submits a different valid email while one change is already pending, when this automation processes the new submission, then the prior pending change and its token are invalidated and replaced by the new target email.

**FEAT-21.SPEC-005-AC-08:** Given a re-verification token is presented at or after its expiry timestamp, when this automation checks it, then the presenter sees "This verification link has expired. Start the change again." and no account field changes.

**FEAT-21.SPEC-005-AC-09:** Given the transactional email delivery capability is unavailable when a re-verification email is due to send, when the delivery capability recovers before the pending change's expiry, then the queued email is delivered and the change remains eligible for confirmation.

**FEAT-21.SPEC-005-AC-10:** Given the transactional email delivery capability remains unavailable past the pending change's expiry, when the expiry check runs, then the change expires per FEAT-21.SPEC-005-AC-05, regardless of the undelivered email.

**FEAT-21.SPEC-005-AC-11:** Given Nadia taps "Resend link" while a prior resend request for the same change is still processing, when the second tap occurs, then it is ignored because FEAT-21.SPEC-003 disables the control during the in-flight request.

**FEAT-21.SPEC-005-AC-12:** Given this automation's commit step fails after a valid token is presented, when the failure occurs, then no sign-in email change is applied, the pending change remains active, and FEAT-21.SPEC-003 shows "Could not complete this action. Check your connection and try again." with Retry.

### User Story 6 - Sign-Out Other Sessions (Priority: P2)

Invalidates every one of Nadia's signed-in sessions except the current one and records the event, when she taps "Sign out other devices".

**Acceptance Scenarios:**

**FEAT-21.SPEC-006-AC-01:** Given Nadia has two other active sessions in addition to her current one, when she confirms "Sign out other devices", then both other sessions are invalidated, her current session remains active, and she sees "All other sessions signed out."

**FEAT-21.SPEC-006-AC-02:** Given Nadia's other session was already invalidated by the time her confirmed request is processed (it ended on its own between load and request), when this automation runs, then it completes with zero sessions affected and she still sees "All other sessions signed out."

**FEAT-21.SPEC-006-AC-03:** Given a signed-out device attempts its next action after invalidation, then that action is rejected as unauthenticated and the device is returned to the sign-in screen.

**FEAT-21.SPEC-006-AC-04:** Given Nadia taps "Sign out other devices" from two of her own sessions at effectively the same time, when the first request's invalidation commits, then the second triggering session is itself signed out and is redirected to sign in again.

**FEAT-21.SPEC-006-AC-05:** Given Nadia taps "Sign out other devices" a second time while the first request is still processing, then the second tap is ignored because the button is in a loading state.

**FEAT-21.SPEC-006-AC-06:** Given the session invalidation step fails, when the failure occurs, then no sessions are invalidated and FEAT-21.SPEC-003 shows "Could not sign out other sessions. Try again."

**FEAT-21.SPEC-006-AC-07:** Given a device Nadia just signed out attempts to reconnect immediately afterward, then it is treated as a fresh, unauthenticated session and must sign in again.

**FEAT-21.SPEC-006-AC-08:** Given this automation completes successfully, then a sign-out event is recorded for the account.

**FEAT-21.SPEC-006-AC-09:** Given Nadia has no other active sessions and the "Sign out other devices" button is therefore not shown on FEAT-21.SPEC-003, then this automation is never triggered from that state.

### User Story 7 - Account Field Validation Rules (Priority: P2)

Enforces required fields, format, and value limits across profile, business details, and payment terms fields on the Freelancer Account.

**Acceptance Scenarios:**

**FEAT-21.SPEC-007-AC-01:** Given Nadia clears the name field, when she blurs it, then she sees "Name is required."

**FEAT-21.SPEC-007-AC-02:** Given Nadia enters a name of exactly 100 characters, when she saves, then it is accepted; entering 101 characters shows "Name must be 100 characters or fewer."

**FEAT-21.SPEC-007-AC-03:** Given Nadia submits a malformed new sign-in email, when she submits the change form, then she sees "Please enter a valid email address."

**FEAT-21.SPEC-007-AC-04:** Given Nadia submits her current sign-in email (including a case-only difference) as the "new" one, when she submits, then she sees "This is already your sign-in email." and no pending change is created.

**FEAT-21.SPEC-007-AC-05:** Given Nadia leaves business_name empty and attempts to save Business Details & Payment Terms, when she saves, then she sees the advisory "Business name is required before invoicing." and the save still succeeds (non-blocking), leaving completeness incomplete.

**FEAT-21.SPEC-007-AC-06:** Given Nadia leaves business_address empty and attempts to save, then she sees the advisory "Business address is required before invoicing." and the save still succeeds, leaving completeness incomplete.

**FEAT-21.SPEC-007-AC-07:** Given Nadia does not select a default_payment_terms value and attempts to save, then she sees the advisory "Choose a default payment term before invoicing." and the save still succeeds, leaving completeness incomplete.

**FEAT-21.SPEC-007-AC-08:** Given Nadia leaves tax_id empty and saves with the other required fields complete, then the save succeeds with no required-field error for tax_id.

**FEAT-21.SPEC-007-AC-09:** Given Nadia enters a tax_id of 51 characters, when she blurs the field, then she sees "Tax ID must be 50 characters or fewer."

**FEAT-21.SPEC-007-AC-10:** Given Nadia has filled business_name, business_address, and default_payment_terms, then FEAT-21.SPEC-009 evaluates business details as complete (the cross-field rule's condition is satisfied).

**FEAT-21.SPEC-007-AC-11:** Given Nadia (Freelancer) is on any Settings screen, when she performs an edit action on her own account, then it is always allowed.

**FEAT-21.SPEC-007-AC-12:** Given Dana (Support Operator) is inside a logged support session, when she looks for any save control on any Settings screen, then none is shown, and a direct attempt to submit a change is refused with "Support sessions are read-only."

**FEAT-21.SPEC-007-AC-13:** Given Dana (Support Operator) is inside a logged support session, when she looks for the sign-in email or signed-in devices, then neither is ever shown to her.

**FEAT-21.SPEC-007-AC-14:** Given Owen (Client Primary Contact) attempts to reach any Settings screen, then he finds none in navigation and no direct access exists.

**FEAT-21.SPEC-007-AC-15:** Given Priya (Client Reviewer Contact) attempts to reach any Settings screen, then she finds none in navigation and no direct access exists.

### User Story 8 - Notification Preference Rules (Priority: P2)

Enforces that only optional notifications can be switched off and that transactional record emails always send (XBR-30).

**Acceptance Scenarios:**

**FEAT-21.SPEC-008-AC-01:** Given Nadia views the Notification Preferences screen, when she looks at a Transactional notification type, then it shows locked-on with no toggle control.

**FEAT-21.SPEC-008-AC-02:** Given Nadia toggles an Optional notification type off, when the toggle saves, then FEAT-14 will not send that type to her going forward until she turns it back on.

**FEAT-21.SPEC-008-AC-03:** Given Nadia toggles an Optional notification type back on, when the toggle saves, then FEAT-14 resumes sending that type to her.

**FEAT-21.SPEC-008-AC-04:** Given a forged or malformed request attempts to toggle a Transactional type off, when this rule evaluates the request, then it is rejected with "Transactional emails cannot be turned off." and the type remains On.

**FEAT-21.SPEC-008-AC-05:** Given FEAT-14 reclassifies a previously Optional type as Transactional, when Nadia's screen next loads, then that type appears in the locked Transactional group with no toggle, regardless of its prior off/on preference.

**FEAT-21.SPEC-008-AC-06:** Given FEAT-14 introduces a new Optional type, when Nadia's screen next loads, then the new type appears with a toggle defaulted to On.

**FEAT-21.SPEC-008-AC-07:** Given Dana (Support Operator) is inside a logged support session, when she views notification preferences, then every toggle -- Transactional or Optional -- is a static, disabled indicator, and a direct change attempt is refused with "Support sessions are read-only."

**FEAT-21.SPEC-008-AC-08:** Given Owen (Client Primary Contact) attempts to reach the Notification Preferences screen, then he finds none in navigation and no direct access exists.

**FEAT-21.SPEC-008-AC-09:** Given a notification of an Optional type was already queued before Nadia turns that type off, when the toggle saves, then the already-queued notification is still delivered.

### User Story 9 - Business Details Completeness Gate (Priority: P2)

Tracks whether business details and payment terms are complete and blocks the first invoice send until they are (XBR-16).

**Acceptance Scenarios:**

**FEAT-21.SPEC-009-AC-01:** Given Nadia has business_name, business_address, and default_payment_terms all filled, when this rule evaluates completeness, then the result is complete and FEAT-21.SPEC-004 shows "Business details are complete."

**FEAT-21.SPEC-009-AC-02:** Given Nadia has business_address empty, when this rule evaluates completeness, then the result is incomplete and FEAT-21.SPEC-004 shows "Business details are incomplete -- required before your first invoice can be sent."

**FEAT-21.SPEC-009-AC-03:** Given Nadia's business details are incomplete, when a first-invoice send is attempted through FEAT-09, then FEAT-09 blocks the send and directs her to complete business details in Settings.

**FEAT-21.SPEC-009-AC-04:** Given Nadia's business details are complete, when a first-invoice send is attempted through FEAT-09, then the send proceeds and is not blocked by this rule.

**FEAT-21.SPEC-009-AC-05:** Given Nadia leaves tax_id empty while the three required fields are filled, when this rule evaluates completeness, then the result is complete (tax_id is not part of the determination).

**FEAT-21.SPEC-009-AC-06:** Given Nadia clears default_payment_terms after previously completing business details, when she saves, then the save succeeds (it is not blocked) and completeness reverts to incomplete.

**FEAT-21.SPEC-009-AC-10:** Given Nadia fills only tax_id and leaves business_name, business_address, and default_payment_terms empty, when she saves on FEAT-21.SPEC-004, then the save succeeds and this rule evaluates completeness as incomplete.

**FEAT-21.SPEC-009-AC-07:** Given Dana (Support Operator) is inside a logged support session, when she views the completeness indicator, then she sees its current state read-only with no ability to change the underlying fields.

**FEAT-21.SPEC-009-AC-08:** Given Owen (Client Primary Contact) attempts to reach any Settings screen showing completeness, then he finds none in navigation and no direct access exists.

**FEAT-21.SPEC-009-AC-09:** Given all three required fields contain only whitespace, when this rule evaluates completeness, then the result is incomplete, since whitespace-only values are treated as empty.

### User Story 10 - Settings Access & Read-Only Scope Rules (Priority: P2)

Enforces that only Nadia can edit her own account, that Dana's support session sees profile/preference/business values read-only and never sign-in credentials, and that client contacts have no settings surface at all.

**Acceptance Scenarios:**

**FEAT-21.SPEC-010-AC-01:** Given Nadia is signed in, when she opens any Settings screen, then she has full view and edit access to her own account.

**FEAT-21.SPEC-010-AC-02:** Given Dana (Support Operator) has no support session open on a freelancer's account, when she attempts to reach that freelancer's Settings, then no path exists -- she has no standing access.

**FEAT-21.SPEC-010-AC-03:** Given Dana (Support Operator) is inside a logged FEAT-31 support session on a freelancer's account, when she views Account Profile, Notification Preferences, or Business Details & Payment Terms, then she sees the current values with no save controls and no destructive actions.

**FEAT-21.SPEC-010-AC-04:** Given Dana (Support Operator) is inside a logged support session, when she attempts to submit a change on any of those three screens through any means, then the attempt is refused with "Support sessions are read-only."

**FEAT-21.SPEC-010-AC-05:** Given Dana (Support Operator) is inside a logged support session, when she looks at the Settings navigation shell, then "Login & Security" does not appear and no direct link reaches it.

**FEAT-21.SPEC-010-AC-06:** Given Dana (Support Operator) is inside a logged support session, when she looks at the Settings navigation shell, then "Close account" does not appear.

**FEAT-21.SPEC-010-AC-07:** Given Owen (Client Primary Contact) is signed in, when he looks for a Settings entry in navigation or follows a direct link to one, then no Settings entry is shown in navigation, and the direct link shows the heading "This link isn't valid anymore" with the line "Sign-in links are single-use and time-limited, and yours has expired, already been used, or was requested again since." and a "Send me a new link" button (FEAT-05.SPEC-002 Error state), with no Settings content exposed.

**FEAT-21.SPEC-010-AC-08:** Given Priya (Client Reviewer Contact) is signed in, when she looks for a Settings entry in navigation or follows a direct link to one, then no Settings entry is shown in navigation, and the direct link shows the identical "This link isn't valid anymore" explanation and "Send me a new link" button, with no Settings content exposed.

**FEAT-21.SPEC-010-AC-09:** Given Dana's support session closes while she has a Settings screen open, when the session ends, then she is redirected out of the freelancer's account view.

**FEAT-21.SPEC-010-AC-10:** Given Nadia has her own Settings open while Dana has a concurrent read-only support session open on the same account, when Nadia saves a change, then Dana's view reflects the new value only on its own next refresh, with no interference to Nadia's save.

**FEAT-21.SPEC-010-AC-11:** Given Dana attempts to construct a direct link into Login & Security while her support session is open, when the link is followed, then the attempt is refused and the screen is not rendered.

**FEAT-21.SPEC-010-AC-12:** Given a client contact follows an old bookmarked Settings URL, when the link is followed, then the same denial applies regardless of the path taken: the heading "This link isn't valid anymore" with the FEAT-05.SPEC-002 explanation line and "Send me a new link" button is shown, and no Settings content is exposed.

### User Story 11 - Account-Critical Change Confirmation Email (Priority: P2)

Sends Nadia a confirmation email, to both her prior and her new sign-in email address, when her sign-in email address or login method actually changes, so a change she did not make is immediately visible to her at the address she held before the change.

**Acceptance Scenarios:**

**FEAT-21.SPEC-011-AC-01:** Given Nadia's sign-in email change is confirmed by FEAT-21.SPEC-005, when this notification fires, then one email with the subject "Your Clientroom sign-in email was changed" is sent to her prior sign-in address and one to her new sign-in address, each naming her prior and new email accurately.

**FEAT-21.SPEC-011-AC-02:** Given Nadia receives this confirmation email, when she taps "Go to Login & Security", then she lands on FEAT-21.SPEC-003 (Login & Security).

**FEAT-21.SPEC-011-AC-03:** Given Nadia changes her sign-in email twice in quick succession, when both changes confirm, then each confirmed change sends its own separate pair of confirmation emails (prior address and new address), never merged.

**FEAT-21.SPEC-011-AC-04:** Given there is no notification preference for this email, when Nadia's account has every optional notification turned off on FEAT-21.SPEC-002, then this confirmation is still sent for any confirmed sign-in email change.

**FEAT-21.SPEC-011-AC-05:** Given delivery of this email fails once for a transient reason, when the delivery capability retries within platform parameter: `transactional-email-retry-window`, then up to platform parameter: `transactional-email-retry-count` retries occur before any failure is surfaced to Nadia.

**FEAT-21.SPEC-011-AC-06:** Given delivery of this email fails after all retries are exhausted, when the final failure is processed, then the delivery-warning banner "We couldn't deliver the confirmation email for your recent sign-in email change. If you didn't make this change, open Login & Security and sign out other devices." appears on Nadia's Account Profile screen (FEAT-21.SPEC-001) until she taps "Dismiss".

**FEAT-21.SPEC-011-AC-07:** Given Nadia's Freelancer Account is deleted shortly after a change is confirmed but before this email is delivered, when delivery proceeds, then the email is still sent to both the prior and the new sign-in email address.

**FEAT-21.SPEC-011-AC-08:** Given FEAT-21.SPEC-005's re-verification link is followed a second time for an already-confirmed change, when that second follow is processed, then no second confirmation email is sent (only the first, successful commit triggers this notification).

**FEAT-21.SPEC-011-AC-09:** Given the new sign-in email address bounces on delivery, when the bounce is reported, then it is handled as any other delivery failure for that address under the Retry on failure rule, the copy to the prior address is unaffected, and the sign-in email change remains in effect regardless.

**FEAT-21.SPEC-011-AC-10:** Given the prior sign-in email address bounces on delivery, when the bounce is reported, then it is retried under the Retry on failure rule, the copy to the new address is still delivered, and after the final failure the delivery-warning banner appears on FEAT-21.SPEC-001.

**FEAT-21.SPEC-011-AC-11:** Given a sign-in email change is confirmed from `{prior_email}` to `{new_email}`, when this notification fires, then the only recipient addresses are `{prior_email}` and `{new_email}`, and no other address receives it.

**FEAT-21.SPEC-011-AC-12:** Given Owen, Priya, or Dana have no sign-in credential on the Freelancer Account to change, then none of them can ever trigger or receive this notification.

### Edge Cases

- **FEAT-21.SPEC-001 (Account Profile):** Unsaved edits trigger the discard confirmation, double taps on Save are ignored, and a server-side failure preserves the name with a retry banner. A concurrent change from a second session is last-write-wins per the Freelancer Account contention note. Source: `docs/blueprint/specifications/FEAT-21-settings-account-management/FEAT-21.SPEC-001-account-profile.md` (section: Edge Cases)
- **FEAT-21.SPEC-002 (Notification Preferences):** Double taps are ignored while a toggle saves, and a failed save reverts the toggle with an inline error and keeps the row interactive. A new optional type added by FEAT-14 appears on next open or refresh, and a toggle from another session shows on the next refresh. Source: `docs/blueprint/specifications/FEAT-21-settings-account-management/FEAT-21.SPEC-002-notification-preferences.md` (section: Edge Cases)
- **FEAT-21.SPEC-003 (Login & Security):** Submitting the current sign-in email is rejected inline with no pending change, and starting a second change replaces the pending banner and invalidates the prior verification. A pending change that expires clears the banner automatically with no data loss, and Sign out other devices is hidden when no other session exists. Source: `docs/blueprint/specifications/FEAT-21-settings-account-management/FEAT-21.SPEC-003-login-security.md` (section: Edge Cases)
- **FEAT-21.SPEC-004 (Business Details & Payment Terms):** Unsaved edits trigger the discard confirmation, double taps are ignored, and a server-side failure preserves entered fields with a retry banner. Concurrent edits from two sessions are last-write-wins per the Freelancer Account contention note. Source: `docs/blueprint/specifications/FEAT-21-settings-account-management/FEAT-21.SPEC-004-business-details-payment-terms.md` (section: Edge Cases)
- **FEAT-21.SPEC-005 (Sign-In Email & Login Method Change):** A re-verification link followed twice (such as email pre-fetching) commits once and the second use shows the no-longer-valid message, and a cancel racing a verification resolves to whichever the system processes first. The expiry check is authoritative, so a token presented at or after the expiry timestamp is treated as expired. Source: `docs/blueprint/specifications/FEAT-21-settings-account-management/FEAT-21.SPEC-005-sign-in-email-login-method-change.md` (section: Edge Cases)
- **FEAT-21.SPEC-006 (Sign-Out Other Sessions):** Signing out other devices with zero other sessions still completes with the confirmation toast, and another session mid-action gets its next request rejected as unauthenticated. Concurrent taps from two sessions resolve to whichever commits first, and double taps from one session are ignored via the loading state. Source: `docs/blueprint/specifications/FEAT-21-settings-account-management/FEAT-21.SPEC-006-sign-out-other-sessions.md` (section: Edge Cases)
- **FEAT-21.SPEC-007 (Account Field Validation Rules):** Boundaries are inclusive: a name of 100 characters and a business address of 500 pass while 101 and 501 fail, and whitespace-only Tax ID is treated as empty and trimmed. A new sign-in email differing from the current one only by case counts as the same email. Source: `docs/blueprint/specifications/FEAT-21-settings-account-management/FEAT-21.SPEC-007-account-field-validation-rules.md` (section: Edge Cases)
- **FEAT-21.SPEC-008 (Notification Preference Rules):** Reclassifying a type from Optional to Transactional discards any existing off preference, while the reverse adds a toggle defaulting to On on the next load. Each toggle is its own field-level write across sessions, and a forged request to turn off a transactional type is rejected with the transactional-emails-cannot-be-turned-off message. Source: `docs/blueprint/specifications/FEAT-21-settings-account-management/FEAT-21.SPEC-008-notification-preference-rules.md` (section: Edge Cases)
- **FEAT-21.SPEC-009 (Business Details Completeness Gate):** FEAT-09's own send-time check is authoritative for an attempt already mid-flight, and clearing business_address makes completeness revert to false immediately for future first-time gate checks without affecting a first invoice already sent (XBR-16). A partial save always succeeds with completeness recomputed. Source: `docs/blueprint/specifications/FEAT-21-settings-account-management/FEAT-21.SPEC-009-business-details-completeness-gate.md` (section: Edge Cases)
- **FEAT-21.SPEC-010 (Settings Access & Read-Only Scope Rules):** An operator's read-only scope is revoked immediately when the support session closes, Login & Security is never rendered inside a support session however it is reached, and a client contact following an old Settings link gets the same None denial with the not-valid-anymore explanation. Two simultaneous support sessions each keep read-only scope independently. Source: `docs/blueprint/specifications/FEAT-21-settings-account-management/FEAT-21.SPEC-010-settings-access-read-only-scope-rules.md` (section: Edge Cases)
- **FEAT-21.SPEC-011 (Account-Critical Change Confirmation Email):** Two quick sign-in email changes are each notified independently with their own pair of emails, and an account deleted before delivery still receives both emails as a final security record. The copy to the prior address with its did-not-make-this-change guidance is the safeguard against a compromised session, and a bounce on either address is retried then surfaced. Source: `docs/blueprint/specifications/FEAT-21-settings-account-management/FEAT-21.SPEC-011-account-critical-change-confirmation-email.md` (section: Edge Cases)

## Requirements (mandatory)

### Functional Requirements

- **FR-001**: The system MUST implement **FEAT-21.SPEC-001** (Account Profile) as specified: Nadia views and edits her name and account profile, and reaches every other Settings section (Notification Preferences, Login & Security, Business Details & Payment Terms, Branding, and account closure) from here; Dana views her name read-only inside a logged support session (never her sign-in email). Full spec: `docs/blueprint/specifications/FEAT-21-settings-account-management/FEAT-21.SPEC-001-account-profile.md`
- **FR-002**: The system MUST implement **FEAT-21.SPEC-002** (Notification Preferences) as specified: Nadia turns optional notifications on or off, with transactional record emails always shown locked-on; Dana views the same preferences read-only inside a logged support session. Full spec: `docs/blueprint/specifications/FEAT-21-settings-account-management/FEAT-21.SPEC-002-notification-preferences.md`
- **FR-003**: The system MUST implement **FEAT-21.SPEC-003** (Login & Security) as specified: Nadia starts a sign-in email or login method change, views and manages her signed-in devices, and signs out other sessions; never surfaced to Dana or any client contact. Full spec: `docs/blueprint/specifications/FEAT-21-settings-account-management/FEAT-21.SPEC-003-login-security.md`
- **FR-004**: The system MUST implement **FEAT-21.SPEC-004** (Business Details & Payment Terms) as specified: Nadia sets her business name, address, tax ID, and default payment terms that print on every invoice; Dana views the same values read-only inside a logged support session. Full spec: `docs/blueprint/specifications/FEAT-21-settings-account-management/FEAT-21.SPEC-004-business-details-payment-terms.md`
- **FR-005**: The system MUST implement **FEAT-21.SPEC-005** (Sign-In Email & Login Method Change) as specified: Processes a pending sign-in email or login method change through re-verification before it takes effect, and expires or lets Nadia resend an unconfirmed change. Full spec: `docs/blueprint/specifications/FEAT-21-settings-account-management/FEAT-21.SPEC-005-sign-in-email-login-method-change.md`
- **FR-006**: The system MUST implement **FEAT-21.SPEC-006** (Sign-Out Other Sessions) as specified: Invalidates every one of Nadia's signed-in sessions except the current one and records the event, when she taps "Sign out other devices". Full spec: `docs/blueprint/specifications/FEAT-21-settings-account-management/FEAT-21.SPEC-006-sign-out-other-sessions.md`
- **FR-007**: The system MUST implement **FEAT-21.SPEC-007** (Account Field Validation Rules) as specified: Enforces required fields, format, and value limits across profile, business details, and payment terms fields on the Freelancer Account. Full spec: `docs/blueprint/specifications/FEAT-21-settings-account-management/FEAT-21.SPEC-007-account-field-validation-rules.md`
- **FR-008**: The system MUST implement **FEAT-21.SPEC-008** (Notification Preference Rules) as specified: Enforces that only optional notifications can be switched off and that transactional record emails always send (XBR-30). Full spec: `docs/blueprint/specifications/FEAT-21-settings-account-management/FEAT-21.SPEC-008-notification-preference-rules.md`
- **FR-009**: The system MUST implement **FEAT-21.SPEC-009** (Business Details Completeness Gate) as specified: Tracks whether business details and payment terms are complete and blocks the first invoice send until they are (XBR-16). Full spec: `docs/blueprint/specifications/FEAT-21-settings-account-management/FEAT-21.SPEC-009-business-details-completeness-gate.md`
- **FR-010**: The system MUST implement **FEAT-21.SPEC-010** (Settings Access & Read-Only Scope Rules) as specified: Enforces that only Nadia can edit her own account, that Dana's support session sees profile/preference/business values read-only and never sign-in credentials, and that client contacts have no settings surface at all. Full spec: `docs/blueprint/specifications/FEAT-21-settings-account-management/FEAT-21.SPEC-010-settings-access-read-only-scope-rules.md`
- **FR-011**: The system MUST implement **FEAT-21.SPEC-011** (Account-Critical Change Confirmation Email) as specified: Sends Nadia a confirmation email, to both her prior and her new sign-in email address, when her sign-in email address or login method actually changes, so a change she did not make is immediately visible to her at the address she held before the change. Full spec: `docs/blueprint/specifications/FEAT-21-settings-account-management/FEAT-21.SPEC-011-account-critical-change-confirmation-email.md`

### Key Entities

- Freelancer Account (update)

## Success Criteria (mandatory)

### Measurable Outcomes

- **SC-001**: Settings updates, notification preference changes, account email changes, business details updates and other-session sign-outs are each observable as distinct signals (settings_updated, notification_preference_changed, account_email_changed, business_details_updated, other_sessions_signed_out); no metric in the success-metrics register connects to this feature, so the outcome is grounded in its Signals alone. Source: `docs/blueprint/features/product-features.md`

## Assumptions

- **ASMP-23**: Freelancers can export and delete their own data, and operator access is read-only and visible to the freelancer. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-24**: Invoices need both parties' business details, which is why business details completeness gates the first invoice. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-18**: Support access is read-only and always visible to the freelancer. Full register: `docs/blueprint/features/assumptions-constraints.md`

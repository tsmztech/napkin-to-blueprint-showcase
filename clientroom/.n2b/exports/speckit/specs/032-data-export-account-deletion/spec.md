# Feature Specification: Data Export & Account Deletion

**Blueprint feature:** FEAT-24
**Priority tier:** Important
**Build order:** 032 of 33
**Depends on:** FEAT-01, FEAT-33
**Blueprint source:** `docs/blueprint/specifications/FEAT-24-data-export-account-deletion/`

## User Scenarios & Testing (mandatory)

### User Story 1 - Data Export Screen (Priority: P2)

Nadia requests a full export of all her own data, tracks the archive's progress, and downloads it once ready.

**Acceptance Scenarios:**

**FEAT-24.SPEC-001-AC-01:** Given Nadia is on FEAT-21.SPEC-001 (Account Profile), when she taps "Close account," then she lands on this screen showing her account's current export status.

**FEAT-24.SPEC-001-AC-02:** Given Nadia is on the Data Export Screen with no prior export, when she taps "Request Export," then FEAT-24.SPEC-003 begins and the status card shows "Preparing your export -- this can take a few minutes for accounts with a lot of history."

**FEAT-24.SPEC-001-AC-03:** Given Nadia's archive is in the Requested/Generating state, when she looks at the Request Export button, then it is disabled and a second tap has no effect.

**FEAT-24.SPEC-001-AC-04:** Given FEAT-24.SPEC-003 reports the archive Ready, when Nadia views this screen, then the status card shows "Your export is ready." with a Download action and "Available until {expiry_date}."

**FEAT-24.SPEC-001-AC-05:** Given Nadia's archive is Ready, when she taps Download, then the file transfer begins through FEAT-16.SPEC-007 and, once it completes, the status card shows "Downloaded on {date}."

**FEAT-24.SPEC-001-AC-06:** Given Nadia's archive has passed its download window unfetched, when she views this screen, then the status card shows "Your last export has expired. Request a new one to download your data again." and the Download action is no longer shown.

**FEAT-24.SPEC-001-AC-07:** Given FEAT-24.SPEC-003 reports generation failure after exhausting its retries, when Nadia views this screen, then she sees the error banner "We couldn't generate your export. Try again." and the Request Export button is enabled.

**FEAT-24.SPEC-001-AC-08:** Given Nadia is on the Data Export Screen, when she taps "Continue to close your account," then she is navigated to FEAT-24.SPEC-002 (Account Deletion Screen).

**FEAT-24.SPEC-001-AC-09:** Given Nadia loses connectivity while this screen is open, when the connection drops, then the banner "You're offline. Reconnect to request or download your export." appears and both Request Export and Download are disabled.

**FEAT-24.SPEC-001-AC-10:** Given Owen (Client Primary Contact) has no navigation path to this screen, when he follows any link in the client portal, then he never reaches it.

**FEAT-24.SPEC-001-AC-11:** Given Dana (Support Operator) is in an active read-only support session, when she attempts to open this screen directly, then she sees "This isn't available during a support session," not a read-only view of Nadia's export.

**FEAT-24.SPEC-001-AC-12:** Given an unauthenticated visitor opens this screen's link, then they are redirected to sign-in and, after signing in, land on FEAT-12 (Freelancer Financial Dashboard), not this screen.

**FEAT-24.SPEC-001-AC-13:** Given Nadia's session expires while this screen is open, when she is shown the expired-session dialog and signs back in, then the screen reloads showing the current archive status exactly as it stood before expiry.

**FEAT-24.SPEC-001-AC-14:** Given Nadia's archive is already Ready, when she taps "Request New Export," then a fresh request supersedes the current archive per FEAT-24.SPEC-003, and the status card returns to Requested/Generating.

**FEAT-24.SPEC-001-AC-15:** Given Nadia has this screen open in two tabs and requests an export in one, when she switches to the other tab without reloading it, then that tab still shows its prior state until she reloads or re-navigates to it.

### User Story 2 - Account Deletion Screen (Priority: P2)

Nadia reviews any open-item warnings and gives the explicit confirmation required to permanently delete her account and all its data.

**Acceptance Scenarios:**

**FEAT-24.SPEC-002-AC-01:** Given Nadia is on FEAT-24.SPEC-001 and taps "Continue to close your account," when this screen loads, then it evaluates open items and retention determinations via FEAT-24.SPEC-006 before showing any warning or confirmation content.

**FEAT-24.SPEC-002-AC-02:** Given Nadia has an unpaid invoice, when this screen finishes loading, then a warning banner describes that specific consequence without disabling the confirmation controls.

**FEAT-24.SPEC-002-AC-03:** Given Nadia has no open items, when this screen finishes loading, then no warning banners appear and only the retention notice and confirmation region are shown.

**FEAT-24.SPEC-002-AC-04:** Given Nadia checks the acknowledgment checkbox but leaves the confirmation text empty, when she looks at "Delete My Account," then it remains disabled.

**FEAT-24.SPEC-002-AC-05:** Given Nadia checks the acknowledgment checkbox and types "delete" (lowercase), when she looks at "Delete My Account," then it remains disabled, since the match is case-sensitive.

**FEAT-24.SPEC-002-AC-06:** Given Nadia checks the acknowledgment checkbox and types "DELETE" exactly, when she looks at "Delete My Account," then it becomes enabled.

**FEAT-24.SPEC-002-AC-07:** Given Nadia has satisfied both confirmation conditions, when she taps "Delete My Account," then FEAT-24.SPEC-009 fires immediately and FEAT-24.SPEC-004 begins, and the screen shows "Deleting your account -- this can take a few minutes for accounts with a lot of history."

**FEAT-24.SPEC-002-AC-08:** Given deletion processing is underway, when Nadia looks at the screen, then every control is disabled and a second tap on Delete My Account has no effect.

**FEAT-24.SPEC-002-AC-09:** Given FEAT-24.SPEC-004 reports successful completion, when processing finishes, then Nadia is signed out and no screen in the product remains for her account.

**FEAT-24.SPEC-002-AC-10:** Given FEAT-24.SPEC-004 reports failure, when processing ends, then the screen shows "We couldn't complete account deletion. Your account has not been changed. Try again." with the checkbox and confirmation text cleared.

**FEAT-24.SPEC-002-AC-11:** Given Nadia taps Cancel at any point before confirming, when she does so, then no deletion action is taken and she returns to FEAT-24.SPEC-001.

**FEAT-24.SPEC-002-AC-12:** Given Owen (Client Primary Contact) has no navigation path to this screen, when he follows any link in the client portal, then he never reaches it.

**FEAT-24.SPEC-002-AC-13:** Given Dana (Support Operator) is in an active read-only support session, when she attempts to open this screen directly, then she sees "This isn't available during a support session."

**FEAT-24.SPEC-002-AC-14:** Given Nadia's session expires while she has partially filled the confirmation region, when she is shown the expired-session dialog and signs back in, then the confirmation region is reset (checkbox unchecked, text cleared) rather than restored.

**FEAT-24.SPEC-002-AC-15:** Given Nadia sends a new unpaid invoice in another tab after this screen has already loaded, when she taps "Delete My Account" here, then the open-item evaluation re-runs at that moment, the tap is rejected, the screen shows the new unpaid-invoice banner and the changed-warnings notice, and neither FEAT-24.SPEC-009 nor FEAT-24.SPEC-004 has fired.

**FEAT-24.SPEC-002-AC-16:** Given Nadia loses connectivity while reviewing this screen, when the connection drops, then the banner "You're offline. Reconnect to continue." appears and the confirmation region is disabled.

**FEAT-24.SPEC-002-AC-17:** Given the screen is in Loaded -- warnings changed after a rejected tap and the checkbox and text are still valid, when Nadia taps "Delete My Account" again and the re-check matches the warnings now displayed, then FEAT-24.SPEC-009 fires, FEAT-24.SPEC-004 begins, and the screen enters Processing.

**FEAT-24.SPEC-002-AC-18:** Given Nadia has 1 unpaid invoice of $1,200.00 displayed and pays it in another tab, when she taps "Delete My Account," then the tap is rejected, the unpaid-invoice banner is removed, the notice "Your open items changed. Review the updated warnings, then tap Delete My Account again." appears, and no notification or deletion is triggered.

**FEAT-24.SPEC-002-AC-19:** Given Nadia has an open invoice and a pending approval, when this screen loads, then the unpaid-invoice banner appears above the pending-approval banner and above the retention notice, each with the exact text defined in FEAT-24.SPEC-006.

### User Story 3 - Data Export Archive Generation (Priority: P2)

Aggregates every client, project, proposal, invoice, and activity record Nadia owns into a single downloadable archive, retries cleanly on failure, and expires the archive after its limited download window.

**Acceptance Scenarios:**

**FEAT-24.SPEC-003-AC-01:** Given Nadia taps Request Export on FEAT-24.SPEC-001 with no prior archive, when this automation runs, then it aggregates every owned record into a new archive and, on success, sets the archive to Ready with a download link/window of platform parameter: `data-export-download-window`.

**FEAT-24.SPEC-003-AC-02:** Given the archive reaches Ready, when this outcome applies, then FEAT-24.SPEC-008 fires and FEAT-24.SPEC-001 shows the Download action.

**FEAT-24.SPEC-003-AC-03:** Given the storage & delivery capability reports a completed download transfer for the archive, when this event arrives, then the archive's status advances from Ready to Downloaded and the Download action remains available.

**FEAT-24.SPEC-003-AC-04:** Given aggregation fails transiently on the first attempt, when the failure occurs, then generation retries automatically without exposing a partial archive to Nadia.

**FEAT-24.SPEC-003-AC-05:** Given every retry up to platform parameter: `data-export-generation-retry-count` fails, when the final retry fails, then the archive carries no download link and FEAT-24.SPEC-001 shows "We couldn't generate your export. Try again."

**FEAT-24.SPEC-003-AC-06:** Given an archive is Ready or Downloaded, when its download window (platform parameter: `data-export-download-window`) elapses, then its status becomes Expired and its underlying stored file is purged via FEAT-16.SPEC-007.

**FEAT-24.SPEC-003-AC-07:** Given Nadia requests a new export while a prior archive is Ready, Downloaded, or Expired, when the new request is received, then the prior archive and its file are discarded and a fresh archive begins generating.

**FEAT-24.SPEC-003-AC-08:** Given a freelancer account with almost no data, when an export is requested, then the archive still generates successfully, containing whatever records exist.

**FEAT-24.SPEC-003-AC-09:** Given the archive's download window elapses at the same moment Nadia's download transfer is already in flight, when both occur, then the in-flight transfer is allowed to complete and the archive is marked Downloaded rather than Expired.

**FEAT-24.SPEC-003-AC-10:** Given the archive's download window elapses before Nadia starts a download, when she then attempts to download, then the attempt fails with "This export has expired. Request a new one." and no file transfer occurs.

**FEAT-24.SPEC-003-AC-11:** Given Nadia requests a new export from two open tabs at effectively the same time, when both requests are processed, then exactly one active archive results, reflecting the later request.

**FEAT-24.SPEC-003-AC-12:** Given a generation run is already in flight when a new request for the same account arrives, when the new request is processed, then it supersedes the in-flight run so that only the new request's resulting archive is ever finalized.

**FEAT-24.SPEC-003-AC-13:** Given this automation aggregates data for one freelancer's export, when it runs, then it never includes another freelancer's records.

### User Story 4 - Account Deletion Processing (Priority: P2)

Cascades the permanent removal of the freelancer's account and all owned data across every data-holding feature once she confirms, disconnecting her payment account, and leaving the account fully intact if processing fails.

**Acceptance Scenarios:**

**FEAT-24.SPEC-004-AC-01:** Given Nadia has confirmed deletion on FEAT-24.SPEC-002, when this automation runs, then the Freelancer Account transitions through Deletion Requested and Deletion Confirmed before the hold phase (step 4) begins.

**FEAT-24.SPEC-004-AC-02:** Given the confirmation is received, when this automation starts, then FEAT-24.SPEC-009 fires in parallel without this automation waiting for that email to send.

**FEAT-24.SPEC-004-AC-03:** Given the cascade reaches an Invoice subject to legal retention, when FEAT-24.SPEC-006 flags it for retention, then that Invoice is held in a retained, inaccessible state rather than hard-deleted, and handed to FEAT-24.SPEC-005.

**FEAT-24.SPEC-004-AC-04:** Given the cascade proceeds, when it reaches the Payment Account Connection, then FEAT-32.SPEC-004 disconnects it in step 5, after every entity has been marked pending-delete, as the commit point of this cascade.

**FEAT-24.SPEC-004-AC-05:** Given the cascade proceeds, when it reaches the Activity Log Entries, then they are removed in step 7, after the commit point, under FEAT-13.SPEC-006's own retention rule, subject to the same financial-record retention exception.

**FEAT-24.SPEC-004-AC-06:** Given the cascade proceeds, when it reaches stored deliverable files and versions, then FEAT-16.SPEC-006 purges every stored byte in step 8, only after the commit point has passed.

**FEAT-24.SPEC-004-AC-07:** Given the commit point succeeds, when this automation reports the commit, then Nadia is signed out; and when every finalization step (7-9) has then succeeded, the Freelancer Account is set to Deleted and hard-deleted.

**FEAT-24.SPEC-004-AC-08:** Given the Payment Account Connection disconnect step fails after exhausting its own retries, when the failure is reported, then the cascade halts, every pending-delete and retained-hold marker is cleared, and the Freelancer Account reverts fully to Active with every entity as before.

**FEAT-24.SPEC-004-AC-09:** Given Nadia has open unpaid invoices or pending approvals at confirmation, when the cascade runs, then deletion still proceeds, with only the retention-flagged records held back.

**FEAT-24.SPEC-004-AC-10:** Given a Client Contact exists with personal data at deletion time, when the cascade reaches Client Contact, then the contact's name and email are removed in full.

**FEAT-24.SPEC-004-AC-11:** Given a confirmed-deletion instruction arrives for an account already in the Deleted state, when this automation processes it, then nothing changes and no error is raised.

**FEAT-24.SPEC-004-AC-12:** Given a second confirmed-deletion instruction for the same account arrives while a first cascade is already in flight, when it arrives, then it is discarded as redundant rather than starting a second concurrent cascade.

**FEAT-24.SPEC-004-AC-13:** Given two confirmed-deletion instructions for the same account arrive at effectively the same time, when both are processed, then only one cascade actually runs to completion.

**FEAT-24.SPEC-004-AC-14:** Given this automation processes a deletion for one freelancer's account, when the cascade runs, then it never touches another freelancer's data.

**FEAT-24.SPEC-004-AC-15:** Given the hold phase (step 4) is running, when Nadia's Client, Project, and Deliverable records are marked pending-delete, then no row or stored byte has been removed and clearing the markers restores each exactly as it was.

**FEAT-24.SPEC-004-AC-16:** Given the payment-account disconnect (step 5) has not yet succeeded, when any earlier step fails, then no Activity Log Entry has been removed and no stored byte has been purged, and the account reverts to Active.

**FEAT-24.SPEC-004-AC-17:** Given the commit point has passed and the stored-byte purge (step 8) fails, when the failure is reported, then the account is not reverted to Active, Nadia's data stays hidden, the step is retried at platform parameter: `deletion-finalization-retry-interval` until it succeeds, and account_deletion_failed is recorded with reverted: no.

**FEAT-24.SPEC-004-AC-18:** Given the hold phase (step 4) cannot write a pending-delete marker for one entity after its retries, when the failure is reported, then no payment-account disconnect is attempted, all markers written so far are cleared, and the account reverts to Active.

### User Story 5 - Legal Retention Purge (Priority: P2)

Purges the Invoice and Payment records held back from an otherwise-completed account deletion once their legal retention period lapses.

**Acceptance Scenarios:**

**FEAT-24.SPEC-005-AC-01:** Given an Invoice was held back by FEAT-24.SPEC-004 and its retention period (platform parameter: `financial-record-legal-retention-period`) has elapsed since deletion completion, when a scheduled sweep runs, then that Invoice is permanently purged.

**FEAT-24.SPEC-005-AC-02:** Given a Payment held back by FEAT-24.SPEC-004 and its retention period has elapsed, when a scheduled sweep runs, then that Payment is permanently purged.

**FEAT-24.SPEC-005-AC-03:** Given a retained record's elapsed time is below the retention threshold, when a scheduled sweep runs, then the record is left untouched.

**FEAT-24.SPEC-005-AC-04:** Given a purge attempt fails on its first try, when the failure occurs, then the record remains retained and is reprocessed at the next sweep.

**FEAT-24.SPEC-005-AC-05:** Given a record's retention period elapses between two scheduled sweeps, when the next sweep runs at or after the threshold, then the record is purged at that sweep, not exactly at the expiry instant.

**FEAT-24.SPEC-005-AC-06:** Given two scheduled sweeps fire close together, when the second sweep encounters a record the first sweep already purged, then no error occurs and the record is simply absent from the candidate set.

**FEAT-24.SPEC-005-AC-07:** Given a sweep is already in flight, when a new sweep is triggered before the first finishes, then the new sweep waits rather than processing the same candidates concurrently.

**FEAT-24.SPEC-005-AC-08:** Given no retained record has reached its threshold at a given sweep, when the sweep runs, then nothing changes.

**FEAT-24.SPEC-005-AC-09:** Given a retained record is purged, when the purge completes, then no restore path exists for it.

**FEAT-24.SPEC-005-AC-10:** Given Activity Log Entries under the same retention exception exist, when this automation runs, then it purges only Invoice and Payment records, leaving those entries to FEAT-13.SPEC-006's own purge.

### User Story 6 - Pre-Deletion Warning & Retention Determination Rules (Priority: P2)

Determines which open items (unpaid invoices, pending approvals) trigger a specific warning without blocking deletion, and which records are legally retained versus immediately deleted.

**Acceptance Scenarios:**

**FEAT-24.SPEC-006-AC-01:** Given Nadia has an Invoice in Sent status, when this determination runs, then open_unpaid_invoice_present evaluates true and FEAT-24.SPEC-002 shows the unpaid-invoice warning.

**FEAT-24.SPEC-006-AC-02:** Given Nadia has every Invoice in Paid status, when this determination runs, then open_unpaid_invoice_present evaluates false and no unpaid-invoice warning is shown.

**FEAT-24.SPEC-006-AC-03:** Given Nadia has a Milestone in Deliverable Uploaded status, when this determination runs, then pending_approval_present evaluates true and FEAT-24.SPEC-002 shows the pending-approval warning.

**FEAT-24.SPEC-006-AC-04:** Given Nadia has no Milestone in Deliverable Uploaded status, when this determination runs, then pending_approval_present evaluates false and no pending-approval warning is shown.

**FEAT-24.SPEC-006-AC-05:** Given Nadia has both an open invoice and a pending approval, when this determination runs, then both warnings are shown together on FEAT-24.SPEC-002.

**FEAT-24.SPEC-006-AC-06:** Given the cascade in FEAT-24.SPEC-004 reaches an Invoice record, when this determination runs, then it is classified Retained regardless of the invoice's own status field.

**FEAT-24.SPEC-006-AC-07:** Given the cascade reaches a Payment record, when this determination runs, then it is classified Retained regardless of the payment's own status field.

**FEAT-24.SPEC-006-AC-08:** Given the cascade reaches a Client Contact record, when this determination runs, then it is classified Immediately Deleted, with no retention exception for the contact's personal data.

**FEAT-24.SPEC-006-AC-09:** Given an invoice's status changes from Sent to Paid in another tab after Nadia loaded FEAT-24.SPEC-002, when she then confirms deletion, then this determination re-runs and the warning reflects the current Paid status.

**FEAT-24.SPEC-006-AC-10:** Given Nadia's account has zero invoices and zero milestones, when this determination runs, then no warnings are shown and no records are classified Retained.

**FEAT-24.SPEC-006-AC-11:** Given an Invoice is in Partially refunded status, when this determination runs, then it counts as open for warning purposes and is separately classified Retained for the cascade.

**FEAT-24.SPEC-006-AC-12:** Given a Milestone is in Reopened status, when this determination runs, then pending_approval_present does not count it as awaiting approval.

**FEAT-24.SPEC-006-AC-13:** Given Nadia (Freelancer) triggers this determination through FEAT-24.SPEC-002 or FEAT-24.SPEC-004, when it runs, then it evaluates only her own account's records.

**FEAT-24.SPEC-006-AC-14:** Given Owen, Priya, or Dana attempts to reach any spec that would trigger this determination, when they try, then it never runs on their behalf, since FEAT-24.SPEC-007 excludes them from every screen and automation that invokes it.

**FEAT-24.SPEC-006-AC-15:** Given Nadia has exactly 1 open Invoice with an outstanding amount of $1,200.00, when this determination runs, then the unpaid-invoice banner reads title "Unpaid invoices" and body "You have 1 unpaid invoice ($1,200.00). Deleting your account will not collect it, and your client will no longer be able to pay it."

**FEAT-24.SPEC-006-AC-16:** Given Nadia has 3 open Invoices totaling $1,000.00 and EUR 300.00, when this determination runs, then the banner body reads "You have 3 unpaid invoices ($1,000.00 + EUR 300.00). Deleting your account will not collect them, and your clients will no longer be able to pay them."

**FEAT-24.SPEC-006-AC-17:** Given Nadia has 2 Milestones in Deliverable Uploaded status, when this determination runs, then the pending-approval banner reads title "Deliverables awaiting approval" and body "2 deliverables are waiting for your clients' approval. Deleting your account will discard them without a decision, and your clients will lose access to them."

**FEAT-24.SPEC-006-AC-18:** Given any open-item state (including none), when FEAT-24.SPEC-002 loads, then the retention notice reads "Invoices and payment records are kept for {retention_period} after deletion, as legally required, and can no longer be accessed by you or your clients. Everything else, including your clients' names and email addresses, is deleted in full." with {retention_period} filled from platform parameter: `financial-record-legal-retention-period`.

### User Story 7 - Export & Deletion Access Rules (Priority: P2)

Restricts every export and deletion action in this feature to Nadia alone, with no access of any kind for client contacts or the Support Operator.

**Acceptance Scenarios:**

**FEAT-24.SPEC-007-AC-01:** Given Nadia (Freelancer) opens FEAT-24.SPEC-001, when the screen loads, then she can view and act on it in full.

**FEAT-24.SPEC-007-AC-02:** Given Nadia (Freelancer) opens FEAT-24.SPEC-002, when the screen loads, then she can view and act on it in full.

**FEAT-24.SPEC-007-AC-03:** Given Nadia triggers FEAT-24.SPEC-003 by requesting an export, when the trigger is received, then authorization passes since it is her own account.

**FEAT-24.SPEC-007-AC-04:** Given Nadia triggers FEAT-24.SPEC-004 by confirming deletion, when the trigger is received, then authorization passes since it is her own account.

**FEAT-24.SPEC-007-AC-05:** Given Owen (Client Primary Contact) attempts to reach FEAT-24.SPEC-001 via a direct link, when he does, then he sees the client portal's standard out-of-scope explanation, never this screen's content.

**FEAT-24.SPEC-007-AC-06:** Given Priya (Client Reviewer Contact) attempts to reach FEAT-24.SPEC-002 via a direct link, when she does, then she sees the same out-of-scope explanation as Owen.

**FEAT-24.SPEC-007-AC-07:** Given Dana (Support Operator) is in an active read-only support session on Nadia's account, when she attempts to open FEAT-24.SPEC-001 or FEAT-24.SPEC-002 directly, then she sees "This isn't available during a support session."

**FEAT-24.SPEC-007-AC-08:** Given Dana is in an active support session, when Nadia's export or deletion actions occur during that session, then Dana receives no indication of them through her support-session view, since this feature is excluded from it entirely.

**FEAT-24.SPEC-007-AC-09:** Given no client contact has any account-level surface to reach this feature from, when Owen or Priya browse their portal, then no navigation element anywhere leads to FEAT-24.

**FEAT-24.SPEC-007-AC-10:** Given Dana's support session grants View access to nearly every other feature, when she looks for an equivalent read-only view of this feature, then none exists -- her access here is None, not View.

**FEAT-24.SPEC-007-AC-11:** Given Nadia's own account is the only account she can act on, when she requests an export or confirms deletion, then the action is always scoped to her own account and never any other freelancer's.

**FEAT-24.SPEC-007-AC-12:** Given a support session is opened on Nadia's account while she has an in-progress (unconfirmed) deletion attempt, when Dana views her session, then Dana still has no access to any FEAT-24 screen or indication of that attempt.

### User Story 8 - Export Ready Notification (Priority: P2)

Emails Nadia when her requested data export archive is ready to download, so she learns about it even if she is away from the product while it generates.

**Acceptance Scenarios:**

**FEAT-24.SPEC-008-AC-01:** Given FEAT-24.SPEC-003 applies a Ready outcome to Nadia's archive, when this notification fires, then she receives an email with subject "Your data export is ready to download."

**FEAT-24.SPEC-008-AC-02:** Given Nadia opens the export-ready email, when she taps "Download your export," then she lands on FEAT-24.SPEC-001 (Data Export Screen).

**FEAT-24.SPEC-008-AC-03:** Given Nadia has no way to opt out of this email, when her notification preferences are checked, then no preference control exists for it and it always sends.

**FEAT-24.SPEC-008-AC-04:** Given the archive reaches Ready at any hour, when this notification fires, then it sends immediately regardless of Nadia's configured quiet hours.

**FEAT-24.SPEC-008-AC-05:** Given delivery fails on the first attempt, when the retry logic runs, then it retries up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`.

**FEAT-24.SPEC-008-AC-06:** Given every retry fails and the archive has since expired, when the final retry would otherwise be attempted, then the email is not sent.

**FEAT-24.SPEC-008-AC-07:** Given Nadia requests a new export before the prior archive's email is delivered, when the new request supersedes the prior archive, then the pending email for the superseded archive is cancelled.

**FEAT-24.SPEC-008-AC-08:** Given Nadia's account is deleted while this email is still queued for retry, when the deletion finalizes, then the pending delivery is cancelled.

**FEAT-24.SPEC-008-AC-09:** Given Owen, Priya, or Dana has no access to this entity, when the archive reaches Ready, then none of them receives any copy of this email.

**FEAT-24.SPEC-008-AC-10:** Given a delayed retry succeeds while the archive is still Ready, when the email is finally delivered, then it shows the archive's actual current expiry date.

**FEAT-24.SPEC-008-AC-11:** Given Nadia has already downloaded the archive by the time a delayed retry succeeds, when the email is delivered, then it still sends as an accurate record of the export becoming ready.

### User Story 9 - Account Deletion Final Warning Notification (Priority: P2)

Emails Nadia a final confirmation notice at the moment she confirms permanent account deletion, giving her a durable record of exactly when she gave that irreversible confirmation.

**Acceptance Scenarios:**

**FEAT-24.SPEC-009-AC-01:** Given Nadia gives explicit confirmation on FEAT-24.SPEC-002, when this notification fires, then she receives an email with subject "Your Clientroom account is being permanently deleted" carrying the exact confirmation timestamp.

**FEAT-24.SPEC-009-AC-02:** Given Nadia opens this email, when she looks for a way to undo or cancel the deletion, then no CTA or link of any kind offers one.

**FEAT-24.SPEC-009-AC-03:** Given Nadia has no way to opt out of this email, when her notification preferences are checked, then no preference control exists for it and it always sends.

**FEAT-24.SPEC-009-AC-04:** Given confirmation is given at any hour, when this notification fires, then it sends immediately regardless of Nadia's configured quiet hours.

**FEAT-24.SPEC-009-AC-05:** Given delivery fails on the first attempt, when the retry logic runs, then it retries up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`.

**FEAT-24.SPEC-009-AC-06:** Given every retry fails and the account has since been fully deleted, when the final retry would otherwise be attempted, then no further action is taken and no delivery warning is surfaced anywhere.

**FEAT-24.SPEC-009-AC-07:** Given this email has already been sent, when FEAT-24.SPEC-004's cascade subsequently fails and reverts the account to Active, then no correction or "never mind" email is sent.

**FEAT-24.SPEC-009-AC-08:** Given two deletion confirmations for the same account are submitted close together, when both reach FEAT-24.SPEC-004, then only the first produces this email.

**FEAT-24.SPEC-009-AC-09:** Given Owen, Priya, or Dana has no access to this entity, when Nadia confirms deletion, then none of them receives any copy of this email.

**FEAT-24.SPEC-009-AC-10:** Given this email is delivered, when Nadia reads it, then it names the exact date and time she confirmed, with no ambiguity about which action it refers to.

### Edge Cases

- **FEAT-24.SPEC-001 (Data Export Screen):** Double taps on Request Export are ignored while disabled, a network failure creates no archive record and returns the screen to its prior state, and the screen is a snapshot so a second tab or an archive expiring while idle updates only on reload or manual refresh. Source: `docs/blueprint/specifications/FEAT-24-data-export-account-deletion/FEAT-24.SPEC-001-data-export-screen.md` (section: Edge Cases)
- **FEAT-24.SPEC-002 (Account Deletion Screen):** Open items and retention determinations are re-checked at the moment of confirmation, so a new invoice sent in another tab is reflected. The deletion cascade continues server-side if she navigates away (she is signed out on return), double taps are ignored, and a pasted confirmation text is treated as typed. Source: `docs/blueprint/specifications/FEAT-24-data-export-account-deletion/FEAT-24.SPEC-002-account-deletion-screen.md` (section: Edge Cases)
- **FEAT-24.SPEC-003 (Data Export Archive Generation):** A near-empty account still generates a normal archive, a deliverable that cannot be included is retried and, if still unavailable after retries, fails generation. An archive expiring at the moment of download resolves to whichever completes first, and concurrent requests from two tabs leave one active archive via discard-and-create. Source: `docs/blueprint/specifications/FEAT-24-data-export-account-deletion/FEAT-24.SPEC-003-data-export-archive-generation.md` (section: Edge Cases)
- **FEAT-24.SPEC-004 (Account Deletion Processing):** Concurrent confirmed-deletion instructions proceed once, with a second instruction discarded as redundant once the account has left Active. A failure at the payment-account disconnect step is retried then halts and reverts (pre-commit), while failures in the stored-byte purge or activity-log removal after the commit point are retried until they succeed. Source: `docs/blueprint/specifications/FEAT-24-data-export-account-deletion/FEAT-24.SPEC-004-account-deletion-processing.md` (section: Edge Cases)
- **FEAT-24.SPEC-005 (Legal Retention Purge):** Overlapping sweeps are safe because an already purged record is simply absent from the next candidate set, a new sweep waits for an in-flight one, and a record whose retention period ends between sweeps is purged at the first sweep at or after the threshold. A failed purge attempt leaves the record retained for the next sweep. Source: `docs/blueprint/specifications/FEAT-24-data-export-account-deletion/FEAT-24.SPEC-005-legal-retention-purge.md` (section: Edge Cases)
- **FEAT-24.SPEC-006 (Pre-Deletion Warning & Retention Determination Rules):** The determination is re-run at confirmation, so an invoice paid in another tab changes the warning. An account with no invoices or milestones shows no warning, Partially refunded and Disputed invoices count as open and are also classified Retained, and a failed or unconfirmed Payment is still Retained because retention follows entity class. Source: `docs/blueprint/specifications/FEAT-24-data-export-account-deletion/FEAT-24.SPEC-006-pre-deletion-warning-retention-determination-rules.md` (section: Edge Cases)
- **FEAT-24.SPEC-007 (Export & Deletion Access Rules):** An operator's direct link to these screens is denied during a support session however it was reached, a client contact gets the portal's out-of-scope explanation (XBR-09), and an account mid-deletion cannot reach a fresh export request. A support session opened after a deletion attempt has started shows the operator no indication of it. Source: `docs/blueprint/specifications/FEAT-24-data-export-account-deletion/FEAT-24.SPEC-007-export-deletion-access-rules.md` (section: Edge Cases)
- **FEAT-24.SPEC-008 (Export Ready Notification):** An archive that expires before delivery (retries exhausted) sends no email and the Expired state shows on the next visit, and requesting a new export cancels the pending email for the superseded archive. Account deletion cancels queued retries, and quiet hours do not apply since the email is transactional. Source: `docs/blueprint/specifications/FEAT-24-data-export-account-deletion/FEAT-24.SPEC-008-export-ready-notification.md` (section: Edge Cases)
- **FEAT-24.SPEC-009 (Account Deletion Final Warning Notification):** The email already sent is not retracted if the cascade later fails and reverts the account to Active, and if every retry fails after the account is fully deleted no further action is taken since no screen remains to surface it. Only the first of two deletion confirmations produces this email, and quiet hours do not apply. Source: `docs/blueprint/specifications/FEAT-24-data-export-account-deletion/FEAT-24.SPEC-009-account-deletion-final-warning-notification.md` (section: Edge Cases)

## Requirements (mandatory)

### Functional Requirements

- **FR-001**: The system MUST implement **FEAT-24.SPEC-001** (Data Export Screen) as specified: Nadia requests a full export of all her own data, tracks the archive's progress, and downloads it once ready. Full spec: `docs/blueprint/specifications/FEAT-24-data-export-account-deletion/FEAT-24.SPEC-001-data-export-screen.md`
- **FR-002**: The system MUST implement **FEAT-24.SPEC-002** (Account Deletion Screen) as specified: Nadia reviews any open-item warnings and gives the explicit confirmation required to permanently delete her account and all its data. Full spec: `docs/blueprint/specifications/FEAT-24-data-export-account-deletion/FEAT-24.SPEC-002-account-deletion-screen.md`
- **FR-003**: The system MUST implement **FEAT-24.SPEC-003** (Data Export Archive Generation) as specified: Aggregates every client, project, proposal, invoice, and activity record Nadia owns into a single downloadable archive, retries cleanly on failure, and expires the archive after its limited download window. Full spec: `docs/blueprint/specifications/FEAT-24-data-export-account-deletion/FEAT-24.SPEC-003-data-export-archive-generation.md`
- **FR-004**: The system MUST implement **FEAT-24.SPEC-004** (Account Deletion Processing) as specified: Cascades the permanent removal of the freelancer's account and all owned data across every data-holding feature once she confirms, disconnecting her payment account, and leaving the account fully intact if processing fails. Full spec: `docs/blueprint/specifications/FEAT-24-data-export-account-deletion/FEAT-24.SPEC-004-account-deletion-processing.md`
- **FR-005**: The system MUST implement **FEAT-24.SPEC-005** (Legal Retention Purge) as specified: Purges the Invoice and Payment records held back from an otherwise-completed account deletion once their legal retention period lapses. Full spec: `docs/blueprint/specifications/FEAT-24-data-export-account-deletion/FEAT-24.SPEC-005-legal-retention-purge.md`
- **FR-006**: The system MUST implement **FEAT-24.SPEC-006** (Pre-Deletion Warning & Retention Determination Rules) as specified: Determines which open items (unpaid invoices, pending approvals) trigger a specific warning without blocking deletion, and which records are legally retained versus immediately deleted. Full spec: `docs/blueprint/specifications/FEAT-24-data-export-account-deletion/FEAT-24.SPEC-006-pre-deletion-warning-retention-determination-rules.md`
- **FR-007**: The system MUST implement **FEAT-24.SPEC-007** (Export & Deletion Access Rules) as specified: Restricts every export and deletion action in this feature to Nadia alone, with no access of any kind for client contacts or the Support Operator. Full spec: `docs/blueprint/specifications/FEAT-24-data-export-account-deletion/FEAT-24.SPEC-007-export-deletion-access-rules.md`
- **FR-008**: The system MUST implement **FEAT-24.SPEC-008** (Export Ready Notification) as specified: Emails Nadia when her requested data export archive is ready to download, so she learns about it even if she is away from the product while it generates. Full spec: `docs/blueprint/specifications/FEAT-24-data-export-account-deletion/FEAT-24.SPEC-008-export-ready-notification.md`
- **FR-009**: The system MUST implement **FEAT-24.SPEC-009** (Account Deletion Final Warning Notification) as specified: Emails Nadia a final confirmation notice at the moment she confirms permanent account deletion, giving her a durable record of exactly when she gave that irreversible confirmation. Full spec: `docs/blueprint/specifications/FEAT-24-data-export-account-deletion/FEAT-24.SPEC-009-account-deletion-final-warning-notification.md`

### Key Entities

- Data Export Archive (create); otherwise acts across all of a freelancer's entities (read, delete) [AUDIT-ADDED: 3 -- inverse check: the archive needed an entity]

## Success Criteria (mandatory)

### Measurable Outcomes

- **SC-001**: Data export requests, ready exports, account deletion requests and completed deletions are each observable as distinct signals (data_export_requested, data_export_ready, account_deletion_requested, account_deletion_completed); no metric in the success-metrics register connects to this feature, so the outcome is grounded in its Signals alone. Source: `docs/blueprint/features/product-features.md`

## Assumptions

- **ASMP-23**: Freelancers can export and delete their own data on request. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-24**: Personal data worldwide is treated as GDPR-class personal data. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-22**: Deliverables retained with version history for the life of the account, so archives can be very large. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-20**: Evidence outlives a contact's erasure request, but only as far as needed. Full register: `docs/blueprint/features/assumptions-constraints.md`

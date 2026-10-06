# Feature Specification: Notifications (Email)

**Blueprint feature:** FEAT-14
**Priority tier:** Core
**Build order:** 026 of 33
**Depends on:** FEAT-02, FEAT-03, FEAT-05, FEAT-06, FEAT-07, FEAT-08, FEAT-09, FEAT-10, FEAT-11, FEAT-18, FEAT-19, FEAT-25, FEAT-31, FEAT-32
**Blueprint source:** `docs/blueprint/specifications/FEAT-14-notifications-email/`

## User Scenarios & Testing (mandatory)

### User Story 1 - Transactional Email Delivery (Priority: P1)

Sends every composed email through the product's transactional email delivery capability and reports delivery, bounce, and failure status back so the product can track and act on outcomes.

**Acceptance Scenarios:**

**FEAT-14.SPEC-001-AC-01:** Given FEAT-14.SPEC-002 hands this integration a fully composed proposal-sent email addressed to Owen, when the send is transmitted, then the capability reports back a delivery outcome and Notification.delivery_status updates accordingly.

**FEAT-14.SPEC-001-AC-02:** Given a notification has been sent, when the capability reports the Delivered event, then Notification.delivery_status is set to Delivered and no further action is taken.

**FEAT-14.SPEC-001-AC-03:** Given a send is rejected because Owen's email address does not exist, when the Bounced event arrives, then Notification.delivery_status is set to Bounced and FEAT-14.SPEC-003 begins its retry-then-warn handling.

**FEAT-14.SPEC-001-AC-04:** Given a send attempt fails for a temporary reason, when the Failed event arrives, then Notification.delivery_status is set to Failed and FEAT-14.SPEC-003 schedules a retry.

**FEAT-14.SPEC-001-AC-05:** Given the capability is slow to confirm a send, when the triggering screen (e.g., FEAT-02.SPEC-005) has already completed its own action, then no screen shows a blocked or degraded state.

**FEAT-14.SPEC-001-AC-06:** Given the capability is down when a notification is queued, when the outage lasts through the standing retry window, then the notification is retried automatically once the capability recovers, per FEAT-14.SPEC-003.

**FEAT-14.SPEC-001-AC-07:** Given a send is rejected for a malformed address, when the rejection is reported, then it is handled identically to a Bounced event and no screen shows an error.

**FEAT-14.SPEC-001-AC-08:** Given Nadia opens her account settings (FEAT-21), when she reviews the delivery-mechanism disclosure, then it names the recipient identity and message content shared with the capability, consistent with this spec's Data Exchanged section.

**FEAT-14.SPEC-001-AC-09:** Given a recipient receives any product email, when they read it, then the sender name identifies the freelancer's business, never a disguised or unrelated origin.

**FEAT-14.SPEC-001-AC-10:** Given a Notification's underlying account has been deleted (FEAT-24), when a delivery event arrives afterward for that notification, then it is discarded silently and no status update is applied.

**FEAT-14.SPEC-001-AC-11:** Given a Notification is already Delivered, when the same Delivered event is reported a second time, then nothing changes and FEAT-14.SPEC-006 is not triggered a second time.

**FEAT-14.SPEC-001-AC-12:** Given a Delivered event for a later attempt arrives before a stale Failed event for the same notification, when both are processed, then the Notification reflects the Delivered outcome by its true occurrence time, not arrival order.

**FEAT-14.SPEC-001-AC-13:** Given the capability goes down mid-send with no confirmation either way, when the outage is detected, then the Notification remains Queued rather than being marked Sent or Delivered without confirmation.

**FEAT-14.SPEC-001-AC-14:** Given a bounce report carries no specific reason, when FEAT-14.SPEC-006 later warns Nadia, then the warning states delivery failed without inventing a reason the capability did not provide.

### User Story 2 - Notification Composition & Dispatch (Priority: P1)

The shared dispatch engine every other feature's triggering event calls -- it resolves the entitled recipient(s) and notification type for one triggering event, creates the Notification record, and hands the composed email to the delivery capability.

**Acceptance Scenarios:**

**FEAT-14.SPEC-002-AC-01:** Given Nadia sends a proposal to Owen, when FEAT-02.SPEC-011 fires this automation, then a Notification record is created for Owen with notification_type "proposal sent" and delivery_status Queued, and the composed email is handed to FEAT-14.SPEC-001.

**FEAT-14.SPEC-002-AC-02:** Given Owen accepts a proposal, when FEAT-03.SPEC-006 fires this automation, then a Notification record is created for the entitled recipients and handed to FEAT-14.SPEC-001.

**FEAT-14.SPEC-002-AC-03:** Given Priya requests a fresh sign-in link, when FEAT-05.SPEC-008 fires this automation, then a Notification record is created for Priya's own contact record and handed to FEAT-14.SPEC-001.

**FEAT-14.SPEC-002-AC-04:** Given a deliverable's upload fully completes, when FEAT-06.SPEC-006 fires this automation, then Notification records are created for every entitled contact on that project.

**FEAT-14.SPEC-002-AC-05:** Given Priya leaves a comment on a deliverable, when FEAT-07.SPEC-003 fires this automation, then a Notification record is created for Nadia.

**FEAT-14.SPEC-002-AC-06:** Given Owen approves a milestone, when FEAT-08.SPEC-007 fires this automation, then Notification records are created for Owen and Nadia per that spec's audience.

**FEAT-14.SPEC-002-AC-07:** Given an invoice is generated and issued, when FEAT-09.SPEC-010 fires this automation, then a Notification record is created for Owen.

**FEAT-14.SPEC-002-AC-08:** Given Owen's payment succeeds, when FEAT-10.SPEC-007 fires this automation, then Notification records are created for Owen and Nadia.

**FEAT-14.SPEC-002-AC-09:** Given an invoice becomes overdue and the day-3 reminder fires, when FEAT-11.SPEC-004 fires this automation, then a Notification record is created for Owen.

**FEAT-14.SPEC-002-AC-10:** Given Owen invites Priya as a Reviewer contact, when FEAT-18's contact-invitation event fires this automation, then a Notification record is created for Priya.

**FEAT-14.SPEC-002-AC-11:** Given Nadia completes sign-up, when FEAT-20's welcome-email event fires this automation, then a Notification record is created for Nadia.

**FEAT-14.SPEC-002-AC-12:** Given Dana opens a support session on Nadia's account, when FEAT-31's session-opened event fires this automation, then a Notification record is created for Nadia, unaffected by any of Nadia's optional preferences (XBR-29).

**FEAT-14.SPEC-002-AC-13:** Given the payment processor reports a chargeback on a paid invoice, when FEAT-25/FEAT-32's dispute event fires this automation, then a Notification record is created for Nadia (XBR-21).

**FEAT-14.SPEC-002-AC-14:** Given Nadia has switched off an optional notification type, when a triggering event of that type fires, then FEAT-14.SPEC-004's entitlement check returns zero recipients and no Notification record is created.

**FEAT-14.SPEC-002-AC-15:** Given a proposal-sent event fires and the client has one Primary contact (Owen) and one Reviewer contact (Priya), when this automation resolves recipients, then a Notification record is created only for Owen, not Priya.

**FEAT-14.SPEC-002-AC-16:** Given this automation cannot complete resolution or composition for a triggering event due to a processing error, when the failure occurs, then a Notification record is still created for the intended recipient with delivery_status set to Failed, and the triggering feature's own action is not blocked or reverted.

**FEAT-14.SPEC-002-AC-17:** Given Nadia turns off an optional notification type at the exact moment a triggering event of that type fires, when this automation evaluates entitlement, then the preference state at evaluation time wins and no Notification record is created.

### User Story 3 - Delivery Status Tracking & Retry (Priority: P1)

Records each notification's delivery outcome, retries a failed send automatically a limited number of times, then hands off to the delivery-failure warning once retries are exhausted or the failure is permanent.

**Acceptance Scenarios:**

**FEAT-14.SPEC-003-AC-01:** Given a Notification is Queued for Owen, when FEAT-14.SPEC-001 reports Delivered on the first attempt, then Notification.delivery_status is set to Delivered and no retry occurs.

**FEAT-14.SPEC-003-AC-02:** Given a Notification's send is reported Bounced, when this automation processes the outcome, then Notification.delivery_status is set to Bounced immediately and no retry is scheduled.

**FEAT-14.SPEC-003-AC-03:** Given a Notification's send is reported Failed for a transient reason, when this automation processes the outcome and the retry count is below platform parameter: `transactional-email-retry-count`, then Notification.delivery_status is set to Failed and a retry is scheduled.

**FEAT-14.SPEC-003-AC-04:** Given a Notification has been retried and reaches platform parameter: `transactional-email-retry-count` attempts with no Delivered outcome, when the final retry's outcome is processed, then Notification.delivery_status is finalized as Failed and FEAT-14.SPEC-006 is triggered.

**FEAT-14.SPEC-003-AC-05:** Given a bounced Notification, when this automation finalizes it, then FEAT-14.SPEC-006 is triggered immediately without waiting for any retry count.

**FEAT-14.SPEC-003-AC-06:** Given a Notification succeeds on its second retry attempt, when the Delivered outcome is processed, then Notification.delivery_status is set to Delivered and no delivery warning is ever generated.

**FEAT-14.SPEC-003-AC-07:** Given two outcome reports for the same Notification arrive at effectively the same time, when this automation processes them, then the outcome that occurred more recently (by its own event time) is the one reflected, regardless of arrival order.

**FEAT-14.SPEC-003-AC-08:** Given a retry outcome report arrives while this automation is still applying the prior report's status change for the same Notification, when both are processed, then they are applied in sequence and the Notification never shows an inconsistent intermediate state.

**FEAT-14.SPEC-003-AC-09:** Given a Notification's final retry succeeds at the exact moment its retry count reaches the limit, when both conditions are evaluated together, then the Delivered outcome takes precedence over marking the notification Failed.

**FEAT-14.SPEC-003-AC-10:** Given the affected project is archived while a Notification is retrying, when retries are later exhausted, then the delivery warning still appears per FEAT-14.SPEC-006.

**FEAT-14.SPEC-003-AC-11:** Given the freelancer's account is deleted while a Notification is mid-retry, when the deletion completes, then retry processing for that Notification stops and the record is removed with the account.

**FEAT-14.SPEC-003-AC-12:** Given this automation cannot resolve a Notification reference due to a processing error, when the outcome report cannot be applied, then no status change is made, no user is blocked, and the report is re-processed on the automation's next run.

### User Story 4 - Notification Type & Recipient Entitlement Rules (Priority: P1)

Governs the one-type-per-event mapping, Access-Matrix-limited recipients, and which notifications a preference can switch off versus which always send.

**Acceptance Scenarios:**

**FEAT-14.SPEC-004-AC-01:** Given a triggering event fires with a notification_type that matches an entry in the registry, when FEAT-14.SPEC-002 checks this spec's rules, then the type is accepted and entitlement resolution proceeds.

**FEAT-14.SPEC-004-AC-02:** Given a Notification record is being created, when its recipient is resolved, then the recipient's role must appear in that notification_type's entitled-role set, or no record is created for that would-be recipient.

**FEAT-14.SPEC-004-AC-03:** Given a Notification's delivery_status is Delivered, when any subsequent event is processed, then delivery_status never regresses to an earlier state (Queued, Sent, Failed, or Bounced).

**FEAT-14.SPEC-004-AC-04:** Given a proposal_sent event fires for a client with Owen as Primary and Priya as Reviewer, when recipients are resolved, then a Notification record is created for Owen and none for Priya, since proposal_sent entitles only the Primary role.

**FEAT-14.SPEC-004-AC-05:** Given a deliverable_ready event fires, when recipients are resolved, then Notification records are created for both Owen and Priya, since deliverable_ready entitles both roles.

**FEAT-14.SPEC-004-AC-06:** Given a proposal_change_requested event fires, when recipients are resolved, then a Notification record is created only for Nadia; Owen and Priya are never resolved as recipients for this type.

**FEAT-14.SPEC-004-AC-07:** Given Nadia (Freelancer) is the subject of a milestone_approved_confirmation event, when recipients are resolved, then she is always entitled and a Notification record is created for her alongside Owen's.

**FEAT-14.SPEC-004-AC-08:** Given Dana (Support Operator) is inside a logged support session, when she looks for a way to view a Notification's full content, then no such view exists -- she can see only the delivery-warning summary defined by FEAT-14.SPEC-006.

**FEAT-14.SPEC-004-AC-09:** Given Owen looks for a way to see his own or another Notification's delivery status as data, when he looks in the portal, then no such view exists for client contacts.

**FEAT-14.SPEC-004-AC-10:** Given every notification_type in the registry is currently classified Transactional, when Nadia opens her notification preferences (FEAT-21), then no toggle is offered for any of them.

**FEAT-14.SPEC-004-AC-11:** Given a hypothetical future notification_type were classified Optional, when Nadia turns it off in her preferences, then FEAT-14.SPEC-002's entitlement check for that type returns zero recipients for her account going forward.

**FEAT-14.SPEC-004-AC-12:** Given a Transactional notification_type, when FEAT-14.SPEC-002 checks entitlement, then the Freelancer Account's notification_preferences are never consulted, and every entitled recipient always resolves.

**FEAT-14.SPEC-004-AC-13:** Given a support_session_notice event fires (Dana opens a session), when recipients are resolved, then Nadia is always entitled, per XBR-29's mandate that support sessions are always announced by email.

**FEAT-14.SPEC-004-AC-14:** Given a chargeback_notice event fires, when recipients are resolved, then Nadia is always entitled, per XBR-21.

**FEAT-14.SPEC-004-AC-15:** Given a new contact is invited via FEAT-18, when the contact_invitation event fires, then the newly invited contact (Owen or Priya, whichever role they were assigned) is the sole entitled recipient.

**FEAT-14.SPEC-004-AC-16:** Given a client has two Primary contacts, when a notification_type entitling the Primary role fires, then each Primary contact receives their own independent Notification record.

**FEAT-14.SPEC-004-AC-17:** Given a contact's role changes from Reviewer to Primary a moment before FEAT-14.SPEC-002 runs its entitlement check, when the check runs, then the updated (Primary) role governs entitlement for that occurrence.

**FEAT-14.SPEC-004-AC-18:** Given no Reviewer contact currently exists for a client, when a notification_type entitling the Reviewer role fires, then a Notification record is created only for the currently existing entitled contact(s), with no error for the missing role.

**FEAT-14.SPEC-004-AC-19:** Given a notification_type outside the registry is somehow supplied to this spec's rules, when FEAT-14.SPEC-002 checks it, then the type is rejected as an internal defect and no Notification record is created under an unregistered type.

**FEAT-14.SPEC-004-AC-20:** Given anyone attempts to delete an individual Notification record within the product, when they look for such an action, then none exists -- Notification records are removed only by FEAT-24's account deletion.

**FEAT-14.SPEC-004-AC-21:** Given any of the 26 Notification specs in the product fires its triggering event (for example FEAT-24.SPEC-008's export-ready event, FEAT-26.SPEC-004's signature-recorded event, or FEAT-32.SPEC-006's needs-attention event), when FEAT-14.SPEC-002 looks up its notification_type, then a registry row exists naming that spec, and recipients resolve per that row (data_export_ready to Nadia only; signed_copy_confirmation to Owen and Nadia; payment_connection_needs_attention to Nadia only; refund_recorded to Owen only).

### User Story 5 - Recognizable & Branded Email Presentation Rules (Priority: P1)

Governs sender name, subject line, branding, and plain, consistent formatting so every email is identifiable as coming from the freelancer's practice and reads as trustworthy, not spam.

**Acceptance Scenarios:**

**FEAT-14.SPEC-005-AC-01:** Given the freelancer has set both a logo and a brand colour, when a client-facing notification is composed, then the email's presentation applies that logo and colour.

**FEAT-14.SPEC-005-AC-02:** Given the freelancer has not set a Branding Profile, when a client-facing notification is composed, then the email falls back to the neutral default logo and colour.

**FEAT-14.SPEC-005-AC-03:** Given the freelancer's brand colour was recently adjusted for legibility by FEAT-19, when the next notification is composed, then it uses the current, already-adjusted colour value.

**FEAT-14.SPEC-005-AC-04:** Given the freelancer has set a business_name, when any notification is composed, then the sender display name is that business_name.

**FEAT-14.SPEC-005-AC-05:** Given the freelancer has not yet set a business_name, when any notification is composed, then the sender display name falls back to her personal account name.

**FEAT-14.SPEC-005-AC-06:** Given a notification is addressed to Owen about a specific project, when its subject line is composed, then it names the project (or the specific invoice/milestone identifier relevant to the event).

**FEAT-14.SPEC-005-AC-07:** Given a notification is addressed to Nadia about a specific client's action, when its subject line is composed, then it names the client company, since Nadia manages several clients.

**FEAT-14.SPEC-005-AC-08:** Given any notification body is composed, when it opens, then it greets the recipient by name ("Hi {first_name},"), never a generic or unaddressed opening.

**FEAT-14.SPEC-005-AC-09:** Given a notification is addressed to Owen or Priya, when it is composed, then it includes the referral mark without overriding the freelancer's branding.

**FEAT-14.SPEC-005-AC-10:** Given a notification is addressed to Nadia herself, when it is composed, then it does not carry the referral mark and does not apply client-facing branding.

**FEAT-14.SPEC-005-AC-11:** Given any notification type, when its content is composed, then it follows the plain, consistent formatting baseline (simple header, personal greeting, clear body, one primary CTA, plain sign-off) rather than promotional styling.

**FEAT-14.SPEC-005-AC-12:** Given Nadia edits her Branding Profile in FEAT-19, when she saves the change, then it is not offered as a per-email override anywhere -- it applies uniformly to every future client-facing send.

**FEAT-14.SPEC-005-AC-13:** Given Nadia looks for a way to turn off the referral mark on client-facing emails, when she checks her settings, then no such control exists, per XBR-32.

**FEAT-14.SPEC-005-AC-14:** Given Dana is inside a logged support session, when she looks for a way to preview a client-facing email's rendered branding, then no such preview action exists for her role.

**FEAT-14.SPEC-005-AC-15:** Given Owen or Priya attempt to edit the freelancer's Branding Profile, when they look for such a control, then none exists anywhere in their portal access.

**FEAT-14.SPEC-005-AC-16:** Given an invoice_issued event addresses both Owen and Nadia in the same occurrence, when each copy is composed, then Owen's copy is branded with the referral mark and Nadia's copy is not.

**FEAT-14.SPEC-005-AC-17:** Given a client has no Reviewer contact on record, when a notification entitling the Reviewer role would otherwise be composed for one, then no presentation rule is violated -- the rule set applies per actual recipient, not per role slot.

**FEAT-14.SPEC-005-AC-18:** Given the freelancer's logo file was accepted by FEAT-19's own size and format validation, when this spec composes an email, then it applies that logo without re-validating its size or format.

**FEAT-14.SPEC-005-AC-19:** Given a Freelancer Account exists (required at sign-up, FEAT-20), when any notification is composed, then a sender name and a recipient greeting name are always available -- neither is ever blank.

### User Story 6 - Delivery Failure Warning to Freelancer (Priority: P1)

Warns Nadia on the affected project when a notification's delivery fails after retries are exhausted (or bounces permanently), so a lost email is surfaced rather than silently vanishing; viewable read-only by Dana inside a support session.

**Acceptance Scenarios:**

**FEAT-14.SPEC-006-AC-01:** Given an invoice_issued notification to Owen bounces permanently, when FEAT-14.SPEC-003 reports the Bounced outcome, then this warning appears on the affected project for Nadia immediately, without waiting for any retry.

**FEAT-14.SPEC-006-AC-02:** Given a proposal_sent notification to Owen fails transiently and exhausts its retries, when the final retry's outcome is processed, then this warning appears on the affected project for Nadia.

**FEAT-14.SPEC-006-AC-03:** Given Nadia opens the affected project's screen, when one open delivery failure exists, then she sees the single-failure variant naming the notification type, the recipient, and, if available, the reason.

**FEAT-14.SPEC-006-AC-04:** Given a project has two open delivery failures of different notification types, when Nadia opens the project's screen, then she sees the batched variant listing both.

**FEAT-14.SPEC-006-AC-05:** Given Nadia taps "Review contact details" on this warning, then she is taken to FEAT-18's client contact list for the affected client.

**FEAT-14.SPEC-006-AC-06:** Given Nadia has no preference control for this warning, when a delivery failure occurs, then the warning always appears -- there is no way to switch it off, per XBR-30.

**FEAT-14.SPEC-006-AC-07:** Given a bounce report carries no specific reason from the delivery capability, when this warning renders, then it states delivery failed without inventing a reason.

**FEAT-14.SPEC-006-AC-08:** Given the same Notification's exhausted-retries outcome is reported twice due to a duplicate event, when this notification is triggered, then only one open warning entry exists for that Notification, not two.

**FEAT-14.SPEC-006-AC-09:** Given Nadia corrects the bouncing contact's email address via FEAT-18, when she returns to the affected project, then the original warning remains open until she separately resends the failed communication through its originating feature's own resend action.

**FEAT-14.SPEC-006-AC-10:** Given the affected project has since been archived, when Nadia opens the archived project, then the open delivery warning still appears on its screen.

**FEAT-14.SPEC-006-AC-11:** Given Nadia's account is deleted, when the deletion completes, then any open delivery warnings tied to her account are removed along with the Notification records they describe.

**FEAT-14.SPEC-006-AC-12:** Given Dana opens a support session on an account with an open delivery warning, when she views the affected project, then she sees the read-only support-session variant with no action control, and never the underlying message's subject or body content.

**FEAT-14.SPEC-006-AC-13:** Given Owen or Priya are viewing their own portal, when a delivery failure occurs on their project, then they never see this warning in any form -- it exists only for Nadia (and Dana, read-only).

**FEAT-14.SPEC-006-AC-14:** Given a proposal is edited and re-sent after its original send bounced (FEAT-02.SPEC-011), when the edited version's own notification succeeds, then the original warning remains as historical context while the current state reflects the successful re-send.

**FEAT-14.SPEC-006-AC-15:** Given this warning is an in-app persistent element rather than a transmitted message, when the affected project's screen loads normally, then the warning renders correctly with no separate delivery/retry mechanics of its own.

### Edge Cases

- **FEAT-14.SPEC-001 (Transactional Email Delivery):** A delivery or bounce event for a Notification that no longer exists is discarded silently, a duplicate delivery event changes nothing and triggers no second warning, and events are applied by occurrence time rather than arrival time. If the capability goes down mid-send the Notification stays Queued and is retried once available. Source: `docs/blueprint/specifications/FEAT-14-notifications-email/FEAT-14.SPEC-001-transactional-email-delivery.md` (section: Edge Cases)
- **FEAT-14.SPEC-002 (Notification Composition & Dispatch):** Each triggering-event occurrence produces its own independent Notification with no coalescing, including near-simultaneous events from different features for one recipient. A resolution retry for the same occurrence reuses the existing record, and archiving the related project or client does not revoke recipient entitlement. Source: `docs/blueprint/specifications/FEAT-14-notifications-email/FEAT-14.SPEC-002-notification-composition-dispatch.md` (section: Edge Cases)
- **FEAT-14.SPEC-003 (Delivery Status Tracking & Retry):** Concurrent outcome reports apply the most recent true outcome by event time and processing for one Notification is serialized. A successful final retry wins over the retries-exhausted determination, and archiving the project between failure and exhaustion does not suppress the delivery warning (FEAT-14.SPEC-006). Source: `docs/blueprint/specifications/FEAT-14-notifications-email/FEAT-14.SPEC-003-delivery-status-tracking-retry.md` (section: Edge Cases)
- **FEAT-14.SPEC-004 (Notification Type & Recipient Entitlement Rules):** Entitlement is unaffected when a client has no Reviewer (a record is created only for entitled contacts that exist), every current Primary gets their own Notification and email, and the role in effect when the entitlement check runs governs. A transactional type can never become a toggle even if offered by a future FEAT-21 change. Source: `docs/blueprint/specifications/FEAT-14-notifications-email/FEAT-14.SPEC-004-notification-type-recipient-entitlement-rules.md` (section: Edge Cases)
- **FEAT-14.SPEC-005 (Recognizable & Branded Email Presentation Rules):** A freelancer with no Branding Profile gets the neutral default logo and colour, one with no business_name falls back to the personal account name, and a copy addressed to the freelancer herself carries no branding or referral mark. The brand colour is always read already legibility-adjusted at composition time. Source: `docs/blueprint/specifications/FEAT-14-notifications-email/FEAT-14.SPEC-005-recognizable-branded-email-presentation-rules.md` (section: Edge Cases)
- **FEAT-14.SPEC-006 (Delivery Failure Warning to Freelancer):** A warning remains as historical context if the underlying record is voided or superseded, failures across notification types batch into one project-level indicator, and correcting a contact's email never auto-retries the failed notification (the freelancer uses the originating feature's resend). An archived project still renders its open warning. Source: `docs/blueprint/specifications/FEAT-14-notifications-email/FEAT-14.SPEC-006-delivery-failure-warning-to-freelancer.md` (section: Edge Cases)

## Requirements (mandatory)

### Functional Requirements

- **FR-001**: The system MUST implement **FEAT-14.SPEC-001** (Transactional Email Delivery) as specified: Sends every composed email through the product's transactional email delivery capability and reports delivery, bounce, and failure status back so the product can track and act on outcomes. Full spec: `docs/blueprint/specifications/FEAT-14-notifications-email/FEAT-14.SPEC-001-transactional-email-delivery.md`
- **FR-002**: The system MUST implement **FEAT-14.SPEC-002** (Notification Composition & Dispatch) as specified: The shared dispatch engine every other feature's triggering event calls -- it resolves the entitled recipient(s) and notification type for one triggering event, creates the Notification record, and hands the composed email to the delivery capability. Full spec: `docs/blueprint/specifications/FEAT-14-notifications-email/FEAT-14.SPEC-002-notification-composition-dispatch.md`
- **FR-003**: The system MUST implement **FEAT-14.SPEC-003** (Delivery Status Tracking & Retry) as specified: Records each notification's delivery outcome, retries a failed send automatically a limited number of times, then hands off to the delivery-failure warning once retries are exhausted or the failure is permanent. Full spec: `docs/blueprint/specifications/FEAT-14-notifications-email/FEAT-14.SPEC-003-delivery-status-tracking-retry.md`
- **FR-004**: The system MUST implement **FEAT-14.SPEC-004** (Notification Type & Recipient Entitlement Rules) as specified: Governs the one-type-per-event mapping, Access-Matrix-limited recipients, and which notifications a preference can switch off versus which always send. Full spec: `docs/blueprint/specifications/FEAT-14-notifications-email/FEAT-14.SPEC-004-notification-type-recipient-entitlement-rules.md`
- **FR-005**: The system MUST implement **FEAT-14.SPEC-005** (Recognizable & Branded Email Presentation Rules) as specified: Governs sender name, subject line, branding, and plain, consistent formatting so every email is identifiable as coming from the freelancer's practice and reads as trustworthy, not spam. Full spec: `docs/blueprint/specifications/FEAT-14-notifications-email/FEAT-14.SPEC-005-recognizable-branded-email-presentation-rules.md`
- **FR-006**: The system MUST implement **FEAT-14.SPEC-006** (Delivery Failure Warning to Freelancer) as specified: Warns Nadia on the affected project when a notification's delivery fails after retries are exhausted (or bounces permanently), so a lost email is surfaced rather than silently vanishing; viewable read-only by Dana inside a support session. Full spec: `docs/blueprint/specifications/FEAT-14-notifications-email/FEAT-14.SPEC-006-delivery-failure-warning-to-freelancer.md`

### Key Entities

- Notification (create, send)

## Success Criteria (mandatory)

### Measurable Outcomes

- **SC-001**: At least 99% of triggered notifications are successfully delivered, with any failure surfaced to the freelancer within minutes rather than silently lost, and fewer than 1 in 100 client contacts report finding a Clientroom email in their spam folder (metric: Notification Delivery Reliability). Source: `docs/blueprint/features/success-metrics.md`
- **SC-002**: Notifications sent, delivery failures and bounces are each observable as distinct signals (notification_sent, notification_delivery_failed, notification_bounced). Source: `docs/blueprint/features/product-features.md`

## Assumptions

- **ASMP-03**: Freelancers and client contacts have reliable, actively monitored email access, and email is the sole client-facing channel. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-26**: Emails that fail to deliver are surfaced to the freelancer within minutes rather than lost. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-29**: Transactional email delivery capability, with delivery and bounce status reported back, is a required dependency. Full register: `docs/blueprint/features/assumptions-constraints.md`

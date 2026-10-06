# Feature Specification: Immutable Activity & Audit Trail

**Blueprint feature:** FEAT-13
**Priority tier:** Core
**Build order:** 025 of 33
**Depends on:** FEAT-03, FEAT-05, FEAT-06, FEAT-08, FEAT-09, FEAT-10, FEAT-11, FEAT-18, FEAT-25, FEAT-31
**Blueprint source:** `docs/blueprint/specifications/FEAT-13-immutable-activity-audit-trail/`

## User Scenarios & Testing (mandatory)

### User Story 1 - Activity Trail (Priority: P1)

Nadia browses the full, permanent chronological history of everything recorded against a project; Dana views the identical trail read-only while a support session is open.

**Acceptance Scenarios:**

**FEAT-13.SPEC-001-AC-01:** Given Nadia opens a project with existing activity, when the screen finishes loading, then she sees every entry for that project listed most-recent-first, each with its description, actor, exact timestamp, and affected-record link.

**FEAT-13.SPEC-001-AC-02:** Given Nadia is viewing the trail, when she taps an entry's affected-record link, then she is taken to that record's own detail screen.

**FEAT-13.SPEC-001-AC-03:** Given Nadia is viewing the trail, when she taps the header "Share" button, then FEAT-13.SPEC-002 opens scoped to the full project trail.

**FEAT-13.SPEC-001-AC-04:** Given Nadia is viewing the trail, when she taps "Share this record" on a single entry, then FEAT-13.SPEC-002 opens scoped to that one entry only.

**FEAT-13.SPEC-001-AC-05:** Given Nadia scrolls to the end of the currently loaded entries in a project with a long history, when more entries exist, then the next page loads with a lightweight indicator at the list's end.

**FEAT-13.SPEC-001-AC-06:** Given Nadia taps the breadcrumb, when the tap registers, then she returns to the FEAT-01 project view.

**FEAT-13.SPEC-001-AC-07:** Given Nadia opens a brand-new project with no recorded events, when the screen loads, then she sees the Empty state explaining entries will appear as milestones progress.

**FEAT-13.SPEC-001-AC-08:** Given Nadia has a trail loaded and a refresh fails, when the failure occurs, then an error banner appears while every previously loaded entry remains visible underneath.

**FEAT-13.SPEC-001-AC-09:** Given Nadia loses connectivity while the trail is open, when connectivity drops, then the last-loaded trail remains visible read-only under an offline banner, and affected-record links are disabled until connectivity returns.

**FEAT-13.SPEC-001-AC-10:** Given Owen or Priya is signed into the client portal, when they look for any way to reach the cross-event activity trail, then no tab, link, or navigation path to this screen exists anywhere in their portal view.

**FEAT-13.SPEC-001-AC-11:** Given Dana has an open support session on Nadia's account, when she opens the activity trail, then she sees the same full trail read-only, with no Share control and no actionable entries.

**FEAT-13.SPEC-001-AC-12:** Given Dana's support session closes while she is viewing the trail, when the session ends, then the screen becomes immediately unreachable to her.

**FEAT-13.SPEC-001-AC-13:** Given Dana's support session opens and closes during a session, when Nadia later opens the same trail, then she sees the session's own opened and closed entries in the list, attributed to Dana (operator).

**FEAT-13.SPEC-001-AC-14:** Given an unauthenticated visitor attempts to reach this screen directly, when the request is made, then they are redirected to sign-in with no project context retained.

**FEAT-13.SPEC-001-AC-15:** Given Nadia's session expires while the trail is open, when she next interacts with the screen, then a "Your session has expired. Sign in to continue." dialog appears and, once she signs back in, the same trail re-displays without her having lost any input (none existed to lose).

### User Story 2 - Printable Record Copy (Priority: P1)

Nadia produces a printable, unalterable copy of a project's full activity trail, or of one selected entry, to show a client during a scope dispute.

**Acceptance Scenarios:**

**FEAT-13.SPEC-002-AC-01:** Given Nadia taps "Share" on the full trail from FEAT-13.SPEC-001, when this screen opens, then it shows every entry for the project at full detail, most-recent-first, under an identifying header naming her business, the client, and the project.

**FEAT-13.SPEC-002-AC-02:** Given Nadia taps "Share this record" on one entry from FEAT-13.SPEC-001, when this screen opens, then it shows only that one entry at full detail, with no other entries present.

**FEAT-13.SPEC-002-AC-03:** Given Nadia is viewing a Ready copy, when she taps "Print / Save," then the button shows "Preparing your copy..." and, on completion, a file is offered for save or the print dialog opens.

**FEAT-13.SPEC-002-AC-04:** Given Nadia is viewing a Ready copy, when she taps "Print / Save" a second time while preparation is still in progress, then the second tap has no effect and the button remains in its "Preparing..." state.

**FEAT-13.SPEC-002-AC-05:** Given Nadia taps the back control, when the tap registers, then she returns to FEAT-13.SPEC-001 (Activity Trail).

**FEAT-13.SPEC-002-AC-06:** Given this screen's scoped content fails to load, when the failure occurs, then the Error state shows "This record couldn't be loaded. Try again." with a Retry control, while the identifying header block remains visible.

**FEAT-13.SPEC-002-AC-07:** Given the Error state is showing, when Nadia taps Retry and loading succeeds, then the Ready state renders with the originally requested scope.

**FEAT-13.SPEC-002-AC-08:** Given Owen or Priya is signed into the client portal, when they look for any way to reach this screen, then no path exists anywhere in their portal view.

**FEAT-13.SPEC-002-AC-09:** Given Dana has an open support session on Nadia's account, when she views the activity trail, then no control to reach this screen is rendered anywhere in her session.

**FEAT-13.SPEC-002-AC-10:** Given Nadia produces a full-trail copy for a project with hundreds of entries, when she taps "Print / Save," then the resulting output includes every entry, spanning multiple pages as needed.

**FEAT-13.SPEC-002-AC-11:** Given Nadia has already produced a copy and new entries are later written to the project's trail, when she views the previously produced copy again, then it still shows only the entries present at the time it was produced.

**FEAT-13.SPEC-002-AC-12:** Given an unauthenticated visitor attempts to reach this screen directly, when the request is made, then they are redirected to sign-in with no scope context retained.

### User Story 3 - Activity Entry Recording (Priority: P1)

Writes one append-only Activity Log Entry whenever any of eleven other features reports a record-worthy event, capturing event type, actor, timestamp, and the affected record.

**Acceptance Scenarios:**

**FEAT-13.SPEC-003-AC-01:** Given Owen accepts a proposal, when the acceptance is recorded, then this automation writes an entry with event_type "proposal accepted," Owen as actor, and the acceptance timestamp.

**FEAT-13.SPEC-003-AC-02:** Given Owen approves a milestone, when the approval is recorded, then this automation writes an entry with event_type "milestone approved," Owen as actor, and the approval timestamp.

**FEAT-13.SPEC-003-AC-03:** Given Nadia reopens a previously approved milestone, when the reopen is recorded, then this automation writes a separate entry with event_type "milestone reopened," distinct from and never overwriting the earlier approval entry.

**FEAT-13.SPEC-003-AC-04:** Given an invoice is sent, whether automatically or as an ad hoc send, when the send completes, then this automation writes an entry with event_type "invoice sent" and the send timestamp.

**FEAT-13.SPEC-003-AC-05:** Given Nadia issues a credit note against a prior invoice, when the credit note is issued, then this automation writes an entry with event_type "credit note issued," Nadia as actor, and a reference to the corrected invoice.

**FEAT-13.SPEC-003-AC-06:** Given Nadia's deliverable upload completes, when the upload finishes, then this automation writes an entry with event_type "deliverable uploaded," Nadia as actor, and the completion timestamp.

**FEAT-13.SPEC-003-AC-07:** Given Nadia withdraws a deliverable, when the removal completes, then this automation writes an entry with event_type "deliverable removed," Nadia as actor, and the removal timestamp.

**FEAT-13.SPEC-003-AC-08:** Given Priya views a deliverable for the first time, when that first view is captured, then this automation writes an entry with event_type "first client view," Priya as actor, and the first-view timestamp; a second view by Priya of the same deliverable writes no further entry.

**FEAT-13.SPEC-003-AC-09:** Given an automatic day-3 reminder is sent, when the send completes, then this automation writes an entry with event_type "reminder sent," actor "Automatic," and the send timestamp.

**FEAT-13.SPEC-003-AC-10:** Given Nadia sends a manual reminder, when the send completes, then this automation writes an entry with event_type "reminder sent," Nadia as actor, and the send timestamp.

**FEAT-13.SPEC-003-AC-11:** Given Owen invites Priya as a Reviewer contact, when the invitation is saved, then this automation writes an entry with event_type "contact role changed" (added), Owen as the acting party.

**FEAT-13.SPEC-003-AC-12:** Given Nadia records a refund on a disputed invoice, when the refund is recorded, then this automation writes a new entry with event_type "refund/reversal/cancellation recorded" that preserves the original disputed entry unchanged.

**FEAT-13.SPEC-003-AC-13:** Given Nadia records an off-platform payment, when the record is saved, then this automation writes an entry with event_type "manual payment recorded," Nadia as actor.

**FEAT-13.SPEC-003-AC-14:** Given Dana opens a support session on Nadia's account, when the session begins, then this automation writes an entry with event_type "support session opened," Dana as actor.

**FEAT-13.SPEC-003-AC-15:** Given Dana's support session ends automatically after inactivity, when the session closes, then this automation writes an entry with event_type "support session closed," Dana as actor, and closure reason "inactivity."

**FEAT-13.SPEC-003-AC-16:** Given an entry write fails on its first attempt, when the automation retries, then it retries at platform parameter: `activity-entry-write-retry-interval` intervals until the write succeeds, and the triggering feature's own action is not reported complete to its user until then.

**FEAT-13.SPEC-003-AC-17:** Given the same triggering event is reported twice due to a retried request from the triggering feature, when the second report arrives, then no second Activity Log Entry is created.

**FEAT-13.SPEC-003-AC-18:** Given two client contacts each record their first view of the same deliverable within moments of each other, when both events are reported, then two independent entries are written, each attributed to its own viewing contact.

**FEAT-13.SPEC-003-AC-19:** Given a client contact's details are later erased through FEAT-18, when Nadia views an entry that already named that contact as actor, then the entry still displays the contact's name exactly as it was at the time of the event.

**FEAT-13.SPEC-003-AC-20:** Given Nadia's Free plan record has just been created at account creation, when FEAT-23.SPEC-002 reports the plan-created event, then this automation writes an entry with event_type "plan created," actor "Automatic," tier Free, status Active, and no project reference, and a retry of that write never delays or reverses her plan record.

**FEAT-13.SPEC-003-AC-21:** Given Nadia's plan moves from Paid, Active to Free, Lapsed at period end, when FEAT-23.SPEC-004 reports the change, then this automation writes an entry with event_type "plan tier/status changed," actor "Automatic," the prior and new tier and status, and the change timestamp; a routine renewal that changes nothing writes no entry.

**FEAT-13.SPEC-003-AC-22:** Given the downgrade-eligible flag on Nadia's plan goes from cleared to raised, when FEAT-23.SPEC-005 reports the offer, then this automation writes one entry with event_type "downgrade offer raised" and actor "Automatic," and no further entry is written while the flag stays raised.

**FEAT-13.SPEC-003-AC-23:** Given Nadia cancels her Paid subscription, when FEAT-23.SPEC-006 reports the recorded cancellation, then this automation writes an entry with event_type "plan cancellation recorded," Nadia as actor, the prior and new status, and the period end date, and a retry of that write never reverses the cancellation.

### User Story 4 - Entry Immutability, Content & Attribution Rules (Priority: P1)

Defines the required fields every Activity Log Entry must carry, the fixed vocabulary of recordable events, and the hard prohibition against ever editing or deleting a written entry.

**Acceptance Scenarios:**

**FEAT-13.SPEC-004-AC-01:** Given a reported event with a recognized event_type, when the entry is composed, then the write proceeds.

**FEAT-13.SPEC-004-AC-02:** Given a reported event with an unrecognized event_type, when the write is attempted, then it is rejected with "Entry rejected: unrecognized event type," and no entry is created.

**FEAT-13.SPEC-004-AC-03:** Given a reported event with no actor supplied, when the write is attempted, then it is rejected with "Entry rejected: actor is required."

**FEAT-13.SPEC-004-AC-04:** Given a reported event whose occurred_at timestamp is in the future, when the write is attempted, then it is rejected with "Entry rejected: invalid timestamp."

**FEAT-13.SPEC-004-AC-05:** Given a reported "milestone approved" event whose affected_record references an Invoice instead of a Milestone, when the write is attempted, then it is rejected with "Entry rejected: affected record does not match event type."

**FEAT-13.SPEC-004-AC-06:** Given a reported "invoice sent" event with no project reference, when the write is attempted, then it is rejected with "Entry rejected: project reference required for this event type."

**FEAT-13.SPEC-004-AC-07:** Given a reported "support session opened" event with no project reference, when the write is attempted, then it succeeds -- project is not required for this event type.

**FEAT-13.SPEC-004-AC-08:** Given Nadia is viewing a written entry, when she looks for any way to edit it, then no edit control is rendered anywhere for the entry.

**FEAT-13.SPEC-004-AC-09:** Given Dana is viewing an entry during a support session, when she looks for any way to edit or delete it, then no such control is rendered -- her access is read-only, consistent with FEAT-13.SPEC-005.

**FEAT-13.SPEC-004-AC-10:** Given any written entry, when any role attempts to reach a delete action for it in-product, then no such action exists anywhere in the product; removal exists only through account deletion under FEAT-13.SPEC-006.

**FEAT-13.SPEC-004-AC-11:** Given a triggering feature reports an event without its own explicit event-moment timestamp, when the entry is composed, then occurred_at is set to the current time at write.

**FEAT-13.SPEC-004-AC-12:** Given a triggering feature reports an event with its own explicit event-moment timestamp (e.g., the exact instant Owen accepted a proposal), when the entry is composed, then occurred_at is set to that supplied timestamp rather than the write time.

**FEAT-13.SPEC-004-AC-13:** Given Priya views the same deliverable twice on different days, when each view is reported, then only the first view produces an entry with event_type "first client view" -- the second view produces no new entry of that type (per FEAT-13.SPEC-003's duplicate-report handling operating on this spec's identity rule).

**FEAT-13.SPEC-004-AC-14:** Given a milestone is approved and later reopened, when both events are reported, then two distinct entries exist -- event_type "milestone approved" and event_type "milestone reopened" -- and neither is overwritten by the other.

**FEAT-13.SPEC-004-AC-15:** Given a client contact's details are erased through FEAT-18 after they are named as actor on an existing entry, when the erasure is processed, then no field on that existing entry is rewritten.

**FEAT-13.SPEC-004-AC-16:** Given a deliverable named in an entry's affected_record is later withdrawn, when Nadia opens that entry, then its recorded content is unchanged even though the deliverable's own current state differs.

**FEAT-13.SPEC-004-AC-17:** Given a reported event is missing a required field, when FEAT-13.SPEC-003 attempts the write, then the rejection is treated as a validation failure and is not subject to the write-durability retry behavior.

**FEAT-13.SPEC-004-AC-18:** Given Nadia is the sole role permitted to produce a printable copy, when Owen looks for any equivalent control in his portal, then none is rendered.

**FEAT-13.SPEC-004-AC-19:** Given a reported "reminder sent" event with actor "Automatic," when the entry is composed, then actor is recorded exactly as "Automatic," distinct from any human actor value.

**FEAT-13.SPEC-004-AC-20:** Given a refund is recorded against a disputed invoice, when the new entry is written, then the original disputed entry's fields remain exactly as they were, and the refund appears as a separate, additional entry.

### User Story 5 - Activity Trail Access & Visibility Rules (Priority: P1)

Governs who may see the cross-event activity trail and its entries -- Nadia Full, Dana View (session-scoped), Owen and Priya None -- versus who sees only the outcome of their own actions within their own scoped views elsewhere in the product.

**Acceptance Scenarios:**

**FEAT-13.SPEC-005-AC-01:** Given Nadia opens the activity trail for any of her own projects, when the screen loads, then she sees the full cross-event trail with every field of every entry intact.

**FEAT-13.SPEC-005-AC-02:** Given Owen is signed into his company's portal, when he looks for any way to reach the cross-event activity trail, then no tab, link, or navigation path to it exists anywhere in his portal view.

**FEAT-13.SPEC-005-AC-03:** Given Priya is signed into her company's portal, when she looks for any way to reach the cross-event activity trail, then no tab, link, or navigation path to it exists anywhere in her portal view.

**FEAT-13.SPEC-005-AC-04:** Given Owen just accepted a proposal, when he looks at his own portal, then he sees the outcome of that action within FEAT-03's own confirmation, never the cross-event trail.

**FEAT-13.SPEC-005-AC-05:** Given Dana has an open support session on Nadia's account, when she opens the trail, then she sees the full trail read-only, with every field intact and no editable or actionable controls.

**FEAT-13.SPEC-005-AC-06:** Given Dana has no open support session, when she attempts to navigate directly to a trail URL, then the screen is unreachable, identical to the unauthenticated experience.

**FEAT-13.SPEC-005-AC-07:** Given Dana's support session closes, when she attempts to continue viewing the trail, then access ends immediately.

**FEAT-13.SPEC-005-AC-08:** Given Nadia is the only role permitted to produce a printable copy, when Dana looks for an equivalent control during her session, then none is rendered.

**FEAT-13.SPEC-005-AC-09:** Given Owen or Priya attempts to reach a trail entry through a stale or guessed direct link, when the request is made, then they land on their own portal home rather than any trail content.

**FEAT-13.SPEC-005-AC-10:** Given a person is a Primary contact for one freelancer and a Reviewer contact for a different freelancer, when they are signed into the first freelancer's portal, then their access to that freelancer's trail remains None, unaffected by their role at the other freelancer.

**FEAT-13.SPEC-005-AC-11:** Given Dana's support session opens and later closes, when Nadia opens her trail afterward, then she sees both the opened and closed entries for that session, each attributed to Dana.

**FEAT-13.SPEC-005-AC-12:** Given Nadia views an entry she has access to, when she reads it, then every field (event description, actor, timestamp, affected-record reference) is shown -- no field is hidden from a role that can see the entry at all.

**FEAT-13.SPEC-005-AC-13:** Given Dana's open session spans a freelancer account with several projects, when she opens the trail for any of those projects, then her View access applies identically across all of them.

**FEAT-13.SPEC-005-AC-14:** Given Nadia's account has only a single project, when she opens the trail, then her Full access applies exactly as it would for an account with many projects.

**FEAT-13.SPEC-005-AC-15:** Given Owen attempts to reach the printable record copy screen directly, when the request is made, then the screen is unreachable to him, identical to his experience with the trail itself.

### User Story 6 - Retention & Account-Deletion Purge Rule (Priority: P1)

Governs how Activity Log Entries are retained for the life of the freelancer's account, removed only on account deletion subject to legal financial-record retention, and how a client contact's erasure request keeps their name on entries that are evidence.

**Acceptance Scenarios:**

**FEAT-13.SPEC-006-AC-01:** Given a freelancer account with activity entries of any age, when no account-deletion action has been taken, then no entry is ever automatically purged for age or volume alone.

**FEAT-13.SPEC-006-AC-02:** Given Nadia looks for a way to delete a single activity entry, when she searches every screen this feature offers, then no such control exists anywhere.

**FEAT-13.SPEC-006-AC-03:** Given Nadia initiates full account deletion through FEAT-24, when the deletion executes, then every activity entry with no financial-record character is removed as part of it.

**FEAT-13.SPEC-006-AC-04:** Given Nadia's account has invoice-sent and payment-recorded activity entries subject to legal financial-record retention, when her account deletion executes, then those specific entries are retained for platform parameter: `financial-record-legal-retention-period` rather than being purged immediately with the rest of her data.

**FEAT-13.SPEC-006-AC-05:** Given a client contact submits an erasure request through FEAT-18, when the request is processed, then their Client Contact details are removed while every existing entry that names them as actor keeps that name unchanged.

**FEAT-13.SPEC-006-AC-06:** Given a client contact was erased in the past, when Nadia opens an entry naming that contact today, then the entry still displays their name exactly as it was written.

**FEAT-13.SPEC-006-AC-07:** Given an erasure request is submitted moments before Nadia initiates account deletion, when both actions process, then the erasure's name-retention behavior applies first, and the subsequent deletion purges entries exactly as it otherwise would.

**FEAT-13.SPEC-006-AC-08:** Given Nadia generates a data export before deleting her account, when the export includes activity entries, then any entry naming a since-erased contact still shows that contact's retained name in the export.

**FEAT-13.SPEC-006-AC-09:** Given two contacts at the same client company each submit separate erasure requests, when both are processed, then each contact's name is independently retained on their own entries with no interaction between the two requests.

**FEAT-13.SPEC-006-AC-10:** Given a freelancer account with no invoices ever sent, when the account is deleted, then every activity entry is removed outright, since no entry qualifies for the legal-retention carve-out.

**FEAT-13.SPEC-006-AC-11:** Given the legal financial-record retention period for a retained entry elapses after account deletion, when the period ends, then the entry is removed with nothing further retained.

**FEAT-13.SPEC-006-AC-12:** Given Nadia looks for a way to schedule or configure automatic time-based cleanup of her activity trail, when she searches account or trail settings, then no such capability exists anywhere in the product.

**FEAT-13.SPEC-006-AC-13:** Given a support session entry (opened/closed) exists with no financial-record character, when the account is deleted, then it is removed with the rest of the non-financial entries.

**FEAT-13.SPEC-006-AC-14:** Given an entry references a record (e.g., a deliverable) that some other feature's own rules later remove independently of account deletion, when that other record is removed, then the activity entry referencing it is unaffected and remains exactly as written.

**FEAT-13.SPEC-006-AC-15:** Given Nadia's account deletion is in progress, when she looks for any way to selectively preserve or exclude specific entries from the purge, then no such control exists -- the purge (subject to the legal-retention carve-out) applies uniformly.

**FEAT-13.SPEC-006-AC-16:** Given the freelancer's account has already been deleted and its legal-retention window has not yet elapsed, when any role attempts to view a retained entry through the product, then no screen or access path exists, since the account itself no longer exists.

### Edge Cases

- **FEAT-13.SPEC-001 (Activity Trail):** A very long history loads in pages with a loading indicator at the list's end, and entries never change while their links open the affected record's current state. The trail is a live view where new entries appear at the top without refresh, and an operator's view becomes unreachable the moment the support session closes. Source: `docs/blueprint/specifications/FEAT-13-immutable-activity-audit-trail/FEAT-13.SPEC-001-activity-trail.md` (section: Edge Cases)
- **FEAT-13.SPEC-002 (Printable Record Copy):** A long history prints in full across multiple pages rather than truncating, and a record later deleted or superseded does not affect an already produced copy. Double taps on Print / Save are ignored, and a scoped entry that no longer resolves shows the could-not-load Error state with Retry. Source: `docs/blueprint/specifications/FEAT-13-immutable-activity-audit-trail/FEAT-13.SPEC-002-printable-record-copy.md` (section: Edge Cases)
- **FEAT-13.SPEC-003 (Activity Entry Recording):** Concurrent events each write their own entry and a duplicate report of the same event is made idempotent by the duplicate check. A failed write is retried at the platform-parameter interval until it succeeds (the triggering action is not complete until its entry is durable), and an actor whose details were later erased keeps their name as it stood at the event (FEAT-13.SPEC-006). Source: `docs/blueprint/specifications/FEAT-13-immutable-activity-audit-trail/FEAT-13.SPEC-003-activity-entry-recording.md` (section: Edge Cases)
- **FEAT-13.SPEC-004 (Entry Immutability, Content & Attribution Rules):** When no explicit event moment is supplied occurred_at equals the write-time timestamp, and entries with the same event type, actor and record but different times are kept distinct. An entry's reference and content stand exactly as written even if the referenced record later changed, and the project reference may be omitted only for genuinely account-level support-session events. Source: `docs/blueprint/specifications/FEAT-13-immutable-activity-audit-trail/FEAT-13.SPEC-004-entry-immutability-content-attribution-rules.md` (section: Edge Cases)
- **FEAT-13.SPEC-005 (Activity Trail Access & Visibility Rules):** A client contact reaching for the trail by a stale or guessed link is denied as any unauthorized attempt, and a closed support session grants no residual access. A person who is a contact for several freelancers still has no access to any trail, and an open support session extends view access across every project on the account being viewed. Source: `docs/blueprint/specifications/FEAT-13-immutable-activity-audit-trail/FEAT-13.SPEC-005-activity-trail-access-visibility-rules.md` (section: Edge Cases)
- **FEAT-13.SPEC-006 (Retention & Account-Deletion Purge Rule):** Invoice-related activity entries subject to legal financial-record retention survive account deletion, and a contact erasure processed just before deletion keeps the contact's name on existing entries. A data export before deletion includes entries with the name as retained, and each of two contacts' erasures independently preserves only that contact's own entries. Source: `docs/blueprint/specifications/FEAT-13-immutable-activity-audit-trail/FEAT-13.SPEC-006-retention-account-deletion-purge-rule.md` (section: Edge Cases)

## Requirements (mandatory)

### Functional Requirements

- **FR-001**: The system MUST implement **FEAT-13.SPEC-001** (Activity Trail) as specified: Nadia browses the full, permanent chronological history of everything recorded against a project; Dana views the identical trail read-only while a support session is open. Full spec: `docs/blueprint/specifications/FEAT-13-immutable-activity-audit-trail/FEAT-13.SPEC-001-activity-trail.md`
- **FR-002**: The system MUST implement **FEAT-13.SPEC-002** (Printable Record Copy) as specified: Nadia produces a printable, unalterable copy of a project's full activity trail, or of one selected entry, to show a client during a scope dispute. Full spec: `docs/blueprint/specifications/FEAT-13-immutable-activity-audit-trail/FEAT-13.SPEC-002-printable-record-copy.md`
- **FR-003**: The system MUST implement **FEAT-13.SPEC-003** (Activity Entry Recording) as specified: Writes one append-only Activity Log Entry whenever any of eleven other features reports a record-worthy event, capturing event type, actor, timestamp, and the affected record. Full spec: `docs/blueprint/specifications/FEAT-13-immutable-activity-audit-trail/FEAT-13.SPEC-003-activity-entry-recording.md`
- **FR-004**: The system MUST implement **FEAT-13.SPEC-004** (Entry Immutability, Content & Attribution Rules) as specified: Defines the required fields every Activity Log Entry must carry, the fixed vocabulary of recordable events, and the hard prohibition against ever editing or deleting a written entry. Full spec: `docs/blueprint/specifications/FEAT-13-immutable-activity-audit-trail/FEAT-13.SPEC-004-entry-immutability-content-attribution-rules.md`
- **FR-005**: The system MUST implement **FEAT-13.SPEC-005** (Activity Trail Access & Visibility Rules) as specified: Governs who may see the cross-event activity trail and its entries -- Nadia Full, Dana View (session-scoped), Owen and Priya None -- versus who sees only the outcome of their own actions within their own scoped views elsewhere in the product. Full spec: `docs/blueprint/specifications/FEAT-13-immutable-activity-audit-trail/FEAT-13.SPEC-005-activity-trail-access-visibility-rules.md`
- **FR-006**: The system MUST implement **FEAT-13.SPEC-006** (Retention & Account-Deletion Purge Rule) as specified: Governs how Activity Log Entries are retained for the life of the freelancer's account, removed only on account deletion subject to legal financial-record retention, and how a client contact's erasure request keeps their name on entries that are evidence. Full spec: `docs/blueprint/specifications/FEAT-13-immutable-activity-audit-trail/FEAT-13.SPEC-006-retention-account-deletion-purge-rule.md`

### Key Entities

- Activity Log Entry (create)

## Success Criteria (mandatory)

### Measurable Outcomes

- **SC-001**: In at least 90% of reported scope disputes, the freelancer is able to locate a specific, relevant timestamped record in the activity trail within 2 minutes (metric: Dispute Resolution Confidence). Source: `docs/blueprint/features/success-metrics.md`
- **SC-002**: Activity entries written, trail views and shared records are each observable as distinct signals (activity_entry_written, activity_trail_viewed, activity_record_shared). Source: `docs/blueprint/features/product-features.md`

## Assumptions

- **ASMP-15**: Records are append-only and immutable once created. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-18**: Support access is read-only and every support session is announced to the freelancer and kept in her activity trail. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-20**: Evidence outlives a contact's erasure request, but only as far as needed. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-25**: Financial and evidentiary records are timestamped at creation and never silently altered afterward. Full register: `docs/blueprint/features/assumptions-constraints.md`

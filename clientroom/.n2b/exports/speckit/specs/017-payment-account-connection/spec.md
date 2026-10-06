# Feature Specification: Payment Account Connection

**Blueprint feature:** FEAT-32
**Priority tier:** Core
**Build order:** 017 of 33
**Depends on:** —
**Blueprint source:** `docs/blueprint/specifications/FEAT-32-payment-account-connection/`

## User Scenarios & Testing (mandatory)

### User Story 1 - Payment Connection Screen (Priority: P1)

Nadia connects, views the readiness status of, reconnects, or disconnects her payment-processor account from one settings surface.

**Acceptance Scenarios:**

**FEAT-32.SPEC-001-AC-01:** Given Nadia has no connected payment account, when she opens this screen, then it shows the Empty state with the explanation text and a single "Connect payment account" button, and no "Reconnect" control.

**FEAT-32.SPEC-001-AC-02:** Given Nadia is on the Empty state, when she taps "Connect payment account," then the consent notice dialog appears with the verbatim consent text and "Continue" and "Cancel" buttons, and no data has left the product and no record exists yet.

**FEAT-32.SPEC-001-AC-03:** Given FEAT-32.SPEC-003 applies a Connected outcome, when Nadia is viewing this screen, then it shows "Ready to accept payments" and the list of currently available payment methods, with a "Disconnect" button.

**FEAT-32.SPEC-001-AC-04:** Given FEAT-32.SPEC-003 applies a Needs attention outcome, when Nadia is viewing this screen, then it shows "Needs attention" and the specific reason rendered exactly as stored, with "Reconnect," "Disconnect," and "Contact support."

**FEAT-32.SPEC-001-AC-05:** Given Nadia is in the Needs attention state, when she taps "Reconnect," then the consent notice dialog appears, and after she taps "Continue" the screen enters the Connecting state using the same hand-off as Connect.

**FEAT-32.SPEC-001-AC-06:** Given Nadia is on the Connected state, when she taps "Disconnect," then the dialog "If you disconnect, your open invoices will lose their pay links until you connect a payment account again. Any payment already submitted to your processor will not be affected." appears with "Disconnect" and "Cancel."

**FEAT-32.SPEC-001-AC-07:** Given the disconnect warning dialog is open, when Nadia taps "Cancel," then the dialog closes and the connection is unchanged.

**FEAT-32.SPEC-001-AC-08:** Given the disconnect warning dialog is open, when Nadia taps "Disconnect," then FEAT-32.SPEC-004 removes the connection and the screen returns to the Empty state with "Payment account disconnected."

**FEAT-32.SPEC-001-AC-09:** Given a Connect or Reconnect hand-off fails or is abandoned mid-flow, when the failure is reported, then the previous state's content remains visible beneath the "We couldn't complete the connection. Try again." banner (Empty content for a failed first connect).

**FEAT-32.SPEC-001-AC-10:** Given Nadia is on the Needs attention state, when she taps "Contact support," then she is navigated to FEAT-31's support request entry point.

**FEAT-32.SPEC-001-AC-11:** Given Owen is signed in to his client portal session, when he looks for any route to this screen, then none exists.

**FEAT-32.SPEC-001-AC-12:** Given Dana is inside a logged support session on Nadia's account, when she wants to see the payment connection status, then she sees it on her own support-session surface (FEAT-31), never on this screen.

**FEAT-32.SPEC-001-AC-13:** Given Nadia's session expires while a hand-off is in progress, when she signs back in, then this screen shows exactly the state persisted by FEAT-32.SPEC-002/SPEC-003 at the moment of expiry (Connecting if no outcome has been applied).

**FEAT-32.SPEC-001-AC-14:** Given Nadia loses connectivity while this screen is open, when the loss is detected, then the banner "You're offline -- reconnect to manage your payment account." appears and Connect, Reconnect, and Disconnect are all disabled.

**FEAT-32.SPEC-001-AC-15:** Given Nadia has this screen open in two tabs and disconnects in one, when she then taps Disconnect in the other (now-stale) tab, then the action is a no-op and that tab shows the Empty state on its next refresh.

**FEAT-32.SPEC-001-AC-16:** Given a Payment Account Connection record already exists in any status, when Nadia views this screen, then no "Connect" action is shown; and given no record exists, then no "Reconnect" action is shown -- the offered actions match the current status (Connected: Disconnect; Needs attention: Reconnect and Disconnect; Connecting: none, except "Start over" once the hand-off is older than platform parameter: `payment-connect-handoff-timeout`; Empty: Connect only).

**FEAT-32.SPEC-001-AC-17:** Given Nadia disconnects while a client's payment is already submitted to the processor, when the disconnect completes, then this screen shows the Empty state and the in-flight payment is unaffected.

**FEAT-32.SPEC-001-AC-18:** Given the consent notice dialog is open, when Nadia taps "Continue," then the dialog closes, FEAT-32.SPEC-002 initiates the hand-off, and the screen enters the Connecting state showing "Setting up your connection -- this usually takes a few minutes."

**FEAT-32.SPEC-001-AC-19:** Given the consent notice dialog is open from Connect, Reconnect, or Retry, when Nadia taps "Cancel" or presses Escape, then the dialog closes, no data leaves the product, no record is created or changed, the screen remains in the state it was in before (Empty, Needs attention, or Error banner), and focus returns to the button that opened the dialog.

**FEAT-32.SPEC-001-AC-20:** Given the "We couldn't complete the connection. Try again." banner is showing, when Nadia taps its "Retry" button, then the consent notice dialog appears again before any new hand-off starts.

**FEAT-32.SPEC-001-AC-21:** Given the consent notice dialog opens, when it appears, then focus moves to its heading "Before you connect," stays inside the dialog until it closes, and after Continue moves to the Connecting status line.

**FEAT-32.SPEC-001-AC-22:** Given FEAT-32.SPEC-006 reports a final delivery failure for the email announcing the current Connected or Needs attention status, when Nadia opens this screen, then the banner "We couldn't email you about this change to your payment account. The status shown below is current." with a "Dismiss" button appears above the status line.

**FEAT-32.SPEC-001-AC-23:** Given the delivery warning banner is showing, when Nadia taps "Dismiss," or a new status outcome is applied, or she disconnects, then the banner clears and does not reappear for that status outcome on reload.

**FEAT-32.SPEC-001-AC-24:** Given Nadia is on the Needs attention state, when she taps "Disconnect" and confirms in the warning dialog, then FEAT-32.SPEC-004 removes the connection and the screen returns to the Empty state with "Payment account disconnected."

**FEAT-32.SPEC-001-AC-25:** Given Nadia opens this screen, when the fetch of the current connection record is in progress, then it shows "Loading your payment account..." with no status and no Connect, Reconnect, or Disconnect control.

**FEAT-32.SPEC-001-AC-26:** Given the initial fetch fails, when the failure is detected, then the screen shows "We couldn't load your payment account. Try again." with a "Retry" button and no status line or action controls.

**FEAT-32.SPEC-001-AC-27:** Given the Load failure state is showing, when Nadia taps "Retry" and the fetch succeeds, then the screen shows the state the record holds (Empty, Connecting, Connected, or Needs attention).

**FEAT-32.SPEC-001-AC-28:** Given FEAT-32.SPEC-003 applied Needs attention because a readiness report listed zero payment methods, when Nadia views this screen, then the reason reads "Your payment account has no payment methods turned on yet. Turn on card or bank transfer in your payment account, then reconnect."

**FEAT-32.SPEC-001-AC-29:** Given Nadia tapped Continue and navigates away while the hand-off is in progress, when she reopens this screen before an outcome is applied, then it shows the Connecting state (from the persisted record) rather than Empty.

**FEAT-32.SPEC-001-AC-30:** Given Nadia's hand-off started less than platform parameter: `payment-connect-handoff-timeout` ago and no outcome has been reported, when she views the Connecting state, then it shows only the progress indicator and status line with no action button.

**FEAT-32.SPEC-001-AC-31:** Given the hand-off is older than platform parameter: `payment-connect-handoff-timeout` and no outcome has been reported, when Nadia views the Connecting state (on load, or while the screen stays open), then the line "This is taking too long. You can start over." and a "Start over" button appear, and Connect, Reconnect, and Disconnect remain hidden.

**FEAT-32.SPEC-001-AC-32:** Given the "Start over" button is showing, when Nadia taps it and confirms "Start over" in the dialog, then FEAT-32.SPEC-003 applies the abandoned-hand-off handling: a first connect returns to the Empty content beneath the "We couldn't complete the connection. Try again." banner, and a Reconnect returns to the Needs attention state beneath that banner; and when she taps "Cancel" or presses Escape instead, then the screen stays Connecting and no record changes.

### User Story 2 - Payment Account Connection & Status Reporting (Priority: P1)

Initiates the connect/reconnect hand-off with the payment-processing capability and receives back readiness status, the specific attention reason, available payment methods, and reversal/chargeback notices to relay onward.

**Acceptance Scenarios:**

**FEAT-32.SPEC-002-AC-01:** Given Nadia has no connected payment account, when she initiates the Connect hand-off, then the first-connect disclosure notice appears before any data leaves the product.

**FEAT-32.SPEC-002-AC-02:** Given Nadia has seen and accepted the disclosure notice, when the hand-off proceeds, then her name, business details, and sign-in email are shared with the capability and no card or bank credential data is ever sent.

**FEAT-32.SPEC-002-AC-03:** Given a hand-off is in progress, when the capability reports the account is ready, then this spec routes the readiness confirmation to FEAT-32.SPEC-003.

**FEAT-32.SPEC-002-AC-04:** Given a Connected account, when the capability restricts it or requests more information, then this spec routes the restriction event, including its specific reason, to FEAT-32.SPEC-003.

**FEAT-32.SPEC-002-AC-05:** Given a hand-off is in progress, when it fails or is abandoned before completing, then this spec routes that outcome to FEAT-32.SPEC-003, which applies no change to the existing record.

**FEAT-32.SPEC-002-AC-06:** Given a previously paid invoice, when the capability reports a chargeback or reversal against it, then this spec correlates the notice to that invoice and relays it to FEAT-25.

**FEAT-32.SPEC-002-AC-07:** Given Nadia is mid-hand-off and the capability is slow to respond, when platform parameter: `payment-connection-handoff-slow-threshold` elapses, then FEAT-32.SPEC-001 adds "Still checking your connection -- this is taking longer than usual."

**FEAT-32.SPEC-002-AC-08:** Given the capability is down, when Nadia attempts to Connect or Reconnect, then the button is disabled with "Connecting payment accounts is temporarily unavailable. Try again in a few minutes." and any existing connection's status is unaffected.

**FEAT-32.SPEC-002-AC-09:** Given the capability rejects a hand-off outright, when the rejection is reported, then FEAT-32.SPEC-001 shows "We couldn't complete the connection. Try again." with the previous status (if any) intact.

**FEAT-32.SPEC-002-AC-10:** Given a readiness-confirmed event is delivered twice for the same hand-off, when the second delivery arrives, then FEAT-32.SPEC-003's already-applied guard means nothing further changes and no duplicate confirmation email fires.

**FEAT-32.SPEC-002-AC-11:** Given events arrive out of order, when a stale readiness event arrives after a more recent restriction event, then FEAT-32.SPEC-003 applies the most recent event by report time, leaving the restriction in place.

**FEAT-32.SPEC-002-AC-12:** Given Nadia has already disconnected her account, when a stale status-report event for the removed connection arrives, then it is discarded and no record is recreated.

**FEAT-32.SPEC-002-AC-13:** Given an invoice was removed through FEAT-24 account deletion, when a reversal notice referencing it arrives afterward, then the notice is discarded since it cannot be correlated to an existing record.

**FEAT-32.SPEC-002-AC-14:** Given the capability goes down mid-hand-off before readiness is confirmed, when the outage is detected, then FEAT-32.SPEC-003 deletes the interim Connecting record (first connect) or clears the in-progress marker (Reconnect), no half-confirmed record remains, and FEAT-32.SPEC-001 shows the capability-down message.

**FEAT-32.SPEC-002-AC-15:** Given Nadia's connection is in Needs attention, when she taps Reconnect, then the same disclosure notice appears before any data leaves the product.

**FEAT-32.SPEC-002-AC-16:** Given the disclosure notice is showing, when Nadia taps "Cancel," then no hand-off starts, nothing is sent to the capability, no record is created or changed, and FEAT-32.SPEC-001 remains in its prior state.

**FEAT-32.SPEC-002-AC-17:** Given no record exists and Nadia taps "Continue" in the notice, when the hand-off starts, then this spec creates the record with `status` Connecting and `handoff_started_at` set, before contacting the capability; for a Reconnect it sets only `handoff_started_at` and leaves `status` Needs attention.

**FEAT-32.SPEC-002-AC-18:** Given a first-connect hand-off fails or is abandoned, when FEAT-32.SPEC-003 applies the outcome, then the interim Connecting record no longer exists and no half-created record remains.

### User Story 3 - Connection Status Sync (Priority: P1)

Applies the processor-reported status (Connected, Needs attention, or a failed/abandoned attempt) and available payment methods to the Payment Account Connection record the instant it is reported.

**Acceptance Scenarios:**

**FEAT-32.SPEC-003-AC-01:** Given Nadia has initiated a Connect hand-off, when the capability reports readiness, then this automation sets `status` to Connected, sets `processor_account_reference`, and sets `available_payment_methods` from the report.

**FEAT-32.SPEC-003-AC-02:** Given a Connected account, when the capability reports a restriction with a stated reason, then this automation sets `status` to Needs attention carrying that reason and clears `available_payment_methods` to none.

**FEAT-32.SPEC-003-AC-03:** Given a Reconnect attempt on a Needs attention record fails or is abandoned, when this automation receives that outcome, then it applies no change to `status`, `processor_account_reference`, or `available_payment_methods` and only clears `handoff_started_at`.

**FEAT-32.SPEC-003-AC-04:** Given a restriction event arrives with no stated reason, when this automation evaluates it, then the event is discarded as malformed and the record's previous status stands.

**FEAT-32.SPEC-003-AC-05:** Given a readiness event and a restriction event arrive out of order, when this automation processes them, then it applies them in event-time order, not arrival order, so the true later state wins.

**FEAT-32.SPEC-003-AC-06:** Given Nadia has already disconnected her account, when a stale status-report event for the removed connection arrives, then this automation discards it without recreating a record.

**FEAT-32.SPEC-003-AC-07:** Given a Connected outcome is applied, when FEAT-32.SPEC-001 next loads, then it shows "Ready to accept payments" and the available methods, and FEAT-32.SPEC-006 fires the confirmation email.

**FEAT-32.SPEC-003-AC-08:** Given a Needs attention outcome is applied, when FEAT-32.SPEC-001 next loads, then it shows the specific reason verbatim, and FEAT-32.SPEC-006 fires the alert email.

**FEAT-32.SPEC-003-AC-09:** Given two events fire for the same connection at effectively the same time, when this automation processes them, then it applies them in event-time order and the record reflects the genuinely later state.

**FEAT-32.SPEC-003-AC-10:** Given an event fires while a previous run for the same connection is still in flight, when the second event arrives, then it queues and is applied afterward in event-time order rather than writing concurrently.

**FEAT-32.SPEC-003-AC-11:** Given the same readiness-confirmed event is delivered twice, when the second delivery arrives with a report time equal to the record's `last_event_reported_at`, then step 2 discards it as an already-applied duplicate, no field changes, and no second confirmation email fires.

**FEAT-32.SPEC-003-AC-12:** Given this automation's own write fails on first attempt, when the retry logic runs, then it retries at platform parameter: `connection-status-apply-retry-interval` intervals until it succeeds, with no maximum retry count.

**FEAT-32.SPEC-003-AC-13:** Given Reconnect is initiated while a stale event from a superseded attempt is still in flight, when the stale event arrives, then it is discarded and the new hand-off's own outcome is unaffected.

**FEAT-32.SPEC-003-AC-14:** Given a readiness-confirmed event reports zero available payment methods, when this automation evaluates it, then it sets `status` to Needs attention with the reason "Your payment account has no payment methods turned on yet. Turn on card or bank transfer in your payment account, then reconnect.", sets `available_payment_methods` to none, and triggers FEAT-32.SPEC-006's alert email carrying that reason.

**FEAT-32.SPEC-003-AC-15:** Given Nadia's first Connect hand-off (record status Connecting) fails or is abandoned, when this automation receives that outcome, then it deletes the interim record, FEAT-32.SPEC-001 shows the Error banner over the Empty state, and no record remains.

**FEAT-32.SPEC-003-AC-16:** Given a readiness event and a restriction event carry an identical report time, when both are processed, then the restriction is applied (or remains applied) and the readiness event is discarded.

**FEAT-32.SPEC-003-AC-17:** Given a duplicate event is discarded at step 2, when discarding completes, then FEAT-32.SPEC-006 is not triggered.

**FEAT-32.SPEC-003-AC-18:** Given Nadia's first-connect hand-off (record status Connecting) is older than platform parameter: `payment-connect-handoff-timeout` with no outcome reported, when she confirms "Start over" on FEAT-32.SPEC-001, then this automation deletes the interim record, FEAT-32.SPEC-001 shows the Error banner over the Empty state, and no record remains; and given a Reconnect hand-off in the same condition, then it clears only `handoff_started_at` and the record keeps its Needs attention `status`, `processor_account_reference`, and `available_payment_methods`.

**FEAT-32.SPEC-003-AC-19:** Given a "Start over" request arrives when no hand-off is in progress (an outcome was already applied or the record was removed) or when `handoff_started_at` is not yet older than platform parameter: `payment-connect-handoff-timeout`, when this automation evaluates it at step 5a, then it makes no change to the record and triggers no notification.

### User Story 4 - Disconnect Payment Account (Priority: P1)

Removes Nadia's connection reference on her explicit disconnect action, without cancelling any payment already submitted to the processor.

**Acceptance Scenarios:**

**FEAT-32.SPEC-004-AC-01:** Given Nadia has confirmed the disconnect warning on FEAT-32.SPEC-001, when this automation runs, then the Payment Account Connection record is removed in full and FEAT-32.SPEC-001 returns to the Empty state with "Payment account disconnected."

**FEAT-32.SPEC-004-AC-02:** Given FEAT-24's account-deletion sequence reaches its payment-disconnect step, when this automation runs, then the Payment Account Connection record is removed in full without a separate warning shown by this spec.

**FEAT-32.SPEC-004-AC-03:** Given no Payment Account Connection record exists when Nadia's disconnect is (redundantly) triggered, when this automation runs, then nothing changes and FEAT-32.SPEC-001 remains on the Empty state.

**FEAT-32.SPEC-004-AC-04:** Given a client's payment is already submitted to the processor, when Nadia disconnects, then the disconnect proceeds and the in-flight payment is entirely unaffected.

**FEAT-32.SPEC-004-AC-05:** Given the removal write itself fails, when the failure occurs, then FEAT-32.SPEC-001 shows "We couldn't disconnect your account. Try again." and the record is retained unchanged.

**FEAT-32.SPEC-004-AC-06:** Given Nadia double-taps Disconnect, when the second tap reaches this automation while the first is still processing, then the second tap produces the no-action outcome without error.

**FEAT-32.SPEC-004-AC-07:** Given a FEAT-32.SPEC-003 status-report event is in flight for the same connection, when this automation completes the removal first, then the status-report event later finds no record and is discarded.

**FEAT-32.SPEC-004-AC-08:** Given Nadia reconnects after a disconnect, when she does so, then FEAT-32.SPEC-002's Create operation runs again rather than restoring any prior record.

**FEAT-32.SPEC-004-AC-09:** Given account deletion and Nadia's own manual disconnect are triggered at effectively the same time, when both reach this automation, then whichever reaches step 2 first performs the removal and the other takes the no-action outcome.

**FEAT-32.SPEC-004-AC-10:** Given the connection is removed by this automation, when an open invoice's pay link is opened afterward, then it shows the fallback experience owned by FEAT-09.SPEC-009 and FEAT-10, not a working payment form.

**FEAT-32.SPEC-004-AC-11:** Given this automation is triggered by FEAT-24, when it completes, then FEAT-24's own sequence proceeds to its next step informed that the payment-account removal succeeded.

**FEAT-32.SPEC-004-AC-12:** Given this automation's write fails when triggered by FEAT-24, when the failure occurs, then FEAT-24's sequence is informed the step did not complete and retries per its own process, rather than this automation silently reporting success.

### User Story 5 - Payment Connection Authorization & Validation Rules (Priority: P1)

Governs the one-account-per-freelancer limit, who may connect/reconnect/disconnect versus view only, the disconnect warning, and the processor-authoritative contention rule that protects an in-flight payment from a concurrent disconnect.

**Acceptance Scenarios:**

**FEAT-32.SPEC-005-AC-01:** Given Nadia has no existing Payment Account Connection record, when she attempts to Connect, then the action is allowed.

**FEAT-32.SPEC-005-AC-02:** Given Nadia already has a Payment Account Connection record in any status, when she views FEAT-32.SPEC-001, then no Connect action is shown -- only Reconnect (Needs attention) and/or Disconnect (Connected or Needs attention), and neither while Connecting.

**FEAT-32.SPEC-005-AC-03:** Given Owen (Client Primary Contact) is signed in, when he looks for any way to connect, view, reconnect, or disconnect a payment account, then no such action or view exists anywhere in his portal.

**FEAT-32.SPEC-005-AC-04:** Given Priya (Client Reviewer Contact) is signed in, when she looks for any way to interact with this entity, then no such action or view exists anywhere in her portal.

**FEAT-32.SPEC-005-AC-05:** Given Dana (Support Operator) is inside a logged support session on Nadia's account, when she views the payment connection, then she sees the status only, never the processor account reference or any credential-adjacent detail.

**FEAT-32.SPEC-005-AC-06:** Given Dana is inside a logged support session, when she looks for a Connect, Reconnect, or Disconnect control, then none is shown -- her session is view-only.

**FEAT-32.SPEC-005-AC-07:** Given Nadia's connection is in Needs attention status, when she attempts to Reconnect, then the action is allowed.

**FEAT-32.SPEC-005-AC-08:** Given Nadia has no connection record (post-disconnect or never connected), when she views FEAT-32.SPEC-001, then only Connect is offered and no Reconnect control appears.

**FEAT-32.SPEC-005-AC-09:** Given Nadia's connection is Connected or Needs attention, when she initiates Disconnect, then the warning dialog "If you disconnect, your open invoices will lose their pay links until you connect a payment account again. Any payment already submitted to your processor will not be affected." appears before anything is removed.

**FEAT-32.SPEC-005-AC-10:** Given the disconnect warning dialog is shown, when Nadia taps "Cancel," then the connection is unchanged and no removal occurs.

**FEAT-32.SPEC-005-AC-11:** Given the disconnect warning dialog is shown, when Nadia taps "Disconnect," then the removal proceeds via FEAT-32.SPEC-004.

**FEAT-32.SPEC-005-AC-12:** Given a client's payment is already submitted to the processor, when Nadia confirms a disconnect, then the disconnect is not blocked and the in-flight payment is not cancelled.

**FEAT-32.SPEC-005-AC-13:** Given the connection is removed by disconnect, when a pay link is opened afterward, then it reflects the now-disconnected state per the fallback rule owned by FEAT-09.SPEC-009 and FEAT-10.

**FEAT-32.SPEC-005-AC-14:** Given a readiness-confirmed event reports zero available payment methods, when FEAT-32.SPEC-003 evaluates it (Processing Logic step 3), then the outcome is Needs attention rather than Connected, `available_payment_methods` is none, and the reason is "Your payment account has no payment methods turned on yet. Turn on card or bank transfer in your payment account, then reconnect."

**FEAT-32.SPEC-005-AC-15:** Given the connection's status is set to Needs attention, when the update is applied, then `available_payment_methods` is cleared to none regardless of what was available immediately before.

**FEAT-32.SPEC-005-AC-16:** Given Nadia double-taps Disconnect, when the second tap reaches processing after the first has already removed the reference, then it is a no-op with no error shown.

**FEAT-32.SPEC-005-AC-17:** Given no field on this entity is ever directly editable by a user, when any party attempts to set `status`, `processor_account_reference`, or `available_payment_methods` directly, then no such input path exists -- all three are set only through FEAT-32.SPEC-002, SPEC-003, or SPEC-004.

**FEAT-32.SPEC-005-AC-18:** Given a restriction event (Needs attention) is applied while Nadia is mid-way through confirming a disconnect, when she then confirms the disconnect, then the removal still proceeds regardless of the just-applied Needs attention status.

**FEAT-32.SPEC-005-AC-19:** Given Nadia's hand-off is in progress and `handoff_started_at` is older than platform parameter: `payment-connect-handoff-timeout` with no outcome applied, when she views FEAT-32.SPEC-001, then "Start over" is allowed (and Connect, Reconnect, and Disconnect remain hidden), and confirming it lets FEAT-32.SPEC-003 apply the abandoned-hand-off handling.

**FEAT-32.SPEC-005-AC-20:** Given a hand-off is in progress but `handoff_started_at` is not yet older than platform parameter: `payment-connect-handoff-timeout`, or no hand-off is in progress, when Nadia views FEAT-32.SPEC-001, then no "Start over" control is shown, and a "Start over" request that arrives anyway changes nothing.

**FEAT-32.SPEC-005-AC-21:** Given Owen, Priya, or Dana is signed in, when any of them looks for a "Start over" control, then none exists on any surface available to them.

### User Story 6 - Connection Status Notifications (Priority: P1)

Sends Nadia a confirmation email when her account connects and an alert email when the connection breaks or needs attention, so she learns about a change to her ability to accept payments even when she is away from the product.

**Acceptance Scenarios:**

**FEAT-32.SPEC-006-AC-01:** Given FEAT-32.SPEC-003 applies a Connected outcome, when this notification fires, then Nadia receives an email with subject "Your payment account is connected."

**FEAT-32.SPEC-006-AC-02:** Given FEAT-32.SPEC-003 applies a Needs attention outcome, when this notification fires, then Nadia receives an email with subject "Action needed: your payment account needs attention" carrying the processor's specific reason verbatim.

**FEAT-32.SPEC-006-AC-03:** Given Nadia opens her Connected confirmation email, when she taps "View payment settings," then she lands on FEAT-32.SPEC-001.

**FEAT-32.SPEC-006-AC-04:** Given Nadia opens her Needs attention alert email, when she taps "Reconnect your account," then she lands on FEAT-32.SPEC-001.

**FEAT-32.SPEC-006-AC-05:** Given Nadia has no way to opt out of either email, when her notification preferences are checked, then no preference control exists for either and both always send.

**FEAT-32.SPEC-006-AC-06:** Given a status change occurs at any hour, when this notification fires, then it sends immediately regardless of Nadia's configured quiet hours.

**FEAT-32.SPEC-006-AC-07:** Given delivery fails on the first attempt, when the retry logic runs, then it retries up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, and after the final failure Nadia sees, above the status line on FEAT-32.SPEC-001, the banner "We couldn't email you about this change to your payment account. The status shown below is current." with a "Dismiss" button.

**FEAT-32.SPEC-006-AC-08:** Given a Needs attention alert is still being retried, when the connection returns to Connected before the retry succeeds, then the pending Needs attention alert is cancelled and the new Connected confirmation still sends.

**FEAT-32.SPEC-006-AC-09:** Given a Connected confirmation is queued, when the connection is later disconnected before that email is delivered, then the confirmation still sends, since it was accurate at the moment it was triggered.

**FEAT-32.SPEC-006-AC-10:** Given two Needs attention outcomes are applied in succession with different reasons, when each is applied, then each fires its own separate alert email.

**FEAT-32.SPEC-006-AC-11:** Given Nadia's account is deleted while an email is still queued for retry, when the deletion finalizes, then the pending delivery is cancelled.

**FEAT-32.SPEC-006-AC-12:** Given the same Connected outcome is not re-applied by FEAT-32.SPEC-003 for a duplicate or stale underlying event, when that duplicate event is discarded upstream, then this notification does not fire a second time.

**FEAT-32.SPEC-006-AC-13:** Given Owen or Priya is a contact at Nadia's client company, when a status change occurs on her connection, then neither receives any copy of either email.

**FEAT-32.SPEC-006-AC-14:** Given Nadia's connection is disconnected by her own action on FEAT-32.SPEC-001, when the disconnect completes, then neither of this spec's emails fires, since disconnect is not one of this spec's triggers.

**FEAT-32.SPEC-006-AC-15:** Given the delivery warning banner is showing on FEAT-32.SPEC-001 after a final delivery failure, when Nadia taps "Dismiss," or FEAT-32.SPEC-003 applies a new status outcome, or she disconnects, then the banner clears and does not return for that outcome.

**FEAT-32.SPEC-006-AC-16:** Given FEAT-32.SPEC-003 converts a readiness report with zero available payment methods into Needs attention, when this notification fires, then the alert email's `{attention_reason}` reads "Your payment account has no payment methods turned on yet. Turn on card or bank transfer in your payment account, then reconnect." and is never empty.

**FEAT-32.SPEC-006-AC-17:** Given a Needs attention alert is cancelled as superseded by a return to Connected, when the cancellation occurs, then no delivery warning is shown on FEAT-32.SPEC-001.

### Edge Cases

- **FEAT-32.SPEC-001 (Payment Connection Screen):** Returning mid-hand-off reloads the persisted Connecting state, double taps on Disconnect are ignored while processing, and a disconnect from one tab removes the connection regardless of a second tab's stale state. Processor status changes appear on the next load or refresh. Source: `docs/blueprint/specifications/FEAT-32-payment-account-connection/FEAT-32.SPEC-001-payment-connection-screen.md` (section: Edge Cases)
- **FEAT-32.SPEC-002 (Payment Account Connection & Status Reporting):** A duplicate readiness event changes nothing and sends no second confirmation email, out-of-order events are applied by the time the capability reports they occurred, and events for a since-disconnected connection or an invoice removed by account deletion are discarded without recreating records. Source: `docs/blueprint/specifications/FEAT-32-payment-account-connection/FEAT-32.SPEC-002-payment-account-connection-status-reporting.md` (section: Edge Cases)
- **FEAT-32.SPEC-003 (Connection Status Sync):** Concurrent readiness and restriction events are applied in event-time order and queue behind any in-flight run for the same connection, so the record never reflects a half-applied state. A restriction event with no stated reason is discarded as malformed, and events for a disconnected connection are discarded. Source: `docs/blueprint/specifications/FEAT-32-payment-account-connection/FEAT-32.SPEC-003-connection-status-sync.md` (section: Edge Cases)
- **FEAT-32.SPEC-004 (Disconnect Payment Account):** Disconnect racing with FEAT-24's own disconnect step or with an in-flight status event resolves by whichever removal reaches the record first, and the removal takes precedence over a late status event. A disconnect is not blocked by a client payment already submitted, which continues on its own path, and the disabled button prevents double submission. Source: `docs/blueprint/specifications/FEAT-32-payment-account-connection/FEAT-32.SPEC-004-disconnect-payment-account.md` (section: Edge Cases)
- **FEAT-32.SPEC-005 (Payment Connection Authorization & Validation Rules):** A second disconnect after the first is an idempotent no-op, and the one-account limit offers Connect with zero records and only Reconnect or Disconnect with exactly one record in any status. An operator's read-only session shows the new Not connected state on its next refresh, and an in-flight client payment proceeds independently of a disconnect. Source: `docs/blueprint/specifications/FEAT-32-payment-account-connection/FEAT-32.SPEC-005-payment-connection-authorization-validation-rules.md` (section: Edge Cases)
- **FEAT-32.SPEC-006 (Connection Status Notifications):** A pending Needs attention alert is cancelled if the connection returns to Connected first, while a pending Connected or Needs attention email still sends after a later disconnect since it was accurate when applied. Successive Needs attention outcomes with different reasons each fire their own alert, and account deletion cancels queued retries. Source: `docs/blueprint/specifications/FEAT-32-payment-account-connection/FEAT-32.SPEC-006-connection-status-notifications.md` (section: Edge Cases)

## Requirements (mandatory)

### Functional Requirements

- **FR-001**: The system MUST implement **FEAT-32.SPEC-001** (Payment Connection Screen) as specified: Nadia connects, views the readiness status of, reconnects, or disconnects her payment-processor account from one settings surface. Full spec: `docs/blueprint/specifications/FEAT-32-payment-account-connection/FEAT-32.SPEC-001-payment-connection-screen.md`
- **FR-002**: The system MUST implement **FEAT-32.SPEC-002** (Payment Account Connection & Status Reporting) as specified: Initiates the connect/reconnect hand-off with the payment-processing capability and receives back readiness status, the specific attention reason, available payment methods, and reversal/chargeback notices to relay onward. Full spec: `docs/blueprint/specifications/FEAT-32-payment-account-connection/FEAT-32.SPEC-002-payment-account-connection-status-reporting.md`
- **FR-003**: The system MUST implement **FEAT-32.SPEC-003** (Connection Status Sync) as specified: Applies the processor-reported status (Connected, Needs attention, or a failed/abandoned attempt) and available payment methods to the Payment Account Connection record the instant it is reported. Full spec: `docs/blueprint/specifications/FEAT-32-payment-account-connection/FEAT-32.SPEC-003-connection-status-sync.md`
- **FR-004**: The system MUST implement **FEAT-32.SPEC-004** (Disconnect Payment Account) as specified: Removes Nadia's connection reference on her explicit disconnect action, without cancelling any payment already submitted to the processor. Full spec: `docs/blueprint/specifications/FEAT-32-payment-account-connection/FEAT-32.SPEC-004-disconnect-payment-account.md`
- **FR-005**: The system MUST implement **FEAT-32.SPEC-005** (Payment Connection Authorization & Validation Rules) as specified: Governs the one-account-per-freelancer limit, who may connect/reconnect/disconnect versus view only, the disconnect warning, and the processor-authoritative contention rule that protects an in-flight payment from a concurrent disconnect. Full spec: `docs/blueprint/specifications/FEAT-32-payment-account-connection/FEAT-32.SPEC-005-payment-connection-authorization-validation-rules.md`
- **FR-006**: The system MUST implement **FEAT-32.SPEC-006** (Connection Status Notifications) as specified: Sends Nadia a confirmation email when her account connects and an alert email when the connection breaks or needs attention, so she learns about a change to her ability to accept payments even when she is away from the product. Full spec: `docs/blueprint/specifications/FEAT-32-payment-account-connection/FEAT-32.SPEC-006-connection-status-notifications.md`

### Key Entities

- Payment Account Connection (create, read, update, delete)

## Success Criteria (mandatory)

### Measurable Outcomes

- **SC-001**: At least 90% of freelancers have a connected, ready payment account at the moment their first invoice is sent, and connecting takes under 5 minutes of the freelancer's own time (metric: Payment Readiness Before First Invoice). Source: `docs/blueprint/features/success-metrics.md`
- **SC-002**: Connection starts, successful connections, needs-attention states and disconnections are each observable as distinct signals (payment_account_connect_started, payment_account_connected, payment_account_needs_attention, payment_account_disconnected). Source: `docs/blueprint/features/product-features.md`

## Assumptions

- **ASMP-04**: Freelancers in the target markets can open or already hold an account with an established payment processor. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-14**: The platform never holds or moves client funds; payments go directly into the freelancer's own processor account. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-28**: Payment-processing capability, including per-freelancer account connection and status reporting, is a required dependency. Full register: `docs/blueprint/features/assumptions-constraints.md`

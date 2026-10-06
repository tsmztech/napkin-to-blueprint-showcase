# Feature Specification: Subscription Plan & Billing Management

**Blueprint feature:** FEAT-23
**Priority tier:** Important
**Build order:** 014 of 33
**Depends on:** FEAT-01
**Blueprint source:** `docs/blueprint/specifications/FEAT-23-subscription-plan-billing-management/`

## User Scenarios & Testing (mandatory)

### User Story 1 - Plan & Billing Screen (Priority: P2)

Nadia views her current tier, usage against its client limit, and billing cycle, and initiates upgrade, responds to a downgrade offer, or cancels, from one surface.

**Acceptance Scenarios:**

**FEAT-23.SPEC-001-AC-01:** Given Nadia's plan is Free with active_client_count within the limit, when she opens this screen, then she sees the Free tier, her usage against platform parameter: `free-tier-active-client-limit`, and a Subscribe action.

**FEAT-23.SPEC-001-AC-02:** Given Nadia is at her free-tier client limit and arrives via the FEAT-01.SPEC-008 upgrade prompt, when this screen loads, then Subscribe is immediately visible as the primary action.

**FEAT-23.SPEC-001-AC-03:** Given Nadia is on the Free tier, when she taps Subscribe, selects a billing cycle, accepts the disclosure notice (first time), completes payment entry with the billing capability, and the charge succeeds, then the Current Plan card updates to show Paid with unlimited client capacity.

**FEAT-23.SPEC-001-AC-04:** Given Nadia's downgrade-eligible flag is raised (tier Paid, status Active), when this screen loads, then the downgrade offer banner appears with both "Downgrade" and "Keep my Paid plan" options.

**FEAT-23.SPEC-001-AC-05:** Given the downgrade offer is showing, when Nadia taps "Keep my Paid plan," then the offer is hidden for this session and no plan data changes.

**FEAT-23.SPEC-001-AC-06:** Given the downgrade offer is showing, when Nadia taps "Downgrade," reads the dialog stating the change is immediate, stops billing, and does not refund the current period, taps "Downgrade now," and the billing capability confirms billing has stopped, then the Current Plan card updates to Free with the billing cycle no longer shown and no charge made.

**FEAT-23.SPEC-001-AC-07:** Given Nadia's plan is Paid and Active, when she taps Cancel, confirms "Cancel plan," and the billing capability acknowledges, then the screen shows the Cancelled explanation with the exact period-end date.

**FEAT-23.SPEC-001-AC-08:** Given Nadia's plan status is Charge failed, when this screen loads, then the Charge failed banner shows the specific reason and a Retry action (retries remaining), and her already-active client work elsewhere remains unaffected.

**FEAT-23.SPEC-001-AC-09:** Given Nadia's plan status is Charge failed, when she taps Retry and the charge succeeds, then the banner clears and the Current Plan card confirms Active/Paid.

**FEAT-23.SPEC-001-AC-10:** Given Nadia's plan status is Charge failed with retries remaining, when she taps Retry and the charge fails again, then the banner updates with the new specific reason and Retry remains available within the grace window until the retry limit is reached.

**FEAT-23.SPEC-001-AC-11:** Given Nadia's plan status is Lapsed, when this screen loads, then the Current Plan card shows "Free -- Lapsed," she sees the Lapsed explanation confirming existing clients remain reachable, an Upgrade action, and a usage meter showing her count against the free-tier limit (not "unlimited").

**FEAT-23.SPEC-001-AC-12:** Given the initial plan data load fails, when the screen opens, then a "Couldn't load your plan. Try again." banner appears with a Retry option.

**FEAT-23.SPEC-001-AC-13:** Given Nadia loses connectivity while viewing this screen, when connectivity drops, then the offline banner appears, all mutating controls disable, and her last-loaded plan details remain visible.

**FEAT-23.SPEC-001-AC-14:** Given Nadia regains connectivity after being offline on this screen, when connectivity is restored, then controls re-enable and the plan is re-fetched.

**FEAT-23.SPEC-001-AC-15:** Given Nadia taps Subscribe twice rapidly, when the second tap occurs while the first is in progress, then the second tap is ignored.

**FEAT-23.SPEC-001-AC-16:** Given a network failure occurs while Nadia submits Cancel, when the failure is detected, then she sees "Couldn't complete this action. Check your connection and try again." with Retry, and her plan is unchanged.

**FEAT-23.SPEC-001-AC-17:** Given Nadia's plan state changed in another session since this screen last loaded, when she attempts an action here, then it is rejected as stale and the screen refreshes to show "Your plan status has changed -- here's the latest."

**FEAT-23.SPEC-001-AC-18:** Given Nadia has this screen open in two tabs and cancels in one, when she then attempts to accept a downgrade offer in the other tab, then that attempt is rejected as stale and the second tab refreshes to the Cancelled state.

**FEAT-23.SPEC-001-AC-19:** Given Dana has no open support session, when she attempts to reach this screen directly, then no such navigation path exists for her role.

**FEAT-23.SPEC-001-AC-20:** Given Owen is signed in to his client portal, when he looks for any navigation to this screen, then none exists.

**FEAT-23.SPEC-001-AC-21:** Given an unauthenticated visitor attempts to open this screen's link, when the attempt is made, then they are redirected to sign-in.

**FEAT-23.SPEC-001-AC-22:** Given Nadia's session expires while she has an in-progress Subscribe request not yet confirmed, when she signs back in, then the screen re-checks and displays the plan's actual current state rather than assuming the request completed.

**FEAT-23.SPEC-001-AC-23:** Given Nadia tapped Subscribe and the billing-cycle panel is open with neither option selected, when she views the panel, then "Continue" is disabled, and if she tries to proceed the message "Choose a monthly or yearly billing cycle to continue." appears; once she selects Monthly or Yearly, "Continue" enables and shows the price per platform parameter: `paid-plan-monthly-price` or platform parameter: `paid-plan-yearly-price`.

**FEAT-23.SPEC-001-AC-24:** Given Nadia is subscribing for the first time and has selected a cycle, when she taps "Continue," then the disclosure notice appears with "Continue" and "Cancel"; when she taps "Cancel," she returns to this screen with nothing sent and the plan unchanged; when she taps "Continue," she is taken to the billing capability's payment-entry experience.

**FEAT-23.SPEC-001-AC-25:** Given Nadia has already seen the disclosure notice, when she taps "How billing works," then the notice reopens with a single "Close" button, no data is sent, and "Close" returns focus to the link.

**FEAT-23.SPEC-001-AC-26:** Given Nadia taps Cancel, when the confirmation dialog opens, then it states her plan stays active through {period_end_date} and offers "Cancel plan" and "Keep my plan"; tapping "Keep my plan" closes the dialog with nothing sent and no data changed.

**FEAT-23.SPEC-001-AC-27:** Given Nadia completes payment entry and is returned to this screen, when the capability's outcome has not yet arrived, then the "Confirming your payment..." banner shows (adding "Still confirming -- this is taking longer than usual." after 10 seconds) and the plan card does not change until the outcome arrives.

**FEAT-23.SPEC-001-AC-28:** Given Nadia abandons the billing capability's payment-entry experience and is returned to this screen, when the screen loads, then she sees "You left before completing payment -- your plan hasn't changed." with Subscribe available, and no Charge failed state is shown.

**FEAT-23.SPEC-001-AC-29:** Given Nadia is on the Free tier and her Subscribe attempt fails, when the failure is reported, then the Attempt-failed banner shows "This charge couldn't be completed: {reason}." with "Try again," her plan stays Free with its prior status, no Charge failed banner appears, and reloading the screen clears the banner.

**FEAT-23.SPEC-001-AC-30:** Given Nadia's status is Charge failed and her retry attempts used equal platform parameter: `subscription-charge-retry-count` with the grace window still open, when this screen loads, then the Retry button is replaced by "You've used all your retries. Your plan stays fully usable until {grace_window_end_date}, then it lapses. You can cancel your plan at any time." and Cancel remains available.

**FEAT-23.SPEC-001-AC-31:** Given the load-error banner is showing, when Nadia taps Retry and the plan loads, then the banner is replaced by the Current Plan card and the state that applies to her plan.

**FEAT-23.SPEC-001-AC-32:** Given Nadia taps "Downgrade now" and the billing capability rejects the request, when the rejection is reported, then the Attempt-failed banner shows "We couldn't end your billing: {reason}. Your plan is unchanged." with "Try again," and the plan stays Paid and Active with the offer still available.

**FEAT-23.SPEC-001-AC-33:** Given Nadia's plan status is Charge failed and her active-client count is within the free-tier limit, when this screen loads, then no downgrade offer banner is shown; given a successful retry then returns status to Active, then the offer banner appears.

**FEAT-23.SPEC-001-AC-34:** Given Nadia is Lapsed and taps Upgrade, when she selects a cycle, continues, and the charge succeeds, then the Current Plan card updates to Paid with unlimited capacity and the Lapsed explanation is gone.

**FEAT-23.SPEC-001-AC-35:** Given Nadia's cancellation was acknowledged but the record is still being written, when this screen loads, then the Cancelled explanation area reads "We're finishing your cancellation -- no action needed." until the record completes.

**FEAT-23.SPEC-001-AC-36:** Given Nadia's plan is Paid and Nadia accepts a downgrade while her live client count has risen above the free-tier limit before the capability confirms, when the confirmation is applied, then the Current Plan card shows "Free -- Lapsed" with the over-limit note on the usage meter.

### User Story 2 - Free Plan Auto-Provisioning (Priority: P2)

Creates the Subscription Plan record on the free tier automatically the instant a new Freelancer Account is created, so there is never an explicit "no plan" state.

**Acceptance Scenarios:**

**FEAT-23.SPEC-002-AC-01:** Given Nadia completes sign-up (FEAT-20.SPEC-001), when her Freelancer Account is created, then a Subscription Plan record is created immediately with tier=Free, status=Active, and billing_cycle unset.

**FEAT-23.SPEC-002-AC-02:** Given Nadia's account was just created, when she opens the Plan & Billing Screen (FEAT-23.SPEC-001) for the first time, then it shows the Free tier -- never an empty or "no plan" state.

**FEAT-23.SPEC-002-AC-03:** Given Nadia's account was just created, when she attempts to add her first client (FEAT-01.SPEC-008), then the free-tier limit check evaluates immediately against her already-provisioned plan.

**FEAT-23.SPEC-002-AC-04:** Given account creation succeeds, when the immediate provisioning attempt fails, then Nadia's onboarding sequence and plan screen show "Setting up your plan -- try again in a moment" and the automation retries automatically.

**FEAT-23.SPEC-002-AC-05:** Given the provisioning retry succeeds after an initial failure, when Nadia next views her plan, then it shows the Free tier normally with no trace of the earlier failure.

**FEAT-23.SPEC-002-AC-06:** Given two new Freelancer Accounts are created at effectively the same time, when this automation fires for each, then each account receives its own independent Subscription Plan record with no interference between the two.

**FEAT-23.SPEC-002-AC-07:** Given a Freelancer Account already has a provisioned Subscription Plan, when a duplicate provisioning attempt is made for that same account, then it is rejected and the existing record is left unchanged.

**FEAT-23.SPEC-002-AC-08:** Given Nadia's account is newly created, when any other feature (FEAT-01, FEAT-16) reads her plan before she has taken any billing action, then it reads Free/Active with no distinction from a plan she might have actively chosen.

**FEAT-23.SPEC-002-AC-09:** Given Nadia's plan record was just created, when the commit completes, then a plan-created event (tier Free, status Active, actor "Automatic") is reported to FEAT-13.SPEC-003, and a trail-write retry never delays or reverses the plan or her onboarding.

**FEAT-23.SPEC-002-AC-10:** Given provisioning is still retrying after an initial failure, when Nadia's account deletion begins or completes (FEAT-24.SPEC-004), then the retry stops and no Subscription Plan record is created.

**FEAT-23.SPEC-002-AC-11:** Given Nadia's plan was provisioned before her account deletion began, when deletion completes, then the record is removed by FEAT-24.SPEC-004 and this automation does not run again for that account.

### User Story 3 - Subscription Billing Processing (Priority: P2)

Submits Nadia's own subscribe, stop-billing (downgrade), retry, and cancellation requests to the subscription-billing capability and receives back charge outcomes, renewal and period-end events, and failure reasons.

**Acceptance Scenarios:**

**FEAT-23.SPEC-003-AC-01:** Given Nadia is on the free tier at her client limit, when she taps Subscribe, selects a billing cycle, accepts the disclosure notice, and the capability confirms the charge, then her Subscription Plan is handed to FEAT-23.SPEC-004 with tier=Paid and the selected billing_cycle.

**FEAT-23.SPEC-003-AC-02:** Given Nadia has accepted a downgrade offer, when the capability confirms it has stopped billing, then the confirmation is handed to FEAT-23.SPEC-004 as a Paid-to-Free downgrade, no charge was submitted, and billing_cycle is cleared by that spec.

**FEAT-23.SPEC-003-AC-03:** Given Nadia confirms Cancel, when the cancellation request is submitted, then it is relayed to the capability so the current paid period continues to its scheduled end, and the capability's acknowledgment is handed to FEAT-23.SPEC-006.

**FEAT-23.SPEC-003-AC-04:** Given an existing Paid plan's billing cycle renews successfully, when the renewal-succeeded event arrives, then the plan's status remains Active with no user-facing interruption.

**FEAT-23.SPEC-003-AC-05:** Given a cancelled plan's paid period ends and Nadia's active-client count is within the free-tier limit, when the period-end event arrives, then FEAT-23.SPEC-004 transitions tier to Free (billing_cycle cleared, status Active).

**FEAT-23.SPEC-003-AC-06:** Given a cancelled plan's paid period ends and Nadia's active-client count exceeds the free-tier limit, when the period-end event arrives, then FEAT-23.SPEC-004 sets tier to Free and status to Lapsed instead of Active.

**FEAT-23.SPEC-003-AC-07:** Given a renewal charge fails, when the charge-failed event arrives with a specific reason, then it is handed to FEAT-23.SPEC-004 for the Charge failed status and to FEAT-23.SPEC-008 for the failure alert email.

**FEAT-23.SPEC-003-AC-08:** Given Nadia taps Subscribe while the capability is slow, when 10 seconds pass without a response, then the note "Still working -- this is taking longer than usual." appears and the rest of the screen stays usable.

**FEAT-23.SPEC-003-AC-09:** Given Nadia taps Subscribe while the capability is down, when the request cannot be sent, then Subscribe is disabled with "Billing is temporarily unavailable. Your current plan is unaffected -- try again in a few minutes."

**FEAT-23.SPEC-003-AC-10:** Given Nadia taps Subscribe and the capability rejects the charge, when the rejection is received, then she sees "This charge couldn't be completed: {reason}. Try again or use a different payment method inside your billing details.", her plan is unchanged, no Charge failed status is set, and no email is sent.

**FEAT-23.SPEC-003-AC-11:** Given Nadia taps Cancel while the capability is down, when the request cannot be sent, then Cancel is disabled with "Billing is temporarily unavailable. Try cancelling again in a few minutes." and her plan is not marked Cancelled and continues unaffected.

**FEAT-23.SPEC-003-AC-12:** Given Nadia has never subscribed before, when she has chosen a billing cycle and proceeds, then the disclosure notice appears with "Continue" and "Cancel," and no data leaves the product until she chooses "Continue."

**FEAT-23.SPEC-003-AC-13:** Given a charge-succeeded event was already applied, when the same event is delivered a second time, then nothing changes and no duplicate confirmation email fires.

**FEAT-23.SPEC-003-AC-14:** Given a period-end event arrives for an account already deleted through FEAT-24, when the event is processed, then it is discarded with no effect and no user feedback.

**FEAT-23.SPEC-003-AC-15:** Given the capability goes down after Nadia accepts the disclosure notice but before a charge outcome is confirmed, when she reopens her plan, then it shows her prior tier unchanged -- no half-upgraded state.

**FEAT-23.SPEC-003-AC-16:** Given Nadia accepts a downgrade offer and the capability rejects the stop-billing request, when the rejection is received, then she sees the reason inline, her plan stays Paid and Active, the offer remains available, and no email is sent.

**FEAT-23.SPEC-003-AC-17:** Given Nadia's status is Charge failed, when she taps Retry and the capability confirms the charge, then the retry-succeeded outcome is handed to FEAT-23.SPEC-004 and no email is sent for it.

**FEAT-23.SPEC-003-AC-18:** Given Nadia's status is Charge failed, when she taps Retry and the capability reports a new failure, then the new reason is handed to FEAT-23.SPEC-004, counted as one retry attempt, and no additional failed-charge alert email fires.

**FEAT-23.SPEC-003-AC-19:** Given Nadia's status is Charge failed and the capability is down, when she views the banner, then Retry is disabled with "Billing is temporarily unavailable. Your plan stays usable until {grace_window_end_date} -- try again in a few minutes." and the grace window is not extended.

**FEAT-23.SPEC-003-AC-20:** Given a plan lapses after its grace window, when FEAT-23.SPEC-004 hands a stop-billing request to this spec and the capability is unavailable, then the request is retried at platform parameter: `stop-billing-relay-retry-interval` until acknowledged and Nadia sees no additional state or message.

**FEAT-23.SPEC-003-AC-21:** Given Nadia abandons the capability's payment-entry experience without completing it, when she returns to the Plan & Billing Screen, then her plan is unchanged and no outcome event was handed to FEAT-23.SPEC-004.

**FEAT-23.SPEC-003-AC-22:** Given a Subscribe attempt fails for a plan that is Free with status Lapsed, when the failure is handed to FEAT-23.SPEC-004, then the plan stays Free and Lapsed with no grace window, and Nadia may try again.

### User Story 4 - Plan State Sync (Priority: P2)

Applies the subscription-billing capability's reported outcomes and period-end/renewal events to the Subscription Plan record's tier and status.

**Acceptance Scenarios:**

**FEAT-23.SPEC-004-AC-01:** Given Nadia's subscribe charge is confirmed by FEAT-23.SPEC-003, when this automation processes the event, then tier is set to Paid, billing_cycle is set to her selection, and status is set to Active.

**FEAT-23.SPEC-004-AC-02:** Given Nadia has accepted a downgrade offer and the capability confirms billing has stopped, when this automation processes the event with her live active_client_count within the free-tier limit, then tier is set to Free, status is set to Active, billing_cycle is cleared, and no charge was made.

**FEAT-23.SPEC-004-AC-03:** Given a Paid, Active plan's renewal charge fails, when this automation processes the event, then status is set to Charge failed with the specific reason, first-failure timestamp, and retry attempts used of 0 recorded, and tier and billing_cycle are left unchanged.

**FEAT-23.SPEC-004-AC-04:** Given an existing Paid plan's billing cycle renews successfully, when this automation processes the renewal event, then status remains Active with no confirmation email sent.

**FEAT-23.SPEC-004-AC-05:** Given a cancelled plan's paid period ends and Nadia's active-client count is within the free-tier limit, when this automation processes the period-end event, then tier is set to Free, status is set to Active, and billing_cycle is cleared.

**FEAT-23.SPEC-004-AC-06:** Given a cancelled plan's paid period ends and Nadia's active-client count exceeds the free-tier limit, when this automation processes the period-end event, then tier is set to Free, status is set to Lapsed, and billing_cycle is cleared.

**FEAT-23.SPEC-004-AC-07:** Given a plan has held Charge failed status for platform parameter: `subscription-charge-grace-window-days` with no successful retry, when this automation processes the grace-window-exhausted condition, then tier is set to Free, status is set to Lapsed, billing_cycle is cleared, and a stop-billing request is handed to FEAT-23.SPEC-003.

**FEAT-23.SPEC-004-AC-08:** Given Nadia's plan is Charge failed within the grace window, when she continues using her already-active client work, then nothing about that work is blocked.

**FEAT-23.SPEC-004-AC-09:** Given a plan just transitioned to Lapsed, when Nadia's existing clients are viewed, then no data is lost and every existing client portal remains reachable.

**FEAT-23.SPEC-004-AC-10:** Given a renewal-charge-failed event and a later successful-retry event both arrive for the same plan, when this automation reconciles them by event time, then the later success wins and status returns to Active.

**FEAT-23.SPEC-004-AC-11:** Given a retry succeeds at the exact instant the grace window would otherwise close, when this automation evaluates the boundary, then the retry is honored as a success and no lapse occurs.

**FEAT-23.SPEC-004-AC-12:** Given the sync write for a confirmed event fails, when this automation detects the failure, then the plan's prior state is preserved, no confirmation email fires, and the automation retries automatically.

**FEAT-23.SPEC-004-AC-13:** Given a renewal-succeeded and a charge-failed event for the same plan arrive at effectively the same time, when this automation reconciles them, then the event with the later reported timestamp is authoritative.

**FEAT-23.SPEC-004-AC-14:** Given two different freelancers' plans each receive an event at effectively the same time, when this automation processes both, then each plan is reconciled independently with no interference between them.

**FEAT-23.SPEC-004-AC-15:** Given Nadia's plan is Free (Active or Lapsed) and her subscribe charge fails, when this automation processes the event, then tier, status, and billing_cycle are unchanged, no grace window starts, no failed-charge alert email is sent, and the reason is passed to FEAT-23.SPEC-001 to show inline.

**FEAT-23.SPEC-004-AC-16:** Given Nadia's downgrade request is rejected by the billing capability, when this automation processes the event, then the plan stays Paid and Active, no email is sent, and the reason is passed to FEAT-23.SPEC-001.

**FEAT-23.SPEC-004-AC-17:** Given Nadia's status is Charge failed and her retry succeeds, when this automation processes the retry-succeeded event, then status is set to Active, the first-failure timestamp, reason, and retry attempts used are cleared, billing-period boundaries advance, no plan-change email is sent, and any failed-charge alert still pending delivery is cancelled.

**FEAT-23.SPEC-004-AC-18:** Given Nadia's status is Charge failed and a retry fails, when this automation processes the renewal-charge-failed event, then retry attempts used increase by 1, the reason is replaced, the original first-failure timestamp is kept, and no additional alert email is sent.

**FEAT-23.SPEC-004-AC-19:** Given Nadia's live active_client_count exceeds the free-tier limit at the moment a confirmed downgrade is processed, when this automation applies it, then tier is set to Free, status is set to Lapsed, billing_cycle is cleared, and the lapse email is sent instead of the downgrade email.

**FEAT-23.SPEC-004-AC-20:** Given the capability reports a period end for a Paid plan whose local status still shows Active because the cancellation was acknowledged but not recorded, when this automation processes the event, then it applies the period-end outcome exactly as for a Cancelled plan.

**FEAT-23.SPEC-004-AC-21:** Given this automation commits any tier or status change other than a routine renewal, when the write completes, then it notifies FEAT-23.SPEC-005 and reports the change to FEAT-13.SPEC-003 as an append-only trail event, and a trail-write retry never reverses the plan change.

**FEAT-23.SPEC-004-AC-22:** Given Nadia's account has been hard-deleted through FEAT-24.SPEC-004, when a billing event for her former plan arrives, then it is discarded with no effect and no feedback.

**FEAT-23.SPEC-004-AC-23:** Given Nadia's account is in the pending-delete hold phase, when a billing event arrives for her plan, then it is applied to the plan and no email is sent.

**FEAT-23.SPEC-004-AC-24:** Given Nadia's status is Charge failed and her retry attempts used equal platform parameter: `subscription-charge-retry-count` with the grace window still open, when this automation evaluates the plan, then status stays Charge failed, the Retry control is replaced by the retries-used message on FEAT-23.SPEC-001, and the lapse occurs only when the window ends.

### User Story 5 - Downgrade Eligibility Detection (Priority: P2)

Evaluates the active-client count against the free-tier threshold whenever the count or the plan's tier or status changes, raises the downgrade offer on a Paid, Active plan that sits at or below the threshold, and clears it the moment the plan stops being eligible.

**Acceptance Scenarios:**

**FEAT-23.SPEC-005-AC-01:** Given Nadia is on a Paid, Active plan with active-client count above the free-tier threshold, when she archives a client bringing her count to exactly platform parameter: `free-tier-active-client-limit`, then the downgrade-eligible flag is raised.

**FEAT-23.SPEC-005-AC-02:** Given Nadia's plan has the downgrade-eligible flag raised, when she reactivates a client bringing her count back above the threshold, then the flag is cleared.

**FEAT-23.SPEC-005-AC-03:** Given Nadia is on the Free tier, when her active-client count changes, then no downgrade-eligible flag is ever raised.

**FEAT-23.SPEC-005-AC-04:** Given Nadia's plan status is Cancelled -- ends at period end, when her active-client count drops below the threshold, then no downgrade-eligible flag is raised -- a plan already ending is not offered a further downgrade.

**FEAT-23.SPEC-005-AC-05:** Given Nadia's downgrade offer was cleared once already, when her plan is next eligible again (count at or below the threshold on a Paid, Active plan), then the offer is raised again fresh.

**FEAT-23.SPEC-005-AC-06:** Given Nadia views her Plan & Billing Screen while the downgrade-eligible flag is raised, when the screen loads, then the optional downgrade offer is shown with no forced plan change.

**FEAT-23.SPEC-005-AC-07:** Given Nadia archives one client and reactivates another in quick succession such that her net count stays above the threshold, when both changes settle, then no downgrade offer is ever shown.

**FEAT-23.SPEC-005-AC-08:** Given two of Nadia's client-count changes fire this automation at effectively the same time, when both evaluations complete, then the flag reflects the most recently committed count.

**FEAT-23.SPEC-005-AC-09:** Given this automation fails to complete an evaluation, when Nadia next views her plan, then she sees the offer's last known state with no error message, and the automation retries on the next trigger.

**FEAT-23.SPEC-005-AC-10:** Given Nadia's status is Charge failed (within the grace window) and her active-client count drops below the threshold, when this automation evaluates eligibility, then no downgrade-eligible flag is raised, since the offer exists only while status is Active.

**FEAT-23.SPEC-005-AC-11:** Given Nadia's downgrade offer is raised and she cancels instead, when FEAT-23.SPEC-006 records the cancellation, then this automation's cancellation trigger fires and the flag is cleared immediately, so the offer is not shown on the next view.

**FEAT-23.SPEC-005-AC-12:** Given Nadia's downgrade offer is raised and a renewal charge fails, when FEAT-23.SPEC-004 sets status to Charge failed, then the tier/status-change trigger fires and the flag is cleared; given a later retry returns status to Active with her count still at or below the threshold, then the offer is raised again.

**FEAT-23.SPEC-005-AC-13:** Given Nadia accepts the downgrade offer and FEAT-23.SPEC-004 sets tier to Free, when the tier-change trigger fires, then the flag is cleared and never raised while she is on the Free tier.

**FEAT-23.SPEC-005-AC-14:** Given the downgrade-eligible flag goes from cleared to raised, when it is raised, then one offer-raised event is reported to FEAT-13.SPEC-003 (Activity Entry Recording) and no further report is made while the flag stays raised.

### User Story 6 - Cancel Subscription (Priority: P2)

Cancels Nadia's paid plan by having the subscription-billing capability acknowledge that it will not renew, then recording the cancellation, with the plan remaining Paid and fully usable through the end of the current paid period.

**Acceptance Scenarios:**

**FEAT-23.SPEC-006-AC-01:** Given Nadia's plan is Paid and Active, when she confirms Cancel and the capability acknowledges, then status is set to Cancelled -- ends at period end, and tier remains Paid.

**FEAT-23.SPEC-006-AC-02:** Given Nadia's plan is now Cancelled -- ends at period end, when she views her plan, then she sees a plain explanation that it stays active through the current period's end.

**FEAT-23.SPEC-006-AC-03:** Given Nadia confirms Cancel, when the request is handed to FEAT-23.SPEC-003, then the plan's status is not changed until the capability's acknowledgment arrives.

**FEAT-23.SPEC-006-AC-04:** Given Nadia's plan is Charge failed within the grace window, when she confirms Cancel and the capability acknowledges, then the cancellation is recorded, the failure record is cleared, and the grace window no longer applies.

**FEAT-23.SPEC-006-AC-05:** Given Nadia's plan is already Cancelled -- ends at period end, when she attempts to cancel again, then the attempt is rejected as stale and the screen shows the already-cancelled state.

**FEAT-23.SPEC-006-AC-06:** Given Nadia's plan lapsed between her screen loading and her tapping Cancel, when the cancellation is attempted, then it is rejected as stale, and her screen refreshes to show the Lapsed state.

**FEAT-23.SPEC-006-AC-07:** Given Nadia taps Cancel in two open sessions at the same time, when both requests reach the automation, then only the first to commit records the cancellation and the second sees the already-cancelled state.

**FEAT-23.SPEC-006-AC-08:** Given a cancellation request is awaiting the capability's acknowledgment, when Nadia taps Cancel again, then the second tap is disabled while the first is in flight.

**FEAT-23.SPEC-006-AC-09:** Given Nadia's downgrade offer is showing when she cancels instead, when the cancellation is recorded, then FEAT-23.SPEC-005 is notified and the offer is cleared immediately, not shown on the next view.

**FEAT-23.SPEC-006-AC-10:** Given the billing capability is down when Nadia confirms Cancel, when no acknowledgment arrives, then the plan is not marked Cancelled, no background relay is queued, and she sees "Billing is temporarily unavailable. Try cancelling again in a few minutes."

**FEAT-23.SPEC-006-AC-11:** Given the capability acknowledged the cancellation but the record write fails, when the retries at platform parameter: `plan-record-write-retry-interval` are under way, then Nadia's screen shows "We're finishing your cancellation -- no action needed." and clears it once the write succeeds.

**FEAT-23.SPEC-006-AC-12:** Given the record write fails on all platform parameter: `plan-record-write-retry-count` attempts, when the last attempt fails, then retries stop, the plan status is unchanged, no email fires, Nadia's session shows the "billing partner has received your cancellation" notice with the period end date, and the capability's later period-end event is applied by FEAT-23.SPEC-004.

**FEAT-23.SPEC-006-AC-13:** Given a cancellation was acknowledged but never recorded and Nadia taps Cancel again, when the capability's repeat acknowledgment arrives and the record write succeeds, then the plan shows Cancelled once, with one confirmation email and one trail entry.

**FEAT-23.SPEC-006-AC-14:** Given a cancellation is recorded, when the write commits, then it is reported to FEAT-13.SPEC-003 as an append-only trail event with Nadia as actor, and a trail-write retry never reverses the cancellation.

**FEAT-23.SPEC-006-AC-15:** Given Nadia's account deletion completes while a cancellation acknowledgment or record retry is pending, when the pending work resolves, then it is discarded with no email and no trail entry.

### User Story 7 - Plan Limit & Access Authorization Rules (Priority: P2)

Governs the free-tier client cap that gates client capacity, who may view versus change the Subscription Plan, and the grace/retry handling before a failed charge lapses the plan.

**Acceptance Scenarios:**

**FEAT-23.SPEC-007-AC-01:** Given Nadia is on the free tier with active_client_count exactly at platform parameter: `free-tier-active-client-limit`, when she views her plan, then the usage display shows her at the limit and no error is present.

**FEAT-23.SPEC-007-AC-02:** Given Nadia is on the free tier at platform parameter: `free-tier-active-client-limit`, when she attempts to add one more active client (FEAT-01.SPEC-008), then the save is blocked with "You've reached your plan's active client limit. Upgrade to add more clients."

**FEAT-23.SPEC-007-AC-03:** Given Nadia's plan is Paid, when the plan record is read, then billing_cycle holds a value (monthly or yearly); given her plan is Free, then billing_cycle is unset.

**FEAT-23.SPEC-007-AC-04:** Given Nadia's status is Lapsed, when she attempts to reactivate an archived client that would push active_client_count beyond platform parameter: `free-tier-active-client-limit`, then the attempt is blocked with "Your plan has lapsed. Upgrade to add or reactivate clients beyond your free-tier limit."

**FEAT-23.SPEC-007-AC-05:** Given Nadia is viewing her own plan, when the screen loads, then she sees full tier, usage, billing cycle, and status detail.

**FEAT-23.SPEC-007-AC-06:** Given Dana has opened a support session on Nadia's account (FEAT-31.SPEC-002), when she views the Subscription & Account Data area, then she sees plan status only, with no Subscribe, downgrade, Cancel, or Retry control rendered.

**FEAT-23.SPEC-007-AC-07:** Given Dana has no open support session on any account, when she attempts to reach a freelancer's Subscription Plan, then no such navigation exists -- the plan is unreachable outside a session.

**FEAT-23.SPEC-007-AC-08:** Given Owen is signed in to his client portal, when he looks for any Subscription & Account Data navigation, then none is shown -- the client portal surface contains no such area.

**FEAT-23.SPEC-007-AC-09:** Given Priya is signed in to her client portal, when she looks for any Subscription & Account Data navigation, then none is shown, identically to Owen.

**FEAT-23.SPEC-007-AC-10:** Given Nadia's tier is Free, when she taps Subscribe and selects a billing cycle, then the request proceeds; given she is already Paid, then no Subscribe control is shown.

**FEAT-23.SPEC-007-AC-11:** Given Nadia's tier is Paid, status is Active, and a downgrade offer has been raised (FEAT-23.SPEC-005), when she views her plan, then Accept and Decline controls are both available; given no offer is currently raised, or status is Charge failed, Cancelled -- ends at period end, or Lapsed, then neither control is shown.

**FEAT-23.SPEC-007-AC-12:** Given Nadia's tier is Paid and status is Active, when she taps Cancel, then the cancellation proceeds (FEAT-23.SPEC-006); given her tier is already Free, then no Cancel control is shown.

**FEAT-23.SPEC-007-AC-13:** Given Nadia's status is Charge failed within platform parameter: `subscription-charge-grace-window-days` of the first failure and with retry attempts used below platform parameter: `subscription-charge-retry-count`, when she taps Retry, then the retry is submitted; given the grace window has elapsed, then no Retry control is shown and the plan has already transitioned to Lapsed.

**FEAT-23.SPEC-007-AC-14:** Given a charge fails at the exact instant that would be platform parameter: `subscription-charge-grace-window-days` after the first failure, when Nadia attempts a retry at that exact instant, then the retry is accepted (the boundary is inclusive).

**FEAT-23.SPEC-007-AC-15:** Given a charge fails and no retry succeeds within platform parameter: `subscription-charge-grace-window-days`, when the window closes, then the plan transitions to Lapsed and existing client work stays fully accessible.

**FEAT-23.SPEC-007-AC-16:** Given Nadia's status is Charge failed, when she continues working with an already-active client during the grace window, then nothing about her client work is blocked mid-session.

**FEAT-23.SPEC-007-AC-17:** Given Nadia's active-client count is exactly at platform parameter: `free-tier-active-client-limit` at the moment her plan transitions from Paid to Free at period end, when a client add is attempted immediately afterward, then it succeeds; one more beyond that is blocked.

**FEAT-23.SPEC-007-AC-18:** Given Nadia is mid-Subscribe on her own screen, when Dana opens a support session on the same account, then Dana's read-only view reflects the plan's state as of her session's own load and never interrupts Nadia's in-progress action.

**FEAT-23.SPEC-007-AC-19:** Given Dana has an open support session, when she attempts any action that would mutate the Subscription Plan through direct manipulation, then the attempt is blocked by the same session-wide read-only enforcement (FEAT-31.SPEC-003) applied to every other control.

**FEAT-23.SPEC-007-AC-20:** Given Nadia's status is Lapsed (tier Free), when she views her plan, then the Subscribe/Upgrade action is authorized and available, and a Subscribe request proceeds as it does on any Free plan.

**FEAT-23.SPEC-007-AC-21:** Given Nadia's status is Charge failed and her retry attempts used equal platform parameter: `subscription-charge-retry-count` with the grace window still open, when she views her plan, then no Retry control is shown, the banner reads "You've used all your retries. Your plan stays fully usable until {grace_window_end_date}, then it lapses. You can cancel your plan at any time.", and the plan lapses at the window's end rather than at the moment the count ran out.

**FEAT-23.SPEC-007-AC-22:** Given Nadia's plan lapses by either path, when the lapse is written, then tier is Free, status is Lapsed, and billing_cycle is unset in that same write.

**FEAT-23.SPEC-007-AC-23:** Given Nadia's plan is Free and her Subscribe attempt fails, when the failure is reported, then her status stays as it was (Active or Lapsed), no grace window starts, and she may attempt again with no retry limit.

**FEAT-23.SPEC-007-AC-24:** Given Nadia accepts a downgrade and the billing capability confirms billing has stopped, when FEAT-23.SPEC-004 applies it, then tier is Free, billing_cycle is unset, and no charge was made; given the capability rejects the request, then the plan stays Paid and Active.

### User Story 8 - Plan & Billing Notifications (Priority: P2)

Sends Nadia an email confirmation on every plan change (upgrade, downgrade, cancellation, lapse) and an alert when a subscription charge fails.

**Acceptance Scenarios:**

**FEAT-23.SPEC-008-AC-01:** Given Nadia's plan upgrades to Paid, when the upgrade is applied (FEAT-23.SPEC-004), then she receives an email "Your Clientroom plan is now Paid" naming her new client capacity.

**FEAT-23.SPEC-008-AC-02:** Given Nadia accepts a downgrade and it is applied (tier Free, status Active), when FEAT-23.SPEC-004 commits it, then she receives "Your Clientroom plan is now Free" stating the change is effective now, billing has stopped, and naming her new client limit.

**FEAT-23.SPEC-008-AC-03:** Given Nadia cancels her subscription, when the cancellation is recorded (FEAT-23.SPEC-006), then she receives "Your Clientroom plan cancellation is confirmed" stating the exact date her plan stays active through.

**FEAT-23.SPEC-008-AC-04:** Given Nadia's plan lapses, when the lapse is applied, then she receives "Your Clientroom plan has lapsed" stating that no data is lost and existing clients remain reachable.

**FEAT-23.SPEC-008-AC-05:** Given a Paid, Active plan's renewal charge fails, when the failure is recorded, then Nadia receives "We couldn't process your Clientroom subscription charge" naming the specific reason and the grace-window end date.

**FEAT-23.SPEC-008-AC-06:** Given Nadia's plan renews successfully, when the renewal event is applied, then no confirmation email is sent for it.

**FEAT-23.SPEC-008-AC-07:** Given Nadia's account is deleted before a pending confirmation email is delivered, when the deletion completes, then the pending email is cancelled and never sent.

**FEAT-23.SPEC-008-AC-08:** Given a failed-charge alert is pending delivery, when Nadia's retry succeeds before it sends, then the pending alert is cancelled and no confirmation email is sent for the return to Active status alone.

**FEAT-23.SPEC-008-AC-09:** Given Nadia's plan lapses and she immediately re-subscribes, when both events are recorded, then she receives both the lapse email and the upgrade confirmation email, each independently.

**FEAT-23.SPEC-008-AC-10:** Given a charge fails twice within the same still-open grace window, when the second failure is recorded, then no second failed-charge alert is sent.

**FEAT-23.SPEC-008-AC-11:** Given the lapse confirmation email fails delivery on every retry, when the final retry fails, then the lapse itself remains fully visible on the Plan & Billing Screen and a delivery warning is recorded per XBR-30.

**FEAT-23.SPEC-008-AC-12:** Given these are transactional emails, when Nadia looks in her Notification Preferences (FEAT-21.SPEC-002), then plan-change and failed-charge emails show as always-on with no toggle.

**FEAT-23.SPEC-008-AC-13:** Given a plan-change email is triggered during Nadia's own quiet hours preference window (set for other notification types), when it is due to send, then it sends immediately regardless, since these emails are exempt from quiet hours.

**FEAT-23.SPEC-008-AC-14:** Given a cancelled plan reaches its period end with Nadia's active-client count within the free-tier limit, when FEAT-23.SPEC-004 applies it, then she receives "Your Clientroom paid plan has ended" stating the {plan_end_date}, that she is now on the Free tier with room for {free_tier_client_limit} active clients, and that nothing was lost.

**FEAT-23.SPEC-008-AC-15:** Given a cancelled plan reaches its period end with the count over the free-tier limit, when FEAT-23.SPEC-004 applies it, then she receives only the lapse email, not the paid-plan-ended email.

**FEAT-23.SPEC-008-AC-16:** Given Nadia cancelled earlier and her plan later reaches its period end, when both events have been applied, then she has received two emails in total for the cancellation journey: the cancellation-confirmed email at cancellation and one period-end email (paid-plan-ended or lapse) at the end.

**FEAT-23.SPEC-008-AC-17:** Given Nadia's subscribe charge fails or her downgrade request is rejected, when the outcome is reported, then no email is sent and the reason appears inline on the Plan & Billing Screen only.

**FEAT-23.SPEC-008-AC-18:** Given a failed retry occurs within an open grace window, when FEAT-23.SPEC-004 records it, then no further failed-charge alert is sent, and a successful retry sends no plan-change email.

### Edge Cases

- **FEAT-23.SPEC-001 (Plan & Billing Screen):** Abandoning Subscribe before submission or leaving the billing capability's payment entry changes nothing and shows a plan-hasn't-changed notice; double taps on Subscribe are ignored. If payment confirmation is slow, the Confirming payment banner adds a still-confirming line after 10 seconds and the plan changes only when the outcome arrives. Source: `docs/blueprint/specifications/FEAT-23-subscription-plan-billing-management/FEAT-23.SPEC-001-plan-billing-screen.md` (section: Edge Cases)
- **FEAT-23.SPEC-002 (Free Plan Auto-Provisioning):** If provisioning fails after account creation it retries automatically and dependent screens show a setting-up-your-plan message until it succeeds. Near-simultaneous creations provision independent plans, and deletion during retries stops provisioning or removes a just-created plan. Source: `docs/blueprint/specifications/FEAT-23-subscription-plan-billing-management/FEAT-23.SPEC-002-free-plan-auto-provisioning.md` (section: Edge Cases)
- **FEAT-23.SPEC-003 (Subscription Billing Processing):** A duplicate or repeated charge-succeeded event changes nothing and sends no second confirmation email, and out-of-order events are applied by event time rather than arrival time. A period-end event for an already-deleted account is discarded with no feedback. Source: `docs/blueprint/specifications/FEAT-23-subscription-plan-billing-management/FEAT-23.SPEC-003-subscription-billing-processing.md` (section: Edge Cases)
- **FEAT-23.SPEC-004 (Plan State Sync):** The most recent event by event time wins when a failure and a retry arrive together, and a retry confirmed at or before the inclusive end of the grace window counts as a success. When retries are used up the plan stays Charge failed and usable until the window ends, and a concurrent client-add re-checks the plan at its own commit (reject-with-refresh). Source: `docs/blueprint/specifications/FEAT-23-subscription-plan-billing-management/FEAT-23.SPEC-004-plan-state-sync.md` (section: Edge Cases)
- **FEAT-23.SPEC-005 (Downgrade Eligibility Detection):** The downgrade threshold equals the free-tier client limit inclusively, and a net count back above the threshold clears the flag before the offer is seen. An accepted downgrade proceeds against the plan state at that moment, and a failed renewal charge clears the offer until a successful retry. Source: `docs/blueprint/specifications/FEAT-23-subscription-plan-billing-management/FEAT-23.SPEC-005-downgrade-eligibility-detection.md` (section: Edge Cases)
- **FEAT-23.SPEC-006 (Cancel Subscription):** Duplicate or concurrent Cancel requests record the cancellation once and reject the second as stale-state, and a plan that lapses before or during the request is rejected or treated as moot with the screen refreshed to Lapsed. Source: `docs/blueprint/specifications/FEAT-23-subscription-plan-billing-management/FEAT-23.SPEC-006-cancel-subscription.md` (section: Edge Cases)
- **FEAT-23.SPEC-007 (Plan Limit & Access Authorization Rules):** The free-tier cap is inclusive of the limit itself (one more client triggers the block), and a retry before the end moment of the charge grace window is accepted while a later one is rejected. The grace-window rule and downgrade eligibility evaluate independently, and an operator's read-only session never intercepts the freelancer's own flow. Source: `docs/blueprint/specifications/FEAT-23-subscription-plan-billing-management/FEAT-23.SPEC-007-plan-limit-access-authorization-rules.md` (section: Edge Cases)
- **FEAT-23.SPEC-008 (Plan & Billing Notifications):** Pending confirmation emails are cancelled silently if the account is deleted, and failed subscribes or rejected downgrades send no email because the outcome shows inline. A cancelled plan reaching period end sends one end-of-plan email on top of the earlier cancellation confirmation, and a pending failed-charge alert is cancelled if the retry succeeds first. Source: `docs/blueprint/specifications/FEAT-23-subscription-plan-billing-management/FEAT-23.SPEC-008-plan-billing-notifications.md` (section: Edge Cases)

## Requirements (mandatory)

### Functional Requirements

- **FR-001**: The system MUST implement **FEAT-23.SPEC-001** (Plan & Billing Screen) as specified: Nadia views her current tier, usage against its client limit, and billing cycle, and initiates upgrade, responds to a downgrade offer, or cancels, from one surface. Full spec: `docs/blueprint/specifications/FEAT-23-subscription-plan-billing-management/FEAT-23.SPEC-001-plan-billing-screen.md`
- **FR-002**: The system MUST implement **FEAT-23.SPEC-002** (Free Plan Auto-Provisioning) as specified: Creates the Subscription Plan record on the free tier automatically the instant a new Freelancer Account is created, so there is never an explicit "no plan" state. Full spec: `docs/blueprint/specifications/FEAT-23-subscription-plan-billing-management/FEAT-23.SPEC-002-free-plan-auto-provisioning.md`
- **FR-003**: The system MUST implement **FEAT-23.SPEC-003** (Subscription Billing Processing) as specified: Submits Nadia's own subscribe, stop-billing (downgrade), retry, and cancellation requests to the subscription-billing capability and receives back charge outcomes, renewal and period-end events, and failure reasons. Full spec: `docs/blueprint/specifications/FEAT-23-subscription-plan-billing-management/FEAT-23.SPEC-003-subscription-billing-processing.md`
- **FR-004**: The system MUST implement **FEAT-23.SPEC-004** (Plan State Sync) as specified: Applies the subscription-billing capability's reported outcomes and period-end/renewal events to the Subscription Plan record's tier and status. Full spec: `docs/blueprint/specifications/FEAT-23-subscription-plan-billing-management/FEAT-23.SPEC-004-plan-state-sync.md`
- **FR-005**: The system MUST implement **FEAT-23.SPEC-005** (Downgrade Eligibility Detection) as specified: Evaluates the active-client count against the free-tier threshold whenever the count or the plan's tier or status changes, raises the downgrade offer on a Paid, Active plan that sits at or below the threshold, and clears it the moment the plan stops being eligible. Full spec: `docs/blueprint/specifications/FEAT-23-subscription-plan-billing-management/FEAT-23.SPEC-005-downgrade-eligibility-detection.md`
- **FR-006**: The system MUST implement **FEAT-23.SPEC-006** (Cancel Subscription) as specified: Cancels Nadia's paid plan by having the subscription-billing capability acknowledge that it will not renew, then recording the cancellation, with the plan remaining Paid and fully usable through the end of the current paid period. Full spec: `docs/blueprint/specifications/FEAT-23-subscription-plan-billing-management/FEAT-23.SPEC-006-cancel-subscription.md`
- **FR-007**: The system MUST implement **FEAT-23.SPEC-007** (Plan Limit & Access Authorization Rules) as specified: Governs the free-tier client cap that gates client capacity, who may view versus change the Subscription Plan, and the grace/retry handling before a failed charge lapses the plan. Full spec: `docs/blueprint/specifications/FEAT-23-subscription-plan-billing-management/FEAT-23.SPEC-007-plan-limit-access-authorization-rules.md`
- **FR-008**: The system MUST implement **FEAT-23.SPEC-008** (Plan & Billing Notifications) as specified: Sends Nadia an email confirmation on every plan change (upgrade, downgrade, cancellation, lapse) and an alert when a subscription charge fails. Full spec: `docs/blueprint/specifications/FEAT-23-subscription-plan-billing-management/FEAT-23.SPEC-008-plan-billing-notifications.md`

### Key Entities

- Subscription Plan (create, update)

## Success Criteria (mandatory)

### Measurable Outcomes

- **SC-001**: At least 40% of freelancers who try to add a client beyond the free limit are on a paid plan within 14 days (metric: Free-to-Paid Conversion). Source: `docs/blueprint/features/success-metrics.md`
- **SC-002**: Plan views, upgrades, downgrade offers, failed subscription charges, cancellations and lapses are each observable as distinct signals (plan_viewed, plan_upgraded, plan_downgrade_offered, subscription_charge_failed, subscription_cancelled, plan_lapsed). Source: `docs/blueprint/features/product-features.md`

## Assumptions

- **ASMP-12**: A permanent free tier for one or two active clients is assumed to convert enough freelancers to paid plans. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-17**: No platform fee or cut of payments; revenue comes solely from the freelancer's subscription. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-31**: Subscription-billing capability for the freelancer's own plan is a required dependency, separate from client-side payment processing. Full register: `docs/blueprint/features/assumptions-constraints.md`

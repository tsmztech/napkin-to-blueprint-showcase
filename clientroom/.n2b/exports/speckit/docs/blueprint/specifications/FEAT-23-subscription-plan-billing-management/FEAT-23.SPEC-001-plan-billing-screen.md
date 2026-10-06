---
document_type: spec
spec_type: screen
spec_id: FEAT-23.SPEC-001
spec_name: Plan & Billing Screen
spec_slug: plan-billing-screen
parent_feature: FEAT-23
parent_feature_name: Subscription Plan & Billing Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 36
---

# Screen Spec: Plan & Billing Screen

## Overview

**Name:** Plan & Billing Screen
**ID:** FEAT-23.SPEC-001
**Type:** Screen
**Purpose:** Nadia views her current tier, usage against its client limit, and billing cycle, and initiates upgrade, responds to a downgrade offer, or cancels, from one surface.
**Parent Feature:** FEAT-23 -- Subscription Plan & Billing Management

## Scope and Non-Goals

**In Scope:**
- Displaying the current plan tier, usage against the client limit, and billing cycle
- Initiating a Subscribe (upgrade) action: billing-cycle selection, the one-time disclosure notice, and the hand-off to and return from the billing capability's payment-entry experience
- Surfacing and responding to an optional downgrade offer (Paid to Free, effective immediately)
- Initiating a Cancel action, with a plain end-of-period explanation in a confirmation dialog
- Showing the Charge failed state inline with its specific reason and an immediate Retry action while retries remain
- Showing a failed Subscribe attempt or rejected downgrade inline without changing the plan

**Non-Goals:**
- Entering or storing payment details -- excluded per BRIEF.md's Constraints and scope-boundaries.md (SC-10): payment details are captured entirely inside the subscription-billing capability's own experience, reached through FEAT-23.SPEC-003, never on this screen.
- The mechanics of submitting a charge, receiving outcomes, or applying a confirmed tier change -- owned by FEAT-23.SPEC-003 (Subscription Billing Processing) and FEAT-23.SPEC-004 (Plan State Sync); this screen only initiates the request and displays the result.
- Choosing which invoicing or payment-collection method a client uses -- entirely separate per product-features.md's Interactions field: this screen governs only Nadia's own subscription to Clientroom, never client-facing Invoicing & Payments (FEAT-09/FEAT-10).
- Changing an already-Paid plan's billing cycle without a tier change (e.g., switching monthly to yearly while staying Paid) -- product-features.md's Key Capabilities name only Upgrade, Downgrade offer, and Cancel; a same-tier cycle switch is not a capability the product definition establishes, so it is not offered here.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-21.SPEC-001 (Account Profile) | Nadia taps the Subscription & Account Data area of Settings | None -- screen loads her current plan |
| FEAT-01.SPEC-008 (Active Client Limit Enforcement) | The "Upgrade" action on the blocked add-client message | None -- screen loads with the Subscribe action already prominent since she is at her limit |
| FEAT-16.SPEC-001 (Storage Usage Summary) | The "Review your plan" link on the storage-limit warning | None -- screen loads her current plan; the storage allowance itself is read from her plan tier by FEAT-16, not carried as a parameter here |
| Email CTA (FEAT-23.SPEC-008) | Nadia taps "View plan," "Upgrade," or "Retry now" in a plan-change or failed-charge email | None beyond routing to this screen; a Retry-now link opens directly onto the Charge failed state's Retry action (or, when retries are used up, onto the retries-used message) |
| Return from the billing capability's payment-entry experience | Nadia completes, cancels, or closes the capability's payment-entry experience and is returned by it | The outcome the capability reports (completed, failed, or abandoned); see Interactions |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Subscribe, accept/decline downgrade offer, Cancel, Retry a failed charge -- all actions on her own plan | -- |
| Dana (Support Operator) | Not applicable to this screen -- her view of plan status is delivered entirely through FEAT-31.SPEC-002 (Operator Support Session Console), a separate read-only mirror; she never opens this screen directly | No | She has no navigation path that reaches this screen; attempting to reach it directly (outside a support session) is treated as any other out-of-scope navigation for her role |
| Owen (Client Primary Contact) | No | No | No settings navigation from the client portal reaches this screen; the client portal product surface contains no Subscription & Account Data area (Access Matrix: None) |
| Priya (Client Reviewer Contact) | No | No | Same as Owen -- no client-portal surface exists for this screen |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in, the user lands on their own dashboard, not this screen, unless they navigated here from a link that re-resolves after authentication |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- any in-progress action (e.g., a Subscribe request not yet confirmed) is not assumed to have completed; Nadia re-checks the plan's actual state after signing back in |

## Layout and Content

**Header:** Screen title "Plan & Billing" with a back arrow (returns to Account Profile, FEAT-21.SPEC-001).

**Body, organized top to bottom:**

- **Load-error banner (conditional):** Appears in place of the Current Plan card only when the initial plan load fails. Text "Couldn't load your plan. Try again." with a "Retry" button.
- **Current Plan card:** Shows the tier label ("Free" or "Paid"; a Lapsed plan shows "Free -- Lapsed"), and when Paid, the billing cycle (monthly or yearly) and the next renewal or period-end date. Contains a "How billing works" link at the bottom of the card. Display-only apart from the link.
- **Payment confirmation banner (conditional):** Shown while the screen waits for the capability's outcome after Nadia returns from the payment-entry experience: "Confirming your payment..." with a progress indicator. After 10 seconds it adds "Still confirming -- this is taking longer than usual."
- **Attempt-failed banner (conditional, transient):** Shown after a Subscribe attempt fails or a downgrade request is rejected: "This charge couldn't be completed: {reason}." (Subscribe) or "We couldn't end your billing: {reason}. Your plan is unchanged." (downgrade), with a "Try again" button and a dismiss control. It reflects an attempt, not plan state: it is not stored, and it disappears on reload.
- **Charge failed banner (Error state, conditional):** Appears only when status is Charge failed (a failed renewal charge on a Paid plan). Shows the specific failure reason, and either a "Retry" button (while the grace window is open and retry attempts used are below the retry limit) or, when retries are used up, the message "You've used all your retries. Your plan stays fully usable until {grace_window_end_date}, then it lapses. You can cancel your plan at any time." with no Retry button. Positioned directly below the Current Plan card so it is the first thing Nadia sees if it applies.
- **Downgrade offer banner (conditional):** Appears only when the downgrade-eligible flag is raised (FEAT-23.SPEC-005), which happens only while tier is Paid and status is Active. Shows a plain statement that her client count now fits the free tier, with "Downgrade" and "Keep my Paid plan" actions, both optional. Positioned below the Usage meter.
- **Usage meter:** Shows Nadia's active-client count against her plan's client limit: when Free (Active), the count against the free-tier limit; when Paid (any status), "unlimited"; when Lapsed, the count against the free-tier limit with, if the count exceeds it, the note "Over your free-tier limit -- existing clients stay reachable; adding more is on hold until you upgrade." Display-only.
- **Primary action area:** Shows "Subscribe" when tier is Free and status is Active; shows "Upgrade" when status is Lapsed (the same action as Subscribe); shows "Cancel" when tier is Paid and status is Active or Charge failed; shows nothing (only the plain explanation text) when status is Cancelled -- ends at period end.
- **Billing-cycle selection panel (opens from Subscribe/Upgrade):** Title "Choose your billing cycle." Two mutually exclusive options: "Monthly -- platform parameter: `paid-plan-monthly-price` per month" and "Yearly -- platform parameter: `paid-plan-yearly-price` per year". Neither is preselected. A "Continue" button (disabled until an option is selected) and a "Back" button. If Nadia taps Continue's disabled area or attempts to proceed without a choice, the message "Choose a monthly or yearly billing cycle to continue." appears under the options.
- **Disclosure notice (dialog, shown the first time Nadia proceeds past cycle selection, and on request via "How billing works"):** Text per FEAT-23.SPEC-003 (Consent and Disclosure): "To subscribe, your account reference and the plan you're choosing are shared with our billing partner. Your payment details are entered directly with them and never stored by Clientroom." Buttons "Continue" and "Cancel" when opened as part of Subscribe; a single "Close" button when opened from the "How billing works" link.
- **Downgrade confirmation dialog:** Opens when Nadia taps "Downgrade." Title "Downgrade to Free?" Text: "This takes effect right away and stops your billing -- you won't be charged again, and the rest of your current paid period isn't refunded. You'll have room for {free_tier_client_limit} active clients; your existing clients and portals stay reachable." Buttons "Downgrade now" and "Not now."
- **Cancel confirmation dialog:** Opens when Nadia taps Cancel. Title "Cancel your paid plan?" Text: "Your plan stays active through {period_end_date}. After that, it moves to the free tier or lapses depending on your client count at that time. You can subscribe again at any time." Buttons "Cancel plan" and "Keep my plan."
- **Lapsed explanation (conditional):** Appears only when status is Lapsed. Plain text stating existing clients and portals remain reachable, and an "Upgrade" action to recover capacity.
- **Cancelled explanation (conditional):** Appears only when status is Cancelled -- ends at period end. Plain text stating the exact date the plan remains active through. While a cancellation is being finished (FEAT-23.SPEC-006), this area instead reads "We're finishing your cancellation -- no action needed."

**Footer:** None.

### Responsive Behavior

- **Compact size class:** All cards, banners, and panels stack in a single column, full width, in the order described above; dialogs fill the width with a consistent margin and their two buttons stack vertically.
- **Medium size class and above:** The same single-column stacking is retained, capped at a consistent platform-wide content width and horizontally centered; dialogs are centered at a fixed width; no structural reordering, since this screen has no side-by-side content that benefits from extra width.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-21.SPEC-001 (Account Profile) | Screen closes | Standard navigation transition |
| "How billing works" link | Tap | Opens the disclosure notice inline with a single "Close" button | Notice shown over the screen | Notice text; "Close" dismisses it and returns focus to the link |
| Load-error banner -- "Retry" | Tap | Reloads the plan data | Banner shows a progress state | Success: banner is replaced by the Current Plan card and the state that applies. Failure: the banner stays with the same message |
| Subscribe / Upgrade button | Tap | Opens the billing-cycle selection panel | Panel appears; Subscribe/Upgrade is not resubmittable while the flow is open | Panel with Monthly and Yearly options and their prices |
| Billing-cycle option (Monthly or Yearly) | Tap or select by keyboard | Selects that option (one at a time) | Selected option is marked; "Continue" enables | The selected option is visibly and programmatically marked (not by colour alone) |
| Billing-cycle panel -- "Continue" | Tap (enabled only after a selection) | First time ever: opens the disclosure notice. Every later time: proceeds directly to the hand-off | Panel closes or notice opens | See the disclosure and hand-off rows |
| Billing-cycle panel -- "Back" | Tap | Closes the panel; nothing is sent | Panel closes | Screen unchanged; Subscribe available |
| Disclosure notice -- "Continue" (Subscribe flow) | Tap | Proceeds to the hand-off to the billing capability | Notice closes; the screen shows "Taking you to our billing partner..." | Nadia leaves this screen for the capability's payment-entry experience; no data left the product before this tap |
| Disclosure notice -- "Cancel" (Subscribe flow) | Tap | Returns to this screen with nothing sent | Notice and panel close | Screen unchanged; the notice will be shown again next time |
| Disclosure notice -- "Close" (from "How billing works") | Tap | Dismisses the notice | Notice closes | Focus returns to the "How billing works" link |
| Return from payment entry -- completed | The capability returns Nadia to this screen after she completes payment entry | Shows the Payment confirmation banner until FEAT-23.SPEC-003 reports the outcome | Banner "Confirming your payment..." | Success outcome: banner clears; Current Plan card updates to Paid with unlimited capacity; upgrade email arrives separately (FEAT-23.SPEC-008). Failure outcome: the Attempt-failed banner appears with the specific reason and "Try again"; the plan is unchanged |
| Return from payment entry -- abandoned | Nadia closes, backs out of, or cancels the capability's payment-entry experience, and is returned (or later reopens this screen) | Nothing is submitted and no outcome arrives; the plan is not changed | None | If returned by the capability: note "You left before completing payment -- your plan hasn't changed." with Subscribe still available. If she simply closes the tab, the next view shows her plan as it was |
| Attempt-failed banner -- "Try again" | Tap | Reopens the billing-cycle selection panel (Subscribe) or the downgrade confirmation dialog (downgrade) | Banner is dismissed; panel or dialog opens | No limit on attempts; the plan and status are unaffected by failed attempts |
| Attempt-failed banner -- dismiss | Tap | Hides the banner | Banner hidden | None |
| Downgrade offer -- "Downgrade" | Tap | Opens the downgrade confirmation dialog | Dialog appears | Dialog text stating immediate effect, billing stops, no refund of the current period |
| Downgrade confirmation -- "Downgrade now" | Tap | Submits the stop-billing (downgrade) request through FEAT-23.SPEC-003 | Dialog closes; the offer banner shows a progress state | Success: Current Plan card updates to Free (Active) with the free-tier client limit and billing cycle no longer shown; downgrade email arrives separately. If her client count rose above the limit in the meantime, it shows Free -- Lapsed instead. Failure: the Attempt-failed banner shows the specific rejection reason with "Try again"; the plan stays Paid and Active and the offer remains |
| Downgrade confirmation -- "Not now" | Tap | Closes the dialog | Dialog closes | No data change; offer banner still showing |
| Downgrade offer -- "Keep my Paid plan" | Tap | Dismisses the offer for this view only -- no data change | Offer banner is hidden for the remainder of this session | No confirmation needed; the offer may resurface on a later view if eligibility is re-evaluated as raised (FEAT-23.SPEC-005) |
| Cancel button | Tap | Opens the Cancel confirmation dialog | Dialog appears | Dialog text stating the end-of-period effect |
| Cancel dialog -- "Cancel plan" | Tap | Submits the cancellation through FEAT-23.SPEC-006, which waits for the billing capability's acknowledgment before recording it | Dialog closes; Cancel button shows a progress state | Success: screen shows the Cancelled explanation with the exact period-end date; confirmation email arrives separately. Capability slow or down: FEAT-23.SPEC-003's messages ("Still working -- this is taking longer than usual." / "Billing is temporarily unavailable. Try cancelling again in a few minutes."), plan unchanged. Acknowledged but still being recorded: "We're finishing your cancellation -- no action needed." Stale state: screen refreshes to the plan's current actual state with "Your plan status has changed -- here's the latest." |
| Cancel dialog -- "Keep my plan" | Tap | Closes the dialog | Dialog closes | No data change; nothing sent |
| Charge failed banner -- "Retry" | Tap (visible only while the grace window is open and retry attempts used are below the retry limit) | Resubmits the failed renewal charge through FEAT-23.SPEC-003 | Retry button shows a progress state | Success: Charge failed banner clears and Current Plan card confirms Active/Paid (no email). Failure: banner updates with the new specific reason; if retry attempts are now used up, the Retry button is replaced by the retries-used message; otherwise it remains available |
| Lapsed explanation -- "Upgrade" | Tap | Same as the Subscribe button -- opens the billing-cycle selection panel and continues through FEAT-23.SPEC-003 | Same as Subscribe | Same as Subscribe |

### Accessibility Notes

- **Focus order:** Back arrow -> load-error banner and its Retry (when present) -> Current Plan card (read order) including the "How billing works" link -> payment confirmation or attempt-failed banner (when present) -> Charge failed banner and its Retry action or retries-used message (when present) -> Usage meter -> Downgrade offer banner and its two actions (when present) -> primary action (Subscribe, Cancel, or Upgrade) -> conditional explanation text. Dialogs and the billing-cycle panel trap focus while open and return it to the control that opened them.
- **Dynamic announcements:** The Charge failed banner, the retries-used message, the attempt-failed banner, the payment confirmation banner, the downgrade offer banner, and the Cancelled/Lapsed explanation are announced to assistive technology when they first appear on load or after an action completes, since each represents a state change relevant to Nadia's billing standing.
- **Action feedback:** Success and failure outcomes for Subscribe, Downgrade, Cancel, and Retry are announced as they occur; on failure, focus moves to the banner containing the specific reason.
- **Keyboard alternatives:** Every action on this screen (Subscribe, billing-cycle options and buttons, disclosure buttons, Downgrade and its dialog, Keep my Paid plan, Cancel and its dialog, Retry, Try again, Upgrade, How billing works, load-error Retry) is reachable and operable by keyboard; there are no pointer-only gestures. Billing-cycle selection and plan states are never distinguished by colour alone.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Current Plan card and Usage meter show a loading placeholder; no actions are available yet | Screen first opens | Plan data finishes loading |
| Free (default) | Current Plan card shows "Free"; Usage meter shows count against platform parameter: `free-tier-active-client-limit`; Subscribe is the primary action | Plan data loads with tier=Free, status=Active | Nadia subscribes successfully |
| Paid (Active) | Current Plan card shows "Paid" with billing cycle and renewal date; Usage meter shows "unlimited"; Cancel is the primary action | Plan data loads with tier=Paid, status=Active | A cancellation, a failed renewal charge, an accepted downgrade, or (rare) a direct status change occurs |
| Downgrade offered | Same as Paid (Active), plus the downgrade offer banner | The downgrade-eligible flag is raised (FEAT-23.SPEC-005) while tier is Paid and status is Active | Nadia accepts, dismisses (for this session), or the flag clears because her count rose above the threshold or the plan's status changed (e.g., Charge failed or Cancelled) |
| Subscribe in progress | Billing-cycle panel, disclosure notice, or the hand-off message is showing; other Subscribe controls are inactive | Nadia taps Subscribe/Upgrade | She goes Back or Cancel (returns to the prior state), or leaves for the payment-entry experience |
| Confirming payment | Payment confirmation banner with progress indicator; Subscribe and Upgrade are inactive | Nadia is returned from the payment-entry experience after completing it | The outcome arrives: Paid (Active) on success, or the Attempt-failed state on failure |
| Attempt failed (transient) | Attempt-failed banner with the specific reason and "Try again"; the plan card shows the unchanged plan | A Subscribe attempt fails or a downgrade request is rejected | Nadia taps Try again, dismisses the banner, or reloads the screen |
| Charge failed (Error) | Charge failed banner shows the specific reason and Retry; the rest of the screen (Current Plan card, client access) remains fully shown and usable; Cancel remains available | A renewal charge on a Paid plan fails (status Charge failed) | A successful retry (returns to Paid/Active), a cancellation, or the grace window ends (moves to Lapsed) |
| Charge failed -- retries used | Same as Charge failed, but the Retry button is replaced by "You've used all your retries. Your plan stays fully usable until {grace_window_end_date}, then it lapses. You can cancel your plan at any time." | Charge failed while retry attempts used equal platform parameter: `subscription-charge-retry-count` and the grace window is still open | The grace window ends (Lapsed) or Nadia cancels |
| Cancelled (pending period end) | Current Plan card still shows "Paid"; the Cancelled explanation text shows the exact period-end date; no Subscribe/Cancel primary action is offered (already cancelled) | Nadia's cancellation is acknowledged and recorded (FEAT-23.SPEC-006) | The period-end event arrives and the plan transitions to Free or Lapsed (FEAT-23.SPEC-004) |
| Lapsed | Current Plan card shows "Free -- Lapsed"; Usage meter shows the count against the free-tier limit (with the over-limit note if applicable); the Lapsed explanation states existing clients remain reachable; Upgrade is the primary action | Grace window exhausted, or period end reached with active-client count over the free-tier limit | Nadia upgrades successfully |
| Error (load failure) | Load-error banner "Couldn't load your plan. Try again." with a Retry button in place of the Current Plan card | The initial plan data load fails | Nadia taps Retry and the load succeeds |
| Offline/Degraded | Banner "You're offline -- plan and billing changes need a connection. You can still view your last-loaded plan details." at the top; Subscribe, billing-cycle Continue, Downgrade, Cancel, and Retry controls are disabled; previously loaded plan data remains visible | Connectivity lost while the screen is open, or the screen is opened without connectivity after a prior successful load | Connectivity is restored -- controls re-enable and the plan is re-fetched to confirm it reflects the latest state |

## Validation Rules

Validation and authorization for every action on this screen are governed by FEAT-23.SPEC-007 (Plan Limit & Access Authorization Rules). See that spec for the free-tier threshold, the grace/retry window and retry limit, and the exact conditions under which Subscribe, downgrade acceptance, Cancel, and Retry are available. This screen applies those conditions by showing or hiding each action per the current plan state. The one input this screen validates itself is the billing cycle: exactly one of Monthly or Yearly must be selected before "Continue" proceeds, otherwise "Choose a monthly or yearly billing cycle to continue."

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-21.SPEC-001 (Account Profile) | FEAT-21 |
| Subscribe / Upgrade action, billing-cycle selection, and disclosure notice | Stay on this screen until the disclosure notice's "Continue" (or, on later subscriptions, the panel's "Continue") | -- |
| Disclosure notice "Continue" / panel "Continue" (after the first time) | The subscription-billing capability's own payment-entry experience (through FEAT-23.SPEC-003); the capability returns Nadia to this screen with the outcome, or she abandons it and the plan is unchanged | Subscription-billing capability (external) |
| Successful Cancel | Stays on this screen, now showing the Cancelled explanation | -- |
| "How billing works" link | Opens the disclosure notice inline; no navigation away from this screen | -- |

## Data Model

**Creates:** None.
**Reads:** Subscription Plan -- tier, billing_cycle, active_client_count, status, downgrade-eligible flag, last failure reason and retry attempts used (when Charge failed), grace-window end date (when Charge failed), period-end date (when Cancelled or Paid).
**Updates:** None directly -- all writes happen through FEAT-23.SPEC-003 (billing submission), FEAT-23.SPEC-004 (state sync), FEAT-23.SPEC-005 (flag), and FEAT-23.SPEC-006 (cancellation); this screen only initiates those requests.
**Deletes:** None.

## Business Rules

- The downgrade offer is always optional -- accepting or declining is entirely Nadia's choice; the screen never applies a downgrade automatically (Shared UI Pattern, Feature Breakdown Brief; FEAT-23.SPEC-005). The offer exists only while tier is Paid and status is Active (FEAT-23.SPEC-007); it is not shown for Charge failed, Cancelled, or Lapsed. A downgrade is Paid to Free, effective immediately, with billing stopped and no refund of the remaining period.
- A Charge failed status never removes access to already-active client work shown or reachable from elsewhere in the product during the same session (Shared UI Pattern, Feature Breakdown Brief; FEAT-23.SPEC-007).
- Only a failed renewal charge on a Paid plan produces the Charge failed state. A failed Subscribe attempt or a rejected downgrade is an inline, transient failure that leaves the plan exactly as it was (FEAT-23.SPEC-004).
- XBR-23: while the plan is Lapsed (always tier Free), the primary action offered here is Upgrade, since adding or reactivating clients beyond the free-tier limit requires an active paid plan again; Subscribe is authorized on Lapsed by FEAT-23.SPEC-007.
- All actions on this screen are governed by FEAT-23.SPEC-007's Authorization Rules -- only Nadia can act, and only under the conditions that spec defines for each action.

## Edge Cases

- **Nadia navigates away mid-Subscribe with the billing-cycle panel or disclosure notice open but not yet confirmed** -- No request has been submitted yet, so nothing changes; returning to this screen later shows her prior tier unchanged and Subscribe available to try again.
- **Nadia taps Subscribe twice rapidly** -- The second tap is ignored while the first request is in progress (button in a progress state, or the panel already open).
- **Nadia abandons the billing capability's payment-entry experience** -- Nothing is submitted, the plan is unchanged, and (if the capability returns her here) she sees "You left before completing payment -- your plan hasn't changed." with Subscribe available. No Charge failed state and no email result.
- **Nadia completes payment entry but the outcome takes long to arrive** -- The Confirming payment banner stays, adding "Still confirming -- this is taking longer than usual." after 10 seconds; the plan changes only when the outcome arrives, and if she leaves and returns, the screen shows whatever the plan's actual state is by then.
- **Network failure while submitting a Subscribe, downgrade, or Cancel request** -- Banner: "Couldn't complete this action. Check your connection and try again." with a Retry option; the plan's state is unchanged since no confirmation was received.
- **Nadia's plan state changed since this screen last loaded (e.g., another session already cancelled, or a charge failed) while she attempts an action here** -- The action is rejected as stale (reject-with-refresh, per the dependency map's Contention note for Subscription Plan) and the screen refreshes to show "Your plan status has changed -- here's the latest," reflecting the actual current state.
- **Concurrent-edit conflict: Nadia has this screen open in two tabs and cancels in one while attempting to accept a downgrade offer in the other** -- The second action (downgrade accept) is rejected as stale once the cancellation has committed, since a Cancelled -- ends at period end plan is no longer eligible for a downgrade offer; the second tab refreshes to show the Cancelled state. Resolution: reject-with-refresh, consistent with the dependency map's Contention note that billing-capability status reports (and the most recently committed local action) are authoritative over a concurrent viewer's stale load.
- **Nadia loses connectivity mid-session with a Charge failed banner already showing** -- The banner and its reason remain visible (last-loaded data), but Retry is disabled with the Offline/Degraded banner until connectivity returns.
- **The last retry attempt fails while the grace window is still open** -- The Retry button is replaced by the retries-used message with the window's end date; Cancel remains available; the plan lapses at the window's end, not before.
- **A renewal charge fails while the downgrade offer is showing** -- On next load or refresh the offer is gone (the offer needs status Active) and the Charge failed banner is shown; after a successful retry the offer may reappear if the count is still within the free-tier limit.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-23.SPEC-003 (Subscription Billing Processing) | Triggers (outbound) | Subscribe, downgrade-accept, Retry, and Cancel actions submit requests through this spec; its disclosure and degradation messages surface here |
| FEAT-23.SPEC-004 (Plan State Sync) | References (inbound) | Supplies the tier/status this screen displays after every outcome, and the reasons for failed subscribe attempts and rejected downgrades |
| FEAT-23.SPEC-005 (Downgrade Eligibility Detection) | References (inbound) | Supplies the downgrade-eligible flag this screen surfaces as the offer |
| FEAT-23.SPEC-006 (Cancel Subscription) | Triggers (outbound) | The confirmed Cancel action initiates this automation |
| FEAT-23.SPEC-007 (Plan Limit & Access Authorization Rules) | References (inbound) | Governs every action's availability and conditions on this screen |
| FEAT-23.SPEC-008 (Plan & Billing Notifications) | References (outbound) | Every action this screen completes triggers the corresponding confirmation or alert email |
| FEAT-21.SPEC-001 (Account Profile) | Navigation (inbound/outbound) | Entry point from, and back-arrow return to, Settings |
| FEAT-01.SPEC-008 (Active Client Limit Enforcement) | Navigation (inbound) | The blocked add-client message's "Upgrade" action opens this screen |
| FEAT-16.SPEC-001 (Storage Usage Summary) | Navigation (inbound) | The storage-limit warning's "Review your plan" link opens this screen |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-------------------|
| plan_viewed | tier, status | Screen finishes loading | supports success-metrics.md: "Free-to-Paid Conversion" |
| plan_upgrade_initiated | from_tier, billing_cycle_selected | Nadia taps "Continue" on the billing-cycle panel (and, the first time, on the disclosure notice) and is handed to the payment-entry experience | supports success-metrics.md: "Free-to-Paid Conversion" |
| plan_downgrade_offer_responded | response: accepted / kept_paid | Nadia taps "Downgrade now" in the confirmation dialog or "Keep my Paid plan" on the offer banner | supports success-metrics.md: "Free-to-Paid Conversion" |
| plan_cancel_initiated | from_tier, billing_cycle | Nadia taps "Cancel plan" in the Cancel confirmation dialog | supports success-metrics.md: "Free-to-Paid Conversion" |
| charge_retry_initiated | prior_failure_reason, retry_attempts_used | Nadia taps Retry on the Charge failed banner | N/A -- no distinct Stage 2 metric tracks retry attempts specifically; retained alongside FEAT-23.SPEC-003's subscription_charge_failed event so the retry behavior this feature promises is observable |
| payment_entry_abandoned | billing_cycle_selected | Nadia is returned from the payment-entry experience without completing it | N/A -- no Stage 2 metric measures abandonment inside the billing capability's experience; retained so drop-off between "Continue" and a confirmed charge is observable alongside plan_upgrade_initiated |

## Acceptance Criteria

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

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 23 | 23 |
| States | 13 (loading, free, paid, downgrade offered, subscribe in progress, confirming payment, attempt failed, charge failed, charge failed with retries used, cancelled, lapsed, load error, offline) | 13 |
| Business Rules | 5 | 5 |
| Edge Cases | 10 | 10 |

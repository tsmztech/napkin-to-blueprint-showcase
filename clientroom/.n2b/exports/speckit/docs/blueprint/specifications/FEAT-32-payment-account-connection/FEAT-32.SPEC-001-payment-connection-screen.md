---
document_type: spec
spec_type: screen
spec_id: FEAT-32.SPEC-001
spec_name: Payment Connection Screen
spec_slug: payment-connection-screen
parent_feature: FEAT-32
parent_feature_name: Payment Account Connection
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 32
---

# Screen Spec: Payment Connection Screen

## Overview

**Name:** Payment Connection Screen
**ID:** FEAT-32.SPEC-001
**Type:** Screen
**Purpose:** Nadia connects, views the readiness status of, reconnects, or disconnects her payment-processor account from one settings surface.
**Parent Feature:** FEAT-32 -- Payment Account Connection

## Scope and Non-Goals

**In Scope:**
- The single surface for all of the entity's states: loading, load failure, not connected (Empty), Connecting, Connected, and Needs attention
- The consent notice shown before every Connect, Reconnect, or Retry hand-off, with its Continue and Cancel outcomes
- Initiating the Connect and Reconnect hand-off and showing its progress
- Displaying the current readiness status, available payment methods, and (when applicable) the specific attention reason
- The delivery warning shown when a status email finally fails to deliver (FEAT-32.SPEC-006)
- The Disconnect action (from Connected or Needs attention) and its explicit warning
- Entry from Settings, from the onboarding "Connect payments" step, and from an invoice sent without a connected account

**Non-Goals:**
- The actual connect/reconnect hand-off with the payment-processing capability -- owned by FEAT-32.SPEC-002 (Payment Account Connection & Status Reporting); this screen only shows the consent notice, initiates the hand-off after Continue, and shows its progress.
- Applying a reported status change to the underlying record -- owned by FEAT-32.SPEC-003 (Connection Status Sync); this screen only displays the result once applied.
- Removing the connection reference -- owned by FEAT-32.SPEC-004 (Disconnect Payment Account); this screen only collects the confirmed intent to disconnect.
- Who is allowed to act and the exact disconnect-warning wording -- owned by FEAT-32.SPEC-005 (Payment Connection Authorization & Validation Rules), referenced by this screen rather than restated.
- Sending the confirmation or alert email or retrying its delivery -- owned by FEAT-32.SPEC-006; this screen only shows the warning after SPEC-006 reports a final delivery failure.
- Dana's (Support Operator) view of connection status -- excluded per feature-dependency-map.md's Cross-Feature Touchpoints: Dana views status read-only inside her own support-session surface (FEAT-31), never on this screen; this is a deliberate isolation of the operator's view from the freelancer's own settings surface, not an oversight.
- Displaying or editing anything about how a client pays (card entry, bank details) -- excluded per BRIEF.md's Constraints and scope-boundaries.md (SC-10): the product never sees or stores card numbers or bank credentials, so no such fields exist on this screen.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-21.SPEC-001 / SPEC-002 / SPEC-003 / SPEC-004 (Settings screens, via the shared Settings navigation shell) | Nadia opens "Payment account" from Settings | None -- screen loads the current connection state for her account |
| FEAT-20.SPEC-002 (Onboarding Guided Sequence, FEAT-20 Onboarding / First-Run Setup) | Nadia taps "Connect payments" on the optional onboarding step | None -- same screen, reached mid-setup rather than from Settings |
| FEAT-09 (Invoice Generation & Sending) | Nadia follows the prompt shown after an invoice sends with no connected account | None -- screen loads the current state (Empty, since no account is connected) |
| FEAT-32.SPEC-006 (Connection Status Notifications) | Nadia taps the CTA in either the confirmation or the alert email | None -- screen loads the current (now-changed) status |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Connect, Reconnect, Disconnect, and Start over on a stalled hand-off (her own account only, per FEAT-32.SPEC-005) | -- |
| Owen (Client Primary Contact) | No | No | This screen is not part of the client portal and carries no client-facing route; Owen never reaches it -- he experiences this feature only through the resulting pay-link availability on his own invoices (FEAT-09) |
| Priya (Client Reviewer Contact) | No | No | Same as Owen -- no client-facing route exists to this screen |
| Dana (Support Operator) | No | No | Dana never reaches this screen; her read-only view of connection status is a separate surface inside her own logged support session (FEAT-31.SPEC-002, per feature-dependency-map.md's Cross-Feature Touchpoints), which never renders the processor account reference or credentials |
| Unauthenticated | No | No | Redirected to sign-in; after signing in as Nadia, she lands on this screen only if it was her original destination |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." Any in-progress Connect/Reconnect hand-off is left exactly as recorded by FEAT-32.SPEC-002/SPEC-003 at the moment the session expired -- re-authenticating returns her to this screen showing that same state, never a lost or reset connection |

## Layout and Content

**Header:** Screen title "Payment Account," with a back arrow returning to FEAT-21.SPEC-001 (Account Profile, the Settings default entry of Settings & Account Management).

**Body (content depends on the current state -- see States below):**
- **Loading (initial fetch of the current connection record):** A progress indicator with the line "Loading your payment account..." No status and no action button is shown until the fetch completes, because the current state is not yet known.
- **Load failure (the initial fetch fails):** The line "We couldn't load your payment account. Try again." with a "Retry" button. No status line and no Connect, Reconnect, or Disconnect control is shown, since offering an action against an unknown state could offer Connect when a record exists.
- **Not connected:** A short explanation -- "Connect a payment account so card and bank-transfer payments from your invoices go straight into it." -- above a single "Connect payment account" button. No "Reconnect" control appears in this state.
- **Connecting:** A progress indicator with the line "Setting up your connection -- this usually takes a few minutes." Until the start-over threshold below, no action buttons are shown. If FEAT-32.SPEC-002 reports the slow threshold has elapsed, the line "Still checking your connection -- this is taking longer than usual." is added beneath it. When the hand-off (`handoff_started_at`) is older than platform parameter: `payment-connect-handoff-timeout`, the line "This is taking too long. You can start over." and a single "Start over" button are added beneath the progress indicator (see the Interactions table); before that threshold no action button is shown. Connect, Reconnect, and Disconnect stay hidden while Connecting, including after the threshold.
- **Connected:** A status line reading "Ready to accept payments," below it a list of the currently available payment methods (Card, Bank transfer, or both, from `available_payment_methods`), and a "Disconnect" button beneath the list.
- **Needs attention:** A status line reading "Needs attention," below it the specific reason rendered exactly as stored on the record (never a generic message, per the Brief's Shared UI Patterns: Specific-reason display), then a "Reconnect" button, a "Disconnect" button, and a "Contact support" link. When the reason comes from FEAT-32.SPEC-003's zero-methods rule, it reads: "Your payment account has no payment methods turned on yet. Turn on card or bank transfer in your payment account, then reconnect."
- **Error (failed or abandoned hand-off):** The previous state's content remains visible underneath a banner: "We couldn't complete the connection. Try again." with a "Retry" button in the banner. When the failed attempt was a first connect, the previous state is Empty, so the Empty content stays visible beneath the banner.
- **Delivery warning banner (Connected or Needs attention only):** Placed directly beneath the header and above the status line. Text: "We couldn't email you about this change to your payment account. The status shown below is current." with a "Dismiss" button. Shown only after FEAT-32.SPEC-006 reports a final delivery failure for the email that announced the currently displayed status. Paired with a warning icon.
- **Consent notice dialog (overlay, shown before every Connect, Reconnect, or Retry hand-off):** Heading "Before you connect." Body, verbatim from FEAT-32.SPEC-002 Consent and Disclosure: "To connect a payment account, we'll share your name, business details, and sign-in email with the payment-processing capability so it can open or link your account. We never see or store your card numbers or bank credentials." Two buttons: "Continue" and "Cancel."

Every status line pairs its wording with a distinct icon (not colour alone), per assumptions-constraints.md's accessibility commitment (ASMP-27) and product-features.md's States field.

**Footer:** None -- all actions live in the body.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described above, full width; the delivery warning (when present), status line, method list, and action buttons stack vertically in that order. The consent notice and disconnect warning dialogs fill the width less a 16px gutter.
- **Medium size class and above:** Same single-column structure, capped at a consistent platform-wide form width (the design layer's exact value) and horizontally centered; no structural change beyond width capping. Dialogs are centered over the screen at the same capped width.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-21.SPEC-001 (Account Profile, the Settings default entry) | Screen closes | Standard back transition |
| "Connect payment account" button (Empty state) | Tap | Authorization checked per FEAT-32.SPEC-005 (always allowed for Nadia when no record exists), then opens the consent notice dialog. Nothing is sent and no record is created until she taps Continue | Consent notice dialog overlays the screen; the screen behind stays Empty | Dialog with the consent text and "Continue" and "Cancel" |
| "Reconnect" button (Needs attention state only) | Tap | Authorization checked per FEAT-32.SPEC-005, then opens the same consent notice dialog | Consent notice dialog overlays the screen; the screen behind stays Needs attention | Same dialog as Connect |
| "Retry" button (Error banner) | Tap | Re-opens the consent notice dialog for the same attempt type that failed (Connect or Reconnect) -- the notice is shown before every hand-off, including a retry | Consent notice dialog overlays the screen; the Error banner stays visible behind it | Same dialog as Connect |
| Consent notice -- "Continue" | Tap | Triggers the hand-off via FEAT-32.SPEC-002, which records the hand-off start (creating the interim Connecting record for a first connect) before contacting the payment-processing capability | Dialog closes; screen enters Connecting state | Progress indicator with "Setting up your connection -- this usually takes a few minutes." |
| Consent notice -- "Cancel" (or Escape key) | Tap / Esc | No hand-off is initiated, no data leaves the product, no record is created or changed | Dialog closes; the screen remains exactly as it was before the tap (Empty stays Empty, Needs attention stays Needs attention, an Error banner stays visible) | Focus returns to the button that opened the dialog; no message |
| "Disconnect" button (Connected or Needs attention state) | Tap | Shows the explicit disconnect warning dialog defined by FEAT-32.SPEC-005 | Dialog overlays the screen | Dialog text: "If you disconnect, your open invoices will lose their pay links until you connect a payment account again. Any payment already submitted to your processor will not be affected." with "Disconnect" and "Cancel" buttons |
| Disconnect warning dialog -- "Disconnect" | Tap | Triggers FEAT-32.SPEC-004 (Disconnect Payment Account) | Dialog closes; screen shows a brief in-progress indicator; any delivery warning is cleared | On completion: screen returns to the Empty state with the confirmation "Payment account disconnected." |
| Disconnect warning dialog -- "Cancel" | Tap | No action taken | Dialog closes | Screen remains exactly as before, connection unchanged |
| "Contact support" link (Needs attention state) | Tap | Navigate to FEAT-31.SPEC-001 (Contact Support Screen), starting a support request | Screen closes | Standard navigation transition |
| "Dismiss" button (delivery warning banner) | Tap | Marks the warning dismissed for the currently displayed status outcome | Banner disappears and does not return on reload for this status outcome | Focus moves to the status line |
| "Start over" button (Connecting state, shown only once the hand-off is older than platform parameter: `payment-connect-handoff-timeout`) | Tap | Authorization checked per FEAT-32.SPEC-005, then a confirmation dialog opens: "Start over? Your current connection attempt will be cancelled. Your payment account details are not affected." with "Start over" and "Cancel" buttons | Confirmation dialog overlays the screen; the screen behind stays Connecting | Dialog with the text above; focus moves to its heading |
| Start over confirmation -- "Start over" | Tap | Sends the abandoned-hand-off outcome to FEAT-32.SPEC-003 (which re-checks the age of the hand-off and applies its failed/abandoned handling) | Dialog closes; on a first connect the interim record is removed and the screen shows the Error banner over the Empty content; on a Reconnect the previous state (Needs attention) shows beneath the Error banner | Banner "We couldn't complete the connection. Try again." with Retry; focus moves to the banner |
| Start over confirmation -- "Cancel" (or Escape key) | Tap / Esc | No action taken | Dialog closes; the screen remains Connecting | Focus returns to the "Start over" button; no message |
| "Retry" button (Load failure state) | Tap | Repeats the initial fetch of the current connection record | Screen re-enters Loading state; on success it shows the state the record holds | "Loading your payment account..." |

### Accessibility Notes

- **Focus order:** Back arrow -> delivery warning (when present) -> status line (announced as text, not implied by colour) -> available-methods list (Connected) or attention reason (Needs attention) -> primary action button (Connect / Reconnect) -> secondary controls (Disconnect, then Contact support, when present).
- **Status announcements:** Every transition into Loading, Load failure, Connecting, Connected, Needs attention, or Error is announced to assistive technology as text, matching the visible line exactly -- never conveyed by icon or colour alone (ASMP-27). The delivery warning is announced politely when it appears.
- **Consent notice focus handling:** On open, focus moves to the dialog heading "Before you connect" and stays trapped inside the dialog (heading, Continue, Cancel) until it closes. Escape triggers Cancel. On Cancel, focus returns to the button that opened the dialog (Connect payment account, Reconnect, or the banner's Retry). On Continue, focus moves to the Connecting status line.
- **Disconnect dialog focus:** The disconnect warning dialog moves focus to its heading on open, traps focus the same way, and returns focus to the "Disconnect" button on cancel.
- **Keyboard alternatives:** Every action on this screen (Connect, Reconnect, Disconnect, both dialogs' Continue/Confirm and Cancel, Contact support, both Retry buttons, Start over and its confirmation, Dismiss) is reachable and operable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Progress indicator: "Loading your payment account..."; no status, no actions | Screen opens and begins fetching the current Payment Account Connection record | The fetch succeeds (screen enters the state the record holds) or fails (Load failure) |
| Load failure | "We couldn't load your payment account. Try again." with Retry; no status, no actions | The initial fetch fails | Nadia taps Retry (returns to Loading) |
| Empty (not connected) | Explanation text and single "Connect payment account" button | No Payment Account Connection record exists for Nadia's account, or one was just removed by a disconnect or by a failed first-connect attempt (FEAT-32.SPEC-003) | Nadia taps "Connect payment account" and confirms Continue in the consent notice |
| Consent notice (overlay) | Dialog with the consent text, "Continue" and "Cancel"; the underlying state stays visible and unchanged behind it | Nadia taps Connect, Reconnect, or Retry | Continue (hand-off starts, Connecting) or Cancel/Escape (dialog closes, underlying state unchanged) |
| Connecting | Progress indicator: "Setting up your connection -- this usually takes a few minutes." (plus the slow-threshold line when applicable) | The record holds status Connecting, or a hand-off-in-progress marker (`handoff_started_at`) set by FEAT-32.SPEC-002 after Continue -- so it is also shown when Nadia reopens the screen mid-hand-off | FEAT-32.SPEC-003 applies readiness, a restriction, or the failed/abandoned outcome (including the abandoned outcome from Nadia's confirmed "Start over," available once the hand-off is older than platform parameter: `payment-connect-handoff-timeout`) |
| Connected | "Ready to accept payments" status line, available methods list, "Disconnect" button | FEAT-32.SPEC-003 applies a Connected outcome | The connection later degrades to Needs attention, or Nadia disconnects |
| Needs attention | "Needs attention" status line, the specific reason verbatim, "Reconnect" button, "Disconnect" button, "Contact support" link | FEAT-32.SPEC-003 applies a Needs attention outcome (including the zero-methods case) | Nadia successfully reconnects (returns to Connected), or disconnects (returns to Empty) |
| Error (failed/abandoned hand-off) | Previous state's content remains visible beneath a "We couldn't complete the connection. Try again." banner with Retry | A Connect or Reconnect hand-off fails or is abandoned mid-flow (FEAT-32.SPEC-003 has restored the previous record state) | Nadia taps Retry (consent notice, then Connecting), or navigates away and reopens the screen (re-loads the intact previous state without the banner) |
| Delivery warning (banner over Connected or Needs attention) | Banner "We couldn't email you about this change to your payment account. The status shown below is current." with Dismiss | FEAT-32.SPEC-006 reports a final delivery failure for the email announcing the displayed status | Nadia taps Dismiss; or a new status outcome is applied by FEAT-32.SPEC-003; or Nadia disconnects |
| Offline/Degraded | Banner: "You're offline -- reconnect to manage your payment account." The last-loaded status remains visible; Connect, Reconnect, Disconnect, and the consent notice's Continue are all disabled, since each requires reaching the payment-processing capability | Connectivity is lost while this screen is open | Connectivity is restored -- the banner clears and all actions become available again against the current status |

## Validation Rules

Validation and authorization governed by FEAT-32.SPEC-005 (Payment Connection Authorization & Validation Rules). See that spec for the one-account-per-freelancer limit, the Nadia-only action gate, the Connecting interim state, and the exact disconnect-warning text. Authorization is applied on screen entry (which actions render) and on each action attempt.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-21.SPEC-001 (Account Profile, the Settings default entry) | FEAT-21 (Settings & Account Management) |
| Successful disconnect | This screen, Empty state (no navigation away) | -- |
| "Contact support" link tap (Needs attention) | FEAT-31.SPEC-001 (Contact Support Screen) | FEAT-31 (Operator Support Access) |

## Data Model

**Creates:** None directly -- Nadia's Continue in the consent notice triggers FEAT-32.SPEC-002, which creates the Payment Account Connection record in its interim Connecting status for a first connect (see FEAT-32.SPEC-005 for the entity definition).
**Reads:** Payment Account Connection -- `status`, `available_payment_methods`, `handoff_started_at` (all as currently stored), plus the specific attention reason retained with a Needs attention status; and the email delivery-failure state and its dismissed flag for the latest status email, as reported by FEAT-32.SPEC-006.
**Updates:** Only the delivery warning's dismissed flag (set by Dismiss). FEAT-32.SPEC-003 applies all status changes and FEAT-32.SPEC-004 removes the reference; this screen otherwise only displays the outcome.
**Deletes:** None directly -- the Disconnect action collects Nadia's confirmed intent and triggers FEAT-32.SPEC-004, which performs the removal.

## Business Rules

- Only Nadia may act on this screen; every other role sees no route to it at all (FEAT-32.SPEC-005, Authorization Rules).
- The one-connected-account-per-freelancer limit (FEAT-32.SPEC-005) means Connect is offered only when no record exists (Empty state); once one exists, Connect is never shown. Per status: Connected offers Disconnect; Needs attention offers Reconnect, Disconnect, and Contact support; Connecting (interim) offers no action while the hand-off is in progress, except "Start over" once the hand-off is older than platform parameter: `payment-connect-handoff-timeout` (so Nadia is never stuck if the processor reports no outcome); Empty offers only Connect -- Reconnect never appears in the Empty state.
- No action is offered while the current state is unknown: during Loading and Load failure the screen shows no Connect, Reconnect, or Disconnect control.
- The consent notice is shown before every Connect, Reconnect, and Retry hand-off, without exception; Cancel leaves everything unchanged and Continue is the only path that initiates a hand-off (FEAT-32.SPEC-002, Consent and Disclosure).
- Reconnect always re-runs the same full hand-off as Connect (FEAT-32.SPEC-002) -- there is no partial-state resume.
- Disconnect always shows the explicit warning defined by FEAT-32.SPEC-005 before proceeding; there is no one-tap disconnect.
- XBR-19: this screen's Connected/Needs attention/Empty states are the source of truth that FEAT-09 (Invoice Generation & Sending) and FEAT-10 (Invoice Payment Processing) derive pay-link availability from; this screen itself never renders client-facing pay-link copy.
- A failed or abandoned hand-off (Error state) never overwrites the status shown before the attempt -- the previous state's content stays visible beneath the retry banner (product-features.md, States field); for a first connect, the previous state is Empty and the interim Connecting record is removed by FEAT-32.SPEC-003.
- The delivery warning is informational only: it never changes the connection status, blocks any action, or replaces the status line. It clears on Dismiss, when a new status outcome is applied, or on disconnect, and is never shown for an email whose delivery was cancelled by FEAT-32.SPEC-006 (a superseded alert).

## Edge Cases

- **Nadia navigates away mid-hand-off (Connecting state) and returns** -- The screen re-loads (Loading, then the persisted state): the record still holds status Connecting or the `handoff_started_at` marker, so the Connecting state is shown again, or, if FEAT-32.SPEC-003 has applied an outcome meanwhile, the resulting Connected, Needs attention, or Empty state. No hand-off is lost by navigating away, and a failed first connect returns to Empty because SPEC-003 removes the interim record.
- **Nadia taps Disconnect twice in rapid succession** -- The second tap while the first disconnect is processing is ignored; the Disconnect button is disabled during processing to prevent a double submission.
- **Nadia has this screen open in two browser tabs and disconnects from one** -- The disconnect (FEAT-32.SPEC-004) removes the connection reference regardless of the other tab's state. If Nadia then taps Disconnect in the second, now-stale tab, the action is a no-op (there is nothing left to remove) and that tab shows the Empty state on its next refresh -- resolution: the explicit disconnect always wins over a stale view, consistent with the dependency map's Contention note that processor-reported status, and Nadia's own explicit disconnect, are authoritative over any other open view of the same record.
- **The processor reports a status change (Connected or Needs attention) while Nadia is viewing this screen without taking an action here** -- The screen reflects the current status as of when it was loaded or last refreshed; it re-fetches automatically after any action she takes here, and offers a manual refresh. A status change arriving from elsewhere may not appear until she refreshes or reopens the screen.
- **Nadia taps Disconnect while a client's payment is already submitted to the processor** -- The disconnect is not blocked and the in-flight payment is unaffected (FEAT-32.SPEC-005's contention rule); this screen shows the Empty state immediately after the disconnect completes, with no indication tied to the unrelated in-flight payment.
- **Nadia reaches this screen from the FEAT-20 onboarding step and skips it** -- The screen is simply left in whatever state existed before (typically Empty); skipping produces no error and no partial record. Likewise, opening the consent notice and tapping Cancel creates no record.
- **Connectivity is lost while the consent notice is open** -- The offline banner appears behind the dialog and "Continue" is disabled; "Cancel" remains available. When connectivity returns, Continue is enabled again; nothing was sent while it was disabled.
- **The processor never reports an outcome (Nadia is stuck on Connecting)** -- Once the hand-off is older than platform parameter: `payment-connect-handoff-timeout`, "Start over" appears (the threshold is evaluated on load and while the screen stays open, so it appears without a reload). Confirming it routes to FEAT-32.SPEC-003's abandoned-hand-off handling, so a first connect returns to Empty (with the Error banner) and a Reconnect returns to the intact Needs attention state.
- **Nadia taps "Start over" at the same moment the processor reports an outcome** -- FEAT-32.SPEC-003 applies whichever arrives first in event-time order; if the readiness or restriction outcome is applied first, the abandonment finds no hand-off in progress and is a no-op, and the screen shows the applied outcome instead of the Error banner.
- **A new status outcome is applied while a delivery warning is showing** -- The warning refers to the email for the previous outcome, so it clears as soon as the screen shows the new outcome; a fresh warning appears only if the new outcome's own email also fails finally.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-32.SPEC-002 (Payment Account Connection & Status Reporting) | Triggers (outbound) | Continue in the consent notice initiates the hand-off; the notice wording and Cancel outcome are defined there |
| FEAT-32.SPEC-003 (Connection Status Sync) | References (inbound); Triggers (outbound) | Supplies the current status, attention reason, and available payment methods this screen displays, and restores the previous state after a failed attempt; a confirmed "Start over" on a stalled hand-off triggers its abandoned-hand-off handling |
| FEAT-32.SPEC-004 (Disconnect Payment Account) | Triggers (outbound) | Confirmed Disconnect triggers the removal |
| FEAT-32.SPEC-005 (Payment Connection Authorization & Validation Rules) | References (inbound) | Governs who can act, the one-account limit, the Connecting interim status, and the exact disconnect-warning wording |
| FEAT-32.SPEC-006 (Connection Status Notifications) | Navigation (inbound) | Both emails' CTAs deep-link back to this screen |
| FEAT-32.SPEC-006 (Connection Status Notifications) | References (inbound) | Reports a final email delivery failure, which this screen surfaces as the delivery warning banner |
| FEAT-21.SPEC-001 / SPEC-002 / SPEC-003 / SPEC-004 (Settings & Account Management) | Navigation (inbound/outbound) | Entry point via the Settings "Payment account" navigation item; back arrow returns to FEAT-21.SPEC-001 |
| FEAT-20.SPEC-002 (Onboarding Guided Sequence) | Navigation (inbound) | Entry point via the optional "Connect payments" step |
| FEAT-09 (Invoice Generation & Sending) | Navigation (inbound) | Entry point from the prompt on an invoice sent without a connected account |
| FEAT-31.SPEC-001 (Contact Support Screen) | Navigation (outbound) | "Contact support" link from the Needs attention state |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| payment_connection_screen_viewed | entry source (settings / onboarding / invoice_prompt / email_cta), status shown at load | The initial fetch succeeds and the screen leaves the Loading state | supports success-metrics.md: "Payment Readiness Before First Invoice" |
| payment_connection_screen_load_failed | entry source | The initial fetch fails and Load failure is shown | N/A -- no Stage 2 metric measures screen load reliability; retained so a failed load is observable rather than silent |
| payment_connect_button_tapped | entry state (empty / needs_attention / error_retry -- i.e. Connect, Reconnect, or Retry) | Nadia taps Connect, Reconnect, or Retry (before the consent notice opens) | supports success-metrics.md: "Payment Readiness Before First Invoice" |
| payment_connection_notice_decision | attempt type (connect / reconnect), decision (continue / cancel) | Nadia taps Continue or Cancel in the consent notice | supports success-metrics.md: "Payment Readiness Before First Invoice" |
| payment_connection_delivery_warning_shown | variant (connected / needs_attention) | The delivery warning banner first appears | N/A -- no Stage 2 metric measures email delivery failure visibility; retained as a delivery-quality signal alongside FEAT-32.SPEC-006's own failure signal |
| payment_connect_start_over_confirmed | attempt type (connect / reconnect), hand-off age at confirmation | Nadia confirms "Start over" in the confirmation dialog | N/A -- no Stage 2 metric measures stalled hand-offs; retained so hand-offs that never receive a processor outcome are observable |
| payment_disconnect_confirmed | prior status (connected / needs_attention) | Nadia confirms Disconnect in the warning dialog | N/A -- no Stage 2 metric measures disconnect frequency; retained alongside FEAT-32.SPEC-004's own disconnected signal for observability of this screen's role in the action |

## Acceptance Criteria

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

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 15 | 15 |
| States | 10 (loading, load failure, empty, consent notice, connecting, connected, needs attention, error, delivery warning, offline) | 10 |
| Business Rules | 9 | 9 |
| Edge Cases | 10 | 10 |

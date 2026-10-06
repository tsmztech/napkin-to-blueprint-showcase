# FEAT-32 — Payment Account Connection

This chapter covers Payment Account Connection, a Core-tier feature. It contains the feature breakdown brief followed by every specification in full: 6 specifications carrying 119 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-32.SPEC-001 | Payment Connection Screen | screen | 32 |
| FEAT-32.SPEC-002 | Payment Account Connection & Status Reporting | integration | 18 |
| FEAT-32.SPEC-003 | Connection Status Sync | automation | 19 |
| FEAT-32.SPEC-004 | Disconnect Payment Account | automation | 12 |
| FEAT-32.SPEC-005 | Payment Connection Authorization & Validation Rules | logic-rule | 21 |
| FEAT-32.SPEC-006 | Connection Status Notifications | notification | 17 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Payment Account Connection

## Summary

**Feature:** Payment Account Connection
**ID:** FEAT-32
**Description:** The freelancer connects her own payment-processor account once, so every invoice's pay link sends card and bank-transfer payments straight into her account. She can see whether payments are ready to accept, and reconnect or disconnect the account.
**Priority:** Core
**Phase:** MVP
**Type:** Platform
**Rationale:** BRIEF.md, Ecosystem & Integrations: "an established processor takes card and bank-transfer payments directly into each freelancer's own account," and Business Context: "Payments go straight to the freelancer's own processor account and the platform never holds their money." The draft defined how a client pays (FEAT-10) but no step where the freelancer's own account is linked, so the money had no path from the pay link into her account. Core because no invoice can be paid in the portal without it: the "pay it by card on the spot" moment (BRIEF.md, The Experience) and the "paid noticeably faster" success criterion both depend on it, and the evidence is the brief's explicit, required integration. Research supports the direct-to-freelancer model: platforms that route payments through themselves draw complaints about stacked fees and delayed payouts (HoneyBook, Bonsai; G2 and Trustpilot reviews, MEDIUM). MVP phase: the first deposit invoice needs it. [AUDIT-ADDED: 1 -- Core: the value-flow walk found no feature connecting the freelancer's own processor account, so client money had no path into her account; every in-portal payment depends on this capability]

**Key Capabilities:**
- Connect her payment account -- Nadia links her own existing or new processor account in a guided step
- See connection status -- whether card and bank-transfer payments are ready to accept
- Reconnect or disconnect -- fix a broken connection, or remove it

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-32.SPEC-001 | Payment Connection Screen | Screen | Nadia (Freelancer) | Nadia connects, views the readiness status of, reconnects, or disconnects her payment-processor account from one settings surface |
| FEAT-32.SPEC-002 | Payment Account Connection & Status Reporting | Integration | Nadia (Freelancer) | Initiates the connect/reconnect hand-off with the payment-processing capability and receives back readiness status, the specific attention reason, available payment methods, and reversal/chargeback notices to relay onward |
| FEAT-32.SPEC-003 | Connection Status Sync | Automation | Nadia (Freelancer), Owen (Client Primary Contact) | Applies the processor-reported status (Connected, Needs attention, or a failed/abandoned attempt) and available payment methods to the Payment Account Connection record the instant it is reported |
| FEAT-32.SPEC-004 | Disconnect Payment Account | Automation | Nadia (Freelancer), Owen (Client Primary Contact) | Removes Nadia's connection reference on her explicit disconnect action, without cancelling any payment already submitted to the processor |
| FEAT-32.SPEC-005 | Payment Connection Authorization & Validation Rules | Logic/Rule | Nadia (Freelancer), Owen (Client Primary Contact), Dana (Support Operator) | Governs the one-account-per-freelancer limit, who may connect/reconnect/disconnect versus view only, the disconnect warning, and the processor-authoritative contention rule that protects an in-flight payment from a concurrent disconnect |
| FEAT-32.SPEC-006 | Connection Status Notifications | Notification | Nadia (Freelancer) | Sends Nadia a confirmation email when her account connects and an alert email when the connection breaks or needs attention |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Connect her payment account -- Nadia links her own existing or new processor account in a guided step | FEAT-32.SPEC-001, FEAT-32.SPEC-002, FEAT-32.SPEC-003 | The screen offers the guided Connect action and shows hand-off progress; the integration spec submits the hand-off to the payment-processing capability; the automation applies the confirmed outcome to the connection record | Phase 2 (Explicit) |
| See connection status -- whether card and bank-transfer payments are ready to accept | FEAT-32.SPEC-001, FEAT-32.SPEC-003 | The screen displays "Ready to accept payments," the specific needs-attention reason, or not-connected; the automation keeps that status current | Phase 2 (Explicit) |
| Reconnect or disconnect -- fix a broken connection, or remove it | FEAT-32.SPEC-001, FEAT-32.SPEC-002, FEAT-32.SPEC-003, FEAT-32.SPEC-004, FEAT-32.SPEC-005 | The screen offers Reconnect (re-runs the same hand-off as Connect) and Disconnect (with the explicit pay-link warning governed by the rules spec); the disconnect automation removes the reference | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-32.SPEC-002 | Payment Account Connection & Status Reporting | Phase 4 (External Dependencies lens) | The Dependencies section of assumptions-constraints.md (ASMP-28) names the payment-processing capability this feature relies on for account connection and status reporting; the External Touchpoints row in the dependency map marks the connection, status, and reversal aspects of that capability as this feature's to specify. Per the standalone-spec decision rule, any trigger-response crossing the product boundary belongs to an Integration spec, never inline in a screen |
| FEAT-32.SPEC-004 | Disconnect Payment Account | Phase 3 (Entity-Lifecycle Analysis) | The Delete/Archive cell of the CRUD matrix for Payment Account Connection is a genuine operation (the dependency map states the entity is "Deleted by FEAT-32 (disconnect)"), distinct from the inbound-status-driven Connection Status Sync automation, because it is triggered by Nadia's own action rather than a processor report |
| FEAT-32.SPEC-005 | Payment Connection Authorization & Validation Rules | Phase 5 (Rule-Constraint Discovery) | The Validation & Limits, Access, and Contention fields together produce 5+ interacting rules (one-account-per-freelancer, Nadia-only connect/reconnect/disconnect, Dana's view-only constraint, the explicit disconnect warning, and the processor-status-authoritative/in-flight-payment-protection contention rule) shared across SPEC-001, SPEC-002, SPEC-003, and SPEC-004 -- past the inline-validation threshold |
| FEAT-32.SPEC-006 | Connection Status Notifications | Phase 4 (Notification surfacing) | The Communications field names two emails to Nadia, each with a defined audience, trigger, and content -- neither is a same-screen confirmation with no delivery rules, so both need a Notification spec rather than an inline toast |

## Entity-Lifecycle Coverage Matrix

**Entity: Payment Account Connection**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-32.SPEC-002 | Creates the connection record with its processor account reference when Nadia's connect hand-off is initiated, in an interim "connecting" sub-state until the processor confirms | Also created via the same path when Nadia connects from the FEAT-20 onboarding step |
| Read (single) | FEAT-32.SPEC-001 | Payment Connection Screen shows the current status, the specific needs-attention reason, and available payment methods | Also read by FEAT-09, FEAT-10, FEAT-25, and FEAT-31 (status only), per the dependency map |
| Read (list) | N/A | Validation & Limits sets one connected payment account per freelancer (product-features.md), so there is never more than one record to list -- no list view is needed | -- |
| Update | FEAT-32.SPEC-003 | Transitions status Not connected → Connected / Needs attention as the payment-processing capability reports each stage, and updates available_payment_methods accordingly | A failed or abandoned connection attempt leaves the previous state intact (product-features.md, States field) rather than overwriting it with an incomplete one |
| Delete/Archive | FEAT-32.SPEC-004 | Hard delete: Nadia's explicit disconnect removes the processor_account_reference entirely -- no restore path (reconnecting always runs the Create operation again, since the record holds only a reference with no historical value to restore); no cascade to Invoice or Payment records (their own historical status is untouched); no retention/purge policy applies, since nothing beyond the live reference is ever retained | Also deleted by FEAT-24 on account deletion (XBR-33) |
| State Transition | FEAT-32.SPEC-003 (Not connected → Connected, Connected → Needs attention, Needs attention → Connected), FEAT-32.SPEC-004 (→ Disconnected) | -- | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Invoice | FEAT-32.SPEC-002 | Correlates a processor-reported reversal or chargeback notice to the specific paid invoice before relaying it to Refund & Cancelled Project Handling (FEAT-25) |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Nadia taps Connect (first time or from onboarding) | Initiate the connect hand-off with the payment-processing capability | Standalone Integration | SPEC-002 |
| Nadia taps Reconnect after a needs-attention or disconnected state | Initiate the same hand-off as Connect | Standalone Integration | SPEC-002 |
| The payment-processing capability confirms the account is ready | Mark the connection Connected, set available payment methods, show "Ready to accept payments" | Standalone Automation | SPEC-003 |
| The payment-processing capability restricts the account or asks for more information | Mark the connection Needs attention with the specific reason | Standalone Automation | SPEC-003 |
| A connect or reconnect hand-off fails or is abandoned mid-flow | Leave the previous status intact and offer retry, rather than showing a false or blank state | Inline in SPEC-001 (Error state), driven by SPEC-003 | SPEC-001 / SPEC-003 |
| The connection reaches Connected | Send a confirmation email to Nadia | Standalone Notification | SPEC-006 |
| The connection reaches Needs attention | Send an alert email to Nadia | Standalone Notification | SPEC-006 |
| Nadia taps Disconnect | Show the explicit warning that open invoices will lose their pay links, then remove the connection reference on confirmation | Standalone Automation, warning text governed by SPEC-005 | SPEC-004 / SPEC-005 |
| A disconnect is issued while a client's payment is already submitted to the processor | The in-flight payment is not cancelled; a pay link opened afterward shows "online payment is temporarily unavailable" instead (reject-with-refresh on the pay page) | Standalone Logic/Rule | SPEC-005 |
| The payment-processing capability reports a chargeback or reversal on a paid invoice | Relay the reversal notice to Refund & Cancelled Project Handling (FEAT-25), which owns the Disputed status and freelancer notice | Standalone Integration (inbound event), then cross-feature | SPEC-002 / FEAT-25 responsibility |
| The connection's status or available payment methods change | Invoice pay-link availability is derived from the new status | Cross-feature -- owned by Invoice Generation & Sending (FEAT-09), which derives pay_link availability | FEAT-09 responsibility |
| The connection is disconnected | Any invoice opened afterward shows a working "pay Nadia directly" fallback instead of a card/bank-transfer form | Cross-feature -- owned by FEAT-09 (issuing) and FEAT-10 (pay screen), per XBR-19 | FEAT-09 / FEAT-10 responsibility |
| Nadia's account is deleted | The payment connection is disconnected as part of account deletion | Cross-feature -- owned by Data Export & Account Deletion (FEAT-24), per XBR-33 | FEAT-24 responsibility |
| A connect, reconnect, disconnect, or needs-attention event occurs | Write an append-only Activity Log Entry | Cross-feature -- owned by Immutable Activity & Audit Trail (FEAT-13) | FEAT-13 responsibility |

## Shared Context

**Shared Entities:**
- Payment Account Connection -- created and updated by SPEC-002 and SPEC-003 (connect/reconnect path), deleted by SPEC-004 (disconnect), read/displayed by SPEC-001, and referenced by SPEC-005 for its one-account limit and contention rule. Fields: `processor_account_reference` (reference only, never card or bank credentials), `status` (Not connected, Connected, Needs attention, Disconnected), `available_payment_methods` (derived).
- Invoice -- read (not owned) by SPEC-002 to correlate a reversal notice to the right invoice before relaying it to FEAT-25; this feature never writes any Invoice field.

**Shared UI Patterns:**
- Single-surface settings pattern -- SPEC-001 is the one screen for this feature's entire capability set (connect, status, reconnect, disconnect), following the Platform-feature heuristic of fewer, denser screens rather than splitting connect/status/disconnect into separate views; its states (Empty, Loading, Error) are all instances of the same screen rather than distinct specs.
- Specific-reason display -- the needs-attention state always shows the processor's stated reason verbatim (never a generic "something went wrong"), consistently referenced by SPEC-001's display and SPEC-006's alert email content, so Nadia sees the same explanation in both places.

**Shared Validation:**
- SPEC-005 defines the one-account-per-freelancer limit, the Nadia-only action gate, the disconnect warning requirement, and the processor-authoritative contention rule. SPEC-001, SPEC-002, SPEC-003, and SPEC-004 all reference SPEC-005 rather than restating these rules.

## Internal Dependency Map

```
SPEC-001 (Payment Connection Screen) -> [Nadia taps Connect or Reconnect] -> SPEC-005 (Payment Connection Authorization & Validation Rules) -> [pass] -> SPEC-002 (Payment Account Connection & Status Reporting)
SPEC-002 (Payment Account Connection & Status Reporting) -> [processor reports readiness, an attention reason, or a reversal] -> SPEC-003 (Connection Status Sync)
SPEC-003 (Connection Status Sync) -> [status becomes Connected] -> SPEC-001 (Payment Connection Screen shows "Ready to accept payments")
SPEC-003 (Connection Status Sync) -> [status becomes Needs attention] -> SPEC-001 (Payment Connection Screen shows the specific reason and a reconnect action)
SPEC-003 (Connection Status Sync) -> [status becomes Connected or Needs attention] -> SPEC-006 (Connection Status Notifications)
SPEC-001 (Payment Connection Screen) -> [Nadia taps Disconnect] -> SPEC-005 (Payment Connection Authorization & Validation Rules) -> [pass, warning acknowledged] -> SPEC-004 (Disconnect Payment Account)
SPEC-004 (Disconnect Payment Account) -> [connection reference removed] -> SPEC-001 (Payment Connection Screen returns to the Empty state)
SPEC-002 (Payment Account Connection & Status Reporting) -> [reversal or chargeback reported] -> FEAT-25 (Refund & Cancelled Project Handling)
```

**Default Entry:** SPEC-001 (Payment Connection Screen) -- reached from the Branding, Onboarding & Settings area, the optional "Connect payments" step in Onboarding / First-Run Setup (FEAT-20), or the prompt to connect on an invoice sent without a connected account (FEAT-09).

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-32.SPEC-001 | Inbound | FEAT-20 (Onboarding / First-Run Setup) | Nadia reaches the Connect step from the guided onboarding flow (optional step) | Nadia reaches the "Connect payments" onboarding step |
| FEAT-32.SPEC-001 | Inbound | FEAT-09 (Invoice Generation & Sending) | Nadia is prompted to connect from an invoice sent without a connected account | An invoice issues and sends with no connected payment account |
| FEAT-32.SPEC-003 | Outbound | FEAT-09 (Invoice Generation & Sending) | FEAT-09 derives pay_link availability directly from this feature's connection status and available payment methods (XBR-19) | The connection's status or available payment methods change |
| FEAT-32.SPEC-002, FEAT-32.SPEC-003 | Outbound | FEAT-10 (Invoice Payment Processing) | Gates whether Owen's pay screen shows a working card/bank-transfer form or "online payment is temporarily unavailable"; a successful payment lands directly in Nadia's connected account | Owen opens a pay link |
| FEAT-32.SPEC-002 | Outbound | FEAT-25 (Refund & Cancelled Project Handling) | Relays a processor-reported reversal or chargeback notice; FEAT-25 owns the resulting Disputed status and freelancer notice (XBR-21) | The payment-processing capability reports a reversal or chargeback |
| FEAT-32.SPEC-001 | Inbound | FEAT-31 (Support Access) | Dana views the connection status read-only inside a logged support session that FEAT-31 owns; she never reaches SPEC-001 itself and never sees account credentials | Dana opens a support session |
| FEAT-32.SPEC-004 | Inbound | FEAT-24 (Data Export & Account Deletion) | Account deletion disconnects the payment account as part of its warn-then-remove sequence (XBR-33) | Nadia deletes her account |
| FEAT-32.SPEC-006 | Outbound | FEAT-14 (Notifications & Email) | Both emails are sent through the transactional email delivery capability owned by FEAT-14; this feature carries no Integration spec of its own for that capability | The connection reaches Connected or Needs attention |
| FEAT-32.SPEC-002, FEAT-32.SPEC-003, FEAT-32.SPEC-004 | Outbound | FEAT-13 (Immutable Activity & Audit Trail) | Connect, reconnect, disconnect, and needs-attention events each write an append-only trail entry | Any connection status change |

## Non-Functional Notes

**Data volumes / growth:** At most one Payment Account Connection record per freelancer (Validation & Limits, product-features.md); volume scales one-to-one with the freelancer roster (a few thousand freelancers in year one, per scope-boundaries.md SC-21), so this feature carries no growth concern of its own. This feature emits `payment_account_connect_started`, `payment_account_connected`, `payment_account_needs_attention`, and `payment_account_disconnected` signals (product-features.md, Signals field); SPEC-002 fires the started signal, SPEC-003 fires connected/needs-attention, and SPEC-004 fires disconnected.

**Responsiveness:** Connecting is expected to take under 5 minutes of Nadia's own time, with at least 90% of freelancers connected and ready by the time their first invoice sends (success-metrics.md, Payment Readiness Before First Invoice); the connection hand-off shows real progress while the processor confirms rather than a silent wait (product-features.md, States field; assumptions-constraints.md, ASMP-27).

**Data sensitivity / privacy:** The Payment Account Connection record holds a financial-account linkage -- a reference and a readiness status only. The product never sees or stores card numbers or bank credentials; that handling belongs entirely to the payment-processing capability (BRIEF.md, Constraints; assumptions-constraints.md, ASMP-24; scope-boundaries.md, SC-10). Dana's support-session view of this feature is status-only and never exposes the reference or credentials (feature-dependency-map.md, Entity: Payment Account Connection, Data Sensitivity).

**Compliance flags:** The financial-account linkage is GDPR-class personal data tied to Nadia's identity (assumptions-constraints.md, ASMP-24). The Payment Connection Screen must remain usable with a screen reader and keyboard and never rely on colour alone to distinguish Connected from Needs attention (assumptions-constraints.md, ASMP-27; product-features.md, States field).

## Non-Goals

- **The product seeing or storing card numbers or bank credentials** -- Excluded per BRIEF.md's Constraints and scope-boundaries.md (SC-10): the product stores only a reference and a readiness status; all credential handling belongs to the payment-processing capability.
- **More than one connected payment account per freelancer, or connecting several processor accounts concurrently** -- Excluded per product-features.md's Validation & Limits field, which sets exactly one connected payment account per freelancer; this is a product decision, not an oversight, and is enforced by SPEC-005.
- **Issuing refunds or fighting chargebacks inside Clientroom** -- Excluded per scope-boundaries.md (SC-18): refunds and dispute responses happen in the freelancer's own processor account; this feature only relays the resulting reversal notice to Refund & Cancelled Project Handling (FEAT-25), which owns recording it.
- **Restoring a disconnected payment account without repeating the connect flow** -- Intentional lifecycle decision surfaced by the CRUD matrix: disconnect is a hard delete of the reference with no restore path, because the record holds no historical value beyond the live reference (feature-dependency-map.md, Entity: Payment Account Connection); reconnecting always runs the Create operation again through SPEC-002.
- **Scoped or delegated access to payment connection settings for other staff** -- Excluded per scope-boundaries.md (SC-01): the product models solo freelancer accounts only, with no internal-staff seat model, so only Nadia herself ever connects, reconnects, or disconnects.



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



# Integration Spec: Payment Account Connection & Status Reporting

## Overview

**Name:** Payment Account Connection & Status Reporting
**ID:** FEAT-32.SPEC-002
**Type:** Integration
**Purpose:** Initiates the connect/reconnect hand-off with the payment-processing capability and receives back readiness status, the specific attention reason, available payment methods, and reversal/chargeback notices to relay onward.
**Parent Feature:** FEAT-32 -- Payment Account Connection

## Scope and Non-Goals

**In Scope:**
- Initiating a connect or reconnect hand-off for Nadia's own processor account
- Receiving the capability's reported readiness status, specific attention reason, and available payment methods
- Receiving a reported payment reversal or chargeback and correlating it to the affected invoice before relaying it onward
- User-facing behavior when the capability is slow, unavailable, or rejects the hand-off
- Disclosure to Nadia about what data is shared with the capability to open or link her account

**Non-Goals:**
- Choosing the payment-processing vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md's Ecosystem & Integrations section names only the category ("an established processor").
- Applying a reported status or methods change to the Payment Account Connection record -- owned entirely by FEAT-32.SPEC-003 (Connection Status Sync); this spec defines only the request/response contract with the capability.
- Removing the connection reference -- owned by FEAT-32.SPEC-004 (Disconnect Payment Account); disconnecting is Nadia's own in-product action and involves no request to this capability.
- Submitting a card or bank-transfer payment, or reporting its outcome -- owned entirely by FEAT-10.SPEC-003 (Card & Bank-Transfer Payment Processing); this spec's contract covers only the connection and its status, never a specific payment attempt.
- Recording the Disputed status or notifying Nadia about a reversal -- owned entirely by FEAT-25 (Refund & Cancelled Project Handling, XBR-21); this spec's responsibility ends at correlating the reported reversal to the right invoice and relaying the notice.
- Rendering the client-facing "online payment is temporarily unavailable" pay-link fallback -- owned by FEAT-09.SPEC-009 and FEAT-10 (XBR-19); this spec only supplies the status those specs read.

## Capability Category

**Category:** Payment processing (into each freelancer's own account)
**Dependency Source:** ASMP-28 -- Dependencies entry in assumptions-constraints.md naming the payment-processing capability
**External Touchpoint:** "Payment processing into each freelancer's own account -- connection, card and bank-transfer payment, status, pending transfers, reversals" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-09, FEAT-10, FEAT-20, FEAT-25, FEAT-32)
**Vendor Mandate:** None -- BRIEF.md, Ecosystem & Integrations names only the category ("an established processor takes card and bank-transfer payments directly into each freelancer's own account"); vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Nadia links her own existing or new processor account in a guided step | Connect her payment account | FEAT-32.SPEC-001 (Payment Connection Screen), FEAT-32.SPEC-003 (Connection Status Sync) |
| Nadia sees whether card and bank-transfer payments are ready to accept, and the specific reason when they are not | See connection status | FEAT-32.SPEC-001, FEAT-32.SPEC-003 |
| Nadia fixes a broken connection by reconnecting | Reconnect or disconnect | FEAT-32.SPEC-001, FEAT-32.SPEC-003 |
| A reported chargeback or reversal on a paid invoice reaches Nadia as a Disputed status and a notice | Cross-feature: XBR-21 | FEAT-25.SPEC-005 (Payment Reversal (Chargeback) Recording), which fires FEAT-25.SPEC-008 (Payment Reversal Notification) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Freelancer's name and business details | Freelancer Account -- name, business_name, business_address | Nadia initiates a Connect or Reconnect hand-off | The capability must identify who it is opening or linking an account for |
| Freelancer's sign-in email | Freelancer Account -- sign-in email | Nadia initiates a Connect or Reconnect hand-off | Account identification and the capability's own account-recovery contact |

**Record lifecycle around a hand-off (this spec's creation step):** Nadia's Continue in the consent notice on FEAT-32.SPEC-001 is the only event that starts a hand-off. Immediately before contacting the capability, this spec (1) for a first connect (no record exists), creates the Payment Account Connection record with `status` Connecting, no `processor_account_reference`, `handoff_started_at` set to the Continue time, and `last_event_reported_at` set to the same time; or (2) for a Reconnect (record in Needs attention), leaves `status` unchanged and sets `handoff_started_at` and advances `last_event_reported_at` to the Continue time. The Connecting status and `handoff_started_at` are what FEAT-32.SPEC-001 reads to show the Connecting state again after Nadia navigates away and returns. If the hand-off fails or is abandoned, FEAT-32.SPEC-003 deletes the interim Connecting record (first connect) or clears `handoff_started_at` (Reconnect), so no half-created record is left behind. Cancel in the consent notice creates and changes nothing.

Card numbers, bank credentials, client data, invoice content, and payment amounts never leave the product through this spec -- credential entry happens entirely inside the payment-processing capability's own flow, and payment-level data is exchanged separately by FEAT-10.SPEC-003.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Readiness confirmation | The hand-off completes and the capability reports the account is ready | Routed to FEAT-32.SPEC-003, which sets Payment Account Connection -- `status` (Connected), `processor_account_reference`, `available_payment_methods` |
| Restriction or information request, with a specific reason | The capability restricts the account or asks for more information, at any point after connecting | Routed to FEAT-32.SPEC-003, which sets Payment Account Connection -- `status` (Needs attention, carrying the reason as descriptive text) |
| Hand-off failure or abandonment | The connect/reconnect attempt does not complete | Routed to FEAT-32.SPEC-003, which deletes the interim Connecting record (first connect) or clears `handoff_started_at` and leaves the rest of the record unchanged (Reconnect); surfaced inline on FEAT-32.SPEC-001 |
| Reversal or chargeback notice | The capability reports a dispute on a previously paid invoice | Correlated against Invoice (read-only, by the invoice reference the notice carries) and relayed to FEAT-25, which owns the resulting Disputed status; no field of Payment Account Connection is changed by this event |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Connection confirmed ready | The capability reports the account is ready to accept payments, following a Connect or Reconnect hand-off | Routed to FEAT-32.SPEC-003 for multi-step application (status, reference, methods) | Applied by FEAT-32.SPEC-003; then shown on FEAT-32.SPEC-001 as "Ready to accept payments" -- or, if the report lists zero available payment methods, as Needs attention with SPEC-003's defined zero-methods reason | FEAT-32.SPEC-003 |
| Connection restricted / more information requested | The capability restricts the account or asks for more information, at any point (including after previously being Connected) | Routed to FEAT-32.SPEC-003 for multi-step application (status, reason, methods) | Applied by FEAT-32.SPEC-003; then shown on FEAT-32.SPEC-001 with the specific reason | FEAT-32.SPEC-003, FEAT-32.SPEC-006 |
| Hand-off fails or is abandoned mid-flow | Nadia's connect or reconnect attempt does not complete, technically or because she exits before completion | Routed to FEAT-32.SPEC-003, which removes the interim record of a first connect or clears the in-progress marker of a Reconnect | FEAT-32.SPEC-001 shows the Error state with Retry over the previous state; no false Connected or Needs-attention state is shown | FEAT-32.SPEC-001, FEAT-32.SPEC-003 |
| Reversal or chargeback reported | The capability reports a dispute on an invoice already recorded as Paid, at any time after payment | The affected Invoice is identified by the reference the notice carries; the notice is relayed to FEAT-25.SPEC-005 | No feedback from this spec directly -- FEAT-25 owns the Disputed status display and Nadia's notice (XBR-21) | FEAT-25.SPEC-005 (Payment Reversal (Chargeback) Recording) |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-32.SPEC-001 (Payment Connection Screen) | The Connecting state continues showing "Setting up your connection -- this usually takes a few minutes."; if the hand-off has not confirmed within platform parameter: `payment-connection-handoff-slow-threshold`, the screen adds "Still checking your connection -- this is taking longer than usual." Nothing else on the screen is blocked. | The Connect/Reconnect button is disabled with "Connecting payment accounts is temporarily unavailable. Try again in a few minutes." Any existing connection's last-known status remains displayed unchanged. | The Error state applies: "We couldn't complete the connection. Try again." The previous status (if any) remains intact underneath; the interim Connecting record created for a first connect is deleted by FEAT-32.SPEC-003 on the failure event, so no half-created connection record persists. |

## Consent and Disclosure

- **First connect/reconnect disclosure** -- Before a hand-off is initiated (whether Connect, Reconnect, or Retry after a failed attempt), a notice appears: "To connect a payment account, we'll share your name, business details, and sign-in email with the payment-processing capability so it can open or link your account. We never see or store your card numbers or bank credentials." with "Continue" and "Cancel" options. Shown before every hand-off, since each one is a distinct, deliberate action Nadia takes. The dialog itself (heading, buttons, focus handling, Escape) is defined in FEAT-32.SPEC-001's Layout, Interactions, and Accessibility Notes.
- **Cancel outcome** -- "Continue" starts the hand-off and creates or marks the record as described under Data Exchanged. "Cancel" (or Escape) closes the notice: no hand-off is initiated, nothing is sent to the capability, no record is created or changed, and FEAT-32.SPEC-001 stays in the state it was in before the tap, with no error and no message.
- **What is never shared** -- Client data, invoice content, and payment amounts stay inside the product for this spec's purposes; card numbers and bank credentials are entered entirely inside the capability's own flow and never pass through the product (BRIEF.md, Constraints; ASMP-24; SC-10).

## Edge Cases

- **The readiness-confirmed event is delivered twice for the same hand-off** -- FEAT-32.SPEC-003's already-applied guard means the second delivery changes nothing; the record already shows Connected and no duplicate confirmation email fires (FEAT-32.SPEC-006).
- **Events arrive out of order (a restriction event followed by a stale readiness event from an earlier moment)** -- FEAT-32.SPEC-003 applies the most recent event by the time the capability reports it occurred, not by arrival order; a late-arriving, now-stale readiness report does not overwrite a more recent restriction.
- **A status-report event arrives for a connection Nadia has since disconnected** -- The event finds no record to update (FEAT-32.SPEC-004 hard-deletes the reference on disconnect) and is discarded without recreating a record; reconnecting always starts a fresh hand-off through this spec rather than resuming a stale report.
- **A reversal notice arrives for an invoice that no longer exists (removed via FEAT-24 account deletion)** -- The notice cannot be correlated to a record that no longer exists and is discarded; there is no freelancer left to notify.
- **The capability goes down mid-hand-off, after Nadia has already been asked to authorize but before readiness is confirmed** -- The outage is reported as a failed hand-off, so FEAT-32.SPEC-003 deletes the interim Connecting record (first connect) or clears the in-progress marker (Reconnect); no connection record is left in a half-confirmed state; FEAT-32.SPEC-001 shows the capability-down message and any pre-existing record keeps its prior status.
- **Nadia navigates away mid-hand-off and returns** -- The interim Connecting record (or the Reconnect's `handoff_started_at`) persists, so FEAT-32.SPEC-001 shows the Connecting state again until the capability reports an outcome.
- **Nadia initiates Reconnect while a status-report event from the previous connection attempt is still in flight** -- The new hand-off is a distinct attempt; FEAT-32.SPEC-003 applies whichever event is most recent by report time, so a stale event from the superseded attempt cannot overwrite the new hand-off's outcome.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-32.SPEC-001 (Payment Connection Screen) | Triggered by (inbound) | Connect and Reconnect taps initiate a hand-off through this spec |
| FEAT-32.SPEC-001 (Payment Connection Screen) | Affects (outbound) | Hand-off progress, the Error state, and the disclosure notice (with its Continue and Cancel outcomes) surface here |
| FEAT-32.SPEC-003 (Connection Status Sync) | Triggers (outbound) | Every inbound event above is routed to this automation for application |
| FEAT-32.SPEC-005 (Payment Connection Authorization & Validation Rules) | References (inbound) | The Nadia-only gate and one-account-per-freelancer limit are checked before a hand-off reaches this spec |
| FEAT-25 (Refund & Cancelled Project Handling) | Affects (outbound) | Reversal and chargeback notices are relayed here (XBR-21) |
| FEAT-09.SPEC-009 (Pay-Link Availability & No-Account Fallback Rule) | References (outbound) | Reads the connection status this spec ultimately produces, via FEAT-32.SPEC-003, to derive pay-link availability |
| FEAT-10.SPEC-003 (Card & Bank-Transfer Payment Processing) | References (outbound) | Depends on the connection's readiness and available methods before a payment can be submitted |

## Analytics and Success Signals

- **payment_account_connect_started** (attempt type: connect / reconnect) -- supports success-metrics.md: "Payment Readiness Before First Invoice"
- **payment_account_handoff_outcome_received** (outcome: ready / restricted / failed_abandoned) -- supports success-metrics.md: "Payment Readiness Before First Invoice"
- **payment_account_reversal_notice_relayed** (invoice reference) -- N/A -- no Stage 2 metric measures reversal frequency; retained so a relayed dispute is observable rather than silent, per XBR-21's evidentiary intent.
- **payment_account_degradation_shown** (condition: slow / down / rejected) -- N/A -- no Stage 2 metric measures capability trouble frequency directly; retained so the product's tolerance for capability trouble during the readiness-critical connect flow is observable.

## Acceptance Criteria

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

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 4 | 4 |
| Inbound Events | 4 | 4 |
| Degradation Paths | 3 (1 screen) | 3 |
| Consent and Disclosure | 3 | 3 |
| Edge Cases | 7 | 7 |



# Automation Spec: Connection Status Sync

## Overview

**Name:** Connection Status Sync
**ID:** FEAT-32.SPEC-003
**Type:** Automation
**Purpose:** Applies the processor-reported status (Connected, Needs attention, or a failed/abandoned attempt) and available payment methods to the Payment Account Connection record the instant it is reported.
**Parent Feature:** FEAT-32 -- Payment Account Connection

## Scope and Non-Goals

**In Scope:**
- Applying a readiness confirmation to the Payment Account Connection record
- Applying a restriction/needs-more-information report, including its specific reason
- Treating a hand-off that Nadia abandons through "Start over" (available once it is older than platform parameter: `payment-connect-handoff-timeout`) as an abandoned hand-off
- Leaving an existing record's status unchanged when a Reconnect hand-off fails or is abandoned, and removing the interim Connecting record when a first connect fails or is abandoned
- Treating a readiness report with zero available payment methods as Needs attention with a defined reason
- Resolving events that arrive out of order or twice for the same underlying change

**Non-Goals:**
- Initiating the hand-off itself, or defining what data is exchanged with the payment-processing capability -- owned by FEAT-32.SPEC-002 (Payment Account Connection & Status Reporting); this automation only consumes events that spec already receives.
- Removing the connection reference -- owned by FEAT-32.SPEC-004 (Disconnect Payment Account), which is triggered by Nadia's own explicit action, not by a processor report.
- Displaying the resulting status -- owned by FEAT-32.SPEC-001 (Payment Connection Screen), which reads this automation's output.
- Sending the confirmation or alert email -- owned by FEAT-32.SPEC-006 (Connection Status Notifications), which this automation triggers but does not compose.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Connection confirmed ready | FEAT-32.SPEC-002 (Payment Account Connection & Status Reporting) | Fires when the payment-processing capability reports the account is ready, following a Connect or Reconnect hand-off (an empty methods list is handled as a restriction, step 3) | processor_account_reference, list of currently available payment methods (may be empty), event report time |
| Connection restricted / more information requested | FEAT-32.SPEC-002 (Payment Account Connection & Status Reporting) | Fires when the capability restricts the account or asks for more information, at any point after a hand-off began, including after a prior Connected state | Specific attention reason (plain-language text), event report time |
| Hand-off fails or is abandoned mid-flow | FEAT-32.SPEC-002 (Payment Account Connection & Status Reporting) | Fires when a Connect or Reconnect attempt does not complete, technically or because Nadia exits before it finishes | Event report time; no reason data required for this outcome |
| Hand-off timed out -- Nadia starts over | FEAT-32.SPEC-001 (Payment Connection Screen) | Fires when Nadia confirms "Start over" while the screen shows Connecting, i.e. no outcome was reported within platform parameter: `payment-connect-handoff-timeout` of `handoff_started_at` | Confirmation time; the record's `handoff_started_at` |

## Processing Logic

1. Receive the reported event and its data from FEAT-32.SPEC-002, including the event's report time.
2. If no Payment Account Connection record exists, discard the event without creating one and stop (Edge Cases: disconnected connection). Otherwise, for a readiness or restriction event, compare its report time against the record's `last_event_reported_at` (set to the hand-off start time when a hand-off begins, and to each applied event's report time):
   - If the event's report time is **older than** `last_event_reported_at`, discard it as stale (Edge Cases: out-of-order arrival) and stop.
   - If the event's report time is **equal to** `last_event_reported_at`, it is an already-applied duplicate: discard it as a no-op and stop -- no field changes and no notification fires (step 6 is never reached). The single exception is a restriction event whose report time equals that of an applied readiness event while `status` is Connected: the restriction is applied (restriction takes precedence over readiness at the same instant, since a needs-attention account cannot be relied on for payment).
   - Failure/abandonment events are not compared by report time; they are applied only while a hand-off is in progress (status Connecting or `handoff_started_at` present), and are a no-op otherwise (step 5).
3. If the event reports readiness: check the reported list of available payment methods.
   - If the list contains at least one method: set `status` to Connected, confirm or set `processor_account_reference`, set `available_payment_methods` to the reported list, clear `handoff_started_at`, and set `last_event_reported_at` to the event's report time.
   - If the list is empty (zero methods): treat the event as a restriction. Set `status` to Needs attention with the product-defined reason "Your payment account has no payment methods turned on yet. Turn on card or bank transfer in your payment account, then reconnect.", confirm or set `processor_account_reference`, set `available_payment_methods` to none, clear `handoff_started_at`, and set `last_event_reported_at`. Because this reason is supplied by the product, step 4's missing-reason check does not apply to this path.
4. If the event reports a restriction: check that a specific reason accompanies it. If no reason is present, treat the event as malformed, discard it, and leave the record unchanged (Edge Cases). If a reason is present, set `status` to Needs attention (retaining the reason for display), set `available_payment_methods` to none -- a needs-attention account cannot be relied on for payment -- clear `handoff_started_at`, and set `last_event_reported_at` to the event's report time.
5. If the event reports a failed or abandoned hand-off: if no hand-off is in progress, apply nothing and stop. Otherwise, if the record's `status` is Connecting (a first connect, with no earlier known-good state), delete the interim record so the screen returns to Empty and no half-created record remains; if the record has any other status (a Reconnect), clear only `handoff_started_at` and leave `status`, `processor_account_reference`, and `available_payment_methods` exactly as they were before the attempt.
5a. If the trigger is Nadia's confirmed "Start over" (FEAT-32.SPEC-001): if no hand-off is in progress (an outcome was applied first, or the record no longer exists), apply nothing and stop. If a hand-off is in progress but `handoff_started_at` is **not yet older than** platform parameter: `payment-connect-handoff-timeout` (the threshold was not actually reached), reject the request as a no-op and leave the record unchanged. Otherwise treat it exactly as an abandoned hand-off and apply step 5 (first connect: delete the interim record; Reconnect: clear only `handoff_started_at`). A processor event reported later for the abandoned hand-off is handled by step 2 (discarded for a deleted record; for a Reconnect, ordered by report time against `last_event_reported_at`, and applied only if it reports a genuinely later state).
6. When a change is applied (step 3 or 4, including the zero-methods path), trigger FEAT-32.SPEC-006 (Connection Status Notifications) for the resulting outcome. Steps that discard the event or apply no status change trigger nothing.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Connected | A readiness-confirmed event with at least one available method is applied | `status` set to Connected; `processor_account_reference` confirmed/set; `available_payment_methods` set from the report; `handoff_started_at` cleared; `last_event_reported_at` advanced | FEAT-32.SPEC-001 shows "Ready to accept payments" and the available methods | FEAT-32.SPEC-001, FEAT-32.SPEC-006, FEAT-09 (pay-link availability) |
| Needs attention | A restriction event with a stated reason is applied | `status` set to Needs attention (carrying the reason); `available_payment_methods` cleared to none; `handoff_started_at` cleared; `last_event_reported_at` advanced | FEAT-32.SPEC-001 shows "Needs attention" with the reason verbatim, plus Reconnect, Disconnect, and Contact support | FEAT-32.SPEC-001, FEAT-32.SPEC-006, FEAT-09 and FEAT-10 (pay-link marked unavailable) |
| Needs attention (zero methods) | A readiness-confirmed event reports zero available payment methods | Same data changes as Needs attention, with the product-defined reason "Your payment account has no payment methods turned on yet. Turn on card or bank transfer in your payment account, then reconnect." and `processor_account_reference` confirmed/set | FEAT-32.SPEC-001 shows "Needs attention" with that reason; FEAT-32.SPEC-006's alert email carries the same text as `{attention_reason}` | FEAT-32.SPEC-001, FEAT-32.SPEC-006, FEAT-09 and FEAT-10 |
| Hand-off failed or abandoned | A failure/abandonment event arrives while a hand-off is in progress | First connect (`status` Connecting): the interim record is deleted, returning Nadia to Empty. Reconnect (any other status): only `handoff_started_at` is cleared; `status`, `processor_account_reference`, and `available_payment_methods` retain their prior values. With no hand-off in progress: no change | FEAT-32.SPEC-001 shows its Error state with Retry over the previous state (Empty for a first connect); no false Connected or Needs-attention flash | FEAT-32.SPEC-001 |
| Hand-off timed out, Nadia starts over | Nadia confirms "Start over" on FEAT-32.SPEC-001 and the hand-off is older than platform parameter: `payment-connect-handoff-timeout` with no outcome applied | Same data changes as Hand-off failed or abandoned: first connect deletes the interim record; Reconnect clears only `handoff_started_at`. If no hand-off is in progress, or the threshold has not elapsed: no change | FEAT-32.SPEC-001 shows its Error banner over the previous state (Empty for a first connect); no false Connected or Needs-attention flash. A no-op request leaves the screen on the state the record holds | FEAT-32.SPEC-001 |
| Malformed restriction event discarded | A restriction event arrives with no stated reason | None -- record unchanged | No user-visible feedback from this event; the previous state stands | FEAT-32.SPEC-001 (unaffected) |
| Stale or duplicate event discarded | An event's report time is older than, or equal to (except restriction-over-readiness), the record's `last_event_reported_at` | None -- no field changes and no notification fires | No user-visible feedback | FEAT-32.SPEC-001 (unaffected), FEAT-32.SPEC-006 (not triggered) |
| Automation failure | This automation cannot durably apply a genuinely valid, current event (an internal processing error) | None persisted on this attempt; the event is not silently dropped -- it is retried | FEAT-32.SPEC-001 continues showing the last durably-applied status; no false update is shown | FEAT-32.SPEC-001 |

## Data Model

**Reads:** Payment Account Connection -- the record's current `status`, `handoff_started_at`, and `last_event_reported_at` (the report time of the most recently applied event, or the hand-off start), to resolve ordering and duplicates.
**Creates:** None -- the record is first created with `status` Connecting by FEAT-32.SPEC-002 when Nadia confirms Continue on FEAT-32.SPEC-001's consent notice; this automation never creates the record itself.
**Updates:** Payment Account Connection -- `status`, `processor_account_reference`, `available_payment_methods`, `handoff_started_at`, `last_event_reported_at`, as defined in Outcome Definitions.
**Deletes:** Payment Account Connection -- only the interim Connecting record of a first connect whose hand-off failed or was abandoned (step 5), so no half-created record remains.

## Business Rules

- Processor-reported status is authoritative over any client-visible expectation or Nadia's own prior screen state (feature-dependency-map.md, Entity: Payment Account Connection, Contention).
- A failed or abandoned hand-off never overwrites the last known-good status (product-features.md, States field); the only state it removes is the interim Connecting record of a first connect, which never held a known-good status.
- An event whose report time equals the `last_event_reported_at` already applied is a duplicate and is a no-op (no field change, no email); the only equal-time exception is a restriction event over an applied readiness event, where the restriction wins.
- `available_payment_methods` is fully derived from the capability's latest report and is never set by direct user input.
- Events are applied strictly in the order the capability reports them occurring (event time), never in the order they happen to arrive.
- XBR-19: this automation's output is the sole source FEAT-09 and FEAT-10 read to derive pay-link availability -- this automation itself renders no pay-link copy.

## Edge Cases

- **Concurrent trigger firing (a readiness event and a restriction event both fire for the same connection at effectively the same time)** -- Both are applied in event-time order, not arrival order; whichever genuinely occurred later determines the final state, and the earlier one is superseded rather than lost (it was still validly applied first, then overwritten by the later, true state).
- **Trigger fires while a previous run is still in flight** -- A second event for the same connection record queues behind the first rather than writing concurrently; each is applied in turn, in event-time order, so the record never reflects a half-applied state.
- **A restriction event arrives with no stated reason** -- Treated as malformed and discarded; the record's previous status stands, since the product's Specific-reason display pattern (feature-overview.md, Shared UI Patterns) never falls back to a generic "something went wrong."
- **A processor-reported event arrives for a connection Nadia has since disconnected** -- The record no longer exists (FEAT-32.SPEC-004 hard-deletes it), so the event has nothing to update and is discarded without recreating a record.
- **Reconnect is initiated while a stale event from a prior, superseded attempt is still in flight** -- FEAT-32.SPEC-002 advances `last_event_reported_at` to the new hand-off's start time, so the stale event's older report time fails step 2's ordering check and is discarded; the new hand-off's outcome is unaffected.
- **The same event is delivered twice, or two events carry an identical report time** -- The second delivery has a report time equal to `last_event_reported_at`, so step 2 discards it as an already-applied duplicate: nothing changes and no second confirmation or alert email fires. If the two events are a readiness and a restriction with an identical report time, the restriction is applied and the readiness is discarded.
- **The processor never reports any outcome for a hand-off** -- The record stays Connecting (or Needs attention with `handoff_started_at` set, for a Reconnect) until Nadia confirms "Start over" on FEAT-32.SPEC-001 once the hand-off is older than platform parameter: `payment-connect-handoff-timeout`; step 5a then applies the abandoned-hand-off handling, so no connection can remain stuck indefinitely.
- **Nadia confirms "Start over" while the processor's outcome is being applied** -- Both are serialised behind the in-flight run; if the outcome is applied first, `handoff_started_at` is already cleared and step 5a stops with no change; if "Start over" is applied first, a later readiness or restriction event is handled by step 2 (discarded for a deleted first-connect record).
- **A readiness event reports zero available methods** -- Step 3 applies Needs attention with the product-defined reason rather than Connected, so Nadia sees a specific, actionable reason and never a "Connected" state with nothing payable.
- **This automation's own write fails on first attempt** -- Retried automatically at platform parameter: `connection-status-apply-retry-interval` intervals until it succeeds; there is no maximum retry count, since a permanently unapplied status update could leave Nadia believing she is ready to accept payments when she is not, or the reverse.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-32.SPEC-002 (Payment Account Connection & Status Reporting) | Triggered by (inbound) | Every readiness, restriction, and failure/abandonment event fires this automation |
| FEAT-32.SPEC-001 (Payment Connection Screen) | Triggered by (inbound) | Nadia's confirmed "Start over" on a Connecting hand-off older than the timeout fires the abandoned-hand-off handling |
| FEAT-32.SPEC-001 (Payment Connection Screen) | Affects (outbound) | Displays the applied status, reason, and available methods |
| FEAT-32.SPEC-006 (Connection Status Notifications) | Triggers (outbound) | The Connected and Needs attention outcomes fire this notification |
| FEAT-09 (Invoice Generation & Sending) | Affects (outbound) | Derives pay-link availability from this automation's applied status (via FEAT-09.SPEC-009) |
| FEAT-10 (Invoice Payment Processing) | Affects (outbound) | Reads the applied status and available methods before allowing a payment attempt |

## Analytics and Success Signals

- **payment_account_connected** (methods_count) -- supports success-metrics.md: "Payment Readiness Before First Invoice"
- **payment_account_needs_attention** (has_prior_connection: yes/no; reason_source: processor / zero_methods) -- supports success-metrics.md: "Payment Readiness Before First Invoice"
- **connection_status_malformed_event_discarded** (event type: restriction_no_reason) -- N/A -- no Stage 2 metric measures malformed-event frequency; retained so a discarded processor report is observable rather than silently lost.
- **connection_handoff_start_over_applied** (attempt_type: connect/reconnect; result: applied / rejected_not_timed_out / no_handoff) -- N/A -- no Stage 2 metric measures stalled hand-off recovery; retained so timed-out hand-offs are observable.
- **connection_status_apply_failed** (retry_count) -- N/A -- no Stage 2 metric measures this automation's own reliability directly; retained for observability of the correctness-critical retry guarantee.

## Acceptance Criteria

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

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 4 | 4 |
| Outcome Paths | 8 | 8 |
| Business Rules | 6 | 6 |
| Edge Cases | 10 | 10 |



# Automation Spec: Disconnect Payment Account

## Overview

**Name:** Disconnect Payment Account
**ID:** FEAT-32.SPEC-004
**Type:** Automation
**Purpose:** Removes Nadia's connection reference on her explicit disconnect action, without cancelling any payment already submitted to the processor.
**Parent Feature:** FEAT-32 -- Payment Account Connection

## Scope and Non-Goals

**In Scope:**
- Removing the Payment Account Connection record (hard delete, no restore path) when Nadia confirms disconnect from FEAT-32.SPEC-001
- Removing the same record when account deletion (FEAT-24) reaches its payment-disconnect step
- Leaving any payment already submitted to the processor unaffected

**Non-Goals:**
- Showing the disconnect warning and collecting Nadia's confirmation -- owned by FEAT-32.SPEC-001 (Payment Connection Screen); this automation only executes once that confirmation, gated by FEAT-32.SPEC-005, is already given.
- The account-deletion warning sequence itself (unpaid invoices, pending approvals) -- owned entirely by FEAT-24 (Data Export & Account Deletion); this automation only performs the payment-account-specific removal step within that sequence.
- Cancelling or reversing a payment already submitted to the processor -- excluded per feature-overview.md's Side-Effect Inventory: an in-flight payment is not cancelled by a disconnect; that payment's own outcome is owned entirely by FEAT-10.SPEC-003.
- Restoring a disconnected account without repeating the connect flow -- excluded per feature-overview.md's Non-Goals: the record holds no historical value beyond the live reference, so reconnecting always runs FEAT-32.SPEC-002's Create operation again.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nadia confirms Disconnect | FEAT-32.SPEC-001 (Payment Connection Screen) | Fires only after FEAT-32.SPEC-005's authorization passes (Nadia, her own account) and she has confirmed the explicit disconnect warning | The current `processor_account_reference` to be removed |
| Account deletion reaches the payment-disconnect step | FEAT-24.SPEC-004 (Account Deletion Processing, FEAT-24) | Fires when FEAT-24's own warn-then-remove sequence reaches the payment-account step, which is that cascade's commit point (FEAT-24.SPEC-004 step 5; XBR-33); no separate warning re-confirmation is required here, since FEAT-24's own sequence has already warned about disconnecting the payment account | The freelancer account's existing Payment Account Connection record, if any |

## Processing Logic

1. Receive the disconnect instruction, noting its source: Nadia's own confirmed action (FEAT-32.SPEC-001) or account-deletion processing (FEAT-24.SPEC-004).
2. Check whether a Payment Account Connection record currently exists for this freelancer account. If none exists, stop -- there is nothing to remove (Edge Cases: idempotent no-op).
3. Set `status` to Disconnected as the record's terminal value, then remove the record entirely: `processor_account_reference`, `status`, and `available_payment_methods` are all deleted together in the same operation -- Disconnected is never a value a live, readable record persists in; it names the outcome of this step, not a lingering state. There is no partial removal and no retained history beyond this point.
4. Any payment already submitted to the processor before this step is left entirely unaffected; this automation makes no request to the payment-processing capability at all.
5. Confirm completion to the triggering source: FEAT-32.SPEC-001 (for Nadia's own action) or FEAT-24.SPEC-004 (for account deletion), so each can proceed with its own next step.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Disconnected (Nadia's own action) | Nadia's confirmed disconnect is processed and a record existed | Payment Account Connection record deleted in full | FEAT-32.SPEC-001 returns to the Empty state with "Payment account disconnected." | FEAT-32.SPEC-001, FEAT-09 (pay-link fallback), FEAT-10 (pay-screen fallback) |
| Disconnected (account deletion) | FEAT-24's sequence reaches this step and a record existed | Payment Account Connection record deleted in full | No independent feedback from this automation -- FEAT-24 owns the account-deletion confirmation the freelancer sees | FEAT-24 |
| No-action (already disconnected) | No Payment Account Connection record exists when either trigger fires | None | FEAT-32.SPEC-001 (if the trigger was Nadia's) already shows the Empty state -- nothing changes; FEAT-24 proceeds with its sequence unaffected | FEAT-32.SPEC-001, FEAT-24 |
| Failure | The deletion write itself fails (an internal processing error) | None -- record retained unchanged | FEAT-32.SPEC-001 shows "We couldn't disconnect your account. Try again." and the Disconnect action remains available for Nadia to retry; for the account-deletion trigger, FEAT-24's own sequence is informed the step did not complete and it retries per its own process | FEAT-32.SPEC-001, FEAT-24 |

## Data Model

**Reads:** Payment Account Connection -- checks for the record's existence before attempting removal.
**Creates:** None.
**Updates:** None.
**Deletes:** Payment Account Connection -- the entire record (`processor_account_reference`, `status`, `available_payment_methods`) is removed together; no restore path exists, per the entity's hard-delete lifecycle (feature-dependency-map.md, Entity: Payment Account Connection).

## Business Rules

- Disconnect never cancels a payment already submitted to the processor (feature-overview.md, Side-Effect Inventory); this automation makes no request to the payment-processing capability.
- Once removed, any pay link opened afterward shows the "temporarily unavailable" fallback owned by FEAT-09.SPEC-009 and FEAT-10 (XBR-19) -- this automation itself renders no such copy.
- FEAT-32.SPEC-005's contention rule (processor-authoritative, in-flight-payment protection) governs the fact that disconnect is never blocked by an in-flight payment; this automation always proceeds once triggered.
- Reconnecting after a disconnect always runs FEAT-32.SPEC-002's Create operation again -- there is no resume or restore of the removed record.
- XBR-33: account deletion's own warn-then-remove sequence owns the warning shown for this step when triggered by FEAT-24; this automation performs the removal itself without re-warning.

## Edge Cases

- **Concurrent trigger firing (Nadia confirms Disconnect at the same moment FEAT-24's sequence reaches its own disconnect step, e.g. she starts account deletion right after disconnecting)** -- Whichever trigger's removal reaches step 2 first finds the record and removes it; the second trigger then finds no record (per step 2) and takes the no-action outcome. Neither trigger errors on finding nothing to remove.
- **Trigger fires while a previous run is still in flight (Nadia double-taps Disconnect, or the confirmation dialog is submitted twice)** -- FEAT-32.SPEC-001 disables the Disconnect button while the first request is processing, preventing a genuine second in-flight run from the same source; if a second request somehow reaches this automation, it is the no-action outcome described above (idempotent).
- **A disconnect is issued while a client's payment is already submitted to the processor** -- Per FEAT-32.SPEC-005's contention rule, the disconnect is not blocked and proceeds exactly as in the standard outcome; the in-flight payment continues to its own outcome (FEAT-10.SPEC-003) entirely independent of this removal.
- **A FEAT-32.SPEC-003 status-report event is in flight for this connection at the same moment it is disconnected** -- The disconnect's removal takes precedence: once the record is deleted, the status-report event finds nothing to update and is discarded (FEAT-32.SPEC-003's own edge-case handling), rather than recreating a partial record.
- **Account deletion is later cancelled or reverted before finalizing (a scenario FEAT-24 itself would define)** -- Outside this spec's scope: this automation performs an unconditional removal once FEAT-24's own sequence reaches this specific step; any decision to not reach that step at all is entirely FEAT-24's to make before triggering this automation.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-32.SPEC-001 (Payment Connection Screen) | Triggered by (inbound) | Nadia's confirmed disconnect (after the warning dialog) fires this automation |
| FEAT-32.SPEC-001 (Payment Connection Screen) | Affects (outbound) | Returns to the Empty state on success, or shows the failure message |
| FEAT-32.SPEC-005 (Payment Connection Authorization & Validation Rules) | References (inbound) | Authorization and the in-flight-payment contention rule gate and shape this automation's behavior |
| FEAT-24 (Data Export & Account Deletion) | Triggered by (inbound) | Account deletion's warn-then-remove sequence fires this automation as its payment-disconnect step (XBR-33) |
| FEAT-32.SPEC-003 (Connection Status Sync) | References (outbound) | A status-report event arriving after this automation's removal finds no record, per that spec's own edge-case handling |
| FEAT-09 (Invoice Generation & Sending) | Affects (outbound) | Open invoices' pay links switch to the fallback experience once this automation removes the connection |
| FEAT-10 (Invoice Payment Processing) | Affects (outbound) | The pay screen switches to the "temporarily unavailable" fallback once this automation removes the connection |

## Analytics and Success Signals

- **payment_account_disconnected** (trigger_source: nadia_manual / account_deletion) -- N/A -- no Stage 2 metric measures disconnect frequency directly; retained per product-features.md's named signal for observability of connection churn against "Payment Readiness Before First Invoice."
- **payment_account_disconnect_failed** (trigger_source, retry_attempted: yes/no) -- N/A -- no Stage 2 metric measures this automation's own failure rate; retained as a standard reliability signal.

## Acceptance Criteria

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

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 | 2 |
| Outcome Paths | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |



# Logic/Rule Spec: Payment Connection Authorization & Validation Rules

## Overview

**Name:** Payment Connection Authorization & Validation Rules
**ID:** FEAT-32.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs the one-account-per-freelancer limit, who may connect/reconnect/disconnect versus view only, the disconnect warning, and the processor-authoritative contention rule that protects an in-flight payment from a concurrent disconnect.
**Parent Feature:** FEAT-32 -- Payment Account Connection
**Governed Entity:** Payment Account Connection

## Scope and Non-Goals

**In Scope:**
- Field-level rules for the Payment Account Connection record's fields
- The one-connected-account-per-freelancer limit
- Authorization for Connect, View, Reconnect, Disconnect, and Start over (the exit from a stalled hand-off) for every role in the Access Matrix
- The exact disconnect-warning wording and when it gates the disconnect action
- The processor-authoritative contention rule protecting an in-flight payment from a concurrent disconnect

**Non-Goals:**
- Executing the connect/reconnect hand-off -- owned by FEAT-32.SPEC-002 (Payment Account Connection & Status Reporting); this spec defines only who may initiate it and under what condition.
- Applying a reported status change -- owned by FEAT-32.SPEC-003 (Connection Status Sync); this spec governs authorization and the contention rule, not the status-application logic itself.
- Executing the removal -- owned by FEAT-32.SPEC-004 (Disconnect Payment Account); this spec defines the gate (authorization and the acknowledged warning) that must pass before that automation runs.
- The exact client-facing "temporarily unavailable" pay-link copy -- owned by FEAT-09.SPEC-009 and FEAT-10; this spec states the underlying rule (an in-flight payment is protected and a disconnected account's pay links stop working), not the rendered wording on those screens.

## Governed Entity

**Entity:** Payment Account Connection
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| processor_account_reference | text (reference) | A reference to the freelancer's linked processor account; never card numbers or bank credentials; present only when a connection exists |
| status | enum | Not connected (represented by the absence of a record), Connecting (interim -- a first-connect hand-off is in progress and no readiness has yet been reported), Connected ("Ready to accept payments"), Needs attention (with the specific reason), or Disconnected (terminal outcome name only; never persisted) |
| available_payment_methods | derived (enum set) | Which of card and/or bank transfer are currently available on this freelancer's pay links |
| handoff_started_at | timestamp | When Nadia's latest Continue started a Connect or Reconnect hand-off; present only while that hand-off is in progress (also the marker that drives the Connecting screen state during a Reconnect, when `status` remains Needs attention); cleared when an outcome is applied |
| last_event_reported_at | timestamp | The report time of the most recently applied processor event, or the hand-off start time when a hand-off begins; used by FEAT-32.SPEC-003 to discard stale and duplicate events |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-32.SPEC-001 | Payment Connection Screen | Authorization on screen entry (which actions render for the viewer) and on each action attempt; the disconnect-warning dialog is shown before Disconnect proceeds |
| FEAT-32.SPEC-002 | Payment Account Connection & Status Reporting | Authorization check (Nadia, her own account) and the one-account limit check before a hand-off is initiated |
| FEAT-32.SPEC-003 | Connection Status Sync | Applies the processor-authoritative rule when reconciling a reported status against the current record |
| FEAT-32.SPEC-004 | Disconnect Payment Account | Executes only once the disconnect-warning acknowledgment and authorization gate here have passed; applies the in-flight-payment contention rule (the disconnect is never blocked by it) |
| FEAT-31.SPEC-002 (Operator Support Session Console, FEAT-31) | Dana's read-only connection view | Enforces the status-only visibility rule -- the processor account reference and any credential-adjacent detail are never rendered in her view |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| processor_account_reference | No validation beyond data type -- entirely supplied by the payment-processing capability once a hand-off confirms readiness; never entered or edited directly by any user | Always | -- | -- | -- |
| status | Must be one of Not connected (no record), Connecting, Connected, Needs attention, Disconnected (terminal, never persisted); Connecting is set only by FEAT-32.SPEC-002 when it creates the record for a first connect, and Connected/Needs attention only by FEAT-32.SPEC-003 (status reports); the record is removed by FEAT-32.SPEC-004 (disconnect) or by FEAT-32.SPEC-003 (failed first connect) -- never directly editable by any user | Always | On every hand-off start, processor-reported event, or disconnect action | N/A -- not a user-facing field; there is no direct-edit path to produce an invalid value | No (system-enforced, not user validation) |
| handoff_started_at | Set only by FEAT-32.SPEC-002 when Nadia confirms Continue in the consent notice; cleared only by FEAT-32.SPEC-003 when it applies any outcome; never entered by a user | Always | On Continue and on every applied outcome | N/A -- system-set | No (system-enforced) |
| last_event_reported_at | Set only by FEAT-32.SPEC-002 (to the hand-off start time) and FEAT-32.SPEC-003 (to an applied event's report time); never moves backward | Always | On every hand-off start and applied event | N/A -- system-set | No (system-enforced) |
| available_payment_methods | No user-facing validation -- fully derived from the payment-processing capability's latest report; never set by direct user input | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Needs attention implies no available methods | status, available_payment_methods | When status is Needs attention, available_payment_methods is set to none, regardless of what was available immediately before the restriction was reported | N/A -- system-derived, not a user-facing validation failure |
| Connected implies at least one available method | status, available_payment_methods | A readiness-confirmed event that reports zero available methods is treated the same as a restriction: FEAT-32.SPEC-003 (Processing Logic step 3) applies Needs attention with `available_payment_methods` none and the product-defined reason "Your payment account has no payment methods turned on yet. Turn on card or bank transfer in your payment account, then reconnect.", since "Connected" with nothing payable is not a meaningful ready state and the reason is never empty | N/A -- system-derived, not a user-facing validation failure; the reason text above is what Nadia sees on FEAT-32.SPEC-001 and in FEAT-32.SPEC-006's alert email |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Connect (create) | Nadia (Freelancer) | Always, provided no Payment Account Connection record currently exists for her account (the one-account-per-freelancer limit) | -- |
| Connect (create) | Nadia (Freelancer) | When a record already exists in any status (including interim Connecting) | Connect is not shown at all -- FEAT-32.SPEC-001 offers Reconnect and/or Disconnect as the status allows (none while Connecting); this is a structural absence, not a denial with an error message |
| Connect (create) | Owen (Client Primary Contact), Priya (Client Reviewer Contact) | Never | The action does not exist anywhere in the client portal -- no Payment Account settings surface is reachable by either role |
| Connect (create) | Dana (Support Operator) | Never | The Connect control is never shown inside Dana's read-only support session (FEAT-31); she has no path to initiate a connection |
| View status (Connected / Needs attention / Not connected, and available payment methods) | Nadia (Freelancer) | Always, her own account | -- |
| View status | Owen (Client Primary Contact), Priya (Client Reviewer Contact) | Never, directly | Neither role has any view of this entity; each experiences its effect only indirectly, as pay-link availability on their own company's invoices (FEAT-09), which is outside this entity entirely |
| View status (status field only) | Dana (Support Operator) | Always, inside a logged support session (FEAT-31), status only | -- |
| View processor_account_reference or any credential-adjacent detail | Dana (Support Operator) | Never | Dana's support-session view renders status only; the reference is never displayed to her, per feature-dependency-map.md's Data Sensitivity note ("the operator sees status only") |
| Reconnect | Nadia (Freelancer) | Always, when the record exists in Needs attention status. Reconnect is never offered when no record exists (post-disconnect or never-connected) -- that case offers Connect instead -- nor while Connecting or Connected | Not shown outside Needs attention -- a structural absence on FEAT-32.SPEC-001, not an error message |
| Reconnect | Owen, Priya, Dana | Never | Same as Connect -- no surface exists for these roles |
| Disconnect | Nadia (Freelancer) | Her own account, when the record is in Connected or Needs attention status, provided she has acknowledged the explicit disconnect warning (Business Rules, below). Not offered while Connecting or when no record exists | If she dismisses or cancels the warning instead of confirming, the disconnect does not proceed and the connection is unchanged; outside Connected and Needs attention the control is not shown (structural absence, no error message) |
| Disconnect | Owen, Priya, Dana | Never | Same as Connect and Reconnect -- no disconnect surface exists for these roles |
| Start over (abandon a stalled hand-off) | Nadia (Freelancer) | Her own account, only while a hand-off is in progress (`status` Connecting, or `handoff_started_at` present) and `handoff_started_at` is older than platform parameter: `payment-connect-handoff-timeout`; she must confirm the "Start over" dialog. Allowed even though Connect, Reconnect, and Disconnect are all hidden while Connecting | Before the threshold has elapsed, or with no hand-off in progress, the control is not shown (structural absence, no error message); if the request still arrives (for example from a stale tab) FEAT-32.SPEC-003 makes no change and the screen shows the state the record holds. Cancelling the dialog changes nothing |
| Start over | Owen, Priya, Dana | Never | Same as Connect -- no surface exists for these roles |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| status | The absence of a Payment Account Connection record represents the Not connected state; for a first connect, FEAT-32.SPEC-002 creates the record with `status` Connecting when Nadia confirms Continue; thereafter `status` is set only by FEAT-32.SPEC-003 (reported events) and the record is removed by FEAT-32.SPEC-003 (failed or abandoned first connect) or FEAT-32.SPEC-004 (disconnect) | On every hand-off, status report, or disconnect | No |
| handoff_started_at | Set to the Continue time by FEAT-32.SPEC-002 on every Connect or Reconnect hand-off; cleared by FEAT-32.SPEC-003 when any outcome is applied | On hand-off start and on every applied outcome | No |
| last_event_reported_at | Initialised to the hand-off start time by FEAT-32.SPEC-002; advanced by FEAT-32.SPEC-003 to each applied event's report time | On hand-off start and on every applied event | No |
| available_payment_methods | Derived entirely from the payment-processing capability's most recently applied report (FEAT-32.SPEC-003); cleared to none whenever status is Needs attention | On every applied status update | No |
| processor_account_reference | Supplied by the payment-processing capability once a hand-off's readiness is confirmed; removed entirely on disconnect | On Connect/Reconnect confirmation; removed on Disconnect | No |

## Business Rules

- One connected payment account per freelancer at a time (product-features.md, Validation & Limits) -- enforced structurally: FEAT-32.SPEC-002 never initiates a second Create while a record exists, and FEAT-32.SPEC-001 only ever offers Connect when none exists.
- **Connecting is an interim, persisted status:** for a first connect, FEAT-32.SPEC-002 creates the record with `status` Connecting (no `processor_account_reference` yet) when Nadia confirms Continue; this is what lets the screen show Connecting after she navigates away and returns. FEAT-32.SPEC-003 replaces it with Connected or Needs attention, or deletes the record if the first attempt fails or is abandoned, so no half-created record survives a failed first attempt. For a Reconnect the existing record keeps its `status` and only `handoff_started_at` is set.
- XBR-19: with no connected account, invoices still issue with fallback instructions for paying Nadia directly; a Needs attention connection shows the pay screen's "temporarily unavailable" fallback; disconnecting warns that open invoices lose their pay links until reconnection.
- **Disconnect Warning:** Before a disconnect proceeds, Nadia is shown the exact dialog: "If you disconnect, your open invoices will lose their pay links until you connect a payment account again. Any payment already submitted to your processor will not be affected." with "Disconnect" and "Cancel" options. Disconnect proceeds only on "Disconnect"; "Cancel" leaves the connection unchanged.
- **Processor-authoritative contention rule:** A disconnect issued while a client's payment is already submitted to the processor is never blocked by that in-flight payment, and the in-flight payment is never cancelled by the disconnect (feature-dependency-map.md, Entity: Payment Account Connection, Contention). Once the disconnect completes, any pay link opened afterward reflects the now-removed connection, per the fallback rule FEAT-09.SPEC-009 and FEAT-10 own (XBR-19); this spec defines the underlying protection, not the rendered pay-screen copy.
- **Stalled hand-off exit:** Connecting is never a dead end. Once `handoff_started_at` is older than platform parameter: `payment-connect-handoff-timeout`, Nadia alone may confirm "Start over" on FEAT-32.SPEC-001; FEAT-32.SPEC-003 then applies its abandoned-hand-off handling (first connect: interim record deleted; Reconnect: only `handoff_started_at` cleared). Start over is the only action allowed while Connecting, and only past the threshold; Connect, Reconnect, and Disconnect stay unavailable in that status.
- Reconnect always re-runs the full Connect hand-off (FEAT-32.SPEC-002) rather than resuming a partial or prior connection -- there is no partial-state resume, consistent with the entity's hard-delete-on-disconnect lifecycle.
- Processor-reported status is authoritative over any other party's expectation of the connection's state (feature-dependency-map.md, Contention) -- applied by FEAT-32.SPEC-003.

## Edge Cases

- **Nadia attempts to disconnect twice in rapid succession (double-tap)** -- The second attempt, once the first has already removed the reference, finds nothing left to remove and is a no-op (FEAT-32.SPEC-004's idempotent handling); no error is shown for the second attempt.
- **The one-account limit at its exact boundary** -- Nadia has zero records: Connect is offered (and Reconnect is not). Nadia has exactly one record, in any status: Connect is never offered, only Reconnect and/or Disconnect as the status allows (neither while Connecting). No value between zero and one is possible, since the entity carries no count field and no list view exists (product-features.md, Validation & Limits).
- **Dana's read-only support session is open on the connection status at the exact moment Nadia disconnects** -- Dana's session reflects the new state (Not connected) on her next refresh; her session never continues showing a reference or status for a record that no longer exists.
- **A disconnect is confirmed while a client's payment is mid-submission to the processor** -- The in-flight payment proceeds to its own outcome entirely independent of the disconnect (FEAT-10.SPEC-003); the connection reference is removed regardless, and any pay link opened even moments later reflects the removed connection.
- **Nadia cancels the disconnect warning dialog** -- No data changes; the connection remains exactly as it was, and FEAT-32.SPEC-004 is never triggered.
- **A restriction event is applied (Needs attention) while Nadia is mid-way through confirming a disconnect** -- The disconnect, once confirmed, still proceeds and removes the record regardless of the just-applied Needs attention status; her own explicit disconnect is not blocked by an intervening status change.

## Acceptance Criteria

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

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 14 | 14 |
| Defaults/Derivations | 5 | 5 |
| Business Rules | 8 | 8 |
| Edge Cases | 6 | 6 |



# Notification Spec: Connection Status Notifications

## Overview

**Name:** Connection Status Notifications
**ID:** FEAT-32.SPEC-006
**Type:** Notification
**Purpose:** Sends Nadia a confirmation email when her account connects and an alert email when the connection breaks or needs attention, so she learns about a change to her ability to accept payments even when she is away from the product.
**Parent Feature:** FEAT-32 -- Payment Account Connection

## Scope and Non-Goals

**In Scope:**
- The confirmation email sent when the connection reaches Connected
- The alert email sent when the connection reaches Needs attention
- Delivery, retry, and expiry behavior for both emails

**Non-Goals:**
- Deciding when the connection reaches Connected or Needs attention -- owned entirely by FEAT-32.SPEC-003 (Connection Status Sync); this spec begins where that automation's trigger fires.
- Notifying anyone about a failed or abandoned hand-off -- the Brief's Side-Effect Inventory routes that outcome to FEAT-32.SPEC-001's inline Error state, not to an email; a failed attempt that leaves the prior state intact is not the record-worthy change these two emails exist to announce.
- Notifying anyone about a disconnect -- product-features.md's Communications field for this feature names only the connected and needs-attention events as triggering email; Nadia performed the disconnect herself on FEAT-32.SPEC-001 and sees its confirmation there directly, with no separate email needed to tell her what she just did.
- Notifying Owen or Priya -- neither role has any access to this entity (Access Matrix: None for both); a connection-status email carries content about Nadia's own financial-account readiness that no client contact is entitled to see.
- In-app notification center delivery -- product-features.md's Communications field names only the email channel for this feature; an in-app surface is owned separately by In-App Notification Center (FEAT-29, Later phase).

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, to Nadia, when the connection reaches Connected or Needs attention | Nadia works from a laptop or desktop throughout her day and is not necessarily inside the product at the moment the payment-processing capability reports a change (user-persona.md, Behavioral Context); a needs-attention change in particular can silently block every client's ability to pay her until she notices, so email reaches her even when she is away |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Connection reaches Connected | FEAT-32.SPEC-003 (Connection Status Sync) | Fires when FEAT-32.SPEC-003 applies the Connected outcome, whether from an initial connect or a reconnect | Freelancer Account name and sign-in email |
| Connection reaches Needs attention | FEAT-32.SPEC-003 (Connection Status Sync) | Fires when FEAT-32.SPEC-003 applies the Needs attention outcome, whether degrading from Connected or failing during an initial connect | Freelancer Account name and sign-in email; the attention reason (non-empty in every case: the processor's specific reason verbatim, or, when FEAT-32.SPEC-003 converted a zero-methods readiness report into Needs attention, that spec's product-defined reason) |

## Audience and Preferences

**Recipients:** Nadia (the Freelancer) only. The Access Matrix entitles only Nadia to any content about this entity; Owen and Priya have None access, and Dana's read-only support-session view of connection status (FEAT-31) is a view surface, not a notification recipient path.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| N/A -- always sent | -- | Always on | -- (both are transactional record emails, not optional notifications) |

Both emails are transactional and core to the record (XBR-30): a change to whether Nadia can accept payment is not something she can switch off, since it directly affects the "get paid faster" promise (BRIEF.md, Experience narrative) and the Payment Readiness Before First Invoice metric. Neither can be turned off through her notification preferences.

**Quiet Hours:** N/A -- the product defines quiet hours for optional, non-transactional notifications only (XBR-30); both emails are transactional and send immediately regardless of the time of day, consistent with how quickly a needs-attention state can affect a client trying to pay.

## Content Definition

**Email (Connected):**
- **Subject:** Your payment account is connected
- **Body:**
  Hi {nadia_first_name},

  Your payment account is connected and ready to accept payments on your invoices.
- **CTA (button):** View payment settings -- deep-links to FEAT-32.SPEC-001 (Payment Connection Screen)

**Email (Needs attention):**
- **Subject:** Action needed: your payment account needs attention
- **Body:**
  Hi {nadia_first_name},

  Your payment account needs attention: {attention_reason}

  Until this is resolved, online payments on your invoices are temporarily unavailable.
- **CTA (button):** Reconnect your account -- deep-links to FEAT-32.SPEC-001 (Payment Connection Screen)

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {nadia_first_name} | Freelancer Account -- name (first name portion) | Nadia | Greeting renders as "Hi," |
| {attention_reason} | Payment Account Connection -- status (the Needs attention value's associated reason: the processor's reason verbatim, or FEAT-32.SPEC-003's product-defined zero-methods reason "Your payment account has no payment methods turned on yet. Turn on card or bank transfer in your payment account, then reconnect.") | We need a copy of your government ID to verify your account. | Never empty -- FEAT-32.SPEC-003 discards a restriction event with no stated reason as malformed, and its zero-methods path supplies its own defined reason, so no Needs attention outcome is ever applied without one and this email's trigger never fires without one |

## Delivery Rules

**Batching:** None -- each status change (Connected, Needs attention) is a distinct, permanent account-readiness event confirmed by FEAT-32.SPEC-003, and each is sent individually as it happens. Since only one Payment Account Connection record exists per freelancer, there is never more than one pending instance of either email to batch.
**Deduplication:** At most one email per FEAT-32.SPEC-003 outcome application. FEAT-32.SPEC-003's own already-applied and ordering guards (its Edge Cases) are the deduplication boundary: a duplicate or stale underlying processor event that FEAT-32.SPEC-003 does not apply never produces a second email.
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001). After the final failure, this spec records a delivery-failure state for that email's status outcome, and FEAT-32.SPEC-001 surfaces it as the delivery warning banner it defines (text "We couldn't email you about this change to your payment account. The status shown below is current.", placed above the status line on the Connected or Needs attention state, with a "Dismiss" button), since this alert has no project to attach a warning to the way other transactional emails do. The warning clears when Nadia dismisses it, when FEAT-32.SPEC-003 applies a new status outcome, or when she disconnects; it is never raised for an email cancelled as superseded (Edge Cases).
**Expiry:** Neither email expires in the sense of becoming pointless to send late: the status each reports remains true until the next status change, so a delayed delivery (after retries) still carries accurate information whenever it lands. There is no withholding cutoff; the retry window above is the only limit, after which delivery is treated as failed (surfaced as a warning) rather than expired.

## Edge Cases

- **The connection returns to Connected while the Needs attention alert is still being retried** -- The pending Needs attention alert is cancelled rather than delivered late alongside a now-contradicting Connected confirmation; the Connected confirmation for the new outcome still sends on its own trigger. A stale "needs attention" email arriving after Nadia already sees "connected" would read as the product not paying attention.
- **The connection is disconnected before a pending Connected or Needs attention email is delivered** -- The pending email still sends: it was accurate at the moment FEAT-32.SPEC-003 applied that outcome, and a later disconnect (Nadia's own separate action) does not retroactively make it false, mirroring how FEAT-10.SPEC-007 treats a confirmation that predates a later reversal.
- **Two Needs attention outcomes are applied in succession with different reasons (the processor changes what it is asking for)** -- Each is a materially new instance of information Nadia needs, so each fires its own alert email rather than being batched or suppressed as a repeat.
- **Nadia's account is deleted (FEAT-24) while an email for either outcome is still queued for retry** -- The pending delivery is cancelled once account deletion finalizes and removes her Freelancer Account and its Notifications, per the dependency map's Notification lifecycle (Deleted by FEAT-24).
- **Quiet hours colliding with expiry** -- N/A, since both emails are transactional and exempt from quiet hours, so there is no quiet-hours hold to collide with an expiry cutoff.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-32.SPEC-003 (Connection Status Sync) | Triggered by (inbound) | The Connected and Needs attention outcomes fire this notification |
| FEAT-32.SPEC-001 (Payment Connection Screen) | Navigation (outbound) | Both CTAs deep-link here |
| FEAT-32.SPEC-001 (Payment Connection Screen) | Affects (outbound) | A final delivery failure surfaces as the delivery warning banner defined in SPEC-001's Layout and Content (text, placement above the status line, Dismiss, and clearing rules are specified there) |
| FEAT-14.SPEC-001 (Transactional Email Delivery) | References (outbound) | Underlying delivery, retry, and bounce/failure reporting capability this notification is sent through |

## Analytics and Success Signals

- **payment_account_connected_email_delivered** () -- supports success-metrics.md: "Payment Readiness Before First Invoice"
- **payment_account_needs_attention_email_delivered** () -- supports success-metrics.md: "Payment Readiness Before First Invoice"
- **payment_account_status_email_delivery_failed** (variant: connected / needs_attention, retry_count) -- N/A -- no Stage 2 metric measures this notification's own delivery-failure rate directly; retained as a standard delivery-quality signal.
- **payment_account_status_email_opened** (variant: connected / needs_attention) -- N/A -- no Stage 2 metric measures open rates for these specific emails; retained as a standard delivery-quality signal.

## Acceptance Criteria

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

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 2 | 2 |
| Preference States | 1 (always on -- no preference exists) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |

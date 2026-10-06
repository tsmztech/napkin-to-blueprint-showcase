---
document_type: feature-overview
feature_number: FEAT-32
feature_name: Payment Account Connection
feature_slug: payment-account-connection
priority_tier: Core
feature_type: Platform
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 6
screen_count: 1
automation_count: 2
logic_rule_count: 1
integration_count: 1
notification_count: 1
---

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

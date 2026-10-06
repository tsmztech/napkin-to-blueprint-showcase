---
document_type: feature-overview
feature_number: FEAT-28
feature_name: Payout Account Connection & Payout Visibility
feature_slug: payout-account-connection-payout-visibility
priority_tier: Core
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 7
screen_count: 2
automation_count: 1
logic_rule_count: 2
integration_count: 1
notification_count: 1
---

# Feature Breakdown Brief: Payout Account Connection & Payout Visibility

## Summary

**Feature:** Payout Account Connection & Payout Visibility
**ID:** FEAT-28
**Description:** The Pro connects their own payout account with the payment-processing capability so every client deposit lands directly with them, and can see at a glance what came in, what was refunded, what the card processor charged, and when money reaches their bank -- with nothing taken by Chairtime.
**Priority:** Core
**Phase:** MVP
**Type:** User-Facing
**Rationale:** BRIEF.md's Business Context states the client's deposit "goes to the pro (the payment processor handles payout to the pro)" and that "the platform takes no cut of any of it"; its Success Criteria demand that nobody "has ever had... a lost deposit." The draft specified where a deposit enters (FEAT-07) but not where it lands, so the value flow was open. Core because it is load-bearing: no deposit can be taken for a Pro without an active payout account, so the product's headline loop cannot close without it. MVP for the same reason. Visibility is part of the feature, not an extra: fee unpredictability is the dominant trust complaint in this market (3 competitors, HIGH confidence) and payouts held without explanation are a frequently mentioned complaint about one competitor (MEDIUM confidence), so the Pro must be able to see every deposit, refund and processor fee plainly. [AUDIT-ADDED: 1 -- Core: value-flow walk found no destination for the deposit money; without a connected payout account the deposit, refund and no-show loop that BRIEF.md's Vision and Success Criteria rest on cannot operate. Supported by market research: integrated card processing with payout to the professional is present in all 5 profiled competitors (Common Features), and fee-transparency complaints are HIGH confidence]

**Key Capabilities:**
- Connect a payout account during onboarding through the payment processor's own secure identity and bank verification -- the Pro never types bank details into Chairtime itself
- See payout account status -- verification pending, active, or action required
- See a simple money list -- deposits received, refunds sent, the processor's card fees, and upcoming and past payouts to the bank
- Resolve a flagged verification or bank-detail problem -- a direct path back into the processor's own flow

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-28.SPEC-001 | Payout Account Connection | Screen | The Pro | Pro launches the payment processor's own secure identity and bank verification flow during onboarding and sees the outcome (pending/active/failed handoff) before continuing setup |
| FEAT-28.SPEC-002 | Payout Status & Money Dashboard | Screen | The Pro, Platform Operator (Support) | Pro's ongoing view of payout account status (with an action-required banner and resolution path) and the money list of deposits, refunds, processor fees, and payouts, in all its data states |
| FEAT-28.SPEC-003 | Payout Account Status Processing | Automation | The Pro | Creates and updates the Payout Account record from the payment processor's reported status changes, drives the go-live gate signal, and triggers the status notification |
| FEAT-28.SPEC-004 | Payout Account Eligibility & Constraints | Logic/Rule | The Pro | Governs one-payout-account-per-Pro, country/currency matching to the Pro Account, and the standing zero-Chairtime-fee rule that the money list and go-live gate both depend on |
| FEAT-28.SPEC-005 | Money List Composition & Net Calculation | Logic/Rule | The Pro, Platform Operator (Support) | Defines how deposits, refunds, processor fees, and payouts are assembled into the money list and how net amount received per period is derived |
| FEAT-28.SPEC-006 | Payout Account Connection & Verification | Integration | The Pro | Handles the outbound handoff to, and inbound status/money data from, the payment-processing capability for account connection, identity and bank verification, action-required resolution, and payout/fee reporting |
| FEAT-28.SPEC-007 | Payout Status Notification | Notification | The Pro | Notifies the Pro when verification completes and the link can go live, and when the payout account needs action |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Connect a payout account during onboarding through the payment processor's own secure identity and bank verification -- the Pro never types bank details into Chairtime itself | FEAT-28.SPEC-001, FEAT-28.SPEC-006 | The Screen launches the handoff; the Integration spec carries the Pro to the processor's own verification flow and never collects bank/identity data itself | Phase 2 (Explicit) |
| See payout account status -- verification pending, active, or action required | FEAT-28.SPEC-002, FEAT-28.SPEC-003 | The dashboard displays the status the Automation spec keeps current from processor-reported events | Phase 2 (Explicit) |
| See a simple money list -- deposits received, refunds sent, the processor's card fees, and upcoming and past payouts to the bank | FEAT-28.SPEC-002, FEAT-28.SPEC-005 | The dashboard renders the list; the Logic/Rule spec defines how it is composed and the net figure derived | Phase 2 (Explicit) |
| Resolve a flagged verification or bank-detail problem -- a direct path back into the processor's own flow | FEAT-28.SPEC-002, FEAT-28.SPEC-006 | The action-required banner on the dashboard links directly into the same processor flow the Integration spec owns | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-28.SPEC-003 | Payout Account Status Processing | Phase 4 (Trigger-Response) | The processor's status events (Verification Pending -> Active -> Action Required -> Disconnected) are cross-entity, cross-feature side-effects -- they create/update the Payout Account record, unlock the go-live gate (XBR-06) that FEAT-15/FEAT-07 read, and fire a notification -- too consequential to leave inline in a screen |
| FEAT-28.SPEC-004 | Payout Account Eligibility & Constraints | Phase 5 (Rule Discovery) | The Validation & Limits field names four interacting conditions (one account per Pro, country/currency match, standing zero-platform-fee, processor-fee-only deduction) shared across the connection screen, the dashboard, and cross-feature gates (XBR-06, XBR-07, XBR-25) -- past the inline-validation threshold |
| FEAT-28.SPEC-005 | Money List Composition & Net Calculation | Phase 5 (Rule Discovery) | The Data Notes field names a derived field (net amount received per period) sourced from two entities (Deposit Transaction, and from v1 Balance Payment) plus the processor's own records -- a non-trivial derivation shared by every money-list-consuming context, not a one-line inline validation |
| FEAT-28.SPEC-006 | Payout Account Connection & Verification | Phase 4 (External Dependencies lens) | The Dependencies section of assumptions-constraints.md (ASMP-31) names the payment-processing capability's identity/bank verification and payout-status reporting as an external dependency this feature must specify |
| FEAT-28.SPEC-007 | Payout Status Notification | Phase 4 (Notification surfacing) | The Communications field names messages with real delivery rules (channel, audience, and content tied to a status transition), which the Phase 4 disposition rule requires as a standalone Notification spec rather than an inline confirmation |

## Entity-Lifecycle Coverage Matrix

**Entity: Payout Account**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-28.SPEC-003 | The record is created the moment the processor first reports a connection outcome (Verification Pending or Active) after the Pro completes the handoff initiated by SPEC-001 | Exactly one Payout Account per Pro Account (SPEC-004) |
| Read (single) | FEAT-28.SPEC-002, FEAT-28.SPEC-004 | The dashboard displays the Pro's own status; the eligibility rule reads it to gate the go-live check (XBR-06) and to answer FEAT-07/FEAT-19's read requests | -- |
| Read (list) | N/A | The Validation & Limits field fixes exactly one Payout Account per Pro Account, so no list view is meaningful | -- |
| Update | FEAT-28.SPEC-003 | Status changes reported by the processor (last-write-wins by the processor's event time, per the dependency map's Contention rule) are written by the Automation spec; the Pro never edits bank details inside Chairtime | -- |
| Delete/Archive | N/A -- explicit non-goal | The Payout Account reference is never independently deleted inside this feature; it is soft-removed only as part of Pro Account closure (FEAT-29's 30-day cooling-off), with no cascade to already-created Deposit Transactions and no separate retention/purge window of its own -- recorded as an explicit non-goal below | -- |
| State Transition | FEAT-28.SPEC-003 | Not Connected -> Verification Pending -> Active -> Action Required -> Disconnected, driven entirely by processor-reported events | Active is the only state from which a deposit can be taken (XBR-06) |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Deposit Transaction | FEAT-28.SPEC-002, FEAT-28.SPEC-005 | Deposits, refunds, and processor fees on the money list are drawn from Deposit Transaction records (created by FEAT-07, updated by FEAT-09/FEAT-11/FEAT-30) |
| Balance Payment | FEAT-28.SPEC-002, FEAT-28.SPEC-005 | From v1, balance payments and their refunds also appear on the money list once In-App Balance Payment (FEAT-22) exists |
| Pro Account | FEAT-28.SPEC-004 | Country and currency are read from the Pro Account and must match the Payout Account (XBR-25); currency becomes locked at the Pro Account's first deposit, not by this feature |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Pro reaches the "getting paid" step of onboarding | Launch the payment processor's own identity and bank verification flow | Standalone Integration | FEAT-28.SPEC-006 |
| Processor reports the account is Active | Create/update the Payout Account record to Active; signal the go-live gate (XBR-06) is satisfied; notify the Pro that the link can go live | Standalone Automation, then Standalone Notification | FEAT-28.SPEC-003, FEAT-28.SPEC-007 |
| Processor reports the account needs action (e.g., rejected bank details) | Update the Payout Account record to Action Required; existing/new bookings continue, payouts held by the processor; show a prominent banner; notify the Pro with exactly what to do | Standalone Automation, then Standalone Notification | FEAT-28.SPEC-003, FEAT-28.SPEC-007 |
| Pro taps the action-required banner's resolution link | Hand the Pro directly back into the processor's own flow for the specific problem | Standalone Integration | FEAT-28.SPEC-006 |
| Deposit is captured on a booking | Deposit appears in the money list the moment it is paid | Cross-feature (Deposit Transaction created by FEAT-07), composed for display by this feature | FEAT-28.SPEC-005 |
| A refund is due but the Pro's processor balance cannot cover it yet | Refund is retried automatically and flagged to the Pro; money list shows it as "in progress" until it clears | Cross-feature (retry and Pro-facing flag owned by FEAT-09/FEAT-30 per XBR-10); "in progress" money-list display owned by this feature | FEAT-28.SPEC-005 (display); FEAT-09/FEAT-30 (retry, flag) |
| Pro opens the money list | Load and assemble deposits, refunds, processor fees, and payouts; compute net amount received per period | Standalone Logic/Rule, inline data load in the triggering screen | FEAT-28.SPEC-005, inline in FEAT-28.SPEC-002 |
| Money list cannot be retrieved | Show the last-loaded figures with the time they were loaded and a retry action | Inline in triggering screen | FEAT-28.SPEC-002 |
| Pro has no bookings yet | Show "your deposits will appear here after your first booking" instead of a zero-filled table | Inline in triggering screen | FEAT-28.SPEC-002 |
| Connectivity is lost while viewing the dashboard | Most recently loaded money list stays viewable, read-only | Inline in triggering screen | FEAT-28.SPEC-002 |
| A pro account or currency/country mismatch is attempted for the payout account | Connection or update is blocked before it reaches the processor step | Standalone Logic/Rule | FEAT-28.SPEC-004 |

The Communications field names three messages. Two -- verification completes, and account needs action -- are status transitions this feature owns outright and are dispositioned into the standalone FEAT-28.SPEC-007 Notification spec. The third -- "a refund cannot be completed yet" -- is a cross-feature disposition: the retry and the flag-to-the-Pro are owned by FEAT-09/FEAT-30 under XBR-10 (FEAT-09 owns refund triggering), consistent with the journey's own Architect note that "the retry itself is triggered by FEAT-09/FEAT-30 ... and the dashboard flag belongs to FEAT-12." This feature's role in that third message is limited to reflecting the "in progress" and later "cleared" state in the money list (FEAT-28.SPEC-005), which is why `notification_count: 1` -- not 3 -- is correct rather than an omission.

## Shared Context

**Shared Entities:**
- Payout Account -- created and updated exclusively by SPEC-003 from processor-reported events; read by SPEC-002 (display) and SPEC-004 (eligibility gating), and by FEAT-07 (must be Active), FEAT-19 (status only), FEAT-22, FEAT-30 per the dependency map. Fields: processor_account_reference, status, country/currency, payout_schedule and recent payouts.
- Deposit Transaction (read-only here) -- composed into the money list by SPEC-005 and displayed by SPEC-002. Fields relevant here: amount/currency, status, processor_fee, outcome_reason/timestamps.

**Shared UI Patterns:**
- Status banner -- SPEC-002 shows the same status treatment (pending / active / action required, with its resolution link) that SPEC-001 first surfaces at the end of onboarding; the two screens should describe this element consistently rather than as two different components.
- Money list row -- a single row pattern (deposit, refund, processor fee, or payout, each with amount, date, and running status) used throughout SPEC-002's money list per the States field's data-state rules (Empty/Loading/Error/Offline-degraded).

**Shared Validation:**
- SPEC-004 defines all eligibility and constraint rules (one account per Pro, country/currency match, zero-Chairtime-fee standing rule). SPEC-001, SPEC-002, and SPEC-006 all reference it rather than duplicating the checks; FEAT-07 and FEAT-15 also read this spec's XBR-06 gate rather than re-deriving it.
- SPEC-005 defines money list composition and net-per-period derivation once; SPEC-002 is its only display consumer.

## Internal Dependency Map

```
SPEC-001 (Payout Account Connection) -> [Pro reaches "getting paid" step] -> SPEC-006 (Payout Account Connection & Verification) -> [processor outcome] -> SPEC-003 (Payout Account Status Processing)
SPEC-003 (Payout Account Status Processing) -> [status becomes Active] -> SPEC-007 (Payout Status Notification)
SPEC-003 (Payout Account Status Processing) -> [status becomes Action Required] -> SPEC-007 (Payout Status Notification)
SPEC-003 (Payout Account Status Processing) -> [status recorded] -> SPEC-002 (Payout Status & Money Dashboard) (status display refreshes)
SPEC-002 (Payout Status & Money Dashboard) -> [Pro taps resolution link on Action Required banner] -> SPEC-006 (Payout Account Connection & Verification) -> [processor outcome] -> SPEC-003 (Payout Account Status Processing)
SPEC-002 (Payout Status & Money Dashboard) -> [Pro opens money list] -> SPEC-005 (Money List Composition & Net Calculation) -> [assembled list] -> SPEC-002
SPEC-001 (Payout Account Connection) -> [validates against] -> SPEC-004 (Payout Account Eligibility & Constraints)
SPEC-002 (Payout Status & Money Dashboard) -> [status/gate checks against] -> SPEC-004 (Payout Account Eligibility & Constraints)
```

**Default Entry:** SPEC-001 (Payout Account Connection) on first use, reached only from the Pro Onboarding & Setup Wizard's "getting paid" step; SPEC-002 (Payout Status & Money Dashboard) thereafter, reached from navigation or from the attention list (FEAT-12).

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-28.SPEC-001 | Inbound | FEAT-15 (Pro Onboarding & Setup Wizard) | Pro reaches the "getting paid" step and is handed into payout account connection | Pro continues setup |
| FEAT-28.SPEC-004 | Outbound | FEAT-15 (Pro Onboarding & Setup Wizard) | Payout account status feeds the go-live gate (XBR-06, XBR-26) -- the booking link cannot go live until this feature reports the account Active | Setup wizard evaluates go-live readiness |
| FEAT-28.SPEC-004 | Outbound | FEAT-07 (Deposit Payment at Booking) | Eligibility check confirms the Payout Account is Active before any deposit charge is attempted (XBR-06) | Client attempts to pay a deposit |
| FEAT-28.SPEC-005 | Inbound | FEAT-07 (Deposit Payment at Booking) | Captured deposits and the processor's card fee become money-list entries | Deposit is captured |
| FEAT-28.SPEC-005 | Inbound | FEAT-09 (Cancellation & No-Show Policy Engine) | Automatic refunds appear in the money list against the deposits they reverse | Cancellation triggers an automatic refund |
| FEAT-28.SPEC-005 | Inbound | FEAT-30 (Pro Booking Management) | Pro-initiated goodwill refunds appear in the money list | Pro issues a refund |
| FEAT-28.SPEC-005 | Inbound | FEAT-22 (In-App Balance Payment, v1) | Balance payments and their refunds join the money list once this feature exists | Client pays or is refunded a balance in-app |
| FEAT-28.SPEC-002 | Outbound | FEAT-25 (Booking & Revenue Insights) | Money list and payout data feed revenue insights | Insights screen is opened |
| FEAT-28.SPEC-002 | Outbound | FEAT-19 (Platform Support Read-Only Access) | Support's read-only view of status and the money list, never bank or identity details | Support opens a Pro account during a help request |
| FEAT-28.SPEC-002 | Inbound | FEAT-12 (Pro Daily Schedule Dashboard) | Pro taps a money-related attention item or the money list navigation entry | Pro taps attention item or navigates to money list |
| FEAT-28.SPEC-004 | Inbound | FEAT-27 (Pro Profile & Booking Page Settings) | Country and currency, owned by FEAT-27, must match the Payout Account (XBR-25) | Payout account is connected or currency/country is evaluated |
| FEAT-28.SPEC-001 / SPEC-002 | Inbound | FEAT-29 (Pro Sign-In & Account Lifecycle) | Anyone not signed in as the Pro is sent to the Pro sign-in screen (XBR-29) | Unauthenticated access attempt |
| FEAT-28.SPEC-003 | Outbound | FEAT-16 (Booking & Payment Activity Record) | Status-change signals feed the append-only activity record for dispute evidence (XBR-22) | Payout account status changes |

## Non-Functional Notes

**Data volumes / growth:** A few hundred pros in year one, each producing one Payout Account and roughly 20-40 bookings a week worth of Deposit Transaction rows on the money list over multiple years of history; the money list must stay equally responsive as that history accumulates (assumptions-constraints.md, ASMP-22).

**Responsiveness:** The money list must let a Pro find any deposit, refund, processor fee, or bank payout from the last 90 days within a few seconds (success-metrics.md, Payout Transparency); per ASMP-27, the dashboard's Loading state shows an in-place indicator rather than a blank page, and nothing on it appears tappable before real data has loaded.

**Data sensitivity / privacy:** Financial data -- bank and identity details are never held by the product; they stay exclusively with the payment-processing capability (Data Notes field; dependency map's Payout Account Data Sensitivity line). Support sees status and the money list only, never bank or identity details (Access field; Access Matrix, Payouts row; ASMP-20-class restriction). The money list and status are visible only to the Pro (Full) and, view-only, to Platform Operator (Support); Clients never see any of it (Access field).

**Compliance flags:** N/A -- no compliance regime is named for this feature specifically; the payment-processing capability (ASMP-31) itself owns whatever identity-verification and financial-services obligations attach to connecting and holding a payout account, keeping that scope out of this product's own code, consistent with the hard card-data boundary in SC-11.

## Non-Goals

- **Handling or storing bank or identity details within the product itself** -- Excluded per the Data Notes field and ASMP-31: bank and identity details stay exclusively with the payment-processing capability; this feature only ever holds a reference to the processor's account and its status.
- **Support acting on a Pro's payout account** -- Excluded per SC-05: Platform Operator (Support) has view-only access to status and the money list and can never edit, resolve, or reconnect a payout account on the Pro's behalf; a Pro must resolve an action-required flag themselves through the processor's own flow.
- **Partial refunds or tiered cancellation schedules on the money list** -- Excluded per SC-18: the cancellation rule whose outcomes appear on this feature's money list is binary (refunded or kept), so the money list never displays a partial-percentage refund line.
- **Chairtime adjudicating a payment dispute** -- Excluded per SC-17: a Disputed deposit shown on the money list (via XBR-22) is resolved between the client and their card issuer through the payment processor's own dispute process; this feature only reflects the outcome.
- **More than one payout account, or one spanning multiple countries/currencies, per Pro** -- Excluded per the Validation & Limits field and SC-20: exactly one payout account per Pro Account, matching the Pro Account's single country and currency; multi-country payout support is deferred to the geography phase-in, not built into this feature now.
- **Independent deletion of the Payout Account record** -- Intentional lifecycle decision surfaced by the CRUD matrix: the record has no standalone delete path inside this feature and is only removed as part of Pro Account closure (FEAT-29), consistent with SC-22's retention posture for financial-adjacent records.
- **In-App Balance Payment and Tipping at Checkout as money-list sources today** -- Adjacency exclusion: both are named, explicitly deferred capabilities (Balance Payment targeted for v1, Tipping at Checkout targeted for Later per the deferral notes); this feature's Connected Entities already anticipate Balance Payment "from v1," but neither contributes money-list rows until its owning feature ships.

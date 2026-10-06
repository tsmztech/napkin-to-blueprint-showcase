# FEAT-28 — Payout Account Connection & Payout Visibility

This chapter covers Payout Account Connection & Payout Visibility (FEAT-28), a Core-tier feature. It carries 7 specifications carrying 127 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-28.SPEC-001 | Payout Account Connection | screen | 18 |
| FEAT-28.SPEC-002 | Payout Status & Money Dashboard | screen | 22 |
| FEAT-28.SPEC-003 | Payout Account Status Processing | automation | 14 |
| FEAT-28.SPEC-004 | Payout Account Eligibility & Constraints | logic-rule | 19 |
| FEAT-28.SPEC-005 | Money List Composition & Net Calculation | logic-rule | 20 |
| FEAT-28.SPEC-006 | Payout Account Connection & Verification | integration | 20 |
| FEAT-28.SPEC-007 | Payout Status Notification | notification | 14 |

The feature breakdown brief follows, then every specification in full.


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



# Screen Spec: Payout Account Connection

## Overview

**Name:** Payout Account Connection
**ID:** FEAT-28.SPEC-001
**Type:** Screen
**Purpose:** Talia launches the payment-processing capability's own secure identity and bank verification flow from the "getting paid" step of setup, and sees the outcome of that handoff (pending, active, or a failed handoff) before continuing.
**Parent Feature:** FEAT-28 -- Payout Account Connection & Payout Visibility

## Scope and Non-Goals

**In Scope:**
- Explaining, in plain words, why Talia is connecting a payout account and that Chairtime never sees her bank or identity details
- Launching the payment-processing capability's own identity and bank verification flow (FEAT-28.SPEC-006)
- Showing the outcome of that handoff (verification pending, active, or the handoff itself failed to start) before Talia continues setup
- Letting Talia continue the rest of onboarding even if verification is still pending

**Non-Goals:**
- Collecting or displaying any bank account number, routing number, or identity document -- excluded per the Data Notes field and ASMP-31: those details are entered directly into the payment-processing capability's own flow and never pass through this screen or the product's own code
- The ongoing status view, money list, and action-required resolution after this first connection -- owned by FEAT-28.SPEC-002 (Payout Status & Money Dashboard); this screen is reached once, on first use, and never again once a Payout Account exists
- Resolving a later "Action Required" flag -- owned by FEAT-28.SPEC-002's banner and FEAT-28.SPEC-006's resolution hand-off; this screen only covers the very first connection attempt during onboarding
- Deciding whether the booking link may go live -- owned by FEAT-15 (Pro Onboarding & Setup Wizard), which reads this feature's status (XBR-06, XBR-26) but makes the go-live decision itself

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-15.SPEC-001 (Setup Wizard Shell) (Pro Onboarding & Setup Wizard), setup step: getting paid | Talia reaches the "getting paid" step of the setup wizard | None -- no Payout Account exists yet for this Pro Account |
| FEAT-15.SPEC-003 (Go-Live Preview & Booking Link Hand-Over) | Talia taps "Finish verifying" on the go-live screen | None -- the existing Payout Account and its current verification status load fresh |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Launch the verification handoff and continue setup once an outcome is shown | -- |
| Platform Operator (Support) | No | No | This screen is reached only from the live onboarding wizard on the Pro's own device; Support's read-only view shows the resulting Payout Account status through FEAT-28.SPEC-002 instead, never this connection screen itself |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29, XBR-29); after signing in, a Pro who has not yet reached this step in setup is returned to FEAT-15 at their current step, not directly to this screen |
| Expired session | No | No | "Your session has expired. Sign in to continue." -- no in-progress handoff exists to preserve, since the identity/bank flow itself runs entirely inside the payment-processing capability, not on this screen |

## Layout and Content

**Header:** Setup wizard's standard step header, showing "Getting paid" as the current step within FEAT-15's overall progress indicator, with a back control to the previous setup step.

**Body:** A single-column explanation panel above one primary action:
- A short paragraph stating why this step exists in plain words: money from every deposit needs somewhere to go, and Chairtime never sees or stores Talia's bank or identity details -- that stays with the payment processor.
- A single primary action: "Connect payout account."
- Once the handoff has been launched at least once, a status region appears below the action, showing the current outcome (Verification Pending, Active, or Handoff Failed) per the States section.
- A secondary text link, "Why do you need my bank details?", which expands an inline explanation (no navigation) covering the same plain-words disclosure as FEAT-28.SPEC-006's Consent and Disclosure section.

**Footer:** A "Continue setup" action, enabled once any outcome (including Verification Pending) has been recorded; disabled before the first handoff attempt.

### Responsive Behavior

- **Compact breakpoint:** Single-column panel as described, full width, primary action full-width beneath the explanation text.
- **Medium size class and above:** Panel content is capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| "Connect payout account" | Tap | Triggers the outbound handoff via FEAT-28.SPEC-006 to the payment-processing capability's identity and bank verification flow | Screen enters the Launching state | The capability's own flow opens (in-flow or as a full-screen takeover); on return, the status region reflects the reported outcome |
| "Why do you need my bank details?" | Tap | Expands an inline disclosure panel in place | No navigation; panel expands below the link | Disclosure text becomes visible; a second tap collapses it |
| Status region (display-only) | -- | Non-interactive; reflects the current outcome | -- | -- |
| "Continue setup" | Tap | Advances FEAT-15 to its next setup step | Screen closes | Setup wizard advances to the next step (calendar connection) |
| Back control | Tap | Returns to the previous FEAT-15 setup step | Screen closes | Setup wizard shows the previous step (cancellation policy) |

### Accessibility Notes

- **Focus order:** Back control -> explanation text -> "Connect payout account" -> "Why do you need my bank details?" -> status region (when present) -> "Continue setup."
- **Dynamic announcements:** When the status region first appears or its outcome changes (Launching -> Verification Pending / Active / Handoff Failed), the new state is announced to assistive technology.
- **Keyboard alternatives:** Every action on this screen, including launching the external handoff, is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Not Started (default) | Explanation text and "Connect payout account" shown; no status region; "Continue setup" disabled | Talia reaches this step with no prior handoff attempt | She taps "Connect payout account" |
| Launching | "Connect payout account" shows a brief loading state while the handoff is initiated | Tap on "Connect payout account" | The payment-processing capability's flow opens, or the handoff fails to start |
| Verification Pending | Status region shows "Verification pending -- we'll let you know as soon as it's ready." "Continue setup" is enabled | The capability reports the identity/bank verification was submitted but not yet confirmed | The capability later reports Active or Action Required (reflected on FEAT-28.SPEC-002 from that point forward) |
| Active | Status region shows "Payout account connected and active." "Continue setup" is enabled | The capability reports verification is complete | Talia continues setup or leaves the screen |
| Handoff Failed | Status region shows "We couldn't start the connection process. Try again." with a "Try again" action; "Continue setup" remains disabled until a successful handoff attempt is recorded | The handoff itself could not be initiated (see FEAT-28.SPEC-006 Degradation Behavior) | Talia taps "Try again" and the handoff launches successfully |
| Offline/Degraded | Banner "You're offline -- connecting a payout account needs an internet connection." The primary action is disabled until connectivity returns; no partial handoff is started or retried automatically | Connectivity is lost before or during the handoff launch | Connectivity is restored -- the screen returns to its state before the loss (Not Started or the last recorded outcome) |

## Validation Rules

Validation governed by FEAT-28.SPEC-004 (Payout Account Eligibility & Constraints). This screen enforces the one-payout-account-per-Pro-Account rule implicitly by never being reachable a second time once a Payout Account exists for this Pro Account (see Edge Cases); no field-level validation exists on this screen since no bank or identity fields are entered here.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| "Connect payout account" tap | FEAT-28.SPEC-006 (Payout Account Connection & Verification) | -- |
| "Continue setup" tap | FEAT-15, next setup step (calendar connection) | FEAT-15 (Pro Onboarding & Setup Wizard) |
| Back control tap | FEAT-15, previous setup step (cancellation policy) | FEAT-15 (Pro Onboarding & Setup Wizard) |

## Data Model

**Creates:** None directly -- the Payout Account record itself is created by FEAT-28.SPEC-003 the moment the payment-processing capability first reports a connection outcome; this screen only initiates the handoff that produces that report.
**Reads:** Payout Account -- status, once a handoff has been launched at least once in this session, to render the status region.
**Updates:** None.
**Deletes:** None.

## Business Rules

- XBR-06: no deposit can be taken, and the booking link cannot go live, until this feature reports the Payout Account Active; "Continue setup" being enabled at Verification Pending does not itself satisfy XBR-06 -- FEAT-15's own go-live check (XBR-26) still requires Active before publishing the booking link.
- FEAT-28.SPEC-004 governs that exactly one Payout Account exists per Pro Account; this screen is reachable only when no Payout Account yet exists for the signed-in Pro.
- FEAT-28.SPEC-004 governs the country/currency match between the Pro Account and the Payout Account (XBR-25); any mismatch is a rejection reported back by the capability itself during the handoff (FEAT-28.SPEC-006), not a check performed on this screen.
- Bank and identity details never reach this screen or the product's own code (SC-11-class boundary, ASMP-31); the handoff hands Talia directly to the capability's own entry surface.

## Edge Cases

- **Talia navigates away mid-handoff (before any outcome is reported)** -- No Payout Account record is created; returning to this step shows the Not Started state again, and she can launch the handoff again without penalty.
- **Talia reaches this step a second time after a Payout Account already exists (e.g., she goes back in the wizard after completing this step)** -- The screen instead shows the recorded outcome directly (Verification Pending / Active) rather than the Not Started state, since FEAT-28.SPEC-004 prevents a second Payout Account from being created for the same Pro Account.
- **The handoff itself fails to launch (the capability cannot be reached at all)** -- Handoff Failed state shown with "Try again"; no Payout Account record is created since no outcome was ever reported.
- **Talia taps "Connect payout account" twice in rapid succession** -- The second tap is ignored while the first handoff launch is in progress (button in loading state).
- **The capability reports Action Required on this very first attempt (e.g., a rejected bank detail on first submission)** -- The status region shows the same "needs action" wording FEAT-28.SPEC-002 uses, with the same resolution link into FEAT-28.SPEC-006; "Continue setup" remains enabled, since existing product behavior lets the Pro finish setup with a non-Active payout account and resolve it later, consistent with the Brief's Alternate flow.
- **Talia's device loses connectivity while the capability's flow is open** -- Handled entirely inside that flow, which is outside this screen's own connectivity boundary; on return to this screen, the reported outcome (if any) is shown, or Handoff Failed if none was received.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-28.SPEC-006 (Payout Account Connection & Verification) | Triggers (outbound) | "Connect payout account" initiates the outbound handoff |
| FEAT-28.SPEC-004 (Payout Account Eligibility & Constraints) | References (inbound) | One-account-per-Pro and country/currency match rules govern this screen's reachability and outcomes |
| FEAT-28.SPEC-003 (Payout Account Status Processing) | Triggers (outbound) | The reported handoff outcome is what that automation writes to the Payout Account record |
| FEAT-28.SPEC-002 (Payout Status & Money Dashboard) | Navigation (outbound) | The same status treatment continues on the ongoing dashboard once setup is complete |
| FEAT-15.SPEC-001 (Setup Wizard Shell, Step Navigation & Guidance), FEAT-15.SPEC-003 (Go-Live Preview & Booking Link Hand-Over), FEAT-15.SPEC-004 (Setup Progress Tracking & Resume) -- within FEAT-15 (Pro Onboarding & Setup Wizard) | Navigation (inbound / outbound) | Entry point and the "Continue setup" / back destinations |
| FEAT-29 (Pro Sign-In & Account Lifecycle) | Navigation (outbound) | Unauthenticated access is redirected here (XBR-29) |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| payout_account_connect_started | outcome of prior attempt, if any | Talia taps "Connect payout account" | supports success-metrics.md: "Setup-to-Live-Link Completion" |
| payout_account_connect_outcome_shown | outcome (verification_pending / active / handoff_failed) | The status region first shows a reported outcome | supports success-metrics.md: "Setup-to-Live-Link Completion" |
| payout_account_setup_continued | outcome at time of continuing | Talia taps "Continue setup" | supports success-metrics.md: "Setup-to-Live-Link Completion" |

## Acceptance Criteria

**FEAT-28.SPEC-001-AC-01:** Given Talia reaches the "getting paid" step of setup for the first time, when the screen loads, then she sees the plain-words explanation and the "Connect payout account" action, with "Continue setup" disabled.

**FEAT-28.SPEC-001-AC-02:** Given Talia is on this screen, when she taps "Connect payout account", then the handoff to the payment-processing capability launches (FEAT-28.SPEC-006).

**FEAT-28.SPEC-001-AC-03:** Given the capability reports the connection as Verification Pending, when the outcome is received, then the status region shows "Verification pending -- we'll let you know as soon as it's ready." and "Continue setup" becomes enabled.

**FEAT-28.SPEC-001-AC-04:** Given the capability reports the connection as Active, when the outcome is received, then the status region shows "Payout account connected and active."

**FEAT-28.SPEC-001-AC-05:** Given the handoff fails to launch at all, when Talia taps "Connect payout account", then she sees "We couldn't start the connection process. Try again." with a "Try again" action, and "Continue setup" stays disabled.

**FEAT-28.SPEC-001-AC-06:** Given Talia is at Verification Pending, when she taps "Continue setup", then FEAT-15 advances to the calendar connection step.

**FEAT-28.SPEC-001-AC-07:** Given Talia taps "Why do you need my bank details?", when the tap registers, then an inline disclosure panel expands in place with no navigation away from this screen.

**FEAT-28.SPEC-001-AC-08:** Given Talia has not launched a handoff yet, when she looks for "Continue setup", then it is disabled.

**FEAT-28.SPEC-001-AC-09:** Given Talia taps "Connect payout account" twice rapidly, when the second tap registers, then it is ignored while the first launch is in progress.

**FEAT-28.SPEC-001-AC-10:** Given Talia loses connectivity while viewing this screen before launching a handoff, when she taps "Connect payout account", then the offline banner appears and no handoff is initiated.

**FEAT-28.SPEC-001-AC-11:** Given Talia already has a Payout Account from a prior visit to this step, when she reaches this step again, then the screen shows the recorded outcome directly instead of the Not Started state.

**FEAT-28.SPEC-001-AC-12:** Given the capability reports Action Required on the very first handoff attempt, when the outcome is received, then the status region shows the needs-action wording with a resolution link into FEAT-28.SPEC-006, and "Continue setup" remains enabled.

**FEAT-28.SPEC-001-AC-13:** Given Talia navigates away before any outcome is reported, when she returns to this step, then no Payout Account record exists and the Not Started state is shown again.

**FEAT-28.SPEC-001-AC-14:** Given an unauthenticated visitor somehow reaches this screen's URL directly, when the screen attempts to load, then they are redirected to the Pro sign-in screen (FEAT-29).

**FEAT-28.SPEC-001-AC-15:** Given Talia's session expires while she is on this screen, when she next interacts with it, then she sees "Your session has expired. Sign in to continue." with no in-progress handoff to preserve.

**FEAT-28.SPEC-001-AC-16:** Given Talia taps the back control, when the tap registers, then FEAT-15 shows the previous setup step (cancellation policy) and any recorded Payout Account outcome is unaffected.

**FEAT-28.SPEC-001-AC-17:** Given the connection reaches Active before Talia taps "Continue setup", then FEAT-15's own go-live check (XBR-26) still evaluates all remaining requirements before the booking link may go live -- this screen's "Continue setup" being enabled does not itself publish the link.

**FEAT-28.SPEC-001-AC-18:** Given Platform Operator (Support) attempts to view this screen for a Pro, when the attempt is made, then it is not reachable to Support at all -- Support instead sees status through FEAT-28.SPEC-002.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 6 (not started, launching, verification pending, active, handoff failed, offline) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Screen Spec: Payout Status & Money Dashboard

## Overview

**Name:** Payout Status & Money Dashboard
**ID:** FEAT-28.SPEC-002
**Type:** Screen
**Purpose:** Talia's ongoing view of her payout account's status (with an action-required banner and resolution path when needed) and the money list of deposits, refunds, processor fees, and payouts, in all its data states.
**Parent Feature:** FEAT-28 -- Payout Account Connection & Payout Visibility

## Scope and Non-Goals

**In Scope:**
- The status banner (pending / active / action required), consistent with the treatment first shown by FEAT-28.SPEC-001
- The money list: deposits received, refunds sent, processor fees, and upcoming and past payouts, assembled by FEAT-28.SPEC-005
- Every data state the money list can be in: empty (no bookings yet), loading, error (with last-loaded figures and retry), and offline/degraded
- A direct resolution link from the action-required banner into the processor's own flow

**Non-Goals:**
- The initial connection handoff itself -- owned by FEAT-28.SPEC-001; this screen is reached only once a Payout Account already exists
- Composing or deriving the money list's rows and net figure -- owned by FEAT-28.SPEC-005 (Money List Composition & Net Calculation); this screen only displays what that spec assembles
- Editing bank or identity details -- excluded per SC-05 and the Data Notes field: this screen never collects or shows bank/identity details, only status and money figures
- Support taking any action on a Pro's payout account -- excluded per SC-05: Support's access here is View-only, per the Access Matrix

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-12 (Pro Daily Schedule Dashboard), navigation | Talia taps the money list navigation entry | None -- opens to the default (most recent) view |
| FEAT-12 (Pro Daily Schedule Dashboard), attention list | Talia taps a money-related attention item (refund in progress / payout action required) | The specific attention condition, so the relevant banner or list entry is highlighted on load |
| FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) (Platform Support Read-Only Access) | Support opens a Pro account after a help request | Read-only context; Support's view omits any action affordance |
| FEAT-28.SPEC-007 (Payout Status Notification) | Talia taps "View payouts" or "Resolve now" in a payout status notification | None -- opens to the default view; for "Resolve now" the action-required item is surfaced for FEAT-28.SPEC-006's resolution flow |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen (status banner and full money list) | Tap the resolution link on the action-required banner, retry a failed load | -- |
| Platform Operator (Support) | Status banner and money list, exactly as the Pro sees them | None -- no resolution link, no retry action; view is read-only per ASMP-20 and XBR-24 | Attempting to tap the resolution link (if somehow present) has no effect; the interface shows no such control for Support in the first place |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29, XBR-29) |
| Expired session | No | No | "Your session has expired. Sign in to continue." -- the most recently loaded money list is not shown until re-authentication succeeds, since it reflects financial data |

## Layout and Content

**Header:** Screen title "Payouts" with a back control to FEAT-12, consistent with the setup wizard's status treatment carried forward from FEAT-28.SPEC-001.

**Body, top region -- Status banner:**
- Shown when status is Verification Pending: "Verification pending -- we'll let you know as soon as it's ready."
- Shown when status is Action Required: a prominent banner "Your payout account needs attention" with a one-line reason drawn from the capability's report and a "Resolve now" action.
- Not shown when status is Active (no banner; the money list occupies the full body).

**Body, main region -- Money list:**
- A reverse-chronological list of rows, each one of: a deposit (client name, amount, processor fee, net, date, status), a refund (client name, amount, date, status including "in progress"), a processor payout to the bank (amount, date, status: upcoming or completed).
- A summary strip above the list showing net amount received for the current period (this month by default), per FEAT-28.SPEC-005's derivation.
- Each row shows a running status consistent with the Shared UI Pattern "money list row" from the Brief's Shared Context.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Status banner and summary strip stack above the single-column money list, full width.
- **Medium size class and above:** The summary strip and money list remain single-column but are capped at a consistent platform-wide content width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back control | Tap | Navigate to FEAT-12 (Pro Daily Schedule Dashboard) | Screen closes | Returns to the schedule dashboard |
| "Resolve now" (action-required banner) | Tap (Pro only) | Navigate into the processor's own resolution flow via FEAT-28.SPEC-006 | Screen transitions to the external flow | The capability's own resolution surface opens |
| Money list row | Tap | Expands the row in place to show the full detail already summarized (no navigation to another spec) | Row expands | Additional detail (e.g., the full outcome_reason for a refund, or the payout's arrival estimate) becomes visible |
| Retry (Error state) | Tap (Pro and Support) | Re-attempts loading the money list | List returns to Loading | Success shows the refreshed list; repeated failure keeps the Error state with the same last-loaded figures |
| Summary strip period selector | Tap | Switches the net-figure period (e.g., this month / last 90 days) | Summary strip recalculates | Updated net figure and label shown, per FEAT-28.SPEC-005 |

### Accessibility Notes

- **Focus order:** Back control -> status banner (when present) -> "Resolve now" (when present) -> summary strip period selector -> money list rows in displayed order -> retry action (when present).
- **Dynamic announcements:** A newly appearing action-required banner is announced to assistive technology; a successful retry that refreshes the list announces "Money list updated."
- **Keyboard alternatives:** Row expansion and the period selector are reachable and operable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Active, populated | No status banner; summary strip and full money list shown | Payout Account status is Active and at least one money-list entry exists | Status changes to Action Required, or the list becomes empty (never occurs once populated, per SC-22 retention) |
| Verification Pending | Informational banner shown; money list area shows the Empty (no bookings yet) sub-state, since a deposit cannot be taken before Active (XBR-06) | Payout Account status is Verification Pending | Status changes to Active or Action Required |
| Action Required | Prominent banner with "Resolve now"; money list below remains visible and populated as normal -- existing and new bookings continue per the Brief's Alternate flow | The payment-processing capability reports the account needs action | Talia resolves the flag through FEAT-28.SPEC-006 and the capability reports Active |
| Empty (no bookings yet) | Money list area shows "Your deposits will appear here after your first booking." instead of a zero-filled table | No Deposit Transaction exists yet for this Pro Account | The first deposit is captured (FEAT-07) |
| Loading | A brief in-place indicator over the money list area; nothing in it appears tappable before real data has loaded (ASMP-27) | Screen opens, or a retry is triggered | Data loads successfully or the load fails |
| Error | The last-loaded money list figures remain visible, with the time they were loaded and a "Retry" action | The money list cannot be retrieved on this load attempt | Talia or Support taps Retry and the load succeeds |
| Offline/Degraded | The most recently loaded money list stays viewable, read-only; a banner reads "You're offline -- showing the last money list we loaded." No action (resolution link, retry) is available while offline | Connectivity is lost while this screen is open, or the screen is opened without connectivity | Connectivity is restored and the list refreshes automatically |

## Validation Rules

N/A -- this screen has no user-entered fields; all data is read-only. Composition and derivation rules for the money list are governed by FEAT-28.SPEC-005; eligibility and status rules are governed by FEAT-28.SPEC-004.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back control tap | FEAT-12.SPEC-001 (Today's & Upcoming Schedule) | FEAT-12 (Pro Daily Schedule Dashboard) |
| "Resolve now" tap | FEAT-28.SPEC-006 (Payout Account Connection & Verification) | -- |

## Data Model

**Creates:** None.
**Reads:** Payout Account -- status, payout_schedule, recent payouts; Deposit Transaction -- amount, currency, status, processor_fee, outcome_reason, timestamps; Balance Payment -- amount, tip, state (from v1, once FEAT-22 exists). All composed into the displayed list by FEAT-28.SPEC-005.
**Updates:** None -- this screen never writes to the Payout Account or any Deposit Transaction; resolution happens entirely inside the processor's own flow (FEAT-28.SPEC-006).
**Deletes:** None.

## Business Rules

- XBR-06 / XBR-26: while status is Verification Pending or Action Required, existing and new bookings continue exactly as the Brief's Alternate flow describes; this screen never implies bookings have stopped.
- FEAT-28.SPEC-004 governs that the only deduction ever shown on a deposit row is the processor's own card fee -- Chairtime's own fee is always zero and is never rendered as a line item (XBR-07).
- FEAT-28.SPEC-005 governs money list composition and the net-per-period derivation; this screen is its only display consumer, per the Brief's Shared Validation section.
- FEAT-19.SPEC-004 (Support Session Scope & Access Rules) is the rule spec for Support-session exclusions; this screen enforces it on render by never showing bank or identity details and offering Support no resolution or retry controls.
- Support's View access (Access Matrix, Payouts) never exposes bank or identity details on this screen, since those are never held by the product in the first place (ASMP-31).

## Edge Cases

- **Talia opens the screen with no connectivity at all** -- The Offline/Degraded state shows the most recently loaded money list if one was ever cached on this device, or the Error state's "cannot be retrieved, retry" wording if none exists yet.
- **The action-required banner's condition resolves while Talia is viewing the screen** -- The banner disappears and the screen transitions to the Active, populated state without requiring a manual refresh, since FEAT-28.SPEC-003 updates the Payout Account status the moment the capability reports the change.
- **A refund shown as "in progress" completes while Talia is viewing the list** -- The row updates in place to its completed status the next time the list refreshes (on this screen's own refresh cadence); it is not required to update instantaneously mid-view.
- **Support opens this screen while the Pro is also viewing it** -- No conflict exists since neither role writes to any entity from this screen; both see independent, read-only snapshots (dependency map's Payout Account Contention note: the processor-reported status is authoritative for both).
- **Talia taps Retry while offline** -- Retry has no effect while genuinely offline; the offline banner's own wording already states that connectivity is required, and no request is sent.
- **The money list has grown to multiple years of history (per ASMP-22)** -- The list remains equally responsive; older entries are available by scrolling further, and the summary strip's period selector lets Talia narrow to a shorter window without waiting on the full history to load.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-28.SPEC-003 (Payout Account Status Processing) | References (inbound) | Supplies the status this screen's banner reflects |
| FEAT-28.SPEC-004 (Payout Account Eligibility & Constraints) | References (inbound) | Governs the zero-Chairtime-fee display rule and status meaning |
| FEAT-28.SPEC-005 (Money List Composition & Net Calculation) | References (inbound) | Supplies the assembled money list and net-per-period figure this screen displays |
| FEAT-28.SPEC-006 (Payout Account Connection & Verification) | Navigation (outbound) | "Resolve now" hands the Pro into the same processor flow |
| FEAT-12 (Pro Daily Schedule Dashboard) | Navigation (inbound / outbound) | Entry point via navigation or attention list; back control returns here |
| FEAT-19 (Platform Support Read-Only Access) | Navigation (inbound) | Support's read-only entry point after a help request |
| FEAT-19.SPEC-004 (Support Session Scope & Access Rules) | Enforces (inbound) | That rule's Support-specific exclusions (no bank or identity details, read-only scoping) are enforced on this screen's render, jointly with this spec's own scoping |
| FEAT-25 (Booking & Revenue Insights) | Affects (outbound) | Money list and payout data feed that feature's insights, once it exists |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| money_list_viewed | status at time of view (active / verification_pending / action_required) | Screen opens successfully | supports success-metrics.md: "Payout Transparency" |
| payout_account_action_required | reason category | The Action Required banner is shown | supports success-metrics.md: "Payout Transparency" |
| money_list_load_failed | -- | The money list fails to load, showing the Error state | N/A -- no Stage 2 metric measures load-failure frequency directly; retained so the correctness bar (ASMP-26) is observable for this screen's data-fetching path |
| money_list_row_expanded | row type (deposit / refund / payout) | Talia or Support expands a row | supports success-metrics.md: "Payout Transparency" |

## Acceptance Criteria

**FEAT-28.SPEC-002-AC-01:** Given Talia's Payout Account is Active and has at least one deposit, when she opens this screen, then she sees the summary strip and full money list with no status banner.

**FEAT-28.SPEC-002-AC-02:** Given Talia's Payout Account is Verification Pending, when she opens this screen, then she sees the pending banner and the money list area shows the Empty (no bookings yet) message, since no deposit could be taken yet (XBR-06).

**FEAT-28.SPEC-002-AC-03:** Given Talia's Payout Account is Action Required, when she opens this screen, then she sees the prominent banner with a plain reason and a "Resolve now" action, and her existing money list remains fully visible below it.

**FEAT-28.SPEC-002-AC-04:** Given Talia taps "Resolve now", when the tap registers, then she is handed into the processor's own resolution flow (FEAT-28.SPEC-006).

**FEAT-28.SPEC-002-AC-05:** Given Talia has no bookings yet, when she opens this screen, then she sees "Your deposits will appear here after your first booking." instead of a zero-filled table.

**FEAT-28.SPEC-002-AC-06:** Given the money list cannot be retrieved, when the load fails, then the last-loaded figures remain visible with the time they were loaded and a "Retry" action.

**FEAT-28.SPEC-002-AC-07:** Given Talia loses connectivity while viewing a previously loaded money list, when connectivity drops, then the most recently loaded list stays viewable read-only with the offline banner shown.

**FEAT-28.SPEC-002-AC-08:** Given a deposit row is displayed, when Talia looks at its deduction, then only the processor's own card fee is shown -- never a Chairtime fee line, since Chairtime's fee is always zero (XBR-07).

**FEAT-28.SPEC-002-AC-09:** Given Talia taps a deposit row, when the tap registers, then the row expands in place to show its full detail without leaving this screen.

**FEAT-28.SPEC-002-AC-10:** Given Platform Operator (Support) opens this screen for a Pro after a help request, when the banner is Action Required, then Support sees the same banner and reason Talia sees but no "Resolve now" action, since resolution is Pro-only per SC-05; Support's Retry action for a failed load remains available, since retrying a read is not an action on the Pro's account.

**FEAT-28.SPEC-002-AC-11:** Given the Action Required condition resolves while Talia is viewing this screen, when FEAT-28.SPEC-003 records the change, then the banner disappears without a manual refresh.

**FEAT-28.SPEC-002-AC-12:** Given a refund shows as "in progress," when it completes, then the row updates to its completed status on the screen's next refresh.

**FEAT-28.SPEC-002-AC-13:** Given Talia opens this screen with no connectivity and no cached list on this device, when the load is attempted, then the Error state's "cannot be retrieved" wording with Retry is shown rather than the offline banner.

**FEAT-28.SPEC-002-AC-14:** Given Talia is offline and viewing the cached list, when she taps Retry, then no request is sent and the offline state remains unchanged.

**FEAT-28.SPEC-002-AC-15:** Given an unauthenticated visitor attempts to reach this screen, when the attempt is made, then they are redirected to the Pro sign-in screen (FEAT-29).

**FEAT-28.SPEC-002-AC-16:** Given Talia's session expires while viewing this screen, when she next interacts with it, then she sees "Your session has expired. Sign in to continue." and the money list is not shown until she re-authenticates.

**FEAT-28.SPEC-002-AC-17:** Given Talia is on FEAT-12's attention list and taps the payout action-required item, when this screen opens, then the Action Required banner is already visible and, where applicable, highlighted.

**FEAT-28.SPEC-002-AC-18:** Given Talia changes the summary strip's period selector, when the new period is chosen, then the net figure recalculates per FEAT-28.SPEC-005 and the money list itself is unaffected (it always shows full history).

**FEAT-28.SPEC-002-AC-19:** Given Talia's money list has multiple years of history, when she scrolls further back, then older entries continue to load responsively.

**FEAT-28.SPEC-002-AC-20:** Given a money-list load fails and Talia taps Retry, when the retry succeeds, then the refreshed list replaces the last-loaded figures and the "loaded at" timestamp updates.

**FEAT-28.SPEC-002-AC-21:** Given Support views this screen, when Support looks for bank or identity details anywhere on it, then none are present -- only status and money-list figures, consistent with ASMP-20.

**FEAT-28.SPEC-002-AC-22:** Given Talia's Payout Account transitions from Active to Action Required while she is on FEAT-12 rather than this screen, when she next opens this screen (via the attention list), then the Action Required banner and its reason are already reflected.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 7 (active/populated, verification pending, action required, empty, loading, error, offline) | 7 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Automation Spec: Payout Account Status Processing

## Overview

**Name:** Payout Account Status Processing
**ID:** FEAT-28.SPEC-003
**Type:** Automation
**Purpose:** Creates and updates the Payout Account record from the payment-processing capability's reported status changes, drives the go-live gate signal (XBR-06), and triggers the status notification.
**Parent Feature:** FEAT-28 -- Payout Account Connection & Payout Visibility

## Scope and Non-Goals

**In Scope:**
- Creating the Payout Account record the moment the capability first reports a connection outcome
- Updating the Payout Account's status field on every subsequent status report (Verification Pending -> Active -> Action Required -> Disconnected)
- Signaling the go-live gate (XBR-06) whenever status changes to or from Active
- Triggering FEAT-28.SPEC-007 (Payout Status Notification) on the two status transitions this feature owns

**Non-Goals:**
- Requesting or receiving the status report itself from the capability -- owned by FEAT-28.SPEC-006 (Payout Account Connection & Verification); this automation only processes what that integration hands it
- Deciding whether a candidate Payout Account is eligible (one per Pro, country/currency match) -- owned by FEAT-28.SPEC-004 (Payout Account Eligibility & Constraints); this automation applies that spec's rule at creation time rather than re-deriving it
- Displaying the resulting status -- owned by FEAT-28.SPEC-001 (first connection) and FEAT-28.SPEC-002 (ongoing dashboard); this automation only writes the record those screens read
- Refund processing or money-list composition -- owned by FEAT-09/FEAT-30's own integration specs and FEAT-28.SPEC-005 respectively; this automation governs only the Payout Account entity's own status

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Connection outcome first reported | FEAT-28.SPEC-006 (Payout Account Connection & Verification) | Fires the first time the capability reports any outcome (Verification Pending or Active) for a Pro Account with no existing Payout Account | Pro Account reference, processor_account_reference, reported status, country, currency |
| Status change reported | FEAT-28.SPEC-006 (Payout Account Connection & Verification) | Fires whenever the capability reports a status different from the Payout Account's current status | Payout Account reference, new status, event time, and (for Action Required) the capability's reason |
| Resolution outcome reported | FEAT-28.SPEC-006 (Payout Account Connection & Verification) | Fires when Talia completes the processor's own resolution flow for an Action Required account and the capability reports the outcome | Payout Account reference, new status (Active or still Action Required), event time |

## Processing Logic

1. Receive the reported outcome from FEAT-28.SPEC-006, carrying the Pro Account reference, the reported status, and the event time the capability assigns to the report.
2. If no Payout Account exists yet for this Pro Account, create one: set processor_account_reference, status to the reported value, and country/currency from the report, after confirming eligibility per FEAT-28.SPEC-004 (one account per Pro, country/currency match to the Pro Account).
3. If a Payout Account already exists, compare the reported event time to the Payout Account's last-recorded event time. If the reported event is not newer, discard it (see Edge Cases -- out-of-order delivery); otherwise proceed.
4. Update the Payout Account's status field to the reported value.
5. If the new status is Active and the previous status was not Active, signal the go-live gate (XBR-06) that this Pro's payout precondition is now satisfied, and trigger FEAT-28.SPEC-007 with the "verification complete" content.
6. If the new status is Action Required, signal the go-live gate that the precondition is not satisfied (if it was previously satisfied, existing bookings and the booking link's live state are unaffected per the Brief's Alternate flow -- only new go-live evaluations are blocked), and trigger FEAT-28.SPEC-007 with the "needs action" content, carrying the capability's reason.
7. If the new status is Disconnected, signal the go-live gate that the precondition is not satisfied.
8. Record the status change as an Activity Event via FEAT-16 (Booking & Payment Activity Record), per XBR-22's evidence trail for status-change signals.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Payout Account created (pending) | First-ever report is Verification Pending | Payout Account created with status Verification Pending | FEAT-28.SPEC-001 shows the pending status | FEAT-28.SPEC-001, FEAT-28.SPEC-002 |
| Payout Account created (active) | First-ever report is Active | Payout Account created with status Active | FEAT-28.SPEC-001 shows Active; go-live gate satisfied; FEAT-28.SPEC-007 fires | FEAT-28.SPEC-001, FEAT-28.SPEC-002, FEAT-28.SPEC-007, FEAT-15 (go-live evaluation) |
| Status updated to Active | Existing Payout Account moves from Verification Pending or Action Required to Active | Payout Account.status set to Active | FEAT-28.SPEC-002's banner clears; go-live gate satisfied; FEAT-28.SPEC-007 fires with "verification complete" | FEAT-28.SPEC-002, FEAT-28.SPEC-007, FEAT-15 |
| Status updated to Action Required | Existing Payout Account (any prior status) receives an Action Required report | Payout Account.status set to Action Required; reason recorded | FEAT-28.SPEC-002 shows the prominent banner; go-live gate signals not-satisfied for future evaluations; FEAT-28.SPEC-007 fires with "needs action" | FEAT-28.SPEC-002, FEAT-28.SPEC-007, FEAT-15 |
| Status updated to Disconnected | The capability reports the account disconnected | Payout Account.status set to Disconnected | FEAT-28.SPEC-002 shows an equivalent needs-action treatment; go-live gate signals not-satisfied | FEAT-28.SPEC-002, FEAT-15 |
| Stale/out-of-order report discarded | Reported event time is not newer than the Payout Account's last-recorded event time | None | No user feedback -- the discard is silent, per the processor-authoritative resolution rule | None |
| Report for a non-existent, ineligible Payout Account | FEAT-28.SPEC-004's eligibility check fails at creation time (e.g., a second account attempted for the same Pro) | No Payout Account created or updated | FEAT-28.SPEC-001/FEAT-28.SPEC-006 show the eligibility rejection message from FEAT-28.SPEC-004 | FEAT-28.SPEC-004 |

## Data Model

**Reads:** Pro Account -- country, currency (for the eligibility check at creation, per FEAT-28.SPEC-004).
**Creates:** Payout Account -- processor_account_reference, status, country, currency, on first-ever reported outcome for a Pro Account.
**Updates:** Payout Account -- status (and, for Action Required, the recorded reason), on every subsequent reported status change.
**Deletes:** None -- this automation never deletes a Payout Account; disconnection is represented as a status value, not a deletion, consistent with the dependency map's Payout Account lifecycle.

## Business Rules

- The processor-reported status is authoritative and resolved last-write-wins by the processor's own event time (dependency map, Payout Account Contention) -- this automation never lets a locally-cached or stale report overwrite a newer one.
- XBR-06: Active is the only status from which a deposit can be taken; this automation is the sole writer of that status, so FEAT-07's own eligibility check (FEAT-28.SPEC-004) always reads a value this automation last set.
- FEAT-28.SPEC-004 governs eligibility at Payout Account creation (one per Pro, country/currency match); this automation defers to that spec's rule rather than re-implementing it.
- XBR-22: every Payout Account status change is written to the append-only activity record (FEAT-16), giving the Pro and Support a durable trail of when the account's standing changed.
- XBR-26: a status change to or from Active is exactly the signal FEAT-15's go-live check consumes; this automation never itself decides whether the booking link goes live -- it only reports the precondition's current truth.

## Edge Cases

- **The capability reports the same status twice in a row (e.g., two Active reports)** -- The second report is a no-op: status is already Active, no further go-live signal or notification fires, per Deduplication in FEAT-28.SPEC-007.
- **Two status reports for the same Payout Account arrive out of order (an older Active report arrives after a newer Action Required report)** -- The event-time comparison in Processing Logic step 3 discards the stale Active report; the Payout Account remains Action Required, matching the capability's true, more recent state.
- **A report arrives for a Pro Account that has since closed (FEAT-29)** -- The report is still applied to the Payout Account record (which is soft-removed only as part of the 30-day cooling-off closure, per the Brief's CRUD matrix), but no notification fires since there is no active Pro session to notify.
- **Concurrent trigger firing (two status reports for the same Payout Account arrive at effectively the same time)** -- Each is processed against its own event time independently; the one with the later event time wins regardless of arrival order, per the last-write-wins resolution rule, so the two reports never race to an inconsistent final state.
- **A status report arrives while a previous report for the same Payout Account is still being processed** -- Processing for one Payout Account is serialized: the second report waits for the first to finish applying its status change before its own event-time comparison runs, so no two updates to the same record are ever applied out of the intended order.
- **The very first report for a brand-new Payout Account is itself Action Required (e.g., a bank detail rejected on first submission)** -- The Payout Account is still created (per Outcome Definitions), with status Action Required from creation; FEAT-28.SPEC-001 shows this exactly as it shows a later-arising Action Required condition.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-28.SPEC-006 (Payout Account Connection & Verification) | Triggered by (inbound) | Every reported outcome and status change originates from that integration's Inbound Events |
| FEAT-28.SPEC-004 (Payout Account Eligibility & Constraints) | References (inbound) | Eligibility rule applied at Payout Account creation |
| FEAT-28.SPEC-001 (Payout Account Connection) | Affects (outbound) | Displays the first recorded status |
| FEAT-28.SPEC-002 (Payout Status & Money Dashboard) | Affects (outbound) | Displays the ongoing status and its banner |
| FEAT-28.SPEC-007 (Payout Status Notification) | Triggers (outbound) | Fired on transitions to Active and to Action Required |
| FEAT-15.SPEC-005 (Go-Live Evaluation & Booking Link Activation) -- within FEAT-15 (Pro Onboarding & Setup Wizard) | Affects (outbound) | Consumes the go-live gate signal (XBR-06, XBR-26) |
| FEAT-07 (Deposit Payment at Booking) | Affects (outbound) | Reads the Active status this automation maintains before any deposit charge (XBR-06) |
| FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) | Triggers (outbound) | Every status change is recorded as an Activity Event (XBR-22) |

## Analytics and Success Signals

- **payout_account_created** (initial status: verification_pending / active) -- supports success-metrics.md: "Setup-to-Live-Link Completion"
- **payout_account_activated** (previous status) -- supports success-metrics.md: "Payout Transparency"
- **payout_account_action_required** (reason category) -- supports success-metrics.md: "Payout Transparency"
- **payout_account_status_report_discarded** (reason: stale_event_time / ineligible) -- N/A -- no Stage 2 metric measures discarded or rejected status reports; retained so the correctness bar (ASMP-26) for this automation's processor-authoritative rule is observable.

## Acceptance Criteria

**FEAT-28.SPEC-003-AC-01:** Given Talia has no Payout Account yet, when the capability first reports Verification Pending, then a Payout Account is created with that status.

**FEAT-28.SPEC-003-AC-02:** Given Talia has no Payout Account yet, when the capability first reports Active, then a Payout Account is created with status Active, the go-live gate is signaled satisfied, and FEAT-28.SPEC-007 fires.

**FEAT-28.SPEC-003-AC-03:** Given Talia's Payout Account is Verification Pending, when the capability reports Active, then status updates to Active, the go-live gate is signaled satisfied, and FEAT-28.SPEC-007 fires with the "verification complete" content.

**FEAT-28.SPEC-003-AC-04:** Given Talia's Payout Account is Active, when the capability reports Action Required, then status updates to Action Required, the go-live gate is signaled not-satisfied for future evaluations, and FEAT-28.SPEC-007 fires with the "needs action" content and reason.

**FEAT-28.SPEC-003-AC-05:** Given Talia's Payout Account is Action Required, when she resolves it through the processor's flow and the capability reports Active, then status updates to Active and FEAT-28.SPEC-007 fires again with the "verification complete" content.

**FEAT-28.SPEC-003-AC-06:** Given the capability reports Disconnected, when the report is processed, then status updates to Disconnected and the go-live gate is signaled not-satisfied.

**FEAT-28.SPEC-003-AC-07:** Given Talia's Payout Account is already Active, when a second Active report arrives, then nothing changes and no duplicate notification fires.

**FEAT-28.SPEC-003-AC-08:** Given two reports for the same Payout Account arrive out of order, when the older report's event time is earlier than the account's current recorded event time, then the older report is discarded and the account reflects the newer report's status.

**FEAT-28.SPEC-003-AC-09:** Given a candidate second Payout Account is reported for a Pro who already has one, when FEAT-28.SPEC-004's eligibility check runs, then no new Payout Account is created and the rejection is surfaced through FEAT-28.SPEC-001 or FEAT-28.SPEC-006.

**FEAT-28.SPEC-003-AC-10:** Given a status change is processed, when the write completes, then an Activity Event is recorded for that Payout Account per XBR-22.

**FEAT-28.SPEC-003-AC-11:** Given a status report arrives for a Pro Account that has since closed, when the report is processed, then the Payout Account record is updated but no notification is delivered.

**FEAT-28.SPEC-003-AC-12:** Given two status reports for the same Payout Account arrive at effectively the same time, when both are processed, then the one with the later event time is the account's final state regardless of arrival order.

**FEAT-28.SPEC-003-AC-13:** Given a report arrives for a Payout Account while a previous report for that same account is still being applied, when the second report's turn comes, then its event-time comparison runs only after the first report's update has fully applied.

**FEAT-28.SPEC-003-AC-14:** Given a brand-new Payout Account's very first report is Action Required, when it is processed, then the account is created directly with status Action Required and FEAT-28.SPEC-007 fires with the "needs action" content.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 (first outcome, status change, resolution outcome) | 3 |
| Outcome Paths | 7 | 7 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Payout Account Eligibility & Constraints

## Overview

**Name:** Payout Account Eligibility & Constraints
**ID:** FEAT-28.SPEC-004
**Type:** Logic/Rule
**Purpose:** Governs one-payout-account-per-Pro, country/currency matching to the Pro Account, and the standing zero-Chairtime-fee rule that the money list and go-live gate both depend on.
**Parent Feature:** FEAT-28 -- Payout Account Connection & Payout Visibility
**Governed Entity:** Payout Account

## Scope and Non-Goals

**In Scope:**
- Field-level rules for every Payout Account field
- The one-account-per-Pro-Account constraint
- The country/currency match rule against the Pro Account (XBR-25)
- The standing zero-Chairtime-fee rule (XBR-07) that governs every deduction ever shown against this entity
- Authorization rules for every action on the Payout Account, per role

**Non-Goals:**
- Writing the Payout Account's status field from processor reports -- owned by FEAT-28.SPEC-003 (Payout Account Status Processing); this spec defines the eligibility gate that automation applies at creation, not the write itself
- Composing the money list or deriving net-per-period -- owned by FEAT-28.SPEC-005 (Money List Composition & Net Calculation); this spec only fixes the zero-fee rule that spec's derivation must respect
- The visual treatment of the eligibility rejection message -- owned by FEAT-28.SPEC-001 and FEAT-28.SPEC-006, which display the exact denied text this spec defines
- Timezone rules -- owned by FEAT-27 (Pro Profile & Booking Page Settings) under XBR-25; this spec only checks that the Payout Account's currency matches the Pro Account's, not timezone

## Governed Entity

**Entity:** Payout Account
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| processor_account_reference | text | Reference to the Pro's account with the payment-processing capability; bank and identity details stay with the capability |
| status | enum | Not Connected \| Verification Pending \| Active \| Action Required \| Disconnected |
| country | enum | Must match the Pro Account's country |
| currency | enum | Must match the Pro Account's currency |
| payout_schedule | derived | The processor's own reported payout cadence -- no validation beyond data type |
| recent payouts | derived | The processor's own reported list of recent bank transfers -- no validation beyond data type |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-28.SPEC-001 | Payout Account Connection | On the first connection handoff; authorization on screen entry |
| FEAT-28.SPEC-002 | Payout Status & Money Dashboard | On every display of status and money-list deductions; authorization on screen entry |
| FEAT-28.SPEC-003 | Payout Account Status Processing | On every candidate Payout Account creation, before writing the record |
| FEAT-28.SPEC-006 | Payout Account Connection & Verification | On the outbound handoff and on the inbound eligibility rejection from the capability |
| FEAT-07 | Deposit Payment at Booking | Reads the Active-status precondition before any deposit charge (XBR-06) |
| FEAT-15 | Pro Onboarding & Setup Wizard | Reads the Active-status precondition for the go-live check (XBR-06, XBR-26) |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| processor_account_reference | Required, non-empty; set only from the capability's own report, never entered by the Pro | Always, once a Payout Account exists | On Payout Account creation | N/A -- not a user-facing field; no direct entry point exists for it | Yes |
| status | Must be one of the five defined enum values | Always | On every write (FEAT-28.SPEC-003) | N/A -- not user-entered; an unrecognized value from the capability is treated as a processing error, not a validation failure shown to the Pro | Yes |
| country | Must exactly match the Pro Account's country at the time of connection | Always, checked at Payout Account creation | On Payout Account creation (via the handoff outcome) | "Your payout account's country doesn't match your Chairtime account's country. Payout accounts must be in the same country you signed up with." | Yes |
| currency | Must exactly match the Pro Account's currency at the time of connection | Always, checked at Payout Account creation | On Payout Account creation (via the handoff outcome) | "Your payout account's currency doesn't match your Chairtime account's currency. Payout accounts must use the same currency as your Chairtime account." | Yes |
| payout_schedule | No validation beyond data type | Always | -- | -- | -- |
| recent payouts | No validation beyond data type | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| One Payout Account per Pro Account | Pro Account reference (implicit), Payout Account existence | A Payout Account can be created for a given Pro Account only when no Payout Account already exists for it | "You already have a connected payout account. Contact support if you need to change it." (shown only in the unreachable case this check is ever triggered outside the normal single-connection flow, since FEAT-28.SPEC-001 is never re-shown once a Payout Account exists) |
| Country/currency match to Pro Account | country, currency, Pro Account.country, Pro Account.currency | Both country and currency on the Payout Account must equal the Pro Account's values at the moment of connection | See Field Validation Rules above (separate messages per field, shown together if both mismatch) |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|----------------------------------------------|
| Connect a Payout Account | The Pro (Talia) | Only when no Payout Account yet exists for this Pro Account | FEAT-28.SPEC-001 is not shown a second time once a Payout Account exists; a direct attempt shows "You already have a connected payout account." |
| View Payout Account status | The Pro (Talia) | Always, own account only | -- |
| View Payout Account status | Platform Operator (Support) | Always, status and money list only (Access Matrix, Payouts = View) | -- |
| View bank or identity details | Platform Operator (Support) | Never -- the product never holds these details at all (ASMP-31) | No control to view them exists anywhere in the product; the interface has no such field to show |
| View bank or identity details | The Client | Never -- Clients have no access to Payouts at all (Access Matrix, Payouts = None) | The Payouts area is not reachable by a Client under any navigation path |
| Resolve an Action Required flag | The Pro (Talia) | Only through the processor's own flow (FEAT-28.SPEC-006), never by editing a field inside the product | "Resolve now" hands Talia directly to the processor; the product has no in-app form to clear this flag itself |
| Resolve an Action Required flag | Platform Operator (Support) | Never | No "Resolve now" or equivalent control is shown to Support, per SC-05 |
| Disconnect a Payout Account | The Pro (Talia) | Only through the processor's own flow, never through a Chairtime control (the dependency map's Payout Account Contention: the Pro acts through the processor's own flow) | No standalone "disconnect" control exists inside the product; disconnection is a status the processor reports, not an action taken here |
| Delete a Payout Account record | Nobody, directly | Never inside this feature -- removed only as part of Pro Account closure (FEAT-29's 30-day cooling-off), per the Brief's CRUD matrix | No delete control exists anywhere for this entity |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| country | Defaulted from the Pro Account's country at the moment of first connection | On Payout Account creation | No -- the Pro changes their Chairtime account's country/currency through FEAT-27, not through this entity |
| currency | Defaulted from the Pro Account's currency at the moment of first connection | On Payout Account creation | No |
| Chairtime's own fee on any deposit, balance, or tip | Always exactly zero (XBR-07); never a field with a stored value, since it never varies | Always | No -- this is a standing product rule, not a per-account setting |

## Business Rules

- XBR-06: no deposit can be taken, and the booking link cannot go live, unless the Payout Account's status is Active; every reader of this entity (FEAT-07, FEAT-15) treats Active as the sole qualifying state.
- XBR-07: money never rests with the platform -- Chairtime's fee on deposits, balances, and tips is always zero; the only deduction ever shown against a Payout Account's transactions is the payment processor's own card fee (FEAT-28.SPEC-005 enforces this at display time; this spec is the standing rule's single source of truth).
- XBR-25: the Payout Account's country and currency must match the Pro Account's; once the Pro Account's currency locks at its first deposit (XBR-25), the Payout Account's currency is likewise fixed for the life of the account.
- The Validation & Limits field (product-features.md, FEAT-28) and SC-20 fix exactly one Payout Account per Pro Account, spanning exactly one country and currency; multi-country payout support is out of scope for this feature (deferred to the geography phase-in).
- A Pro never edits bank details inside Chairtime (dependency map, Payout Account Contention); every write to processor_account_reference and status originates from the capability's own report, applied by FEAT-28.SPEC-003.

## Edge Cases

- **A Pro's Chairtime account country/currency changes after the Payout Account was already connected** -- Cannot occur under normal operation: FEAT-27 locks currency at the first deposit (XBR-25) and country is not user-editable after account creation, so no post-connection mismatch scenario exists; if the capability itself ever reports a changed country/currency for an existing account, that report is treated as an Action Required condition requiring the Pro to reconnect, rather than silently accepted.
- **A candidate connection reports a country/currency combination Chairtime does not yet support (outside the US at MVP)** -- Rejected with the same country mismatch message; SC-20's geography phase-in note applies -- this rule does not change until that phase-in ships.
- **Two connection attempts race for the same Pro Account (e.g., two browser tabs)** -- The first committed report wins and creates the Payout Account; the second is evaluated against the one-account-per-Pro rule and rejected, consistent with reject-with-refresh handling elsewhere in the product.
- **The zero-Chairtime-fee rule is checked against a transaction that somehow carries a non-zero platform fee value (a data anomaly)** -- Treated as a processing error, never displayed to the Pro as a fee; FEAT-28.SPEC-005's composition never renders a Chairtime fee line under any circumstance, per XBR-07.
- **Support attempts to view or infer bank/identity details indirectly (e.g., by cross-referencing the processor_account_reference)** -- The reference itself carries no bank or identity information; it is an opaque pointer to the capability's own record, so no inference is possible from data the product holds.
- **A Pro Account closes while its Payout Account is Action Required** -- The Payout Account is not independently deleted; it is soft-removed only as part of FEAT-29's 30-day cooling-off closure, with no cascade to already-created Deposit Transactions, per the explicit non-goal in the Brief.

## Acceptance Criteria

**FEAT-28.SPEC-004-AC-01:** Given Talia has no Payout Account, when a connection is reported with country and currency matching her Pro Account, then the Payout Account is created successfully.

**FEAT-28.SPEC-004-AC-02:** Given a candidate connection reports a country different from Talia's Pro Account, when the eligibility check runs, then no Payout Account is created and "Your payout account's country doesn't match your Chairtime account's country. Payout accounts must be in the same country you signed up with." is shown.

**FEAT-28.SPEC-004-AC-03:** Given a candidate connection reports a currency different from Talia's Pro Account, when the eligibility check runs, then no Payout Account is created and the currency mismatch message is shown.

**FEAT-28.SPEC-004-AC-04:** Given Talia already has a Payout Account, when a second connection attempt is made, then it is rejected with "You already have a connected payout account. Contact support if you need to change it."

**FEAT-28.SPEC-004-AC-05:** Given a captured deposit is composed for the money list, when the deduction is shown, then only the processor's own card fee appears -- never a Chairtime fee line.

**FEAT-28.SPEC-004-AC-06:** Given Talia's Payout Account status is anything other than Active, when FEAT-07 checks eligibility for a deposit charge, then the charge is blocked per XBR-06.

**FEAT-28.SPEC-004-AC-07:** Given Talia's Payout Account status is Active, when FEAT-15 evaluates go-live readiness, then this precondition is satisfied.

**FEAT-28.SPEC-004-AC-08:** Given the Client (Riley) attempts to reach any Payouts-related screen, when the attempt is made, then no such navigation path exists for her role.

**FEAT-28.SPEC-004-AC-09:** Given Platform Operator (Support) views a Pro's Payout Account, when Support looks for bank or identity details, then none are shown, since the product never holds them.

**FEAT-28.SPEC-004-AC-10:** Given Talia's Payout Account is Action Required, when she looks for an in-app control to clear the flag directly, then none exists -- only "Resolve now," which hands her to the processor's own flow.

**FEAT-28.SPEC-004-AC-11:** Given Support views a Pro's Action Required banner, when Support looks for a resolution control, then none is shown, per SC-05.

**FEAT-28.SPEC-004-AC-12:** Given Talia wants to disconnect her Payout Account, when she looks for a Chairtime control to do so, then none exists -- disconnection is reflected only from the processor's own report.

**FEAT-28.SPEC-004-AC-13:** Given Talia's Pro Account currency locks at her first deposit (XBR-25), when the Payout Account's currency is evaluated afterward, then it remains fixed and is never independently editable.

**FEAT-28.SPEC-004-AC-14:** Given a candidate connection reports a country/currency combination outside the US at MVP, when the eligibility check runs, then it is rejected with the same country mismatch message, per SC-20's phase-in note.

**FEAT-28.SPEC-004-AC-15:** Given two connection attempts race for the same Pro Account, when both are evaluated, then only the first-committed report creates the Payout Account and the second is rejected under the one-account-per-Pro rule.

**FEAT-28.SPEC-004-AC-16:** Given a Pro Account closes while its Payout Account is Action Required, when FEAT-29's closure process runs, then the Payout Account is soft-removed only as part of that closure, with no independent deletion beforehand.

**FEAT-28.SPEC-004-AC-17:** Given Talia's Payout Account has never been connected (Not Connected), when FEAT-07 checks eligibility, then the deposit charge is blocked exactly as it would be for any other non-Active status.

**FEAT-28.SPEC-004-AC-18:** Given the capability reports processor_account_reference and status values for a new account, when FEAT-28.SPEC-003 processes them, then no user-facing validation error can occur on these fields, since they are never directly entered by the Pro.

**FEAT-28.SPEC-004-AC-19:** Given a data anomaly reports a non-zero platform fee on a transaction, when the money list composes that row, then no Chairtime fee line is ever rendered, per XBR-07.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 6 | 6 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 9 | 9 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Money List Composition & Net Calculation

## Overview

**Name:** Money List Composition & Net Calculation
**ID:** FEAT-28.SPEC-005
**Type:** Logic/Rule
**Purpose:** Defines how deposits, refunds, processor fees, and payouts are assembled into the money list and how net amount received per period is derived.
**Parent Feature:** FEAT-28 -- Payout Account Connection & Payout Visibility
**Governed Entity:** Money List row (a derived, read-only composition over Deposit Transaction, Balance Payment, and the Payout Account's reported payouts -- not a stored entity of its own)

## Scope and Non-Goals

**In Scope:**
- Row-composition rules for each money-list row type (deposit, refund, processor fee display, payout)
- Ordering, grouping, and the "in progress" display rule for a refund that has not yet cleared
- The net-amount-received-per-period derivation
- Authorization rules for who may view the composed list

**Non-Goals:**
- Creating or updating the underlying Deposit Transaction, Balance Payment, or Payout Account records -- owned by FEAT-07, FEAT-09, FEAT-30, FEAT-22, and FEAT-28.SPEC-003 respectively; this spec only reads and composes them for display
- The screen that renders the composed list -- owned by FEAT-28.SPEC-002 (Payout Status & Money Dashboard); this spec supplies the assembled data, not the layout
- Determining Payout Account eligibility or the zero-Chairtime-fee standing rule itself -- owned by FEAT-28.SPEC-004; this spec applies that rule when composing a deposit row's deduction, it does not define the rule
- Partial or tiered refund display -- excluded per SC-18: the underlying cancellation rule is binary, so no partial-percentage refund row is ever composed

## Governed Entity

**Entity:** Money List row -- a derived, read-only view composed at display time from three source entities. No independent field list of its own; instead, each row type's source fields are enumerated below.

**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| Deposit Transaction.amount / currency | number / enum | The captured deposit amount and currency |
| Deposit Transaction.status | enum | Authorized \| Captured \| Applied \| Refunded \| Refund in Progress \| Forfeited \| Disputed |
| Deposit Transaction.processor_fee | number | The processor's own card fee on this transaction |
| Deposit Transaction.outcome_reason / timestamps | text / date | Which rule or action produced the current status, and when |
| Balance Payment.amount | number | Price minus deposit, from v1 (FEAT-22) |
| Balance Payment.tip | number | Optional, non-negative, Later (FEAT-23) |
| Balance Payment.state | enum | Attempted \| Succeeded \| Failed \| Refunded |
| Payout Account.payout_schedule | derived | The processor's reported payout cadence |
| Payout Account.recent payouts | derived | The processor's reported list of recent bank transfers (amount, date, status) |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-28.SPEC-002 | Payout Status & Money Dashboard | On every money list load and refresh; authorization on screen entry |
| FEAT-25 | Booking & Revenue Insights | Reads this spec's composed data and net derivation once that feature exists |

## Field Validation Rules

Every source field is written by its owning feature (FEAT-07, FEAT-09, FEAT-30, FEAT-22, FEAT-28.SPEC-003) under that feature's own validation rules; this spec only composes and derives from already-valid data, so no field below carries composition-time validation of its own beyond confirming it is present. The Cross-Field Rules section governs how these fields combine into displayed rows, which is where this spec's own logic lives.

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| Deposit Transaction.amount / currency | No validation beyond data type -- validity is owned by FEAT-07.SPEC-003 at capture time | Always | -- | -- | -- |
| Deposit Transaction.status | No validation beyond data type -- validity is owned by FEAT-07, FEAT-09, FEAT-11, FEAT-30 as they write it | Always | -- | -- | -- |
| Deposit Transaction.processor_fee | Treated as zero for display if missing or null (see Edge Cases), rather than failing composition | When the field is absent | On composition | N/A -- not a user-facing validation error; a display fallback, not a blocking failure | No |
| Deposit Transaction.outcome_reason / timestamps | No validation beyond data type | Always | -- | -- | -- |
| Balance Payment.amount / tip / state | No validation beyond data type -- validity is owned by FEAT-22 (and FEAT-23 for tip) | Always | -- | -- | -- |
| Payout Account.payout_schedule | No validation beyond data type -- shown exactly as the processor reports it | Always | -- | -- | -- |
| Payout Account.recent payouts | No validation beyond data type -- shown exactly as the processor reports it | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Deposit row composition | Deposit Transaction.amount, processor_fee, status | Every Deposit Transaction in status Captured, Applied, Refunded, Refund in Progress, Forfeited, or Disputed produces exactly one deposit row, showing the gross amount, the processor_fee as the only deduction, and the net (amount minus processor_fee); a Deposit Transaction still Authorized (not yet captured) produces no row | N/A -- display composition, not a user-facing error |
| Refund row composition | Deposit Transaction.status, outcome_reason, timestamps | A Deposit Transaction whose status is Refunded or Refund in Progress additionally produces a linked refund row beneath its deposit row, showing the refunded amount, the refund date (if Refunded) or "in progress" (if Refund in Progress), and referencing the original deposit row | N/A |
| Forfeited deposit display | Deposit Transaction.status = Forfeited | A forfeited deposit shows no refund row; the deposit row itself shows "Kept" as its running status, per the binary outcome rule (SC-18) | N/A |
| Disputed deposit display | Deposit Transaction.status = Disputed | The deposit row (and its refund row, if any) carries a "Disputed" overlay alongside its existing status, per XBR-22; the underlying outcome (kept, refunded, or refund in progress) remains visible beneath the overlay | N/A |
| Balance payment row composition (from v1) | Balance Payment.amount, tip, state | Once FEAT-22 exists, a Balance Payment in state Succeeded produces a balance-payment row; a Refunded state additionally produces a linked refund row, following the same pattern as deposits | N/A |
| Payout row composition | Payout Account.recent payouts | Each entry in the processor's reported recent payouts produces one payout row (amount, date, status: upcoming or completed); no Chairtime-side computation of payout amounts occurs -- the processor's own reported figures are shown as-is | N/A |
| Net-per-period derivation | Deposit Transaction.amount, processor_fee, status; Balance Payment.amount, tip, state (from v1) | Net amount received for a selected period = sum of (amount minus processor_fee) for every deposit row with status Captured, Applied, or Refund in Progress dated within the period, minus the amount of every completed refund dated within the period, plus (from v1) the net of any Balance Payment rows in the same period; a Forfeited deposit's full amount minus its processor_fee counts toward net, since it is kept | N/A |
| Zero-Chairtime-fee enforcement at composition | Deposit Transaction.processor_fee | The composition logic has no field or code path that renders a Chairtime-owned fee on any row, per XBR-07 and FEAT-28.SPEC-004's standing rule | N/A |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|----------------------------------------------|
| View composed money list | The Pro (Talia) | Own Payout Account's data only, always | -- |
| View composed money list | Platform Operator (Support) | Same composed data Talia sees (Access Matrix, Payouts = View), never bank or identity details (none of which are among this spec's source fields in any case) | -- |
| View composed money list | The Client (Riley) | Never -- Clients have no access to Payouts at all | The money list is not reachable by any Client navigation path |
| Trigger recomposition (open the money list / change the period selector) | The Pro, Platform Operator (Support) | Always, read-only action | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Deposit row net amount | amount minus processor_fee | Computed at every composition | No |
| Net amount received per period | See the net-per-period cross-field rule above | Recomputed whenever the money list loads or the period selector changes | Yes -- the Pro (and Support, view-only) may change the selected period, which recomputes the figure; the underlying rows never change |
| Default period | This month, from the first day of the current calendar month in the Pro's timezone (XBR-25) to the current date | On money list load, before any period change | Yes -- changeable via the period selector on FEAT-28.SPEC-002 |
| Refund row's "in progress" label | Derived directly from Deposit Transaction.status = Refund in Progress | Whenever a linked refund row is composed | No -- the label always mirrors the source status exactly |

## Business Rules

- XBR-07: Chairtime's own fee is always zero; the only deduction this composition ever shows on any row is the payment processor's own card fee, per FEAT-28.SPEC-004's standing rule.
- FEAT-23.SPEC-003 (Tip Payout & Refund Rule) is enforced here: a succeeded Balance Payment's `tip` contributes in full to its row and to the net-per-period figure, with no fee line deducted.
- XBR-10: a refund that cannot complete immediately is shown as "in progress," never as failed or silently dropped, and the underlying retry (owned by FEAT-09/FEAT-30) is what eventually moves the row to its completed refund state.
- SC-18: the cancellation rule producing these outcomes is binary (refunded or kept); the composition never renders a partial-percentage refund row.
- XBR-22: a Disputed overlay never erases the underlying outcome; the composition shows both together.
- ASMP-22 / ASMP-27: the composition remains equally responsive as history accumulates over multiple years, and every load shows an in-place loading indicator with nothing tappable until real data has loaded, matching FEAT-28.SPEC-002's States section.

## Edge Cases

- **A Deposit Transaction transitions from Refund in Progress to Refunded while the money list is open** -- The linked refund row updates from "in progress" to its completed status and date on the list's next refresh (FEAT-28.SPEC-002), and the net-per-period figure recalculates to include it if the refund date falls in the selected period.
- **A deposit and its refund fall in different periods (deposit last month, refund this month)** -- Each event counts toward the net figure of the period its own date falls in; the deposit's full net (minus fee) counted toward last month's total is not retroactively reversed -- this month's total instead reflects the refund as a negative entry.
- **A Deposit Transaction is Disputed while also Refund in Progress** -- Both overlays compose onto the same row: the "in progress" refund status and the "Disputed" overlay are shown together, since a dispute never erases the underlying refund-in-progress outcome (XBR-22).
- **Balance Payment rows before FEAT-22 ships** -- No Balance Payment rows or net contribution are composed at all; the money list and net figure are computed entirely from Deposit Transaction data until FEAT-22 exists, consistent with the Brief's Non-Goals.
- **The processor reports a recent payout for an amount that does not match the sum of the Deposit Transaction rows composed for that period** -- The payout row is shown exactly as the processor reports it (amount, date, status), since the processor's own payout timing may span multiple periods or batching windows; no reconciliation or discrepancy flag is computed by this spec, and the payout row and the deposit/refund rows are never forced to reconcile to the same total.
- **The money list is opened for a Pro with multiple years of history (per ASMP-22)** -- Composition and the net-per-period derivation operate only over the selected period's rows plus the always-shown full history list; performance does not degrade as total history grows, since the period derivation never requires scanning the Pro's entire multi-year history to compute a shorter period's figure.

## Acceptance Criteria

**FEAT-28.SPEC-005-AC-01:** Given a Deposit Transaction is Captured, when the money list composes, then a deposit row shows the gross amount, the processor's own fee, and the net (amount minus fee).

**FEAT-28.SPEC-005-AC-02:** Given a Deposit Transaction is Authorized but not yet Captured, when the money list composes, then no row is shown for it.

**FEAT-28.SPEC-005-AC-03:** Given a Deposit Transaction is Refunded, when the money list composes, then a linked refund row appears beneath the deposit row showing the refunded amount and refund date.

**FEAT-28.SPEC-005-AC-04:** Given a Deposit Transaction is Refund in Progress, when the money list composes, then the linked refund row shows "in progress," never a failure state.

**FEAT-28.SPEC-005-AC-05:** Given a Deposit Transaction is Forfeited, when the money list composes, then the deposit row shows "Kept" and no refund row is composed.

**FEAT-28.SPEC-005-AC-06:** Given a Deposit Transaction is Disputed and also Refund in Progress, when the money list composes, then the row shows both the "in progress" refund status and the "Disputed" overlay together.

**FEAT-28.SPEC-005-AC-07:** Given the processor reports a recent payout, when the money list composes, then a payout row shows the amount, date, and status (upcoming or completed) exactly as reported.

**FEAT-28.SPEC-005-AC-08:** Given Talia opens the money list with the default period, when the net figure is computed, then it covers from the first day of the current calendar month (her timezone) to today.

**FEAT-28.SPEC-005-AC-09:** Given Talia changes the period selector, when the new period is applied, then the net figure recomputes for that period and the full money list itself is unaffected.

**FEAT-28.SPEC-005-AC-10:** Given a deposit is captured in one period and refunded in a later period, when both periods' net figures are computed, then the deposit's net counts toward the period it was captured in and the refund counts as a negative entry in the period it completed in.

**FEAT-28.SPEC-005-AC-11:** Given a deposit row is composed, when its deduction is displayed, then only the processor's own card fee ever appears -- never a Chairtime fee line.

**FEAT-28.SPEC-005-AC-12:** Given FEAT-22 does not yet exist for this Pro, when the money list composes, then no Balance Payment rows or net contribution appear.

**FEAT-28.SPEC-005-AC-13:** Given the Client (Riley) attempts to view any Pro's money list, when the attempt is made, then no such access path exists for her role.

**FEAT-28.SPEC-005-AC-14:** Given Support views a Pro's money list, when composed, then Support sees exactly the same rows Talia sees, since none of this spec's source fields are bank or identity data.

**FEAT-28.SPEC-005-AC-15:** Given the processor's reported recent payout amount does not match the sum of a period's deposit rows, when the money list composes, then the payout row is shown exactly as reported with no reconciliation discrepancy flag.

**FEAT-28.SPEC-005-AC-16:** Given Talia's history spans multiple years, when she selects a one-month period, then the net figure computes only over that period's rows without scanning her full history.

**FEAT-28.SPEC-005-AC-17:** Given a refund transitions from in progress to completed while the list is open, when the list next refreshes, then the row updates and the net-per-period figure recalculates to reflect it if the refund date falls in the selected period.

**FEAT-28.SPEC-005-AC-18:** Given a cancellation rule produces a refund outcome, when the money list composes it, then only a full refund or a kept deposit is ever shown -- never a partial-percentage row, per SC-18.

**FEAT-28.SPEC-005-AC-19:** Given a Deposit Transaction's processor_fee field somehow carries a null or missing value, when the money list composes its row, then the row still renders with the amount and net shown as the amount itself (fee treated as zero for display), rather than the row failing to compose.

**FEAT-28.SPEC-005-AC-20:** Given a Deposit Transaction that is Disputed but not refunded or forfeited, when the money list composes, then the deposit row shows its existing status (e.g., Captured) with the Disputed overlay, and the underlying outcome remains visible beneath it.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 7 | 7 |
| Cross-Field Rules | 8 | 8 |
| Authorization Rules | 4 | 4 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Integration Spec: Payout Account Connection & Verification

## Overview

**Name:** Payout Account Connection & Verification
**ID:** FEAT-28.SPEC-006
**Type:** Integration
**Purpose:** Handles the outbound handoff to, and inbound status/money data from, the payment-processing capability for account connection, identity and bank verification, action-required resolution, and payout/fee reporting.
**Parent Feature:** FEAT-28 -- Payout Account Connection & Payout Visibility

## Scope and Non-Goals

**In Scope:**
- Handing Talia into the payment-processing capability's own identity and bank verification flow, both for the first connection and for resolving a flagged Action Required condition
- Receiving the capability's reported connection and status outcomes (Verification Pending, Active, Action Required, Disconnected)
- Receiving the capability's reported payout schedule and recent payouts data for display on the money list
- User-facing behavior when the capability is slow, unavailable, or rejects the connection or resolution attempt
- Disclosure to Talia about what data is shared with the capability during connection

**Non-Goals:**
- Writing the Payout Account record from the reported outcomes -- owned by FEAT-28.SPEC-003 (Payout Account Status Processing); this spec only reports what the capability says, it does not decide what to persist
- Checking eligibility (one account per Pro, country/currency match) -- owned by FEAT-28.SPEC-004; this spec carries the capability's own rejection when its checks fail, it does not perform the check itself
- The original deposit charge and its own payout routing -- owned by FEAT-07.SPEC-005 (Card Deposit Charge & Payout Routing), which routes an already-captured deposit to an already-active account; this spec only connects and verifies the account itself
- Refund requests -- owned by FEAT-09.SPEC-005 and FEAT-30's own refund integration behavior against the same capability; this spec covers only account connection, verification, and payout/fee reporting

## Capability Category

**Category:** Payment processing
**Dependency Source:** ASMP-31 -- "Payment-processing capability -- required to take client deposits, verify each pro's identity and bank details for a connected payout account, pay deposits out to the pro, issue refunds, notify the product of card-issuer disputes, and bill the pro's own monthly subscription." (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Payment processing -- connected payout accounts with identity and bank verification" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-28, FEAT-15, FEAT-07, FEAT-22, FEAT-23; Integration Specs column names FEAT-28.SPEC-006 directly for "hand-off into the processor's own identity and bank verification, action-required resolution, and inbound account-status, payout and processor-fee reporting; FEAT-15 reaches it through its getting-paid step")
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision. BRIEF.md's Constraints establish only the hard boundary that bank and identity details are never held by the product's own code, not a named vendor.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Talia connects a payout account through the capability's own secure identity and bank verification flow, never typing bank details into Chairtime itself | Connect a payout account during onboarding through the payment processor's own secure identity and bank verification | FEAT-28.SPEC-001 (Payout Account Connection) |
| Talia sees her payout account status -- verification pending, active, or action required | See payout account status | FEAT-28.SPEC-001, FEAT-28.SPEC-002 (Payout Status & Money Dashboard) |
| Talia resolves a flagged verification or bank-detail problem through a direct path back into the processor's own flow | Resolve a flagged verification or bank-detail problem | FEAT-28.SPEC-002 |
| Talia's money list shows the processor's own reported payout schedule and recent payouts | See a simple money list -- upcoming and past payouts to the bank | FEAT-28.SPEC-002, FEAT-28.SPEC-005 (Money List Composition & Net Calculation) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Pro Account reference | Pro Account -- an internal reference sufficient to tie the capability's account to this Pro | Talia taps "Connect payout account" (FEAT-28.SPEC-001) or "Resolve now" (FEAT-28.SPEC-002) | Lets the capability associate the connected account, and any resolution, with the correct Pro |
| Country and currency | Pro Account -- country, currency | Same moments as above | The capability must verify the new payout account against the same country and currency Chairtime already has on file (XBR-25) |
| Identity and bank details Talia enters | Not a product entity -- entered directly into the capability's own verification flow and never received by the product's own code, per ASMP-31 | While Talia completes the capability's own flow | The capability needs these details to verify identity and connect the bank account; the product never touches or stores them |

Talia's client list, bookings, service prices, and every other Pro Account or Booking field never leave the product through this integration -- only a Pro Account reference and the country/currency needed to verify the new payout account are shared.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Connection/status outcome (Verification Pending / Active / Action Required / Disconnected) | The capability resolves a connection or resolution attempt, or reports a later status change | Consumed by FEAT-28.SPEC-003, which writes Payout Account -- status |
| processor_account_reference | The capability confirms the account connection | Payout Account -- processor_account_reference (written by FEAT-28.SPEC-003) |
| Action Required reason (plain-language category) | The capability reports the account needs action | Consumed by FEAT-28.SPEC-003 and surfaced on FEAT-28.SPEC-002's banner; not persisted as a separate field, since the current reason is always the latest report |
| payout_schedule and recent payouts | The capability reports its own payout cadence and history | Payout Account -- payout_schedule, recent payouts (consumed for display by FEAT-28.SPEC-005) |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Connection outcome reported (Verification Pending or Active) | The capability resolves Talia's connection attempt from FEAT-28.SPEC-001 | Consumed by FEAT-28.SPEC-003, which creates or updates the Payout Account | FEAT-28.SPEC-001 shows the reported status | FEAT-28.SPEC-003 |
| Status changed to Active | The capability confirms verification is complete (first connection or after resolving Action Required) | Consumed by FEAT-28.SPEC-003, which sets status to Active | FEAT-28.SPEC-002's banner clears; FEAT-28.SPEC-007 fires the "verification complete" notification | FEAT-28.SPEC-003, FEAT-28.SPEC-007, FEAT-15 (go-live evaluation) |
| Status changed to Action Required | The capability flags a problem (e.g., a rejected bank detail) | Consumed by FEAT-28.SPEC-003, which sets status to Action Required and records the reason | FEAT-28.SPEC-002 shows the prominent banner with the plain-language reason; FEAT-28.SPEC-007 fires the "needs action" notification | FEAT-28.SPEC-003, FEAT-28.SPEC-007 |
| Status changed to Disconnected | The capability reports the account is disconnected | Consumed by FEAT-28.SPEC-003, which sets status to Disconnected | FEAT-28.SPEC-002 shows an equivalent needs-action treatment | FEAT-28.SPEC-003 |
| Payout schedule / recent payouts updated | The capability reports its own payout cadence or a new completed/upcoming payout | Consumed for display by FEAT-28.SPEC-005 | Talia's money list (FEAT-28.SPEC-002) shows the updated payout row on its own refresh cadence | FEAT-28.SPEC-005, FEAT-28.SPEC-002 |
| Payout figure reported | The capability reports a new or updated payout figure for a Pro (a new completed or upcoming payout, or a changed payout amount) | Consumed for display by FEAT-28.SPEC-005; also forwarded as an external-event trigger to FEAT-25.SPEC-004, which reads it and makes no change to its own aggregates (payout figures are read by FEAT-25 through FEAT-28.SPEC-005) | Talia's money list (FEAT-28.SPEC-002) shows the updated payout row on its own refresh cadence | FEAT-28.SPEC-005, FEAT-28.SPEC-002, FEAT-25.SPEC-004 |
| Resolution outcome reported | Talia completes the capability's own resolution flow for an Action Required account | Consumed by FEAT-28.SPEC-003, which sets status to Active (resolved) or leaves it Action Required (still unresolved, with a possibly updated reason) | FEAT-28.SPEC-002 reflects the new status | FEAT-28.SPEC-003 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-28.SPEC-001 (Payout Account Connection) | "Connect payout account" shows a brief loading state; past a short threshold it adds "Still working -- this is taking a little longer than usual." | The button shows "We couldn't start the connection process. Try again." (the screen's Handoff Failed state); no Payout Account record is created | The capability's own eligibility rejection (e.g., country/currency mismatch, per FEAT-28.SPEC-004) is surfaced in plain language on this screen; no Payout Account record is created from a rejected attempt |
| FEAT-28.SPEC-002 (Payout Status & Money Dashboard) | The status banner and money list continue showing the last-known data with no new waiting state, since this screen does not itself send requests -- it only displays reported outcomes | The banner and money list show the last-known data with the Error state's "cannot be retrieved" wording and a Retry action if the initial load itself is affected; an already-loaded screen is unaffected by a later outage in reporting | N/A -- this screen never sends a request the capability can reject; it only displays reported outcomes and hands Talia into FEAT-28.SPEC-006's resolution flow, whose own rejection is handled there |
| "Resolve now" resolution flow (via FEAT-28.SPEC-002, delivered through this spec) | The resolution flow shows its own in-flow loading state, matching the capability's own experience; Talia's dashboard is unaffected while she is inside it | "We can't reach the payout account service right now. Please try resolving this again in a few minutes." with a return path back to FEAT-28.SPEC-002; the account remains Action Required, unchanged | The capability's own specific rejection reason for the resolution attempt is shown in plain language, and the account remains Action Required until a subsequent attempt succeeds |

## Consent and Disclosure

- **First connection disclosure** -- Before Talia is handed into the capability's own flow from FEAT-28.SPEC-001, the screen states: "Connecting a payout account shares your Chairtime account reference, country, and currency with our payment processor. Your bank and identity details go directly to them -- Chairtime never sees or stores them." Shown every time the handoff is launched, since the flow may be re-entered if the first attempt fails.
- **Resolution disclosure** -- The "Resolve now" link on FEAT-28.SPEC-002 carries the same underlying disclosure; no new consent screen is needed since resolving an existing account reuses the connection Talia already consented to.
- **What is never shared** -- Talia's client list, bookings, service prices, and every Pro Account or Booking field beyond the internal reference, country, and currency stay inside the product; this boundary is stated in the first-connection disclosure's plain wording.
- **What the Pro is told about ongoing reporting** -- The first time Talia views her money list with at least one reported payout (FEAT-28.SPEC-002), a one-time notice states: "Your payout schedule and recent payouts are reported directly by your payment processor." No separate consent is required for this ongoing reporting, since it is inherent to the connection Talia already made.

## Edge Cases

- **The same status-changed event is delivered twice** -- FEAT-28.SPEC-003's own deduplication (status already at the reported value) means the second delivery changes nothing and no duplicate notification fires.
- **A status-changed event arrives for a Payout Account that no longer exists (Pro Account closed)** -- The event is applied to the retained record during FEAT-29's 30-day cooling-off period if it has not yet been removed, or discarded silently if the closure has completed; no user feedback fires in either case, since there is no active Pro session to notify.
- **Events arrive out of order (an older Active report after a newer Action Required report)** -- FEAT-28.SPEC-003's event-time comparison discards the stale report; the account reflects the newer, true status.
- **The connection handoff succeeds but the immediate status report is delayed** -- FEAT-28.SPEC-001 shows the Launching state until a report is received; if no report arrives within the capability's own expected window, the screen falls back to its Handoff Failed treatment, and any report that does eventually arrive is still applied normally by FEAT-28.SPEC-003.
- **Capability goes down mid-resolution, after Talia starts but before completing the resolution flow** -- The Payout Account remains Action Required (no half-resolved state); Talia is returned to FEAT-28.SPEC-002 with the capability-down message, and can retry the resolution once the capability recovers.
- **Talia attempts to connect a second payout account (already has one)** -- The capability itself is never asked, since FEAT-28.SPEC-004's eligibility check and FEAT-28.SPEC-001's own reachability rule prevent the handoff from being offered a second time; this is a product-side rejection, not a capability rejection.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-28.SPEC-001 (Payout Account Connection) | Triggered by (inbound) | "Connect payout account" initiates the outbound handoff |
| FEAT-28.SPEC-001 (Payout Account Connection) | Affects (outbound) | Launching, outcome, and degradation states surface this spec's reported results |
| FEAT-28.SPEC-002 (Payout Status & Money Dashboard) | Triggered by (inbound) | "Resolve now" initiates the resolution handoff |
| FEAT-28.SPEC-002 (Payout Status & Money Dashboard) | Affects (outbound) | Status banner and payout/fee data surface this spec's reported results |
| FEAT-28.SPEC-003 (Payout Account Status Processing) | Triggers (outbound) | Every reported outcome and status change feeds that automation |
| FEAT-28.SPEC-004 (Payout Account Eligibility & Constraints) | References (inbound) | The capability's own eligibility rejection is surfaced using that spec's exact denied messages |
| FEAT-28.SPEC-005 (Money List Composition & Net Calculation) | Affects (outbound) | Payout schedule and recent payouts feed that spec's composition |
| FEAT-25.SPEC-004 (Historical Aggregate Maintenance) | Triggers (outbound) | A reported payout figure (Inbound Events, "Payout figure reported") is cited as an external-event trigger source by that automation |
| FEAT-15 (Pro Onboarding & Setup Wizard) | Triggered by (inbound) | The "getting paid" setup step reaches this integration through FEAT-28.SPEC-001 |
| FEAT-07 (Deposit Payment at Booking) | References (outbound) | FEAT-07.SPEC-005 routes captured deposits to the account this spec connects and verifies |

## Analytics and Success Signals

- **payout_account_handoff_launched** (context: first_connection / resolution) -- supports success-metrics.md: "Setup-to-Live-Link Completion"
- **payout_account_handoff_outcome_received** (outcome: verification_pending / active / action_required / rejected) -- supports success-metrics.md: "Payout Transparency"
- **payout_account_handoff_degraded** (condition: slow / down / rejected; screen: spec ID) -- N/A -- no Stage 2 metric measures degradation frequency for this integration; retained so the product's tolerance for capability trouble is observable, consistent with FEAT-07.SPEC-005's Analytics section.
- **payout_reporting_updated** (schedule_changed: yes/no; new_payout_count) -- N/A -- no Stage 2 metric measures payout-reporting update frequency directly; retained so the money list's "last few seconds" responsiveness target (Payout Transparency) is traceable to a fresh reporting event when it is checked.

## Acceptance Criteria

**FEAT-28.SPEC-006-AC-01:** Given Talia taps "Connect payout account", when the handoff launches, then a request carrying her Pro Account reference, country, and currency is sent to the payment-processing capability.

**FEAT-28.SPEC-006-AC-02:** Given the capability reports Verification Pending after the first connection, when the report is received, then FEAT-28.SPEC-003 creates the Payout Account with that status and FEAT-28.SPEC-001 reflects it.

**FEAT-28.SPEC-006-AC-03:** Given the capability reports Active, when the report is received, then FEAT-28.SPEC-003 updates status to Active and FEAT-28.SPEC-007 fires the verification-complete notification.

**FEAT-28.SPEC-006-AC-04:** Given the capability flags the account Action Required with a specific reason, when the report is received, then FEAT-28.SPEC-002's banner shows that plain-language reason.

**FEAT-28.SPEC-006-AC-05:** Given Talia taps "Resolve now" on an Action Required banner, when the resolution flow launches, then she is handed directly into the capability's own flow for that specific problem.

**FEAT-28.SPEC-006-AC-06:** Given Talia completes the resolution flow successfully, when the capability reports Active, then FEAT-28.SPEC-003 updates status to Active and the banner clears.

**FEAT-28.SPEC-006-AC-07:** Given the capability reports an updated payout schedule or a new recent payout, when the report is received, then FEAT-28.SPEC-005 composes it into the money list on its next refresh.

**FEAT-28.SPEC-006-AC-08:** Given Talia is on FEAT-28.SPEC-001 for the first time, when she reaches the point of launching the handoff, then she sees the first-connection disclosure naming exactly what is shared: her account reference, country, and currency.

**FEAT-28.SPEC-006-AC-09:** Given the handoff request is in flight, when more than the capability's expected response window passes, then FEAT-28.SPEC-001 shows "Still working -- this is taking a little longer than usual."

**FEAT-28.SPEC-006-AC-10:** Given the capability is unreachable when Talia taps "Connect payout account", when the request cannot be sent, then FEAT-28.SPEC-001 shows "We couldn't start the connection process. Try again." and no Payout Account is created.

**FEAT-28.SPEC-006-AC-11:** Given the capability rejects the connection for a country/currency mismatch, when the rejection is reported, then FEAT-28.SPEC-004's exact denied message is shown on FEAT-28.SPEC-001.

**FEAT-28.SPEC-006-AC-12:** Given the capability is unreachable when Talia taps "Resolve now", when the request cannot be sent, then she sees "We can't reach the payout account service right now. Please try resolving this again in a few minutes." and the account remains Action Required.

**FEAT-28.SPEC-006-AC-13:** Given the same status-changed event is delivered twice, when the second delivery is processed, then nothing changes and no duplicate notification fires.

**FEAT-28.SPEC-006-AC-14:** Given a status-changed event arrives for a Pro Account that has since closed, when the event is processed, then no user feedback fires, since no active Pro session exists to receive it.

**FEAT-28.SPEC-006-AC-15:** Given an older Active report and a newer Action Required report arrive out of order, when both are processed, then the account reflects the newer Action Required status.

**FEAT-28.SPEC-006-AC-16:** Given the connection handoff succeeds but no status report arrives within the capability's expected window, when the window elapses, then FEAT-28.SPEC-001 falls back to its Handoff Failed treatment.

**FEAT-28.SPEC-006-AC-17:** Given the capability goes down mid-resolution, when Talia is returned from the flow, then the Payout Account remains Action Required with no half-resolved state, and she sees the capability-down message.

**FEAT-28.SPEC-006-AC-18:** Given Talia already has a Payout Account, when she looks for a way to connect a second one, then no handoff is ever offered -- the rejection is product-side, not a capability response.

**FEAT-28.SPEC-006-AC-19:** Given Talia views her money list with at least one reported payout for the first time, when the one-time reporting notice appears, then it reads "Your payout schedule and recent payouts are reported directly by your payment processor."

**FEAT-28.SPEC-006-AC-20:** Given a connection or resolution request is composed, when it is sent, then it carries only the Pro Account reference, country, and currency -- never client, booking, or pricing data.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 4 | 4 |
| Inbound Events | 6 | 6 |
| Degradation Paths | 8 (3 screens x 3 conditions, one N/A cell excluded) | 8 |
| Consent and Disclosure | 4 | 4 |
| Edge Cases | 6 | 6 |



# Notification Spec: Payout Status Notification

## Overview

**Name:** Payout Status Notification
**ID:** FEAT-28.SPEC-007
**Type:** Notification
**Purpose:** Notifies Talia when verification completes and the booking link can go live, and when the payout account needs action, so she never discovers either condition only by happening to open the dashboard.
**Parent Feature:** FEAT-28 -- Payout Account Connection & Payout Visibility

## Scope and Non-Goals

**In Scope:**
- The "verification complete" notification, fired when the Payout Account first becomes Active
- The "needs action" notification, fired when the Payout Account becomes Action Required
- In-app delivery for both, matching the account-health pattern already used for calendar connection alerts (FEAT-04)
- Preference, deduplication, and expiry behavior for both notifications

**Non-Goals:**
- The "a refund cannot be completed yet" message named in product-features.md's Communications field -- this is a cross-feature disposition owned by FEAT-09/FEAT-30 (the retry and the Pro-facing flag) under XBR-10, delivered through FEAT-08.SPEC-006 (Pro Attention Alert); this feature's role in that message is limited to reflecting the "in progress" state in the money list (FEAT-28.SPEC-005), which is why this spec covers exactly two notifications, not three
- Routine booking activity of any kind -- owned by FEAT-08.SPEC-005 (Pro Booking Activity Notification); this spec is reserved for the Payout Account's own two status transitions
- Displaying the status banner itself -- owned by FEAT-28.SPEC-002 (Payout Status & Money Dashboard); this spec only fires the notification that draws Talia's attention to it

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always, for both notifications | Matches the product's established pattern for account-health conditions (product-features.md, FEAT-04: "A dashboard alert (not a text/email) when a connection needs reconnecting"); the payout account's own status is exactly this kind of account-health condition, and Talia's attention list (FEAT-12) is the canonical place she checks for anything needing action |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Payout Account becomes Active | FEAT-28.SPEC-003 (Payout Account Status Processing) | Fires when status transitions to Active from any other status, whether on first connection or after resolving an Action Required condition | Pro Account reference, previous status |
| Payout Account becomes Action Required | FEAT-28.SPEC-003 (Payout Account Status Processing) | Fires when status transitions to Action Required from any other status | Pro Account reference, the capability's reported reason |

## Audience and Preferences

**Recipients:** The Pro (Talia) whose Payout Account the condition concerns (Access Matrix: Payouts = Full for the Pro). Platform Operator (Support) has View-only access to the resulting status through FEAT-28.SPEC-002 but is never a recipient of this notification, consistent with SC-05 and the Access Matrix.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| N/A -- this notification cannot be turned off | Always on, in-app only | Always on | N/A -- consistent with FEAT-04's calendar reconnection alert and FEAT-08.SPEC-006's treatment of in-app delivery, an account-health condition affecting whether Talia can be paid at all is never a mutable preference |

**Quiet Hours:** N/A -- product-features.md defines no quiet-hours window for Pro-facing account-health alerts (ASMP-29's quiet-hours discipline applies to client-facing reminders, not Pro account status), and a condition that gates whether Talia's booking link can go live or take deposits is exactly the kind of time-sensitive information a delay would work against.

## Content Definition

**In-app (verification complete):**
- **Title:** Your payout account is active
- **Body:** Verification is complete. If your booking link isn't live yet, you're ready to finish setup and start taking bookings.
- **CTA:** View payouts -- deep-links to FEAT-28.SPEC-002 (Payout Status & Money Dashboard)

**In-app (needs action):**
- **Title:** Your payout account needs attention
- **Body:** {reason}. Resolve it to keep your booking link live and your money flowing.
- **CTA:** Resolve now -- deep-links to FEAT-28.SPEC-002 (Payout Status & Money Dashboard), which carries Talia into FEAT-28.SPEC-006's resolution flow

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {reason} | Payout Account -- the capability's reported Action Required reason, surfaced via FEAT-28.SPEC-006's Inbound Events | Your bank account details couldn't be verified | If the capability reports no specific reason, renders as "Your payout account needs a quick update" |

## Delivery Rules

**Batching:** Not batched -- each of the two notifications concerns a distinct, singular condition on the one Payout Account a Pro can have (FEAT-28.SPEC-004); there is never more than one open instance of either notification to batch.
**Deduplication:** At most one active "needs action" notification per open Action Required condition -- a status report that leaves the account at Action Required (e.g., a second rejected attempt with a different reason) updates the existing notification's content rather than creating a new one, consistent with FEAT-08.SPEC-006's treatment of a persistently unresolved condition. The "verification complete" notification fires once per transition to Active; if Active is reported again with no intervening non-Active status (per FEAT-28.SPEC-003's deduplication), no second notification fires.
**Retry on failure:** N/A -- in-app delivery has no separate retry: the notification is written to Talia's in-app attention list and is delivered the moment she next opens the product, with no failure mode of its own beyond the product being unreachable, which is outside this notification's own scope.
**Expiry:** Neither notification expires undelivered. "Needs action" remains on the attention list until Talia resolves the condition (it clears then, not before). "Verification complete" remains visible until Talia acknowledges it by viewing FEAT-28.SPEC-002 or explicitly dismissing it; either way, the underlying Active status is durably visible on FEAT-28.SPEC-002 regardless of whether the notification itself is ever opened.

## Edge Cases

- **The Payout Account's Pro Account closes before the notification is acted on** -- The notification is not delivered, since no active Pro session exists to receive it (mirroring FEAT-28.SPEC-006's edge case for the same underlying event); the closure process itself governs the account's fate independently.
- **Status flips from Action Required back to Active and then back to Action Required again in quick succession** -- Each transition fires its own notification per the Trigger table; a "verification complete" notification and a later "needs action" notification are never merged into one, since they name opposite conditions.
- **Talia dismisses the "needs action" notification without resolving the underlying condition** -- Dismissing the notification does not clear the Action Required status; the banner on FEAT-28.SPEC-002 remains until the condition actually resolves, and dismissing again is possible if the notification re-surfaces after further reason updates.
- **Two "needs action" reports with different reasons arrive close together for the same open condition** -- Per Deduplication, the single active notification's content updates to the latest reported reason rather than producing two notifications.
- **The account reaches Active for the very first time (first connection, not a resolution)** -- The same "verification complete" content and CTA apply; there is no separate first-time variant, since the message ("verification is complete... you're ready to finish setup") already covers both the first-connection and post-resolution cases identically.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-28.SPEC-003 (Payout Account Status Processing) | Triggered by (inbound) | Both status transitions this spec covers originate from that automation |
| FEAT-28.SPEC-002 (Payout Status & Money Dashboard) | Navigation (outbound) | Both CTAs deep-link here |
| FEAT-28.SPEC-006 (Payout Account Connection & Verification) | References (outbound) | The "needs action" reason placeholder is sourced from that spec's Inbound Events |
| FEAT-15 (Pro Onboarding & Setup Wizard) | References (outbound) | The "verification complete" body references finishing setup, which is FEAT-15's own remaining flow |
| FEAT-12 (Pro Daily Schedule Dashboard) | References (inbound) | Both notifications surface as items on that feature's attention list |
| FEAT-08.SPEC-006 (Pro Attention Alert) | References (outbound) | The distinct "refund cannot be completed yet" message is delivered there instead, per this spec's Non-Goals |

## Analytics and Success Signals

- **payout_status_notification_sent** (condition: verification_complete / needs_action) -- supports success-metrics.md: "Payout Transparency"
- **payout_status_notification_opened** (condition) -- supports success-metrics.md: "Payout Transparency"
- **payout_status_notification_cta_tapped** (condition) -- supports success-metrics.md: "Setup-to-Live-Link Completion" (the verification-complete CTA leads directly back into finishing setup)

## Acceptance Criteria

**FEAT-28.SPEC-007-AC-01:** Given Talia's Payout Account transitions to Active for the first time, when FEAT-28.SPEC-003 fires this notification, then she receives an in-app notification titled "Your payout account is active" with a "View payouts" CTA to FEAT-28.SPEC-002.

**FEAT-28.SPEC-007-AC-02:** Given Talia's Payout Account transitions to Action Required with a specific reason, when FEAT-28.SPEC-003 fires this notification, then she receives an in-app notification titled "Your payout account needs attention" whose body includes that exact reason.

**FEAT-28.SPEC-007-AC-03:** Given Talia taps "Resolve now" on the needs-action notification, when the tap registers, then she lands on FEAT-28.SPEC-002 and can proceed into FEAT-28.SPEC-006's resolution flow.

**FEAT-28.SPEC-007-AC-04:** Given Talia's account resolves from Action Required back to Active, when FEAT-28.SPEC-003 fires this notification again, then she receives the same "verification complete" content as a first-time activation.

**FEAT-28.SPEC-007-AC-05:** Given Talia has no preference control for this notification, when she looks in her notification settings, then no on/off toggle exists for it -- it is always on, in-app only.

**FEAT-28.SPEC-007-AC-06:** Given the capability reports no specific reason for an Action Required condition, when the notification renders, then the body shows "Your payout account needs a quick update" instead of a blank reason.

**FEAT-28.SPEC-007-AC-07:** Given Talia's account is already Active, when a second Active report arrives with no intervening non-Active status, then no duplicate notification fires.

**FEAT-28.SPEC-007-AC-08:** Given Talia's account is Action Required and a second report arrives with an updated reason, when the update is processed, then the existing notification's content updates to the new reason rather than a second notification appearing.

**FEAT-28.SPEC-007-AC-09:** Given Talia's Pro Account closes before this notification is delivered, when the closure completes, then the notification is not delivered.

**FEAT-28.SPEC-007-AC-10:** Given a needs-action condition flips to Active and back to Action Required in quick succession, when each transition is processed, then each fires its own distinct notification.

**FEAT-28.SPEC-007-AC-11:** Given Talia dismisses the needs-action notification without resolving the condition, when she next opens FEAT-28.SPEC-002, then the Action Required banner is still shown, since dismissing the notification does not clear the underlying status.

**FEAT-28.SPEC-007-AC-12:** Given this feature's third named communication concerns a refund that cannot be completed yet, when that condition occurs, then it is delivered through FEAT-08.SPEC-006, not this spec.

**FEAT-28.SPEC-007-AC-13:** Given Talia views this notification during any time of day, when it is delivered, then no quiet-hours delay is applied.

**FEAT-28.SPEC-007-AC-14:** Given Talia's needs-action notification remains unresolved for several days, when she checks her attention list, then it is still present, since neither notification ever expires undelivered.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (in-app) | 1 |
| Trigger Paths | 2 (verification complete, needs action) | 2 |
| Preference States | 1 (always on, no variation) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |


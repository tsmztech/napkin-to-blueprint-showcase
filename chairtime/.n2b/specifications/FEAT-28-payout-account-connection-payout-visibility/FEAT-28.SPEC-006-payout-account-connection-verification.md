---
document_type: spec
spec_type: integration
spec_id: FEAT-28.SPEC-006
spec_name: Payout Account Connection & Verification
spec_slug: payout-account-connection-verification
parent_feature: FEAT-28
parent_feature_name: Payout Account Connection & Payout Visibility
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 20
---

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

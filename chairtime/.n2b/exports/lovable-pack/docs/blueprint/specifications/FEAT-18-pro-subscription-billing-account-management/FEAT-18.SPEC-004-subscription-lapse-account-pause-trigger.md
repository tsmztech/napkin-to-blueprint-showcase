---
document_type: spec
spec_type: automation
spec_id: FEAT-18.SPEC-004
spec_name: Subscription-Lapse Account Pause Trigger
spec_slug: subscription-lapse-account-pause-trigger
parent_feature: FEAT-18
parent_feature_name: Pro Subscription Billing & Account Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Automation Spec: Subscription-Lapse Account Pause Trigger

## Overview

**Name:** Subscription-Lapse Account Pause Trigger
**ID:** FEAT-18.SPEC-004
**Type:** Automation
**Purpose:** Triggers the Pro Account's system-imposed pause when the grace period expires unresolved, and lifts it the moment billing is restored.
**Parent Feature:** FEAT-18 -- Pro Subscription Billing & Account Management

## Scope and Non-Goals

**In Scope:**
- Setting the Pro Account's system-imposed Paused state when the grace period expires unresolved
- Clearing that system-imposed pause the moment billing is restored (payment method confirmed and the retry charge succeeds)
- Coordinating with the Pro Account's own pause mechanism so a system-imposed pause is distinguished from a Pro-chosen one

**Non-Goals:**
- Deciding whether a renewal succeeded or failed, or starting the grace clock -- owned by FEAT-18.SPEC-003 (Subscription Renewal & Payment-Failure Processing); this spec only acts on that automation's "grace expires unresolved" outcome
- Any other Pro Account field (profile, other pause reasons, closure) -- owned by FEAT-27 (Pro Profile & Booking Page Settings) and FEAT-29 (Pro Sign-In & Account Lifecycle), per the Feature Breakdown Brief's Referenced Entities note: this feature writes only the Paused state, and only for this one reason
- What a paused account looks like to clients on the public booking page -- owned by FEAT-05 (Public Booking Page & Booking Flow), which reads the Pro Account's Paused state; this spec only sets or clears that state
- Affecting existing bookings, reminders, refunds, or client self-service -- explicitly excluded per XBR-14 and XBR-11: a subscription lapse pauses new bookings only, and this automation never touches Booking, Deposit Transaction, or Messaging Consent records

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Grace period expires unresolved | FEAT-18.SPEC-003 (Subscription Renewal & Payment-Failure Processing) | Fires when a Subscription remains Payment Failed at its recorded grace deadline with no successful charge | Subscription reference, Pro Account reference, grace deadline that passed |
| Billing restored | FEAT-18.SPEC-003 (Subscription Renewal & Payment-Failure Processing) | Fires when a Subscription that was Payment Failed (whether or not the account was already paused) returns to Active via a successful retry | Subscription reference, Pro Account reference |

## Processing Logic

1. On "grace period expires unresolved," read the Pro Account's current status.
2. If the Pro Account is not already Paused for any reason, set its status to Paused with the system-imposed subscription-lapse reason recorded (distinct from a Pro-chosen pause, per the dependency map's Contention note: "a system-imposed subscription pause cannot be cleared by the Pro's 'resume bookings' toggle until billing is restored").
3. If the Pro Account is already Paused for a Pro-chosen reason, record that a subscription lapse also applies, so that clearing the Pro-chosen pause alone does not resume bookings while billing remains unresolved.
4. On "billing restored," read the Pro Account's current status.
5. If the Pro Account's Paused state was set (or is jointly held) for the system-imposed subscription-lapse reason, clear that reason. If no other pause reason remains, set status back to Active. If a Pro-chosen pause reason still applies independently, the account remains Paused for that reason alone.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Account paused (lapse only) | Grace expires unresolved and the account was Active | Pro Account.status set to Paused, reason = subscription lapse | Dashboard attention item added (FEAT-12.SPEC-005); public booking page (FEAT-05) stops offering new bookings | FEAT-12, FEAT-05 |
| Account already paused, lapse reason added | Grace expires unresolved while the account is already Paused for a Pro-chosen reason | Pro Account records the subscription-lapse reason alongside the existing pause reason | No visible change to the booking page (already paused); the Pro's "resume bookings" toggle becomes unable to clear the pause by itself until billing is restored | FEAT-27 |
| Pause lifted (lapse only) | Billing restored and the subscription-lapse reason was the only pause reason | Pro Account.status set back to Active | Public booking page resumes offering new bookings; dashboard attention item clears | FEAT-12, FEAT-05 |
| Lapse reason cleared, Pro-chosen pause remains | Billing restored while a Pro-chosen pause reason also applies | Subscription-lapse reason removed; Pro Account remains Paused for the Pro-chosen reason | No visible change to the booking page (still paused, now only for the Pro's own reason); the Pro's "resume bookings" toggle now works | FEAT-27 |
| No action needed | Billing restored but no subscription-lapse pause reason was ever recorded (grace never expired) | None | None | -- |

## Data Model

**Reads:** Pro Account -- status, pause reason(s); Subscription -- status.
**Creates:** None.
**Updates:** Pro Account -- status (Active <-> Paused), the subscription-lapse pause reason component specifically (this feature never writes any other Pro Account field, per the dependency map's Referenced Entities note).
**Deletes:** None.

## Business Rules

- XBR-14: a paused account (from this trigger or a Pro-chosen pause) takes no new bookings or deposits, while existing bookings keep their reminders, refunds, and client self-service unchanged.
- XBR-11: this automation never cancels or silently changes any existing Booking; any conflict this pause creates for an already-scheduled booking is out of scope here (there is none -- existing bookings are simply unaffected).
- Per the dependency map's Pro Account Contention note: a system-imposed subscription pause cannot be cleared by the Pro's own "resume bookings" toggle (owned by FEAT-27) until billing is restored -- only this automation clears it.
- Governed by FEAT-18.SPEC-005 (Subscription Billing Rules): the 7-day grace threshold (platform parameter: `subscription-payment-failure-grace-period-days`) whose expiry fires this automation, and the rule that a lapse pauses the account rather than deleting it, are defined there; this automation enforces them at grace expiry and does not redefine them.
- This automation is the sole writer of the subscription-lapse pause reason; all other Pro Account fields remain exclusively owned by FEAT-27 and FEAT-29.

## Edge Cases

- **Grace expires while the Pro has simultaneously initiated a payment-method update that has not yet been confirmed** -- The pause is triggered as soon as the grace deadline passes with no confirmed successful charge; if the in-flight update then succeeds moments later, the "billing restored" outcome fires immediately after and lifts the pause -- the Pro Account may be Paused for a brief window but never left paused once billing is genuinely restored.
- **Concurrent trigger firing (grace expiry and billing restoration are reported at effectively the same time, e.g., a very late retry)** -- FEAT-18.SPEC-003 reports at most one outcome per Subscription at a time (per its own concurrency handling), so this automation never receives both triggers simultaneously for the same Subscription; whichever outcome the payment-processing capability actually confirmed is the one applied.
- **Trigger fires while a previous run is in flight for the same Pro Account** -- A second pause/lift evaluation for the same Pro Account waits for the prior one to complete before reading and writing status, so the two writes cannot race and leave an inconsistent combined pause-reason state.
- **The Pro manually pauses their own account (via FEAT-27) while a subscription-lapse pause is already in effect** -- Both reasons are recorded jointly (per Processing Logic step 3); billing being restored later clears only the subscription-lapse reason, leaving the account Paused for the Pro's own reason until the Pro clears it themselves via FEAT-27.
- **Billing is restored for a Pro Account that was never actually paused (grace resolved before the deadline via FEAT-18.SPEC-003's retry path)** -- No pause was ever set by this automation, so "billing restored" is a no-op here; the Subscription itself simply returns to Active per FEAT-18.SPEC-003, without this automation ever having acted.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-18.SPEC-003 (Subscription Renewal & Payment-Failure Processing) | Triggered by (inbound) | "Grace expires unresolved" and "billing restored" outcomes fire this automation |
| FEAT-27 (Pro Profile & Booking Page Settings) | Affects (outbound) | Sets/clears the Pro Account's system-imposed pause; FEAT-27 owns the pause state field itself and the Pro-facing "resume bookings" toggle, which cannot clear this pause alone |
| FEAT-05 (Public Booking Page & Booking Flow) | Affects (outbound) | A pause set here stops the booking page from offering new bookings; existing bookings, reminders, refunds and client self-service continue unchanged (XBR-14, XBR-11) |
| FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) | A payment-failure pause surfaces as a dashboard attention item (FEAT-12.SPEC-005) |
| FEAT-18.SPEC-005 (Subscription Billing Rules) | References (inbound) | Enforces the grace-threshold expiry and pause-not-delete rules defined there (this spec appears in that rule's Enforced By table) |
| FEAT-18.SPEC-002 (Billing & Subscription Management Screen) | Affects (outbound) | The screen's status banner and grace messaging reflect this automation's state |

## Analytics and Success Signals

- **pro_account_paused_subscription_lapse** (days grace was open before pause) -- supports success-metrics.md: "Subscription Retention"
- **pro_account_pause_lifted_billing_restored** (days paused before restoration) -- supports success-metrics.md: "Subscription Retention"

## Acceptance Criteria

**FEAT-18.SPEC-004-AC-01:** Given Talia's subscription grace period expires with no successful charge, when this automation fires, then her Pro Account's status is set to Paused with the subscription-lapse reason recorded.

**FEAT-18.SPEC-004-AC-02:** Given Talia's Pro Account is paused only for a subscription lapse, when billing is restored (a successful retry), then her Pro Account's status returns to Active.

**FEAT-18.SPEC-004-AC-03:** Given Talia's Pro Account is already Paused for a Pro-chosen reason, when her grace period also expires unresolved, then the subscription-lapse reason is recorded alongside the existing pause, and her own "resume bookings" toggle cannot clear the pause until billing is restored.

**FEAT-18.SPEC-004-AC-04:** Given Talia's Pro Account is paused for both a Pro-chosen reason and a subscription lapse, when billing is restored, then only the subscription-lapse reason is cleared and the account remains Paused for her own reason until she clears it via FEAT-27.

**FEAT-18.SPEC-004-AC-05:** Given Talia's Pro Account is paused from a subscription lapse, when a client visits her public booking page, then no new bookings can be started, while any of Talia's existing bookings keep their reminders, refunds, and client self-service unchanged, per XBR-14.

**FEAT-18.SPEC-004-AC-06:** Given Talia's Pro Account is paused from a subscription lapse, when the pause is set, then a dashboard attention item appears per FEAT-12.SPEC-005.

**FEAT-18.SPEC-004-AC-07:** Given a pause-evaluation run is already in flight for Talia's Pro Account, when a second trigger fires for the same account, then the second evaluation waits for the first to complete before reading or writing status.

**FEAT-18.SPEC-004-AC-08:** Given Talia's subscription grace resolves before its deadline via a successful retry, when the Subscription returns to Active, then this automation never sets a pause, since the grace never actually expired.

**FEAT-18.SPEC-004-AC-09:** Given Talia's Pro Account is briefly paused because a late retry had not yet confirmed when the grace deadline passed, when that retry's success is confirmed moments later, then the pause is lifted immediately and the account is not left paused once billing is genuinely restored.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (grace expires unresolved, billing restored) | 2 |
| Outcome Paths | 5 | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |

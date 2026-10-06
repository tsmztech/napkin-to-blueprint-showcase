# FEAT-09 — Cancellation & No-Show Policy Engine

This chapter covers Cancellation & No-Show Policy Engine (FEAT-09), a Core-tier feature. It carries 6 specifications carrying 87 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-09.SPEC-001 | Cancellation Policy Setup | screen | 12 |
| FEAT-09.SPEC-002 | Policy Versioning & Cutoff Rendering | logic-rule | 15 |
| FEAT-09.SPEC-003 | Deposit Outcome Rules | logic-rule | 16 |
| FEAT-09.SPEC-004 | Cancellation & No-Show Outcome Evaluation | automation | 15 |
| FEAT-09.SPEC-005 | Automatic Deposit Refund | integration | 16 |
| FEAT-09.SPEC-006 | Refund Idempotency & Retry Rule | logic-rule | 13 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Cancellation & No-Show Policy Engine

## Summary

**Feature:** Cancellation & No-Show Policy Engine
**ID:** FEAT-09
**Description:** The Pro defines their own cancellation window and what happens to the deposit inside vs. outside it; the system enforces that policy automatically and consistently on every cancellation and no-show, exactly as the client agreed to it at booking.
**Priority:** Core
**Phase:** MVP
**Type:** Platform
**Rationale:** BRIEF.md's Business Context is exact: "a cancellation inside the pro's window forfeits the deposit... a cancellation outside the window refunds the deposit automatically," "under the pro's own cancellation policy that the client agreed to when booking." This is the mechanism behind the founder's headline promise: "the pro never chases a no-show again." [RESEARCH-INFORMED: policy-clarity finding -- clients dispute deposit/cancellation-fee charges when terms were unclear at booking, so the policy must be shown with this booking's exact cut-off time and amount rather than as a generic rule (MEDIUM confidence)]

**Key Capabilities:**
- Set the cancellation/reschedule window (e.g., 24 hours before appointment)
- Automatically refund the deposit for a cancellation made outside the window
- Automatically flag a cancellation inside the window (or a no-show) for deposit forfeiture, applied via FEAT-11
- Apply the counterpart rules: a Pro-initiated cancellation always refunds the client's deposit in full, whatever the timing; a reschedule outside the window carries the deposit over to the new time; a reschedule inside the window is treated as a late cancellation (deposit kept) plus a new booking with its own deposit, shown to the client before confirming

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-09.SPEC-001 | Cancellation Policy Setup | Screen | The Pro | Pro sets or edits the cancellation window (1-168 whole hours) and reviews the resulting plain-language wording before saving |
| FEAT-09.SPEC-002 | Policy Versioning & Cutoff Rendering | Logic/Rule | The Pro, The Client | Governs how policy edits create immutable new versions, how each booking is permanently bound to the version acknowledged at booking time, and how this booking's exact cutoff time and wording are computed for display elsewhere |
| FEAT-09.SPEC-003 | Deposit Outcome Rules | Logic/Rule | The Pro, The Client | The binary, symmetric rule set governing every deposit outcome: client cancel outside/inside the window, no-show, Pro-initiated cancellation, and reschedule outside/inside the window |
| FEAT-09.SPEC-004 | Cancellation & No-Show Outcome Evaluation | Automation | The Pro, The Client | On every cancellation, reschedule, no-show marking, or Pro-initiated cancellation, evaluates the bound policy version against SPEC-003's rules and writes the resulting refund-or-forfeit outcome to the Deposit Transaction |
| FEAT-09.SPEC-005 | Automatic Deposit Refund | Integration | The Pro, The Client | Requests the refund from the payment-processing capability whenever SPEC-004 determines a refund is due, drawing on the Pro's payout account |
| FEAT-09.SPEC-006 | Refund Idempotency & Retry Rule | Logic/Rule | The Pro, The Client | Guarantees a refund completes exactly once per deposit, retries automatically when it cannot complete immediately, and keeps the Pro's dashboard flag and the client's "in progress" status consistent until resolved |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Set the cancellation/reschedule window (e.g., 24 hours before appointment) | FEAT-09.SPEC-001, FEAT-09.SPEC-002 | The Pro sets window_hours (1-168) and reviews the wording on SPEC-001; saving invokes SPEC-002 to create the new policy version | Phase 2 (Explicit) |
| Automatically refund the deposit for a cancellation made outside the window | FEAT-09.SPEC-003, FEAT-09.SPEC-004, FEAT-09.SPEC-005 | SPEC-003 defines the outside-window-refund rule; SPEC-004 evaluates the timing and fires the outcome; SPEC-005 executes the refund through the payment-processing capability | Phase 2 (Explicit) |
| Automatically flag a cancellation inside the window (or a no-show) for deposit forfeiture, applied via FEAT-11 | FEAT-09.SPEC-003, FEAT-09.SPEC-004 | SPEC-003 defines the inside-window/no-show forfeiture rule; SPEC-004 evaluates and writes the forfeiture flag to the Deposit Transaction; the flag's application (moving the transaction to Forfeited) is FEAT-11's responsibility, per the Key Capability's own wording | Phase 2 (Explicit) |
| Apply the counterpart rules (Pro cancellation always refunds in full; reschedule outside carries the deposit over; reschedule inside is a late cancellation plus a new deposit, shown before confirming) | FEAT-09.SPEC-003, FEAT-09.SPEC-004 | SPEC-003 states each counterpart rule explicitly, including the reschedule-inside-window compound outcome; SPEC-004 evaluates which counterpart applies and produces the outcome, including flagging the new booking's own deposit requirement | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-09.SPEC-002 | Policy Versioning & Cutoff Rendering | Phase 3 (Entity-Lifecycle) / Phase 5 (Rule Discovery) | The Update operation on Cancellation Policy ("each edit creates a new version") and the Contention/resolution rule in the dependency map's Cancellation Policy entry (version-changed-between-acknowledgment-and-payment refused-with-refresh) are conditional, cross-consuming rules shared by this feature, FEAT-05, and FEAT-10 -- past the inline threshold and requiring its own spec to keep policy-version authority (XBR-08) in one place |
| FEAT-09.SPEC-005 | Automatic Deposit Refund | Phase 4 (External Dependencies lens) | The Dependencies section of assumptions-constraints.md (ASMP-31) names the payment-processing capability this feature relies on to issue refunds; the outbound refund call and its dispute-notice/insufficient-balance inbound signals need their own Integration spec, distinct from the evaluation logic that decides a refund is owed |
| FEAT-09.SPEC-006 | Refund Idempotency & Retry Rule | Phase 6 (Failure Analysis) | The Error state ("if automatic refund processing fails... the refund is retried automatically, the outcome is flagged clearly on the Pro's dashboard") and XBR-10 ("refunds... happen at most once per deposit... retried automatically, flagged... never dropped") describe cross-cutting reliability behavior spanning the evaluation automation and the refund integration -- not a single spec's concern, so it is elevated to its own rule spec, consistent with the idempotency pattern FEAT-07 uses for deposit capture |

## Entity-Lifecycle Coverage Matrix

**Entity: Cancellation Policy**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-15 (per dependency map) | The dependency map's Cancellation Policy lifecycle line states "Created by FEAT-15" (the default proposed during onboarding) | **Flagged discrepancy, not resolved:** the Navigation connections table shows the Pro completing this step by navigating *into* FEAT-09 ("Pro continues setup" -> FEAT-09, "cancellation window (default offered)"), which would place the initial Create inside this feature's own screen (SPEC-001). The Feature Analyst does not resolve this -- it is flagged here per the Requirements Architect's ownership of the dependency map. |
| Read (single) | FEAT-09.SPEC-001, FEAT-09.SPEC-002, FEAT-09.SPEC-004 | SPEC-001 loads the current version for editing; SPEC-002 reads the version bound to a given booking to render its cutoff and wording; SPEC-004 reads the bound version to evaluate an outcome | -- |
| Read (list) | N/A | This feature exposes no list of historical policy versions | The version-acknowledged-at-booking-time record that appears in a dispute timeline (Talia Handles a No-Show Dispute journey) is owned by FEAT-30/FEAT-16's timeline view, which reads this entity but does not belong to this feature |
| Update | FEAT-09.SPEC-001, FEAT-09.SPEC-002 | Pro edits window_hours/wording on SPEC-001; SPEC-002 enforces that the edit always creates a new version rather than overwriting the current one | Every existing Booking keeps the version it was created against (XBR-08); this is never a destructive update |
| Delete/Archive | N/A | No delete path exists -- per the dependency map's Cancellation Policy lifecycle line, "versions are kept while any booking references them" | Soft-vs-hard delete is not applicable: versions are immutable and permanent once any booking references them; there is no restore, cascade, or purge concept for this entity -- recorded as an explicit non-goal below |
| State Transition | N/A | A policy version has no internal state machine -- it is either the current version or a superseded one, determined structurally by version/effective_from, not by an explicit status field | -- |

**Entity: Deposit Transaction (narrow slice -- outcome fields only, not fully managed by this feature)**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A -- owned by FEAT-07 | Per the dependency map's Deposit Transaction lifecycle line, "Created by FEAT-07" at the moment the deposit is captured | This feature never creates a Deposit Transaction, only updates the outcome of an existing one |
| Read (single) | FEAT-09.SPEC-004, FEAT-09.SPEC-006 | SPEC-004 reads the transaction's current status before writing an outcome; SPEC-006 reads it to enforce the exactly-once refund guarantee | -- |
| Read (list) | N/A | This feature produces no list view of Deposit Transactions | The money list is FEAT-28's screen; the activity record is FEAT-16's |
| Update | FEAT-09.SPEC-004, FEAT-09.SPEC-005 | SPEC-004 writes outcome_reason and the forfeiture flag or the refund-due flag; SPEC-005 writes status Refunded or Refund in Progress and the refund timestamp | Forfeited is the terminal status FEAT-11 applies to the flag this feature raises; a Pro goodwill refund overriding a forfeited outcome is FEAT-30's action, not this feature's |
| Delete/Archive | N/A | No delete/archive path exists for a financial record | Per SC-22, retained for the life of the account and de-identified (never removed) after client deletion or account closure -- recorded as an explicit non-goal below |
| State Transition | FEAT-09.SPEC-004, FEAT-09.SPEC-005, FEAT-09.SPEC-006 | Captured -> Refunded (direct, on outside-window/Pro-cancel outcomes) or Captured -> Refund in Progress -> Refunded (on a retried refund) | Captured -> [forfeiture flagged] -> Forfeited is only initiated (flagged) here; the transition to Forfeited itself is applied by FEAT-11, per the Key Capability's own wording and the dependency map's Updated-by list |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Booking | FEAT-09.SPEC-002, FEAT-09.SPEC-004 | SPEC-002 reads the appointment start_time and the bound policy_version to compute this booking's cutoff time; SPEC-004 reads the cancellation/reschedule/no-show timestamp and who initiated it (client vs. Pro) to determine which counterpart rule applies |
| Payout Account | FEAT-09.SPEC-005 | Refunds draw on the Pro's connected payout account balance; the Integration spec checks this before requesting a refund |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Pro saves an edited cancellation window/wording on the setup screen | Create a new policy version, effective immediately; every already-confirmed booking keeps the version it was bound to | Standalone Logic/Rule | FEAT-09.SPEC-002 |
| Policy version changes between a client's booking-page acknowledgment and payment capture | Refuse the payment attempt with a refresh; client is shown the current wording and must re-acknowledge | Standalone Logic/Rule, cross-feature (the acknowledgment/payment screens themselves belong to FEAT-05/FEAT-07) | FEAT-09.SPEC-002 |
| Client cancels a booking (FEAT-10) | Compare the cancellation timestamp to the booking's bound policy cutoff; determine refund (outside window) or forfeiture flag (inside window) | Standalone Automation | FEAT-09.SPEC-004 |
| Client reschedules a booking outside the window (FEAT-10) | Carry the existing deposit over to the new appointment time -- no new charge, no outcome change | Standalone Automation | FEAT-09.SPEC-004 |
| Client reschedules a booking inside the window (FEAT-10) | Treat as a late cancellation (deposit kept under SPEC-003) plus flag that the new booking requires its own fresh deposit; shown to the client before they confirm | Standalone Automation, cross-feature (the client-facing preview and the new booking's own deposit collection belong to FEAT-10 and FEAT-07) | FEAT-09.SPEC-004 |
| Booking is marked no-show (FEAT-11) | Evaluate as a forfeiture outcome under the policy, regardless of the window (a no-show is always inside-window-equivalent) | Standalone Automation | FEAT-09.SPEC-004 |
| Pro cancels a booking (FEAT-30) | Always evaluate as a full refund, whatever the timing -- the window never applies to a Pro-initiated cancellation | Standalone Automation | FEAT-09.SPEC-004 |
| Outcome evaluation determines a refund is due | Request the refund from the payment-processing capability, drawing on the Pro's payout account | Standalone Integration | FEAT-09.SPEC-005 |
| Refund request cannot complete immediately (e.g., Pro's payout balance cannot cover it yet) | Retry automatically without user action; flag the outcome clearly on the Pro's dashboard; keep the transaction at Refund in Progress until it resolves | Standalone Logic/Rule | FEAT-09.SPEC-006 |
| Refund completes (immediately or after retry) | Deposit Transaction moves to Refunded; the outcome is included in the relevant confirmation message rather than a separate one | Cross-feature -- Notification spec owned by FEAT-08 | FEAT-08 responsibility |
| Outcome evaluation determines a forfeiture flag | Hand the flag to No-Show Marking & Deposit Forfeiture for application | Cross-feature | FEAT-11 responsibility |
| Talia reviews a no-show dispute timeline | Timeline display reads which policy version was shown and acknowledged, and when | Cross-feature -- Screen owned by FEAT-30/FEAT-16 | FEAT-30/FEAT-16 responsibility |

The feature's Communications field is explicit that policy application "is communicated in-flow (at booking and at cancellation), not as a separate standalone message; the outcome (refund confirmed / deposit kept) is included in the relevant confirmation." This resolves the disposition already: every outcome-communication side-effect above is a cross-feature trigger into FEAT-08's own Notification spec (confirmation and cancellation-outcome messages), never duplicated here. `notification_count: 0` is intentional, not an omission.

## Shared Context

**Shared Entities:**
- Cancellation Policy -- read by SPEC-001/SPEC-002/SPEC-004; updated (versioned) by SPEC-001/SPEC-002. Fields: window_hours (1-168), inside_window_outcome (binary: kept), outside_window_outcome (binary: full refund), plain_language_wording, version/effective_from.
- Deposit Transaction (outcome slice only) -- read by SPEC-004/SPEC-006; updated by SPEC-004 (outcome_reason, forfeiture/refund-due flag) and SPEC-005 (status, refund timestamp). This feature never touches amount, currency, or processor_fee -- those are FEAT-07's fields.
- Booking (read-only slice) -- SPEC-002 and SPEC-004 both read start_time, the bound policy_version, and the cancellation/reschedule/no-show timestamp and initiator (client vs. Pro).

**Shared UI Patterns:**
- Single settings surface -- SPEC-001 is the one screen for both the initial window choice (reached via FEAT-15's onboarding navigation) and any later edit; both paths produce the same versioning behavior via SPEC-002.
- Outcome wording consistency -- wherever another feature (FEAT-05, FEAT-10) displays this booking's exact cutoff time and deposit amount, it renders text produced by SPEC-002 rather than re-deriving it, so the wording a client acknowledges at booking always matches the wording shown at cancellation.

**Shared Validation:**
- SPEC-003 defines the complete outcome rule set (all client/Pro/cancel/reschedule/no-show combinations). SPEC-004 and every cross-feature outcome preview (FEAT-10's pre-confirmation display, FEAT-30's pro-cancellation flow) reference it rather than re-deriving the rule.
- SPEC-006 defines the exactly-once-refund guarantee that SPEC-004 and SPEC-005 both rely on, mirroring the idempotency pattern FEAT-07 uses for deposit capture.

## Internal Dependency Map

```
SPEC-001 (Cancellation Policy Setup) -> [Pro saves window/wording] -> SPEC-002 (Policy Versioning & Cutoff Rendering) -> [new version created]
SPEC-002 (Policy Versioning & Cutoff Rendering) -> [renders wording/cutoff, consumed by] -> FEAT-05 / FEAT-10 (external display)
[Client cancels/reschedules via FEAT-10, no-show marked via FEAT-11, or Pro cancels via FEAT-30] -> SPEC-004 (Cancellation & No-Show Outcome Evaluation)
SPEC-004 (Cancellation & No-Show Outcome Evaluation) -> [applies rules from] -> SPEC-003 (Deposit Outcome Rules)
SPEC-004 (Cancellation & No-Show Outcome Evaluation) -> [reads bound version from] -> SPEC-002 (Policy Versioning & Cutoff Rendering)
SPEC-004 (Cancellation & No-Show Outcome Evaluation) -> [refund due] -> SPEC-005 (Automatic Deposit Refund)
SPEC-004 (Cancellation & No-Show Outcome Evaluation) -> [forfeiture flagged] -> FEAT-11 (external application)
SPEC-005 (Automatic Deposit Refund) -> [cannot complete immediately] -> SPEC-006 (Refund Idempotency & Retry Rule) -> [retries] -> SPEC-005
SPEC-005 (Automatic Deposit Refund) -> [refund completes] -> FEAT-08 (external confirmation message)
```

**Default Entry:** SPEC-001 (Cancellation Policy Setup) -- the only screen this feature produces; reached from FEAT-15's onboarding step on first use, and from the Pro's own settings area for any later edit (Interactions field: this feature is otherwise reached only through automations triggered by other features, never through its own standalone navigation entry for evaluation or refund behavior).

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-09.SPEC-001 | Inbound | FEAT-15 (Pro Onboarding & Setup Wizard) | Pro is navigated to the cancellation-window step with a recommended default already proposed | Pro continues the onboarding wizard |
| FEAT-09.SPEC-002 | Outbound | FEAT-05 (Public Booking Page & Booking Flow) | This booking's exact cutoff time, deposit amount, and plain-language wording are displayed for the client to acknowledge before paying | Client reaches the policy-acknowledgment step |
| FEAT-09.SPEC-002 | Outbound | FEAT-10 (Client-Initiated Cancel/Reschedule) | The same wording and cutoff time are shown again at cancellation/reschedule time, and used to preview the outcome before the client confirms | Client opens the cancel/reschedule flow |
| FEAT-09.SPEC-003 | Outbound | FEAT-10 (Client-Initiated Cancel/Reschedule) | The outcome rules this feature owns determine what FEAT-10 previews to the client before they confirm a cancellation or reschedule | Client is about to confirm a cancellation or reschedule |
| FEAT-09.SPEC-003 | Outbound | FEAT-30 (Pro Booking Management) | The Pro-cancellation-always-refunds rule governs the outcome FEAT-30 applies when the Pro cancels | Pro cancels a booking |
| FEAT-09.SPEC-004 | Inbound | FEAT-10 (Client-Initiated Cancel/Reschedule) | A recorded client cancellation or reschedule triggers this feature's outcome evaluation | Client confirms a cancellation or reschedule |
| FEAT-09.SPEC-004 | Inbound | FEAT-11 (No-Show Marking & Deposit Forfeiture) | A recorded no-show marking triggers this feature's outcome evaluation | Pro taps "no-show" on a booking |
| FEAT-09.SPEC-004 | Inbound | FEAT-30 (Pro Booking Management) | A recorded Pro-initiated cancellation triggers this feature's outcome evaluation (always full refund) | Pro cancels a booking |
| FEAT-09.SPEC-004 | Outbound | FEAT-11 (No-Show Marking & Deposit Forfeiture) | The forfeiture flag this feature raises is applied (moved to Forfeited) by FEAT-11 | Outcome evaluation flags forfeiture |
| FEAT-09.SPEC-005 | Outbound | FEAT-28 (Payout Account Connection & Payout Visibility) | Refunds are drawn against the Pro's connected payout account balance | Outcome evaluation determines a refund is due |
| FEAT-09.SPEC-005 / SPEC-006 | Outbound | FEAT-12 (Pro Daily Schedule Dashboard) | A refund that cannot complete immediately is flagged clearly on the Pro's dashboard until it resolves | Refund request fails to complete immediately |
| FEAT-09.SPEC-005 | Outbound | FEAT-08 (Automated Booking Messaging) | The refund outcome (confirmed or in progress) is included in the relevant confirmation message sent to the client | Refund is requested, retried, or completes |
| FEAT-09.SPEC-002 | Outbound | FEAT-16 (Booking & Payment Activity Record) / FEAT-30 (Pro Booking Management) | The policy version shown and acknowledged at booking, and when, is read into a no-show dispute's timeline review | Pro opens a no-show dispute's timeline |

## Non-Functional Notes

**Data volumes / growth:** Cancellation Policy versions are edited rarely per Pro account (a handful over the account's lifetime, per the onboarding-then-occasional-edit pattern); Deposit Transaction outcome updates track Booking cancellation/no-show volume directly, well within a solo Pro's scale of 20-40 bookings a week (SC-19). No distinct growth pattern applies beyond what Booking volume already implies.

**Responsiveness:** "Policy evaluation is instantaneous at the moment of cancellation" (States field) -- the outcome evaluation (SPEC-004) never presents a loading state to either party; only the refund itself (SPEC-005/SPEC-006), when it cannot complete immediately, surfaces a visible "in progress" state rather than appearing instant.

**Data sensitivity / privacy:** The Cancellation Policy itself carries no sensitivity -- it is public terms shown to every client on the booking page (dependency map's Cancellation Policy entry). The Deposit Transaction fields this feature writes (outcome_reason, status) are a financial record tied to an identifiable booking and client, visible only to the Pro (Full) and, view-only, to Platform Operator (Support) for status -- never card data (Access field; Access Matrix, Cancellation & No-Show Handling / Payouts rows; SC-11).

**Compliance flags:** N/A -- no compliance regime is named for this feature specifically; the general financial-record retention expectation (SC-22) governs the Deposit Transaction fields this feature touches, and is addressed in the Entity-Lifecycle Coverage Matrix's Delete/Archive rows rather than as a separate compliance obligation.

## Non-Goals

- **Partial refunds and tiered cancellation schedules** -- Excluded per SC-18: BRIEF.md's Business Context defines a binary rule (full refund outside the window, deposit kept inside it or on a no-show); this feature never computes or offers a partial-percentage outcome, however close to the cutoff a cancellation lands.
- **Card-on-file cancellation fees charged after booking** -- Excluded per SC-13: the product protects the Pro with a deposit paid up front; this feature's entire mechanism is deposit-outcome enforcement (refund or forfeit of an already-captured amount), never a fresh charge to a card kept on file.
- **Chairtime adjudicating cancellation or no-show disputes** -- Excluded per SC-17: this feature provides the trustworthy, timestamped record an outcome was derived from, and the Pro may refund as goodwill (FEAT-30), but the product itself never rules on whether a cancellation was justified; a client who disagrees contests the charge with their card issuer through the payment processor.
- **Automatic detection of "genuine emergency" exceptions** -- Adjacency exclusion, grounded in the Validation & Limits field ("the policy is binary in v1") and the Resolving a No-Show Dispute journey, where the emergency judgment and the resulting goodwill refund are explicitly a manual Pro decision (FEAT-30), never an automated override this feature's evaluation logic attempts to infer.
- **Retaining superseded policy versions or forfeited/refunded transaction outcomes with any purge policy** -- Intentional lifecycle decision surfaced by the CRUD matrix: per the dependency map's Cancellation Policy entry ("versions are kept while any booking references them") and SC-22, both entities are retained for the life of the account (de-identified only after account closure); no automatic purge window applies to either.



# Screen Spec: Cancellation Policy Setup

## Overview

**Name:** Cancellation Policy Setup
**ID:** FEAT-09.SPEC-001
**Type:** Screen
**Purpose:** Talia sets or edits her cancellation/reschedule window in whole hours and reviews the resulting plain-language wording before saving, so every client sees exactly what will happen to their deposit.
**Parent Feature:** FEAT-09 -- Cancellation & No-Show Policy Engine

## Scope and Non-Goals

**In Scope:**
- Capturing and editing the single configurable value of the Cancellation Policy: window_hours (1-168 whole hours)
- Rendering the resulting plain-language wording live as Talia adjusts the window, before she saves
- Saving the edit, which hands off to FEAT-09.SPEC-002 to create a new policy version
- Serving both the first-time setup path (reached from FEAT-15's onboarding) and any later edit, as one shared surface

**Non-Goals:**
- Choosing between a refund or a forfeiture outcome inside vs. outside the window -- excluded per SC-18: the binary outcome (full refund outside, deposit kept inside or on a no-show) is fixed by the product definition and is never a configurable choice on this screen
- Composing or free-typing the plain-language wording -- the wording is always system-derived from window_hours (FEAT-09.SPEC-002); Talia never authors policy text herself, which keeps the wording a client reads at booking and at cancellation always consistent
- Viewing a history of past policy versions -- excluded per the Entity-Lifecycle Coverage Matrix: this feature exposes no list of historical versions; a dispute timeline's read of which version was acknowledged belongs to FEAT-30/FEAT-16
- Creating the very first policy version during onboarding -- per the dependency map's Cancellation Policy lifecycle line, Create is owned by FEAT-15 (the default proposed during onboarding); this screen governs the edit path, reached from FEAT-15's setup step and from Talia's own settings for every edit thereafter

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-15 (Pro Onboarding & Setup Wizard), setup step: policy | Talia continues the onboarding wizard to the cancellation-window step | A recommended default window (platform parameter: `cancellation-window-default-hours`) is pre-filled; no existing Cancellation Policy version exists yet |
| FEAT-27 (Pro Profile & Booking Page Settings) | Talia opens her booking-page settings and selects the cancellation policy item | The current Cancellation Policy version's window_hours is loaded for editing |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Set the window and save (creating a new version) | -- |
| Platform Operator (Support) | Full screen, read-only, for the Pro account under an active help request | None -- the window control and Save button are not shown | Support's attempt to act is impossible because the controls are not rendered; there is no separate denial message because nothing actionable is ever offered |
| The Client (Riley) | No | No | This setup screen is never reached through any Client-facing path; Riley sees only the resulting plain-language wording rendered by FEAT-09.SPEC-002 on the booking page (FEAT-05) and at cancellation (FEAT-10), never this screen |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (XBR-29); after signing in, Talia lands on her Daily Schedule Dashboard (FEAT-12), not directly back on this screen |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- an in-progress window change that was not yet saved is discarded (the last saved version stands); after re-authentication Talia returns to her Daily Schedule Dashboard (FEAT-12) |

## Layout and Content

**Header:** Screen title "Cancellation Policy" with a back arrow (returns to FEAT-15's next setup step during onboarding, or to FEAT-27 during a later edit) and a "Save" action button (right-aligned, disabled until the window value changes from the currently saved version).

**Body:** A single-column form with two elements, in order:
- **Cancellation window** (numeric stepper input, required): a whole-number-of-hours value, from 1 to 168. Labeled "Clients can cancel or reschedule free of charge up until this many hours before their appointment."
- **Policy preview** (read-only text block, below the window input): the plain-language wording FEAT-09.SPEC-002 derives from the current window value, updating live as Talia adjusts the stepper -- for example, "Clients cancelling or rescheduling less than {window_hours} hours before their appointment, or who don't show up, will have their deposit kept. Cancelling earlier refunds the deposit in full." The preview always reflects the value currently shown in the stepper, not yet the saved value.

**Footer:** None -- Save is in the header.

### Responsive Behavior

- **Compact size class:** Single-column form as described above, full width; Save remains in the header.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width (the design layer's decision) and horizontally centered; no structural change beyond width capping.
- **Policy preview block:** Wraps to as many lines as its text requires at any width; never truncated or scrollable within itself.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate away without saving any unsaved change | Screen closes | Animated transition to the calling context (FEAT-15 or FEAT-27) |
| Cancellation window stepper | Increment/decrement or type a value | Updates the local window value; re-renders the policy preview from the new value; validates via FEAT-09.SPEC-002 | Preview text updates immediately; Save button becomes enabled if the value differs from the saved version | Stepper shows the new value; preview text changes in place |
| Cancellation window stepper | Blur with an out-of-range or non-whole value | Triggers field validation via FEAT-09.SPEC-002 | Error state on the field | "Enter a whole number of hours between 1 and 168." below the field; Save remains disabled |
| Save button | Tap | 1. Validate the window value via FEAT-09.SPEC-002. 2. If valid, trigger FEAT-09.SPEC-002's versioning logic to create a new policy version effective immediately. | Button shows a loading state during save | Success: toast "Cancellation policy updated" and navigate to the calling context (FEAT-15's next step, or back to FEAT-27). Failure: inline error banner. |
| Save button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> Cancellation window stepper -> Save.
- **Validation announcements:** When the stepper enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Live preview announcements:** The policy preview text is announced to assistive technology when it changes, so a screen-reader user hears the exact wording a client would see before saving.
- **Keyboard alternatives:** The stepper's increment/decrement is reachable by keyboard (arrow keys or direct numeric entry); there are no pointer-only gestures on this screen.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Default (first use, onboarding) | Stepper pre-filled with the recommended default (platform parameter: `cancellation-window-default-hours`); preview shows the wording for that default; Save enabled | No Cancellation Policy version exists yet for this Pro Account, reached via FEAT-15 | Talia changes the value or taps Save |
| Default (editing) | Stepper pre-filled with the current version's window_hours; preview shows the current wording; Save disabled until a change is made | Screen opens via FEAT-27 with an existing policy version | Talia changes the value |
| Editing | Stepper shows Talia's in-progress value; preview updates live; Save enabled | Talia changes the stepper value | Talia taps Save or navigates away |
| Validation Error | Stepper shows an error state with the message below it; Save disabled | The value is out of range or not a whole number | Talia corrects the value |
| Saving | Save button shows a loading spinner; stepper disabled | Talia taps Save with a valid value | Save completes or fails |
| Error | Error banner at the top of the form with a Retry option; the entered value is preserved | The save operation fails | Talia taps Retry or navigates away |
| Offline/Degraded | N/A -- this screen requires connectivity, consistent with every other Pro setup screen in the product; the Pro is shown the standard Error state if a save is attempted without connectivity, exactly as any other network failure | -- | -- |

## Validation Rules

Validation governed by FEAT-09.SPEC-002 (Policy Versioning & Cutoff Rendering). See that spec for the window_hours field rule and the exact error message. This screen applies validation on field blur and on Save.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap (onboarding entry) | FEAT-15's next setup step | FEAT-15 (Pro Onboarding & Setup Wizard) |
| Back arrow tap (settings entry) | Booking-page settings | FEAT-27 (Pro Profile & Booking Page Settings) |
| Successful save (onboarding entry) | FEAT-15's next setup step | FEAT-15 (Pro Onboarding & Setup Wizard) |
| Successful save (settings entry) | Booking-page settings | FEAT-27 (Pro Profile & Booking Page Settings) |

## Data Model

**Creates:** None directly -- Save triggers FEAT-09.SPEC-002's versioning logic, which is what actually creates the new Cancellation Policy version record.
**Reads:** Cancellation Policy -- the current version's window_hours (editing path) or the recommended default (first-use path, platform parameter: `cancellation-window-default-hours`).
**Updates:** None directly on this screen -- see Creates; every "edit" is a new version, never an overwrite of the existing record, per FEAT-09.SPEC-002.
**Deletes:** None -- no delete path exists for this entity (Entity-Lifecycle Coverage Matrix).

## Business Rules

- Every save creates a new policy version effective immediately (FEAT-09.SPEC-002); this screen never overwrites the current version in place.
- The plain-language wording is always derived from window_hours by FEAT-09.SPEC-002 -- Talia cannot enter custom wording.
- XBR-08: existing bookings keep the policy version they were created against; a save on this screen never changes what a client already booked was told.
- XBR-26: an active cancellation policy is one of the preconditions for the booking link going live; the recommended default proposed during onboarding (FEAT-15) satisfies this precondition until Talia changes it here.

## Edge Cases

- **Talia navigates away with an unsaved change** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Talia taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in loading state).
- **Network failure during save** -- Error banner: "Could not save your cancellation policy. Check your connection and try again." with a Retry button. The entered value is preserved.
- **Talia enters exactly 1 or exactly 168 hours** -- Both are valid boundary values and save normally.
- **Talia edits her policy while a client is mid-checkout on the current version (concurrent-edit conflict)** -- Per the dependency map's Cancellation Policy Contention note, Talia's save always succeeds and creates a new version immediately; there is no conflict on her side. The client's in-progress checkout is the side affected: if their payment capture completes before Talia's save, they keep the version they acknowledged; if Talia's save commits first and the client has not yet captured payment, the client's payment attempt is refused with refresh and they must re-acknowledge the new wording (FEAT-09.SPEC-002's contention rule) -- resolution is reject-with-refresh on the client's side, never a conflict Talia sees on this screen.
- **Talia reopens this screen immediately after saving** -- The stepper and preview reflect the just-saved version, not a stale cached value.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-002 (Policy Versioning & Cutoff Rendering) | Triggers (outbound) | Save action triggers the creation of a new policy version and supplies the field-level validation rule |
| FEAT-15 (Pro Onboarding & Setup Wizard) | Navigation (inbound/outbound) | Onboarding's policy step reaches this screen with a recommended default; Save returns to the wizard's next step |
| FEAT-27 (Pro Profile & Booking Page Settings) | Navigation (inbound/outbound) | Talia reaches this screen from her settings for any later edit; Save returns her there |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| cancellation_policy_updated | entry source (onboarding / settings edit), new window_hours value | Save completes successfully | supports success-metrics.md: "Policy Clarity at Booking" |
| cancellation_policy_save_failed | entry source, attempted window_hours value | Save operation fails | N/A -- no Stage 2 metric measures save failures; retained so save reliability for this setup step is observable |

## Acceptance Criteria

**FEAT-09.SPEC-001-AC-01:** Given Talia is on the Cancellation Policy Setup screen during onboarding with no existing policy, when the screen loads, then the stepper is pre-filled with the recommended default and the preview shows the wording for that default.

**FEAT-09.SPEC-001-AC-02:** Given Talia is on the Cancellation Policy Setup screen with an existing policy of 24 hours, when the screen loads via FEAT-27, then the stepper shows 24 and the preview shows the current wording, with Save disabled until she changes the value.

**FEAT-09.SPEC-001-AC-03:** Given Talia adjusts the stepper to 48 hours, when the value changes, then the preview text updates immediately to reflect a 48-hour window and Save becomes enabled.

**FEAT-09.SPEC-001-AC-04:** Given Talia enters 0 hours and moves focus away from the stepper, then the field shows the error "Enter a whole number of hours between 1 and 168." and Save remains disabled.

**FEAT-09.SPEC-001-AC-05:** Given Talia enters 169 hours and moves focus away from the stepper, then the field shows the error "Enter a whole number of hours between 1 and 168." and Save remains disabled.

**FEAT-09.SPEC-001-AC-06:** Given Talia sets the window to 72 hours and taps Save, when the save succeeds, then a new policy version is created effective immediately, she sees the toast "Cancellation policy updated," and she returns to the calling context.

**FEAT-09.SPEC-001-AC-07:** Given Talia is on this screen with unsaved changes, when she taps the back arrow, then a confirmation dialog appears asking "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.

**FEAT-09.SPEC-001-AC-08:** Given Talia taps Save and the operation fails due to a network error, then an error banner reads "Could not save your cancellation policy. Check your connection and try again." with a Retry button, and her entered value is preserved.

**FEAT-09.SPEC-001-AC-09:** Given Platform Operator (Support) opens this screen for Talia's account under an active help request, when they view it, then the window stepper and Save button are not shown, and the screen is otherwise fully visible read-only.

**FEAT-09.SPEC-001-AC-10:** Given an unauthenticated visitor attempts to reach this screen directly, then they are redirected to the Pro sign-in screen, and after signing in they land on the Daily Schedule Dashboard rather than back on this screen.

**FEAT-09.SPEC-001-AC-11:** Given Talia's session expires while she has an unsaved change on this screen, when the expiry is detected, then the dialog "Your session has expired. Sign in to continue." appears and the unsaved change is discarded, with the last saved version standing.

**FEAT-09.SPEC-001-AC-12:** Given Talia saves a new window while a client's checkout for the current version is not yet captured, when the client attempts to complete payment, then their attempt is refused with refresh per FEAT-09.SPEC-002's contention rule, while Talia's own save on this screen completed without any conflict shown to her.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 6 (default-onboarding, default-editing, editing, validation error, saving, error) plus offline N/A | 7 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Policy Versioning & Cutoff Rendering

## Overview

**Name:** Policy Versioning & Cutoff Rendering
**ID:** FEAT-09.SPEC-002
**Type:** Logic/Rule
**Purpose:** Governs how an edit to the Cancellation Policy always creates a new, immutable version rather than overwriting the current one, how every Booking stays permanently bound to the version it acknowledged, and how this booking's exact cutoff time and plain-language wording are computed wherever another feature needs to display them.
**Parent Feature:** FEAT-09 -- Cancellation & No-Show Policy Engine
**Governed Entity:** Cancellation Policy (versioning and field rules), jointly with the Booking.policy_version binding (read-only slice)

## Scope and Non-Goals

**In Scope:**
- Field validation for window_hours, the only Pro-editable field on the Cancellation Policy
- The immutable-versioning rule: every edit creates a new version effective immediately, never an overwrite
- Deriving plain_language_wording from window_hours (system-derived, never Pro-authored)
- Binding a Booking permanently to the policy version in force at the moment the client acknowledges it, and computing that booking's exact cutoff time (start_time minus window_hours)
- The contention rule when a policy version changes between a client's acknowledgment and payment capture
- Authorization for every action on the Cancellation Policy, across every role in the Access Matrix

**Non-Goals:**
- The Cancellation Policy setup screen's layout and interactions -- owned by FEAT-09.SPEC-001; this spec defines the rules that screen enforces, not its UI
- Deciding what outcome (refund or forfeit) applies to a given cancellation, reschedule, or no-show -- owned by FEAT-09.SPEC-003 (Deposit Outcome Rules); this spec only governs the policy record itself and the cutoff time the outcome rules are evaluated against
- Creating the very first Cancellation Policy version during onboarding -- per the dependency map's Cancellation Policy lifecycle line, Create is owned by FEAT-15; this spec governs every version created from that point forward, including the first Pro-initiated edit
- The client-facing screens that display the rendered wording and cutoff (FEAT-05's policy-acknowledgment step, FEAT-10's cancellation preview) -- owned by those features; this spec defines the rendering logic they consume, not their own layouts

## Governed Entity

**Entity:** Cancellation Policy (primary), with a read-only reference to Booking's policy-binding fields
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|--------------|
| window_hours | number | Whole hours, 1-168, before the appointment; the sole Pro-editable value |
| inside_window_outcome | enum (fixed) | Deposit kept -- binary in v1, never Pro-configurable |
| outside_window_outcome | enum (fixed) | Full refund -- binary in v1, never Pro-configurable |
| plain_language_wording | derived text | The exact text shown to clients, computed from window_hours; never directly editable |
| version / effective_from | derived | The version number and the date/time from which this version applies; system-managed, never directly editable |
| Booking.policy_version (read-only reference) | reference | The specific Cancellation Policy version a given Booking is permanently bound to, set at the moment of client acknowledgment (owned and written by FEAT-05/FEAT-07, read-only here) |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-09.SPEC-001 | Cancellation Policy Setup | On field blur and on Save; authorization on screen entry (controls hidden for non-Pro roles) and on Save |
| FEAT-09.SPEC-003 | Deposit Outcome Rules | Reads the cutoff time this spec computes as the boundary every deposit-outcome comparison is made against |
| FEAT-09.SPEC-004 | Cancellation & No-Show Outcome Evaluation | Reads the Booking's bound policy version and computed cutoff at the moment of evaluation |
| FEAT-16.SPEC-002 (Activity Event Recording) | Cross-feature | On the client's policy acknowledgment, records an Activity Event carrying the Booking reference, the bound Cancellation Policy version, the plain-language wording shown, and the acknowledgment timestamp, reading the version binding this spec defines without altering it |
| FEAT-05 (Public Booking Page & Booking Flow) | Cross-feature | Renders this booking's exact wording and cutoff at the policy-acknowledgment step, and binds Booking.policy_version at that moment |
| FEAT-07 (Deposit Payment at Booking) | Cross-feature | Re-checks that the acknowledged version still matches the current version at payment capture; applies the contention rule below if it does not |
| FEAT-10 (Client-Initiated Cancel/Reschedule) | Cross-feature | Renders the same wording and cutoff again at cancellation/reschedule time, using the booking's bound version, not the current one |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| window_hours | Required; whole number; between 1 and 168 inclusive | Always | On blur and on Save (FEAT-09.SPEC-001) | "Enter a whole number of hours between 1 and 168." | Yes |
| inside_window_outcome | No validation beyond data type -- fixed value, never Pro-input | Always | -- | -- | -- |
| outside_window_outcome | No validation beyond data type -- fixed value, never Pro-input | Always | -- | -- | -- |
| plain_language_wording | No direct validation -- always system-derived from window_hours (Defaults and Derivations below); never accepts direct input | Always | -- | -- | -- |
| version / effective_from | No direct validation -- always system-managed (Defaults and Derivations below); never accepts direct input | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Cutoff derivation | window_hours, Booking.start_time | A Booking's cutoff time = Booking.start_time minus its bound version's window_hours; this is the single value every cancellation/reschedule/no-show outcome check (FEAT-09.SPEC-003, FEAT-09.SPEC-004) and every client-facing display (FEAT-05, FEAT-10) reads, so all four never drift from one another | N/A -- structural derivation, no client-facing error |
| Version-immutability rule | window_hours, version/effective_from | Any change to window_hours is written as a brand-new version record with a new effective_from, never as an update to the existing version's window_hours field | N/A -- structural guarantee, enforced at the point of save, not by a client-facing validation error |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create the first Cancellation Policy version | The Pro (Talia) | Only during onboarding (FEAT-15); owned by that feature, not this spec | -- |
| Edit the cancellation window (create a new version) | The Pro (Talia) | Always | -- |
| View the current version's wording and cutoff for a specific booking | The Pro (Talia), The Client (Riley) | Riley: only for her own booking, via FEAT-05 (booking-time) or FEAT-10 (cancellation-time); Talia: any of her own bookings | Riley attempting to view another client's booking's policy wording is impossible -- FEAT-06's access-link scoping never surfaces another client's booking in the first place |
| View the Cancellation Policy for support purposes | Platform Operator (Support) | View-only, for the Pro account under an active help request | -- |
| Author or edit the plain-language wording directly | The Pro (Talia) | Never -- the wording is always system-derived from window_hours | The wording field is never presented as an input; Talia sees it only as read-only preview text on FEAT-09.SPEC-001 |
| Delete or archive a policy version | Any role | Never -- no delete/archive path exists for this entity | No delete or archive control exists anywhere in the product for this entity |
| Change which version an already-confirmed Booking is bound to | Any role | Never -- XBR-08: existing bookings keep the version they were created against, permanently | No control anywhere in the product allows re-binding a confirmed Booking's policy_version |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| plain_language_wording | Composed from window_hours using a fixed template: "Clients cancelling or rescheduling less than {window_hours} hours before their appointment, or who don't show up, will have their deposit kept. Cancelling earlier refunds the deposit in full." | Whenever window_hours changes (live preview on FEAT-09.SPEC-001) and on every save | No -- always derived, never directly editable |
| version | Incremented from the previous version's number | On every save that changes window_hours | No |
| effective_from | Set to the moment of save | On every save that changes window_hours | No |
| Booking's cutoff time (rendering only, not stored on Cancellation Policy) | Booking.start_time minus the bound version's window_hours | Computed on demand whenever FEAT-05, FEAT-10, FEAT-09.SPEC-003, or FEAT-09.SPEC-004 needs it | No |
| First version's window_hours (onboarding) | Recommended default proposed by FEAT-15 (platform parameter: `cancellation-window-default-hours`) | On account creation, before Talia's first edit | Yes -- Talia may accept the default or change it before her first save, and at any time thereafter via FEAT-09.SPEC-001 |

## Business Rules

- XBR-08: every Booking is governed by the Cancellation Policy version shown and acknowledged at the moment the client books; a later edit to window_hours never changes an already-confirmed Booking's outcome basis.
- **Versioning is edit-only, never in-place:** every save on FEAT-09.SPEC-001 that changes window_hours produces a new version record; the previous version is retained, unchanged, for as long as any Booking references it (Entity-Lifecycle Coverage Matrix; no purge policy applies, per this feature's Non-Goals).
- **Version-changed-during-checkout contention:** if the Cancellation Policy's current version changes between a client's acknowledgment on FEAT-05 and the deposit's capture on FEAT-07, the payment attempt is refused with a refresh; the client is shown the current wording and must re-acknowledge it before payment can proceed. This is the dependency map's Cancellation Policy Contention resolution rule, enforced jointly by FEAT-05 and FEAT-07, which both read this spec's version state to detect the change.
- **Rendering consistency:** wherever another feature displays this booking's cutoff time and wording (FEAT-05 at booking, FEAT-10 at cancellation), it renders the output of this spec's derivation rather than re-deriving it independently, so the wording a client acknowledged always matches the wording shown later.
- Onboarding's recommended default (platform parameter: `cancellation-window-default-hours`) satisfies XBR-26's go-live precondition of an active cancellation policy until Talia changes it.

## Edge Cases

- **window_hours at exactly 1 hour** -- Passes validation; the derived wording reads "less than 1 hour before their appointment."
- **window_hours at exactly 168 hours** -- Passes validation; the boundary value is accepted with no special-casing.
- **window_hours at 0 or 169** -- Fails validation with "Enter a whole number of hours between 1 and 168."
- **window_hours entered as a decimal (e.g., 24.5)** -- Fails validation; only whole numbers are accepted, consistent with the "whole number of hours" rule.
- **A client acknowledges the policy, then Talia edits the window before the client's payment captures** -- The contention rule fires: the client's payment attempt is refused with refresh, and the client re-acknowledges the current wording before paying. Talia's own save is never blocked or delayed by an in-progress client checkout.
- **A client's booking already exists under version 3, and Talia has since saved versions 4 and 5** -- The booking's cutoff and wording are always computed from version 3, its bound version, regardless of how many later versions exist.
- **Two overlapping edits from Talia's own two signed-in devices** -- Last-write-wins between Talia's own sessions, consistent with the dependency map's general pattern for Pro-only-edited entities with no other concurrent writer; whichever save commits last becomes the current version, and the other device's stale form is refreshed to reflect it on its next load.
- **A no-show is evaluated against the cutoff** -- FEAT-09.SPEC-003 treats a no-show as always inside-window-equivalent regardless of the computed cutoff time; this spec still supplies the bound version and wording for display, even though the no-show outcome itself does not depend on the cutoff comparison.

## Acceptance Criteria

**FEAT-09.SPEC-002-AC-01:** Given Talia enters 24 for window_hours and saves, then the plain-language wording reads "Clients cancelling or rescheduling less than 24 hours before their appointment, or who don't show up, will have their deposit kept. Cancelling earlier refunds the deposit in full."

**FEAT-09.SPEC-002-AC-02:** Given Talia enters exactly 1 hour, when she saves, then the value is accepted and a new version is created.

**FEAT-09.SPEC-002-AC-03:** Given Talia enters exactly 168 hours, when she saves, then the value is accepted and a new version is created.

**FEAT-09.SPEC-002-AC-04:** Given Talia enters 0 hours, when validation runs, then the error "Enter a whole number of hours between 1 and 168." is shown and no version is created.

**FEAT-09.SPEC-002-AC-05:** Given Talia enters 169 hours, when validation runs, then the error "Enter a whole number of hours between 1 and 168." is shown and no version is created.

**FEAT-09.SPEC-002-AC-06:** Given Talia's current policy is version 3 with a 24-hour window, when she saves a new 48-hour window, then a new version 4 is created effective immediately, and version 3 is retained unchanged.

**FEAT-09.SPEC-002-AC-07:** Given Riley's booking was created and acknowledged under version 3, when Talia later saves versions 4 and 5, then Riley's booking's cutoff and wording remain computed from version 3.

**FEAT-09.SPEC-002-AC-08:** Given Riley's booking is for an appointment at 3:00 PM and its bound version has a 24-hour window, when FEAT-10 renders her cancellation preview, then the cutoff time shown is 3:00 PM the day before.

**FEAT-09.SPEC-002-AC-09:** Given Riley has acknowledged the current wording on FEAT-05's booking flow, when Talia saves a new version before Riley's deposit captures, then Riley's payment attempt is refused with a refresh, and she is shown the current wording to re-acknowledge before paying.

**FEAT-09.SPEC-002-AC-10:** Given Talia attempts to author custom wording anywhere in the product, when she looks for a wording input field, then none exists -- the wording is always the read-only derived preview shown on FEAT-09.SPEC-001.

**FEAT-09.SPEC-002-AC-11:** Given Platform Operator (Support) opens the Cancellation Policy for Talia's account under an active help request, when they view it, then they see the current version's window and wording read-only, with no edit control.

**FEAT-09.SPEC-002-AC-12:** Given any role looks for a way to delete or archive a policy version, then no such control exists anywhere in the product.

**FEAT-09.SPEC-002-AC-13:** Given any role looks for a way to re-bind a confirmed booking to a different policy version, then no such control exists anywhere in the product, consistent with XBR-08.

**FEAT-09.SPEC-002-AC-14:** Given Talia has two devices signed in and saves an edit from each at effectively the same time, when both saves commit, then the version from whichever save committed last becomes the current version, and the other device's form refreshes to reflect it on next load.

**FEAT-09.SPEC-002-AC-15:** Given Talia has not yet completed onboarding, when FEAT-15 reaches the policy step, then the window is pre-filled with the recommended default (platform parameter: `cancellation-window-default-hours`), which Talia may accept or change before her first save.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 7 | 7 |
| Defaults/Derivations | 5 | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 8 | 8 |



# Logic/Rule Spec: Deposit Outcome Rules

## Overview

**Name:** Deposit Outcome Rules
**ID:** FEAT-09.SPEC-003
**Type:** Logic/Rule
**Purpose:** Defines the complete, binary, symmetric rule set governing what happens to a booking's deposit for every combination of who acts (client or Pro), what they do (cancel, reschedule, no-show), and when they do it relative to the policy's cutoff -- the single source of truth every evaluation and preview in the product reads instead of re-deriving.
**Parent Feature:** FEAT-09 -- Cancellation & No-Show Policy Engine
**Governed Entity:** Deposit Transaction (outcome-determination slice), evaluated jointly against Booking (timing and initiator) and Cancellation Policy (bound version's cutoff)

## Scope and Non-Goals

**In Scope:**
- The complete decision table covering client cancellation (outside/inside the window), no-show, Pro-initiated cancellation, and reschedule (outside/inside the window)
- The exact condition that determines "outside" vs. "inside" the window for each action type
- Authorization over who may view a determined outcome, and the explicit statement that no role may override the automatic determination
- The rationale and boundary conditions for every rule, so downstream evaluation (FEAT-09.SPEC-004) never has to interpret intent

**Non-Goals:**
- Actually evaluating a specific cancellation/reschedule/no-show event and writing its outcome to the Deposit Transaction -- owned by FEAT-09.SPEC-004 (Cancellation & No-Show Outcome Evaluation); this spec defines the rule table that evaluation applies, not the evaluation process itself
- Executing the refund through the payment-processing capability -- owned by FEAT-09.SPEC-005 (Automatic Deposit Refund)
- Applying the forfeiture flag as a terminal Forfeited state -- owned by FEAT-11 (No-Show Marking & Deposit Forfeiture), per the Key Capability's own wording and the dependency map's Updated-by list for Deposit Transaction
- Partial refunds or tiered cancellation schedules -- excluded per SC-18: BRIEF.md's Business Context defines a binary rule; this spec never computes a percentage-based or sliding-scale outcome, however close to the cutoff an action lands
- Automatic detection of a "genuine emergency" exception to any of these rules -- excluded per the Validation & Limits field ("the policy is binary in v1") and SC-17/SC-18; a goodwill override is always a manual Pro decision through FEAT-30, never a rule this spec or FEAT-09.SPEC-004 infers automatically

## Governed Entity

**Entity:** Deposit Transaction (outcome_reason field, and the status transition it sets up for FEAT-09.SPEC-004/FEAT-09.SPEC-005/FEAT-11 to apply), evaluated against Booking and Cancellation Policy
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|--------------|
| Deposit Transaction.status | enum | Current status at the moment of evaluation; this spec's rules apply only while status is Captured (an already-Refunded, Forfeited, Refund in Progress, or Disputed transaction is never re-evaluated, per the dependency map's "once per deposit" Contention rule) |
| Deposit Transaction.outcome_reason | text | Which rule below produced the determination; set by FEAT-09.SPEC-004 from this spec's table, not by this spec directly |
| Booking.start_time | date/time | The appointment's scheduled start, in the Pro's timezone |
| Booking's cancellation/reschedule/no-show timestamp | date/time | When the triggering action occurred, read by FEAT-09.SPEC-004 and compared against the booking's cutoff (FEAT-09.SPEC-002) |
| Booking's cancellation/reschedule initiator | enum (Client \| Pro) | Who performed the action; determines which half of the symmetric rule set applies |
| Cancellation Policy's bound version, window_hours | reference / number | The version acknowledged by this specific booking, and its window, both read via FEAT-09.SPEC-002 to compute the cutoff this spec's rules compare against |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-09.SPEC-004 | Cancellation & No-Show Outcome Evaluation | Applies this spec's rule table at the moment a cancellation, reschedule, or no-show marking is recorded, to determine the outcome it writes |
| FEAT-10 (Client-Initiated Cancel/Reschedule) | Cross-feature | Previews the outcome this spec determines to the client before they confirm a cancellation or reschedule |
| FEAT-30 (Pro Booking Management) | Cross-feature | Applies the Pro-cancellation-always-refunds rule when Talia cancels a booking |

## Field Validation Rules

No input fields exist on this spec's governed slice -- outcome_reason is always system-derived from the rule table below, never entered directly by any role.

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| Deposit Transaction.outcome_reason | No direct validation -- always derived from this spec's rule table by FEAT-09.SPEC-004; never accepts direct input from any role | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Client cancellation, outside the window | timestamp, cutoff, initiator | If initiator = Client and timestamp is at or before the cutoff, outcome = full refund | N/A -- automatic determination, no client-facing error |
| Client cancellation, inside the window | timestamp, cutoff, initiator | If initiator = Client and timestamp is after the cutoff, outcome = deposit kept | N/A -- automatic determination, no client-facing error |
| No-show | Booking marked no-show | A no-show marking always evaluates as deposit kept, regardless of how the computed cutoff compares to the marking time -- a no-show is treated as always inside-window-equivalent (product-features.md, Primary Flows) | N/A -- automatic determination, no client-facing error |
| Pro-initiated cancellation | initiator | If initiator = Pro, outcome = full refund, unconditionally -- the window is never applied to a Pro-initiated cancellation | N/A -- automatic determination, no client-facing error |
| Client reschedule, outside the window | timestamp, cutoff, initiator | If initiator = Client, action = reschedule, and timestamp is at or before the cutoff, the existing deposit carries over to the new appointment time -- no new charge, no outcome change | N/A -- automatic determination, no client-facing error |
| Client reschedule, inside the window | timestamp, cutoff, initiator | If initiator = Client, action = reschedule, and timestamp is after the cutoff, the compound outcome applies: the original deposit is kept (treated as a late cancellation), and the new appointment requires its own fresh deposit, shown to the client before they confirm | N/A -- automatic determination, no client-facing error |
| Pro-initiated reschedule | initiator | If initiator = Pro, action = reschedule, the client is never exposed to the window: the existing deposit always carries over to the new time, regardless of timing | N/A -- automatic determination, no client-facing error |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Trigger an outcome determination (by cancelling, rescheduling, or being marked no-show) | The Client (Riley), The Pro (Talia) | Riley: her own booking only, via FEAT-10; Talia: her own bookings, via FEAT-11 (no-show) or FEAT-30 (cancel/reschedule) | -- |
| View a determined outcome | The Pro (Talia), The Client (Riley) | Talia: any of her own bookings; Riley: her own booking only | -- |
| View determined outcomes for support purposes | Platform Operator (Support) | View-only, for the Pro account under an active help request | -- |
| Override or manually set an outcome different from this spec's rule table | Any role | Never -- the determination is always automatic; a goodwill refund overriding a kept deposit is a distinct, explicit Pro action through FEAT-30, not an override of this spec's determination | No control anywhere lets any role directly set outcome_reason; Talia's only path to a different financial result is FEAT-30's explicit goodwill refund action, which is recorded as its own action, not a correction to this spec's rule |
| Adjudicate whether a cancellation or no-show was "justified" | Any role | Never -- excluded per SC-17; the product provides the timestamped record, never a ruling | No adjudication control exists; a client who disputes an outcome contests the charge with their card issuer through the payment processor |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| outcome_reason | Derived from the rule table in Business Rules below, based on initiator, action type, and the timestamp-vs-cutoff comparison | At the moment FEAT-09.SPEC-004 processes a cancellation, reschedule, or no-show marking | No -- always automatic |
| "Outside" vs. "inside" the window | timestamp at or before the cutoff = outside; timestamp strictly after the cutoff = inside | Every timing-dependent rule row above | No |

## Business Rules

The complete outcome table (XBR-09), evaluated in this order of precedence -- initiator and action type first, then timing where the rule is timing-dependent:

| # | Trigger | Timing | Outcome |
|---|---------|--------|---------|
| 1 | Client cancels | Outside the window (at or before cutoff) | Full refund |
| 2 | Client cancels | Inside the window (after cutoff) | Deposit kept |
| 3 | Booking marked no-show | N/A -- always applies regardless of cutoff comparison | Deposit kept |
| 4 | Pro cancels | N/A -- window never applies | Full refund |
| 5 | Client reschedules | Outside the window (at or before cutoff) | Deposit carries over to the new time; no new charge |
| 6 | Client reschedules | Inside the window (after cutoff) | Original deposit kept (treated as a late cancellation per Rule 2); the new appointment requires its own fresh deposit, shown to the client before they confirm |
| 7 | Pro reschedules | N/A -- window never applies to a Pro-made reschedule | Deposit carries over to the new time; no new charge |

- Rules 1-2 and 5-6 are mutually exclusive and exhaustive for a client-initiated action: every client cancellation or reschedule falls into exactly one timing bucket, with no undefined middle case.
- Rule 3 takes precedence over Rules 1-2: once a booking is marked no-show, it is never re-evaluated as a "late cancellation" -- the two are mutually exclusive triggers.
- Rules 4 and 7 are absolute: no timing comparison is ever performed for a Pro-initiated action, so there is no "Pro cancels inside the window" variant.
- The outcome is always binary -- full refund or deposit kept in full -- per SC-18; no rule in this table produces a partial or percentage-based result.
- This rule table is the single authoritative source FEAT-10's pre-confirmation preview and FEAT-30's pro-cancellation flow both read, rather than re-deriving the rule independently (Shared Validation).

## Edge Cases

- **A cancellation timestamp lands exactly at the cutoff, to the minute** -- Rule 1/5 applies (at or before the cutoff = outside): the boundary favors the client, consistent with "at or before" in the rule definitions above.
- **A client reschedules more than once before the appointment** -- Each reschedule is evaluated independently against the (possibly new) appointment's own cutoff at the moment it occurs; an earlier outside-window reschedule that carried the deposit over does not exempt a later reschedule from its own timing check.
- **A no-show is marked, then the Pro later realizes the client did in fact show up** -- Outside this spec's scope: reversing a no-show marking is FEAT-11's undo mechanism (FEAT-11.SPEC-003); if reversed within its window, the outcome this spec determined is reversed by FEAT-11's own undo logic, not re-evaluated by this spec.
- **A Pro reschedules a booking that a client had already rescheduled inside the window (compound event)** -- Rule 7 applies to the Pro's own reschedule action independently; the earlier client-side late-reschedule outcome (Rule 6) on the original booking already resolved and is not revisited by the Pro's subsequent action on the new booking.
- **A client attempts to cancel a booking that has already been marked no-show** -- Not possible: per XBR-12, a no-show or completed booking cannot be cancelled or rescheduled, so Rules 1-2 and 5-6 can never fire against a booking Rule 3 has already resolved.
- **A booking's Deposit Transaction is already Refunded, Forfeited, Refund in Progress, or Disputed when a second triggering event somehow arrives** -- No rule in this table re-evaluates it; per the dependency map's Deposit Transaction Contention note, a terminal outcome is set once per deposit, and FEAT-09.SPEC-004 (not this spec) is responsible for refusing a second determination.
- **A goodwill refund is issued by Talia after a deposit was kept under Rule 2 or Rule 3** -- This is a distinct FEAT-30 action, not a correction of this spec's determination; the original outcome_reason set by this spec's rule table is never rewritten, and the goodwill refund is recorded as its own event.

## Acceptance Criteria

**FEAT-09.SPEC-003-AC-01:** Given Riley cancels her booking at a time at or before the computed cutoff, then the outcome determined is a full refund (Rule 1).

**FEAT-09.SPEC-003-AC-02:** Given Riley cancels her booking at a time after the computed cutoff, then the outcome determined is deposit kept (Rule 2).

**FEAT-09.SPEC-003-AC-03:** Given Riley's booking is marked no-show, then the outcome determined is deposit kept, regardless of how the marking time compares to the cutoff (Rule 3).

**FEAT-09.SPEC-003-AC-04:** Given Talia cancels Riley's booking, then the outcome determined is a full refund, regardless of timing (Rule 4).

**FEAT-09.SPEC-003-AC-05:** Given Riley reschedules her booking at a time at or before the computed cutoff, then the existing deposit carries over to the new appointment time with no new charge and no outcome change (Rule 5).

**FEAT-09.SPEC-003-AC-06:** Given Riley reschedules her booking at a time after the computed cutoff, then the original deposit is kept as a late cancellation and she is shown, before confirming, that the new appointment requires its own fresh deposit (Rule 6).

**FEAT-09.SPEC-003-AC-07:** Given Talia reschedules Riley's booking, then the existing deposit carries over to the new time with no new charge, regardless of timing (Rule 7).

**FEAT-09.SPEC-003-AC-08:** Given a cancellation timestamp lands exactly at the computed cutoff, then it is treated as outside the window (full refund), per the "at or before" boundary rule.

**FEAT-09.SPEC-003-AC-09:** Given Riley's booking has already been marked no-show, when a cancellation attempt is made against it, then no such attempt can occur (XBR-12 blocks it upstream) and Rule 3's outcome stands.

**FEAT-09.SPEC-003-AC-10:** Given Talia looks for a control to set a different outcome than this spec's rule table would produce, then no such control exists anywhere in the product.

**FEAT-09.SPEC-003-AC-11:** Given Riley disputes that her deposit was correctly kept, when she looks for an in-product way to have Chairtime rule on the dispute, then none exists -- she is directed to contest the charge with her card issuer through the payment processor.

**FEAT-09.SPEC-003-AC-12:** Given Talia issues a goodwill refund through FEAT-30 after a deposit was kept under Rule 2, then the original outcome_reason this spec determined is never rewritten -- the goodwill refund is recorded as its own separate action.

**FEAT-09.SPEC-003-AC-13:** Given Platform Operator (Support) views a determined outcome for Talia's account under an active help request, then they see the outcome read-only, with no action to change it.

**FEAT-09.SPEC-003-AC-14:** Given a Deposit Transaction is already Forfeited, when a second triggering event is somehow recorded against the same booking, then this spec's rule table produces no second determination for it.

**FEAT-09.SPEC-003-AC-15:** Given Riley reschedules twice before her appointment, then each reschedule is evaluated independently against its own appointment's cutoff at the time it occurs.

**FEAT-09.SPEC-003-AC-16:** Given FEAT-10 previews a cancellation outcome to Riley before she confirms, then the preview reflects exactly this spec's rule table rather than a separately derived approximation.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 1 | 1 |
| Cross-Field Rules | 7 | 7 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 7 (rule table rows) | 7 |
| Edge Cases | 7 | 7 |



# Automation Spec: Cancellation & No-Show Outcome Evaluation

## Overview

**Name:** Cancellation & No-Show Outcome Evaluation
**ID:** FEAT-09.SPEC-004
**Type:** Automation
**Purpose:** On every cancellation, reschedule, or no-show marking, evaluates the booking's bound policy version against FEAT-09.SPEC-003's rule set and writes the resulting refund-due or forfeiture-due outcome to the Deposit Transaction, instantaneously and with no user-visible loading state.
**Parent Feature:** FEAT-09 -- Cancellation & No-Show Policy Engine

## Scope and Non-Goals

**In Scope:**
- Evaluating every cancellation, reschedule, or no-show-marking event against FEAT-09.SPEC-003's rule table
- Writing the determined outcome (outcome_reason and its timestamp) to the Deposit Transaction
- Handing a refund-due outcome to FEAT-09.SPEC-005 for execution
- Handing a forfeiture-due outcome to FEAT-11 for application (this automation flags it; it does not itself set the terminal Forfeited status)
- Writing the forfeiture-due determination on the original deposit at the moment FEAT-10.SPEC-004 commits a late reschedule -- the same commit that flags the new booking as requiring its own fresh deposit (that flag itself is set by FEAT-10.SPEC-004, not by this automation)

**Non-Goals:**
- Defining which outcome applies to which trigger -- owned by FEAT-09.SPEC-003 (Deposit Outcome Rules); this automation applies that spec's table, it does not define it
- Computing the booking's cutoff time or reading the bound policy version's wording -- owned by FEAT-09.SPEC-002 (Policy Versioning & Cutoff Rendering); this automation consumes that computation
- Executing the actual refund against the payment-processing capability -- owned by FEAT-09.SPEC-005 (Automatic Deposit Refund); this automation only determines that a refund is due and hands it off
- Setting the Deposit Transaction's terminal Forfeited status or the Booking's No-Show state -- excluded per the Entity-Lifecycle Coverage Matrix: this automation initiates (flags) the forfeiture outcome only; the transition itself is applied by FEAT-11, per the Key Capability's own wording

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Client cancels a booking | FEAT-10 (Client-Initiated Cancel/Reschedule) | Fires when a client-initiated cancellation is recorded against a Confirmed or Awaiting Outcome booking | Booking reference, cancellation timestamp, initiator = Client, the booking's bound policy_version and start_time, the linked Deposit Transaction's current status |
| Client reschedules a booking | FEAT-10 (Client-Initiated Cancel/Reschedule) | Fires when a client-initiated reschedule to a new confirmed time is recorded | Booking reference, reschedule timestamp, initiator = Client, the booking's bound policy_version and start_time, the new appointment's time, the linked Deposit Transaction's current status |
| Booking marked no-show | FEAT-11 (No-Show Marking & Deposit Forfeiture) | Fires when the Pro's no-show marking is recorded against an Awaiting Outcome booking | Booking reference, marking timestamp, initiator = Pro, the linked Deposit Transaction's current status |
| Pro cancels a booking | FEAT-30 (Pro Booking Management) | Fires when a Pro-initiated cancellation is recorded against a Confirmed or Awaiting Outcome booking | Booking reference, cancellation timestamp, initiator = Pro, the linked Deposit Transaction's current status |
| Pro reschedules a booking | FEAT-30 (Pro Booking Management) | Fires when a Pro-initiated reschedule to a new confirmed time is recorded | Booking reference, reschedule timestamp, initiator = Pro, the new appointment's time, the linked Deposit Transaction's current status |

## Processing Logic

1. Receive the triggering event's data: the Booking reference, the action type (cancel or reschedule), the initiator (Client or Pro), and the event timestamp.
2. Read the Deposit Transaction linked to the Booking. If its status is not Captured (already Refunded, Forfeited, Refund in Progress, or Disputed), stop and route to the already-resolved outcome below -- no second determination is ever written.
3. For a cancellation or a reschedule (not a no-show marking), read the Booking's bound policy_version and compute its cutoff time via FEAT-09.SPEC-002 (start_time minus the bound version's window_hours).
4. Compare the initiator, action type, and (where timing-dependent) the event timestamp against the cutoff, applying FEAT-09.SPEC-003's rule table in its stated precedence order (no-show and Pro-initiated rules override timing comparisons).
5. Determine the outcome: full refund due, deposit-kept (forfeiture due), or deposit-carries-over (reschedule outside the window, no outcome change).
6. Write the determined outcome_reason and its timestamp to the Deposit Transaction (Captured status is not changed by this step -- see Outcome Definitions for what changes next).
7. If the outcome is a refund due, hand the Deposit Transaction to FEAT-09.SPEC-005 to execute the refund. Separately, for a client-initiated or automatic cancellation (not a reschedule or no-show marking), if the Booking has a Balance Payment in Succeeded state, also hand the Booking and Balance Payment references to FEAT-09.SPEC-005, which invokes FEAT-22.SPEC-005 to refund the paid balance in full whatever the deposit outcome (XBR-23; a balance is never forfeited). A Pro-initiated cancellation's balance refund is invoked by FEAT-30.SPEC-007 directly, so this automation does not duplicate it.
8. If the outcome is a forfeiture due (deposit kept, from a cancellation, no-show, or a late reschedule), hand the forfeiture flag to FEAT-11 for application to the terminal Forfeited state.
9. If the outcome is the late-reschedule compound case (Rule 6), this evaluation runs as part of the same commit sequence FEAT-10.SPEC-004 already executed the moment Riley tapped "Confirm Reschedule" -- the single confirmation gate for the whole compound outcome (product-features.md, FEAT-10 Alternate Flow: she "sees plainly" both halves "before confirming" and "can back out with nothing changed" if she declines that one tap). The new Booking's fresh-deposit requirement was already flagged by FEAT-10.SPEC-004 at that same commit, not by this step; this automation's own role here is solely to write the forfeiture-due outcome_reason to the *original* Deposit Transaction, finalizing the half of the compound outcome Riley was shown before she confirmed. If Riley instead backs out before that tap (via "Choose a different time" or "Cancel instead" on FEAT-10.SPEC-003), FEAT-10.SPEC-004 never commits anything and this automation never fires for that attempt.
10. If the outcome is deposit-carries-over (reschedule outside the window), no Deposit Transaction change is made beyond re-associating it with the new appointment time on the same, continuing Booking record (FEAT-10.SPEC-004 updates that record in place; no new Booking is created for this outcome) -- no new charge, no outcome_reason change beyond noting the carry-over.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Refund due | Client cancels outside the window, or Pro cancels (any timing) | Deposit Transaction.outcome_reason set to the refund determination and timestamped | No separate loading state; the outcome is included in the relevant confirmation message (FEAT-08), not a standalone one | FEAT-09.SPEC-005 (executes the refund) |
| Forfeiture due (client cancel, inside window) | Client cancels inside the window | Deposit Transaction.outcome_reason set to the forfeiture determination and timestamped | Riley is shown, before confirming the cancellation, exactly what will happen to her deposit (FEAT-10); Talia sees the kept-deposit outcome on her dashboard once resolved | FEAT-11 (applies the terminal Forfeited state) |
| Forfeiture due (no-show) | Booking marked no-show | Deposit Transaction.outcome_reason set to the forfeiture determination and timestamped | Talia sees the kept-deposit outcome on the no-show prompt; Riley's deposit status reflects the outcome in her own view | FEAT-11 (applies the terminal Forfeited state) |
| Deposit carries over (reschedule, outside window) | Client or Pro reschedules outside the window | Deposit Transaction re-associated with the new appointment's Booking record; no status or outcome_reason change | Riley sees no new charge and no change to her deposit status; the appointment time updates | FEAT-10 (or FEAT-30 for a Pro-made reschedule) |
| Late-reschedule compound outcome | Client reschedules inside the window | Original Deposit Transaction's outcome_reason set to the forfeiture determination (as a late cancellation), written as part of the single atomic commit FEAT-10.SPEC-004 performs the moment Riley taps "Confirm Reschedule"; the new Booking's fresh-deposit flag is set by that same FEAT-10.SPEC-004 commit, not by this automation | Riley is shown, before that one confirm tap, that her original deposit will be kept and the new time will need its own deposit; declining before the tap (choosing a different time, or cancelling instead) leaves everything -- including this outcome -- untouched, since nothing has been evaluated yet | FEAT-11 (applies Forfeited to the original), FEAT-10 (shows the preview and performs the commit), FEAT-07 (collects the new deposit after the commit) |
| Already resolved -- no second determination | The linked Deposit Transaction's status is already Refunded, Forfeited, Refund in Progress, or Disputed when a trigger fires | None | No outcome-specific feedback is produced by this automation; the triggering spec (FEAT-10, FEAT-11, or FEAT-30) surfaces its own already-resolved messaging, since this booking-state conflict is that spec's concern, not this automation's | FEAT-10, FEAT-11, FEAT-30 (whichever triggered) |
| Evaluation failure (processing error) | The evaluation step itself cannot complete (e.g., the policy version or cutoff cannot be read) | No outcome_reason is written -- the Deposit Transaction remains Captured, unresolved | The triggering action (cancellation, reschedule, or no-show mark) is not blocked from recording; the outcome is retried automatically and, if retries are exhausted, flagged on Talia's dashboard as needing attention, never silently dropped | FEAT-12 (attention list) |

## Data Model

**Reads:** Booking -- start_time, policy_version, cancellation/reschedule/no-show timestamp, and initiator. Cancellation Policy -- the bound version's window_hours, via FEAT-09.SPEC-002. Deposit Transaction -- current status (must be Captured to proceed).
**Creates:** None.
**Updates:** Deposit Transaction -- outcome_reason and an outcome timestamp for a refund-due or forfeiture-due determination; for the deposit-carries-over outcome (reschedule outside the window), the same Deposit Transaction record is re-associated with the new appointment time on the same, continuing Booking record -- per FEAT-10.SPEC-004's resolution of the Entity-Lifecycle Coverage Matrix, an outside-window reschedule updates the existing Booking in place and never creates a new record, so this re-association carries no status or outcome_reason change of its own. This automation never itself sets status to Refunded, Refund in Progress, or Forfeited; those transitions belong to FEAT-09.SPEC-005 (refund path) and FEAT-11 (forfeiture path) respectively.
**Deletes:** None -- consistent with the dependency map's Deposit Transaction lifecycle, which has no delete path (SC-22).

## Business Rules

- XBR-09: this automation applies FEAT-09.SPEC-003's rule table exactly, with no independent interpretation of timing or intent.
- XBR-08: the cutoff and bound version used for every comparison are always the ones the specific booking acknowledged at booking time (FEAT-09.SPEC-002), never the current policy.
- Evaluation is instantaneous from the point of view of both parties -- neither Talia nor Riley ever sees a loading state for this step (product-features.md, States field); only the refund itself, when it cannot complete immediately, surfaces an "in progress" state (FEAT-09.SPEC-005/FEAT-09.SPEC-006).
- A deposit can be evaluated to a terminal outcome only once (dependency map, Deposit Transaction Contention): once outcome_reason is set and handed to FEAT-09.SPEC-005 or FEAT-11, this automation never re-fires for the same triggering event.
- The forfeiture flag this automation raises is an initiation, not the terminal state: FEAT-11 (not this automation) performs the actual Captured -> Forfeited transition, per the Entity-Lifecycle Coverage Matrix.
- **Single confirmation gate for the late-reschedule compound outcome:** the "Confirm Reschedule" tap on FEAT-10.SPEC-003 is the one point of commitment for the whole compound outcome (product-features.md, FEAT-10 Alternate Flow). Backing out before that tap -- via "Choose a different time" or "Cancel instead" -- means FEAT-10.SPEC-004 never commits and this automation never fires; nothing about the original booking or its Deposit Transaction changes. Once that tap succeeds, FEAT-10.SPEC-004's atomic write (original Booking -> Rescheduled, new Booking created and flagged for a fresh deposit) and this automation's forfeiture-due determination on the original Deposit Transaction happen as one committed sequence -- there is no second confirmation gate, and what later happens to the new Booking's own (unpaid) deposit never reopens or reverses that already-written determination.

## Edge Cases

- **A cancellation and a Pro-initiated reschedule are recorded for the same booking at effectively the same time (concurrent trigger firing)** -- Per the dependency map's Booking Contention rule (reject-with-refresh, first committed state transition wins), only one of the two triggering actions can have actually recorded against the Booking in the first place; this automation only ever receives the one event whose triggering action won that race, so no two evaluations ever run against the same booking concurrently.
- **A second trigger fires while this automation's evaluation for the same booking is still in flight (trigger fires while a previous run is in flight)** -- The Deposit Transaction's status check (step 2) means the second run finds the first run's outcome_reason already being written or written; the second run's own triggering action would only have been possible if the first action's Booking-state transition had not yet committed (per Booking Contention, reject-with-refresh), so a second determination against the same still-Captured transaction from a genuinely distinct action cannot occur -- the two triggers described above (cancel vs. Pro reschedule) are the same scenario, not a separate one.
- **The cutoff computation itself fails (the bound policy version cannot be read)** -- Routed to the Evaluation failure outcome: the triggering action still records, the evaluation retries automatically, and Talia's dashboard is flagged if retries are exhausted.
- **A client reschedules to a time, then reschedules again before the first new time arrives** -- Each reschedule is its own triggering event, evaluated independently against its own appointment's freshly computed cutoff (per FEAT-09.SPEC-003's edge case for repeated reschedules).
- **A no-show marking arrives for a booking whose Deposit Transaction was already set to Refund in Progress by an earlier automatic determination** -- Not possible under normal use: XBR-12 requires the appointment's start time to have passed before a no-show mark, and a booking already resolved to a refund outcome would already be in a terminal or in-progress state that FEAT-11's own eligibility check (FEAT-11.SPEC-004) refuses to re-open; if it is somehow attempted, this automation's status check at step 2 still refuses a second determination.
- **Riley backs out of a late reschedule before tapping "Confirm Reschedule" (via "Choose a different time" or "Cancel instead" on FEAT-10.SPEC-003)** -- There is nothing for this automation to evaluate: FEAT-10.SPEC-004 never commits the compound write, so this automation never fires for that attempt. The original Booking and its Deposit Transaction remain exactly as they were, consistent with the single confirmation gate covering the whole compound outcome (product-features.md, FEAT-10 Alternate Flow: "can back out with nothing changed").
- **Riley taps "Confirm Reschedule" on a late reschedule, but the new Booking that commit creates is never actually paid** -- The original Deposit Transaction's forfeiture-due outcome was already written by this automation as part of the same atomic commit FEAT-10.SPEC-004 performed at the "Confirm Reschedule" tap -- that tap, not the new deposit's payment, was the single confirmation gate for the whole compound outcome. The new Booking's later non-payment and expiration (governed by FEAT-10.SPEC-004/FEAT-03's ordinary hold-and-expiration handling, XBR-02) is a separate, subsequent fact about a different Booking record; it never reopens or reverses this automation's already-written determination on the original.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-002 (Policy Versioning & Cutoff Rendering) | References (outbound) | Reads the bound policy version and computed cutoff for every timing-dependent determination |
| FEAT-09.SPEC-003 (Deposit Outcome Rules) | References (outbound) | Applies the rule table this spec defines |
| FEAT-09.SPEC-005 (Automatic Deposit Refund) | Triggers (outbound) | A refund-due outcome hands off to this spec to execute; a client-initiated or automatic cancellation on a Booking with a Succeeded Balance Payment also hands off the balance refund trigger, which that spec passes to FEAT-22.SPEC-005 (XBR-23) |
| FEAT-10 (Client-Initiated Cancel/Reschedule) | Triggered by (inbound) | FEAT-10.SPEC-004's committed cancellation or reschedule fires this automation; the outcome preview Riley sees before confirming (FEAT-10.SPEC-001/FEAT-10.SPEC-003) is read directly from FEAT-09.SPEC-003's rule table, not from this automation, which runs only after that commit |
| FEAT-11 (No-Show Marking & Deposit Forfeiture) | Triggered by (inbound) / Affects (outbound) | A recorded no-show marking fires this automation; the forfeiture flag this automation raises is applied by FEAT-11 |
| FEAT-30 (Pro Booking Management) | Triggered by (inbound) | A recorded Pro-initiated cancellation or reschedule fires this automation |
| FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) | An evaluation failure that exhausts retries is flagged on the attention list |

## Analytics and Success Signals

- **cancellation_within_window_flagged** (booking reference, days/hours before appointment) -- supports success-metrics.md: "Policy Clarity at Booking"
- **deposit_refund_triggered** (trigger type: client-cancel-outside / pro-cancel / pro-reschedule / client-reschedule-outside) -- supports success-metrics.md: "Automatic Refund Correctness"
- **late_reschedule_treated_as_cancellation** (booking reference) -- supports success-metrics.md: "Policy Clarity at Booking" (measures how often the compound outcome fires, which the disclosure-before-confirming step exists specifically to make unsurprising)
- **deposit_outcome_evaluation_failed** (trigger type, retry count) -- N/A -- no Stage 2 metric measures evaluation-failure frequency directly; retained so the "never silently dropped" guarantee (product-features.md, Error state) is observable

## Acceptance Criteria

**FEAT-09.SPEC-004-AC-01:** Given Riley cancels her booking outside the computed cutoff, when the cancellation is recorded, then this automation writes a refund-due outcome to the Deposit Transaction and hands it to FEAT-09.SPEC-005, with no loading state shown to Riley.

**FEAT-09.SPEC-004-AC-02:** Given Riley cancels her booking inside the computed cutoff, when the cancellation is recorded, then this automation writes a forfeiture-due outcome and hands it to FEAT-11, having already shown Riley the outcome before she confirmed (FEAT-10).

**FEAT-09.SPEC-004-AC-03:** Given Talia marks Riley's booking as no-show, when the marking is recorded, then this automation writes a forfeiture-due outcome regardless of how the marking time compares to the cutoff, and hands it to FEAT-11.

**FEAT-09.SPEC-004-AC-04:** Given Talia cancels Riley's booking, when the cancellation is recorded, then this automation writes a refund-due outcome and hands it to FEAT-09.SPEC-005, regardless of timing.

**FEAT-09.SPEC-004-AC-05:** Given Riley reschedules her booking outside the computed cutoff, when the reschedule is recorded, then this automation re-associates the existing Deposit Transaction with the new appointment with no outcome change and no new charge.

**FEAT-09.SPEC-004-AC-06:** Given Riley taps "Confirm Reschedule" on a chosen time inside the computed cutoff and FEAT-10.SPEC-004's commit succeeds, when the commit is recorded, then this automation writes a forfeiture-due outcome on the original deposit as part of that same commit -- the new appointment's fresh-deposit requirement was already flagged by FEAT-10.SPEC-004, and Riley had already seen both halves of the outcome before that one confirm tap.

**FEAT-09.SPEC-004-AC-07:** Given Talia reschedules Riley's booking, when the reschedule is recorded, then this automation re-associates the existing Deposit Transaction with the new time with no outcome change, regardless of timing.

**FEAT-09.SPEC-004-AC-08:** Given a booking's Deposit Transaction is already Refunded, when a second triggering event somehow arrives for it, then this automation writes no second outcome and produces no duplicate refund or forfeiture flag.

**FEAT-09.SPEC-004-AC-09:** Given the cutoff computation for a triggering event cannot complete due to a processing error, when evaluation is attempted, then the triggering action itself still records, the evaluation is retried automatically, and Talia's dashboard is flagged if retries are exhausted.

**FEAT-09.SPEC-004-AC-10:** Given two actions that could both apply to the same booking are attempted at effectively the same time, when the Booking's own contention rule resolves which one committed, then this automation evaluates only the one event whose action actually recorded, never both.

**FEAT-09.SPEC-004-AC-11:** Given Riley reschedules twice before her first new appointment time arrives, when each reschedule is recorded, then this automation evaluates each one independently against its own freshly computed cutoff.

**FEAT-09.SPEC-004-AC-12:** Given Riley taps "Confirm Reschedule" on the inside-window outcome and FEAT-10.SPEC-004's commit succeeds, when the new Booking that commit creates is later left unpaid and expires per FEAT-03's hold rules, then the original deposit's forfeiture-due outcome this automation wrote at that same commit still stands -- it is not reversed by the new Booking's later expiration, since the "Confirm Reschedule" tap itself, not the new deposit's payment, was the single confirmation gate for the whole compound outcome.

**FEAT-09.SPEC-004-AC-13:** Given a refund-due outcome is written for Riley's cancellation, when FEAT-09.SPEC-005 executes it, then the outcome the client eventually sees is included in her cancellation confirmation message (FEAT-08), never a separate standalone notice from this automation.

**FEAT-09.SPEC-004-AC-14:** Given a forfeiture-due outcome is written from a no-show marking, when FEAT-11 applies the terminal Forfeited state, then this automation's own record shows only the outcome_reason and timestamp it wrote -- the status transition itself is FEAT-11's action, not this automation's.

**FEAT-09.SPEC-004-AC-15:** Given Riley cancels a booking and its Balance Payment is in Succeeded state, when the cancellation is recorded, then this automation hands the balance refund trigger to FEAT-09.SPEC-005 for FEAT-22.SPEC-005 to refund in full, regardless of whether the deposit outcome is refund-due or forfeiture-due.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 5 (client cancel, client reschedule, no-show mark, Pro cancel, Pro reschedule) | 5 |
| Outcome Paths | 7 (refund due, forfeiture due x2 trigger flavors, carries over, late-reschedule compound, already resolved, evaluation failure) | 7 |
| Business Rules | 6 | 6 |
| Edge Cases | 7 | 7 |



# Integration Spec: Automatic Deposit Refund

## Overview

**Name:** Automatic Deposit Refund
**ID:** FEAT-09.SPEC-005
**Type:** Integration
**Purpose:** The product requests a full deposit refund from the payment-processing capability whenever FEAT-09.SPEC-004 determines one is due, drawing on the Pro's connected payout account, and reflects the outcome to both parties without either having to chase it.
**Parent Feature:** FEAT-09 -- Cancellation & No-Show Policy Engine

## Scope and Non-Goals

**In Scope:**
- Requesting a full refund for a specific Deposit Transaction once FEAT-09.SPEC-004 determines a refund is due
- Receiving and applying the refund outcome (succeeded, cannot complete immediately) to the Deposit Transaction
- User-facing behavior when the payment-processing capability is slow, unavailable, or rejects the refund request
- Disclosure of what data this refund request shares with the capability
- Handing a cancellation on a Booking with a Succeeded Balance Payment to FEAT-22.SPEC-005 so the paid balance is refunded in full alongside the deposit (XBR-23); this spec is the trigger and reflects the outcome, and does not call the payment-processing capability for the balance itself

**Non-Goals:**
- Determining that a refund is due in the first place -- owned by FEAT-09.SPEC-004 (Cancellation & No-Show Outcome Evaluation) jointly with FEAT-09.SPEC-003 (Deposit Outcome Rules); this spec only executes a refund already determined
- Retrying a refund that could not complete immediately, or keeping the Pro's dashboard flag and the client's "in progress" status consistent until it resolves -- owned by FEAT-09.SPEC-006 (Refund Idempotency & Retry Rule); this spec defines the single request/response contract that spec's retry loop calls
- Choosing the payment-processing vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate
- Capturing the original deposit charge, or routing it to the Pro's payout account in the first place -- owned by FEAT-07.SPEC-005 (Card Deposit Charge & Payout Routing); this spec only reverses an already-captured amount
- Executing the Balance Payment refund call, its idempotency key, its retry loop, and its own Refunded / refund-in-progress states -- owned by FEAT-22.SPEC-005 (Balance Charge, Payout Routing & Refund); this spec only invokes it and never refunds, forfeits, or partially refunds a balance itself
- Goodwill refunds initiated by the Pro outside this feature's automatic rules -- owned by FEAT-30 (Pro Booking Management), which executes its own refund requests against the same capability for that distinct, manually-triggered case

## Capability Category

**Category:** Payment processing
**Dependency Source:** ASMP-31 -- "Payment-processing capability... required to take client deposits, verify each pro's identity and bank details for a connected payout account, pay deposits out to the pro, issue refunds, notify the product of card-issuer disputes, and bill the pro's own monthly subscription." (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Payment processing -- client card charges and refunds (deposits; from v1 balances; Later tips)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-07, FEAT-09, FEAT-30, FEAT-22, FEAT-23; this spec is named directly as the FEAT-09 Integration Spec covering "automatic full deposit refund on client cancellation outside the window and every Pro cancellation... with not-yet-completable refunds reported back for retry")
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Riley's deposit refunds automatically when she cancels outside Talia's window, with no action from Talia | Automatically refund the deposit for a cancellation made outside the window | FEAT-10 (Client-Initiated Cancel/Reschedule) shows the confirmed refund |
| Riley's deposit always refunds in full when Talia cancels, whatever the timing | Apply the counterpart rule: a Pro-initiated cancellation always refunds the client's deposit in full | FEAT-30 (Pro Booking Management) shows the confirmed refund |
| If a refund cannot complete immediately, both Talia and Riley see it as "in progress" rather than silently failing | Automatically refund the deposit for a cancellation made outside the window (Error-state guarantee) | FEAT-12 (Pro Daily Schedule Dashboard) attention flag; Riley's own booking status view |
| Riley's paid balance refunds in full, with no action from Talia, when Riley's cancellation (or an automatic cancellation) lands on a booking whose balance she already paid in-app | Refunded in full if the appointment is later cancelled by either party -- a balance is never subject to forfeiture (FEAT-22, XBR-23) | FEAT-22.SPEC-005 (executes the balance refund); FEAT-10 shows the outcome |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Refund amount and currency | Deposit Transaction -- amount, currency | FEAT-09.SPEC-004 determines a refund is due | The capability must know exactly how much to return, matching the original captured amount |
| Deposit reference | Deposit Transaction -- the reference tying it to the original capture | Refund is requested | Ties the refund to the specific original charge so the capability reverses the correct transaction |
| Payout account reference | Payout Account -- processor_account_reference | Refund is requested | Identifies which of the Pro's connected accounts the refund draws against |

Booking details (service, client note, appointment time), the Client's contact fields, and every other Deposit Transaction field beyond amount, currency, and the deposit reference never leave the product for this integration.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Refund succeeded confirmation, with refund timestamp | The capability completes the refund | Deposit Transaction -- status, refund timestamp |
| Refund cannot complete immediately (e.g., the Pro's payout balance cannot yet cover it), with a retry-eligibility signal | The capability reports the refund could not be completed on this attempt | Deposit Transaction -- status (Refund in Progress); handed to FEAT-09.SPEC-006 for automatic retry |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Refund succeeded | The capability completes the requested refund | Deposit Transaction.status set to Refunded; refund timestamp recorded | The outcome is included in the relevant confirmation message (FEAT-08.SPEC-004), not a separate standalone notice; Riley sees her deposit refunded on her own booking view | FEAT-10 or FEAT-30 (whichever triggered), FEAT-08.SPEC-004 (confirmation content), FEAT-25.SPEC-004 (historical aggregate update on Refunded) |
| Refund could not complete immediately | The capability reports it cannot complete the refund on this attempt (for example, the Pro's payout balance cannot yet cover it) | Deposit Transaction.status set to Refund in Progress | Talia's dashboard shows a clear attention flag; Riley's booking view shows the refund as "in progress," never as failed or silent | FEAT-09.SPEC-006 (owns the retry loop), FEAT-12.SPEC-005 (attention flag aggregation), FEAT-25.SPEC-004 (historical aggregate update on Refund in Progress) |
| Balance refund due | FEAT-09.SPEC-004 records a client-initiated or automatic cancellation on a Booking that has a Balance Payment in Succeeded state (XBR-23), regardless of the deposit outcome (a balance is never forfeited) | No Deposit Transaction change. This spec hands the Booking and Balance Payment references to FEAT-22.SPEC-005, which executes the balance refund exactly once and owns the Balance Payment's Refunded / refund-in-progress state | Riley's cancellation confirmation (FEAT-08.SPEC-004) states that her paid balance is refunded in full, or in progress if FEAT-22.SPEC-005 reports it cannot complete immediately; Talia's dashboard flags an in-progress balance refund the same way as a deposit refund | FEAT-22.SPEC-005 (executes the refund), FEAT-08.SPEC-004, FEAT-12.SPEC-005, FEAT-25.SPEC-004 |
| Balance refund outcome reported back | FEAT-22.SPEC-005 reports its balance refund as Refunded or as refund-in-progress | None in this feature's entities; the outcome is reflected in the same cancellation confirmation content and attention signal as the deposit outcome | The confirmation reads the combined outcome (deposit and balance) so Riley sees one consistent statement, never a separate notice per refund | FEAT-08.SPEC-004, FEAT-12.SPEC-005 |
| Refund completes after a retry | FEAT-09.SPEC-006's retry succeeds on a later attempt | Deposit Transaction.status set to Refunded; refund timestamp recorded | The Pro's attention flag clears; Riley's "in progress" status updates to refunded, and the confirmation content reflects the completed refund | FEAT-09.SPEC-006, FEAT-12.SPEC-005, FEAT-08.SPEC-004, FEAT-25.SPEC-004 |

## Degradation Behavior

This integration is triggered by FEAT-09.SPEC-004's automation, not directly by a screen action, so no screen sends the refund request itself; the rows below cover the screens where this capability's trouble is visible to a user.

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-12 (Pro Daily Schedule Dashboard, attention list) | No visible change -- the refund request is not user-initiated on this screen, so a slow response produces no waiting state here; the attention flag simply has not yet appeared | If the capability is unreachable when the refund is requested, the attention flag reads "A refund for {client name}'s cancelled booking is in progress and will complete automatically." -- Talia sees no action she needs to take, and the flag persists until FEAT-09.SPEC-006's retry succeeds | If the capability explicitly rejects the refund request (for example, the payout account is no longer valid), the attention flag reads "A refund for {client name}'s cancelled booking needs attention -- your payout account may need reconnecting." with a link into FEAT-28 |
| Riley's own booking status view (FEAT-06/FEAT-10) | No visible change -- Riley sees no waiting state for a refund that has not yet been requested to fail or succeed | Riley's booking shows "Your deposit refund is in progress and will complete automatically." -- never a failure message | Riley's booking shows the same "in progress" wording; she is never shown the capability's rejection reason directly, since the resolution (e.g., reconnecting the payout account) is Talia's action, not hers |

## Consent and Disclosure

- **No new disclosure moment for the refund itself** -- Riley already agreed, at the moment she paid her deposit (FEAT-07), that the payment-processing capability holds and processes her payment method; reversing that same capture through the same capability requires no additional consent screen. The booking-time policy acknowledgment (FEAT-09.SPEC-002) already told her that an outside-window cancellation refunds automatically.
- **Payout account reference disclosure** -- Talia was told, when she connected her payout account (FEAT-28), that it would be used to receive deposits and to fund refunds and payouts; this integration's use of that same reference for a refund draws on that existing disclosure and requires no repeated notice.
- **What is never shared** -- Booking details, the Client's contact fields, and every Deposit Transaction field beyond amount, currency, and the deposit reference stay inside the product; the refund request never carries client contact information to the capability.

## Edge Cases

- **A refund-succeeded event arrives for a Deposit Transaction already marked Refunded** -- The second delivery changes nothing: the Deposit Transaction stays Refunded with its original refund timestamp, and no duplicate confirmation content fires.
- **A refund-could-not-complete event arrives after a refund-succeeded event for the same deposit (out-of-order delivery)** -- The Deposit Transaction reflects the most recent true state, not arrival order: since a deposit can be refunded only once (dependency map, Deposit Transaction Contention), a genuine refund-succeeded event is authoritative and a stale not-yet-completed report arriving late is treated as superseded and produces no status change.
- **The Booking or Client the refund relates to is deleted or de-identified before the refund event arrives** -- The event is still applied to the retained, de-identified financial record (per SC-22); no user feedback fires since there is no longer an active client-facing view to show it to.
- **The capability goes down mid-request, before confirming whether the refund was received** -- The Deposit Transaction is left at Refund in Progress rather than a half-resolved state; FEAT-09.SPEC-006's retry re-attempts the request, and a late success or rejection that eventually arrives is applied exactly as an on-time one would be.
- **A refund is requested twice for the same Deposit Transaction (e.g., an automation retry and a manual retry overlap)** -- The deposit reference sent with the request ties both attempts to the same original capture; the capability recognizes the resubmission as the same refund rather than issuing a second one, consistent with the "refunded only once" guarantee this spec and FEAT-09.SPEC-006 jointly uphold.
- **A cancellation lands on a Booking with a Succeeded Balance Payment and a Captured deposit that is forfeited (inside-window client cancellation)** -- The deposit is kept per FEAT-09.SPEC-003 but the balance is still handed to FEAT-22.SPEC-005 for a full refund; a balance is never forfeited, and the two outcomes are stated separately in the confirmation.
- **A cancellation lands on a Booking with no Succeeded Balance Payment (unpaid, declined, or already Refunded)** -- No hand-off to FEAT-22.SPEC-005 is made; the balance-refund path is skipped without any user feedback, and a cancellation racing an in-flight balance charge follows FEAT-22.SPEC-004's contention rule, not this spec.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-004 (Cancellation & No-Show Outcome Evaluation) | Triggered by (inbound) | A refund-due determination initiates this integration's request |
| FEAT-09.SPEC-006 (Refund Idempotency & Retry Rule) | Triggers (outbound) / References (inbound) | A not-yet-completable refund hands off to that spec's retry loop, which calls back into this spec's request contract |
| FEAT-28 (Payout Account Connection & Payout Visibility) | References (outbound) | Refunds draw on the Pro's connected payout account balance |
| FEAT-12.SPEC-005 (Attention Flag Aggregation) -- within FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) | A refund that cannot complete immediately is flagged clearly on the Pro's dashboard until it resolves |
| FEAT-22.SPEC-005 (Balance Charge, Payout Routing & Refund) -- within FEAT-22 (In-App Balance Payment) | Triggers (outbound) | A client-initiated or automatic cancellation on a Booking with a Succeeded Balance Payment invokes that spec's balance refund (XBR-23); its outcome is reflected back into the shared cancellation confirmation and attention signal |
| FEAT-25.SPEC-004 (Historical Aggregate Maintenance) -- within FEAT-25 (Booking Revenue Insights) | Affects (outbound) | Each Deposit Transaction change to Refund in Progress or Refunded fires that automation's aggregate update |
| FEAT-08.SPEC-004 (Booking Change & Refund Notice) -- within FEAT-08 (Automated Booking Messaging) | Affects (outbound) | The refund outcome (confirmed or in progress) is included in the relevant confirmation message sent to the client |
| FEAT-10 (Client-Initiated Cancel/Reschedule) | Affects (outbound) | Riley's own booking status view reflects the refund outcome |

## Analytics and Success Signals

- **deposit_refund_requested** (trigger type: client-cancel-outside / pro-cancel / pro-reschedule) -- supports success-metrics.md: "Automatic Refund Correctness"
- **deposit_refund_outcome_received** (outcome: succeeded / could-not-complete) -- supports success-metrics.md: "Automatic Refund Correctness"
- **balance_refund_handoff** (trigger type: client-cancel / automatic-cancel; outcome reported: refunded / in-progress) -- supports success-metrics.md: "Automatic Refund Correctness"
- **deposit_refund_failed** (reason category) -- supports success-metrics.md: "Automatic Refund Correctness" (a refund that cannot complete immediately must still be shown as "in progress" and complete without either party chasing it; this event measures how often that path is exercised)

## Acceptance Criteria

**FEAT-09.SPEC-005-AC-01:** Given FEAT-09.SPEC-004 determines Riley's cancellation is a refund-due outcome, when this integration requests the refund and the capability confirms it immediately, then the Deposit Transaction is set to Refunded with a refund timestamp, and Riley sees the confirmation reflecting her refund.

**FEAT-09.SPEC-005-AC-02:** Given a refund request is sent to the capability, when the capability reports it cannot complete on this attempt, then the Deposit Transaction is set to Refund in Progress, Talia's dashboard shows the attention flag, and Riley's booking shows "in progress," never a failure.

**FEAT-09.SPEC-005-AC-03:** Given the capability is unreachable when a refund is requested, then Talia's dashboard shows "A refund for {client name}'s cancelled booking is in progress and will complete automatically." with no action required from her yet.

**FEAT-09.SPEC-005-AC-04:** Given the capability explicitly rejects a refund request because the payout account is no longer valid, then Talia's dashboard shows "A refund for {client name}'s cancelled booking needs attention -- your payout account may need reconnecting." with a link into FEAT-28.

**FEAT-09.SPEC-005-AC-05:** Given a Deposit Transaction is already Refunded, when the same refund-succeeded event is delivered again, then nothing changes and no duplicate confirmation content fires.

**FEAT-09.SPEC-005-AC-06:** Given a refund-could-not-complete event arrives after a refund-succeeded event for the same deposit, then the Deposit Transaction remains Refunded and the late, superseded report produces no status change.

**FEAT-09.SPEC-005-AC-07:** Given the Client whose deposit is being refunded has since been deleted, when the refund event arrives, then it is applied to the retained de-identified financial record with no user feedback fired.

**FEAT-09.SPEC-005-AC-08:** Given the capability goes down mid-request before confirming receipt, then the Deposit Transaction is left at Refund in Progress rather than any half-resolved state, and a later-arriving outcome is applied exactly as an on-time one would be.

**FEAT-09.SPEC-005-AC-09:** Given a refund request for the same Deposit Transaction is submitted twice (an automatic retry overlapping a prior attempt), then the capability recognizes the resubmission via the shared deposit reference and issues only one refund.

**FEAT-09.SPEC-005-AC-10:** Given Riley has never had a deposit refunded before, when her first outside-window cancellation triggers this integration, then no new consent screen appears -- the refund proceeds under the disclosure she already received at booking and at deposit payment.

**FEAT-09.SPEC-005-AC-11:** Given this integration requests a refund, when the request is composed, then only the refund amount, currency, deposit reference, and payout account reference are sent -- Booking details and Client contact fields are never included.

**FEAT-09.SPEC-005-AC-12:** Given Talia looks at the attention flag for a refund in progress, when she reads it, then it never contains technical detail about the capability -- only the plain-language "in progress, will complete automatically" or the "needs attention -- reconnect payout account" wording defined above.

**FEAT-09.SPEC-005-AC-13:** Given a refund that could not complete immediately eventually succeeds after FEAT-09.SPEC-006's retry, then the Deposit Transaction is set to Refunded, Talia's attention flag clears, and Riley's "in progress" status updates to reflect the completed refund.

**FEAT-09.SPEC-005-AC-14:** Given Riley cancels a Booking outside the window and its Balance Payment is in Succeeded state, when FEAT-09.SPEC-004 records the cancellation, then this spec hands the Booking and Balance Payment references to FEAT-22.SPEC-005 alongside the deposit refund, and Riley's confirmation states that both her deposit and her paid balance are refunded in full.

**FEAT-09.SPEC-005-AC-15:** Given Riley cancels a Booking inside the window (deposit forfeited) and its Balance Payment is in Succeeded state, when the cancellation is recorded, then the deposit remains forfeited while the balance is still handed to FEAT-22.SPEC-005 for a full refund, and the confirmation states the two outcomes separately.

**FEAT-09.SPEC-005-AC-16:** Given a cancellation is recorded on a Booking with no Succeeded Balance Payment, when this spec processes the refund-due outcome, then no request is made to FEAT-22.SPEC-005 and no balance-related message is shown to Riley or Talia.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 4 | 4 |
| Inbound Events | 5 | 5 |
| Degradation Paths | 6 (2 screens x 3 conditions) | 6 |
| Consent and Disclosure | 3 | 3 |
| Edge Cases | 7 | 7 |



# Logic/Rule Spec: Refund Idempotency & Retry Rule

## Overview

**Name:** Refund Idempotency & Retry Rule
**ID:** FEAT-09.SPEC-006
**Type:** Logic/Rule
**Purpose:** Guarantees a refund completes exactly once per Deposit Transaction, retries automatically and indefinitely when it cannot complete immediately, and keeps the Pro's dashboard flag and the client's "in progress" status consistent with the true state until the refund resolves.
**Parent Feature:** FEAT-09 -- Cancellation & No-Show Policy Engine
**Governed Entity:** Deposit Transaction (refund-completion invariant: status Refunded / Refund in Progress and the retry state that governs the transition between them)

## Scope and Non-Goals

**In Scope:**
- The guarantee that at most one successful refund is ever recorded against a given Deposit Transaction, regardless of retries or duplicate delivery of the underlying capability's events
- The automatic, indefinite retry cadence applied when a refund request cannot complete immediately
- Keeping Talia's dashboard flag and Riley's "in progress" status consistent with the Deposit Transaction's true state throughout the retry period
- Resolution behavior when a retry succeeds, and when the underlying blocking condition (e.g., the Pro's payout account) is resolved by a Pro action mid-retry

**Non-Goals:**
- Determining that a refund is due, or deciding its amount -- owned by FEAT-09.SPEC-004 (Cancellation & No-Show Outcome Evaluation) and FEAT-09.SPEC-003 (Deposit Outcome Rules); this spec governs correctness *after* a refund has already been determined and requested
- Composing or sending the refund request itself, or defining the capability's inbound events -- owned by FEAT-09.SPEC-005 (Automatic Deposit Refund); this spec defines the guarantee that spec's request/response contract must uphold when a single attempt does not resolve
- Forfeiture correctness or the no-show/cancellation-inside-window path -- excluded per the Entity-Lifecycle Coverage Matrix: this spec governs only the refund path, mirroring the idempotency pattern FEAT-07.SPEC-004 applies to deposit capture, not the forfeiture path FEAT-11 owns
- A Pro-facing manual "retry now" control -- excluded per product-features.md's Error state, which states the refund "is retried automatically" with no manual action described; the Pro's only visible control is the attention flag itself, not a retry trigger

## Governed Entity

**Entity:** Deposit Transaction (refund-completion slice)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|--------------|
| Deposit Transaction (refund-attempt existence) | derived | Whether a refund attempt for a given Deposit Transaction has already succeeded -- this spec guarantees this fact is single-valued (succeeded at most once) |
| Deposit Transaction.status | enum | At this spec's stage: Refund in Progress -> Refunded. This spec guarantees the status set here is the true, final outcome of the one attempt that ultimately succeeds |
| Refund retry state (derived, not a dependency-map field) | derived | Whether a retry is currently scheduled, and when the next attempt will occur; exists only for the duration a Deposit Transaction sits at Refund in Progress |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-09.SPEC-005 | Automatic Deposit Refund | Applies this spec's idempotency key discipline when submitting a refund request, and calls this spec's retry loop whenever the capability reports it cannot complete a refund immediately |
| FEAT-12 (Pro Daily Schedule Dashboard) | Cross-feature | Displays the attention flag this spec keeps consistent with the Deposit Transaction's true state throughout the retry period |

## Field Validation Rules

No input fields exist on this spec's governed slice -- the refund-completion invariant is a system guarantee, never a value any role enters directly.

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| Deposit Transaction.status (Refund in Progress / Refunded) | Must never show as Refunded unless exactly one successful refund has been confirmed by the payment-processing capability for it, and must never remain Refund in Progress once that confirmation exists | Always | Continuously -- this is an invariant this spec's retry loop maintains, not a point-in-time check | N/A -- this is a system invariant, not a validated input | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| At-most-once refund | Deposit Transaction (refund-attempt existence), status | Once a Deposit Transaction reaches Refunded, no further refund request is ever submitted for it, by this spec's retry loop or by any resubmission arriving through FEAT-09.SPEC-005 | N/A -- structural guarantee, no client-facing error |
| Idempotency key discipline | Deposit Transaction (refund-attempt existence), the refund request itself | Every refund request FEAT-09.SPEC-005 submits carries an identifier tied to this specific refund attempt on this specific Deposit Transaction, so a resubmitted request (from a retry or a duplicate delivery) is recognized by the capability as the same attempt rather than a second refund | N/A -- structural guarantee at the integration boundary, not a client-facing error |
| Attention-flag consistency | Deposit Transaction.status, Talia's dashboard flag, Riley's "in progress" status | Both surfaces reflect the Deposit Transaction's true current status at all times during the retry period; neither is ever cleared before the true state changes, and neither ever shows "failed" while a retry is still pending | "A refund for {client name}'s cancelled booking is in progress and will complete automatically." (Talia); "Your deposit refund is in progress and will complete automatically." (Riley) |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View the refund's current status (in progress / refunded) | The Pro (Talia), The Client (Riley) | Talia: any of her own bookings; Riley: her own booking only | -- |
| View refund retry status for support purposes | Platform Operator (Support) | View-only, for the Pro account under an active help request | -- |
| Manually trigger a retry attempt | Any role | Never -- retries are always automatic, per product-features.md's Error state | No manual "retry now" control exists anywhere in the product for this action |
| Manually mark a stuck refund as resolved without the capability's confirmation | Any role | Never -- the Deposit Transaction only moves to Refunded on the capability's own confirmation | No control anywhere lets any role force the status to Refunded; if Talia believes a refund is taking too long, her only path is contacting support, which surfaces the true status, never a manual override |
| Cancel or abandon a pending automatic refund | Any role | Never -- once a refund is due (FEAT-09.SPEC-004), it is retried until it completes; XBR-10 states refunds are "never dropped" | No control exists to cancel or abandon an in-progress automatic refund |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Retry schedule | Retried automatically on a fixed cadence (platform parameter: `refund-retry-interval-hours`) after a not-yet-completable report, continuing indefinitely until the capability confirms success | Whenever a Deposit Transaction sits at Refund in Progress | No |
| Talia's dashboard flag, on screen load | Derived from the Deposit Transaction's true current status at the moment the dashboard loads: shown if status is Refund in Progress, cleared if status is Refunded | Whenever FEAT-12 loads or re-loads | No -- always read fresh from the true state, never cached |
| Riley's "in progress" status, on screen load | Derived identically from the Deposit Transaction's true current status | Whenever her booking view loads or re-loads | No |

## Business Rules

- **Never refund twice:** at most one successful refund is ever recorded against a Deposit Transaction, however many retries occur; a retry that resubmits a request already fulfilled is recognized as the same attempt by the capability rather than processed as a second refund (idempotency key discipline).
- **Never leave an ambiguous outcome:** every refund attempt resolves, from the product's point of view, to exactly one of Refunded or still-in-progress. There is no third "failed and abandoned" state -- XBR-10 requires refunds to be "never dropped," so a not-yet-completable report always leads to another scheduled retry, never a terminal failure state.
- **Automatic, indefinite retry until resolution, never after:** once Refund in Progress, the retry loop continues on the fixed cadence (platform parameter: `refund-retry-interval-hours`) until the capability confirms success; once Refunded, no further retry of that Deposit Transaction ever occurs.
- **Surfaces are always the true state, never assumed:** Talia's dashboard flag and Riley's "in progress" status are both derived fresh from the Deposit Transaction's current status on every load, never from a cached or previously-shown value, so neither party is ever shown a stale "in progress" after the refund has in fact completed, or a stale "refunded" before it has.
- **Consistent with FEAT-07.SPEC-004's idempotency pattern:** this spec mirrors, for the refund path, the same never-double-process and never-ambiguous-outcome guarantees FEAT-07.SPEC-004 applies to deposit capture, using the same idempotency-key discipline at the capability boundary.

## Edge Cases

- **The Pro's payout account is reconnected mid-retry (the blocking condition resolves before the next scheduled attempt)** -- The next scheduled retry (per the fixed cadence) picks up the now-resolved payout account automatically; no separate trigger is needed, and the refund completes on that attempt without any manual action from Talia.
- **The capability reports success for a retry attempt the product had scheduled, but a different retry attempt for the same Deposit Transaction is also in flight (overlapping retries)** -- Only one can result in a Refunded status; the other, whichever resolves second, finds the Deposit Transaction already Refunded (via the idempotency key at the capability boundary) and is treated as a no-op with no duplicate refund and no duplicate confirmation content.
- **Talia views her dashboard while a refund is mid-retry, then reloads moments after it succeeds** -- The reload shows the true current state (Refunded, flag cleared); she is never shown a stale "in progress" flag after the underlying refund has actually completed.
- **Riley closes her booking view while a refund is in progress and reopens it days later** -- Her view re-derives the current status fresh on reopen; if it has since completed, she sees it reflected as refunded, never a stale "in progress."
- **A refund stays at Refund in Progress for an extended period because the underlying blocking condition never resolves** -- The retry loop continues indefinitely on its fixed cadence; the Deposit Transaction is never silently abandoned or moved to a terminal failure state, and Talia's attention flag persists for as long as the condition remains unresolved, consistent with XBR-10's "never dropped" guarantee.
- **The payment-processing capability reports a very late refund success for a retry attempt the product's own retry loop had already superseded with a newer attempt** -- The late success is still applied if no successful refund has yet been recorded for the Deposit Transaction; if a different attempt already succeeded in the meantime, the late report is treated as the duplicate-success case and produces no double refund or double confirmation.

## Acceptance Criteria

**FEAT-09.SPEC-006-AC-01:** Given a refund cannot complete on its first attempt, when the retry loop's next scheduled attempt runs (platform parameter: `refund-retry-interval-hours` after the previous attempt), then a new refund request is submitted carrying the same attempt's idempotency key.

**FEAT-09.SPEC-006-AC-02:** Given a Deposit Transaction is already Refunded, when any further retry logic runs for that same attempt, then no further refund request is submitted -- only the already-resolved state is reflected on any surface that reads it.

**FEAT-09.SPEC-006-AC-03:** Given two overlapping retry attempts for the same Deposit Transaction are both processed, when both resolve, then only one results in a Refunded status, and the other is treated as a no-op with no duplicate refund.

**FEAT-09.SPEC-006-AC-04:** Given Talia reconnects her payout account while a refund sits at Refund in Progress, when the next scheduled retry runs, then the refund completes automatically with no separate action required from Talia beyond having reconnected the account.

**FEAT-09.SPEC-006-AC-05:** Given Talia views her dashboard while a refund is Refund in Progress, then she sees "A refund for {client name}'s cancelled booking is in progress and will complete automatically."

**FEAT-09.SPEC-006-AC-06:** Given Talia reloads her dashboard immediately after a refund has completed, then the attention flag is cleared and no stale "in progress" state is shown.

**FEAT-09.SPEC-006-AC-07:** Given Riley views her booking while her refund is Refund in Progress, then she sees "Your deposit refund is in progress and will complete automatically." -- never a failure message.

**FEAT-09.SPEC-006-AC-08:** Given Riley reopens her booking view days after her refund actually completed, then she sees it reflected as refunded, never a stale "in progress" state.

**FEAT-09.SPEC-006-AC-09:** Given any role looks for a manual "retry now" control for a refund in progress, then none exists anywhere in the product.

**FEAT-09.SPEC-006-AC-10:** Given any role looks for a way to manually mark a stuck refund as resolved, then no such control exists -- the status changes only on the capability's own confirmation.

**FEAT-09.SPEC-006-AC-11:** Given any role looks for a way to cancel or abandon an in-progress automatic refund, then no such control exists anywhere in the product.

**FEAT-09.SPEC-006-AC-12:** Given a refund's blocking condition never resolves, when successive scheduled retries continue to fail, then the Deposit Transaction remains Refund in Progress indefinitely rather than moving to any terminal failure state, and Talia's attention flag persists throughout.

**FEAT-09.SPEC-006-AC-13:** Given a very late refund-success report arrives for an attempt the retry loop had already superseded, when a different attempt already succeeded in the meantime, then the late report produces no second refund and no duplicate confirmation.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 1 | 1 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |


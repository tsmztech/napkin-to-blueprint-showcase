---
document_type: feature-overview
feature_number: FEAT-09
feature_name: Cancellation & No-Show Policy Engine
feature_slug: cancellation-no-show-policy-engine
priority_tier: Core
feature_type: Platform
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 6
screen_count: 1
automation_count: 1
logic_rule_count: 3
integration_count: 1
notification_count: 0
---

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

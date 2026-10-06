---
document_type: feature-overview
feature_number: FEAT-21
feature_name: Recurring/Standing Appointments
feature_slug: recurring-standing-appointments
priority_tier: Nice-to-Have
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 10
screen_count: 3
automation_count: 2
logic_rule_count: 2
integration_count: 0
notification_count: 3
---

# Feature Breakdown Brief: Recurring/Standing Appointments

## Summary

**Feature:** Recurring/Standing Appointments
**ID:** FEAT-21
**Description:** A client can set up a standing appointment pattern (e.g., "every 3 weeks") with the same Pro, generating individual bookings automatically instead of booking fresh each time.
**Priority:** Nice-to-Have
**Phase:** v1
**Type:** User-Facing
**Rationale:** BRIEF.md's Open Questions asks directly whether this is "v1 or later." As Visionary judgment: valuable for retention-style services (lash fills, haircuts) but not required for the founder's three-month first-paying-pro timeline, and it adds real complexity to the availability engine. Phased to v1, once the single-booking core loop is proven reliable. [INFERRED: carried from Visionary draft]

**Key Capabilities:**
- Set up a recurring pattern from an existing booking ("repeat this every N weeks")
- See and manage the upcoming generated occurrences as a group
- Cancel the whole series, or just one upcoming occurrence, independently

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-21.SPEC-001 | Set Up Recurring Series | Screen | The Client | Client opts into a recurring pattern from an existing booking, choosing an interval of every 1-12 weeks |
| FEAT-21.SPEC-002 | My Recurring Series | Screen | The Client | Client views their own series and its upcoming occurrences as a group, with no extra UI when they hold none, and cancels the whole series or one occurrence |
| FEAT-21.SPEC-003 | Recurring Series Setup & Generation Limits | Logic/Rule | The Client, The Pro | Governs the valid interval range (1-12 weeks) and the booking-horizon ceiling on how far ahead occurrences may ever be generated |
| FEAT-21.SPEC-004 | Occurrence Generation & Conflict Handling | Automation | The Client, The Pro, Platform Operator (Support) | Generates each occurrence's Booking within the horizon, subject to the same slot validation as any booking, and hands an occurrence whose usual time is no longer available to a client pick-a-new-time flow without breaking the rest of the series |
| FEAT-21.SPEC-005 | Occurrence Deposit Request & Release | Automation | The Client, The Pro, Platform Operator (Support) | Sends each occurrence's own fresh deposit link about a week before it, and releases the occurrence if the deposit is unpaid by the cancellation cut-off |
| FEAT-21.SPEC-006 | Series & Occurrence Cancellation Rules | Logic/Rule | The Client, The Pro, Platform Operator (Support) | Governs what cancelling the whole series does to its not-yet-occurred occurrences versus cancelling a single occurrence, and how a concurrent Client/Pro change to the same series or occurrence resolves |
| FEAT-21.SPEC-007 | Occurrence Generated Notification | Notification | The Client | Confirms to the client each time a new occurrence is generated |
| FEAT-21.SPEC-008 | Occurrence Time Change Advance Notice | Notification | The Client | Gives the client advance notice, with a prompt to pick a new time, when an occurrence's usual slot is no longer available |
| FEAT-21.SPEC-009 | Occurrence Deposit Lifecycle Notification | Notification | The Client, The Pro | Sends the pre-occurrence deposit-link message and, separately, the both-parties notice when an unpaid occurrence is released |
| FEAT-21.SPEC-010 | Pro Recurring Series Management | Screen | The Pro, The Client | Pro sets up a client's recurring series at the chair, views and manages that series on their own schedule, and cancels a single occurrence or ends the whole series |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Set up a recurring pattern from an existing booking ("repeat this every N weeks") | FEAT-21.SPEC-001, FEAT-21.SPEC-010, FEAT-21.SPEC-003 | The client setup screen creates the series from an existing booking; the Pro's series management screen does the same for a client at the chair; the generation-limits rule governs the valid interval and horizon before either is accepted | Phase 2 (Explicit) |
| See and manage the upcoming generated occurrences as a group | FEAT-21.SPEC-002, FEAT-21.SPEC-010, FEAT-21.SPEC-006 | My Recurring Series groups a series' occurrences on one screen for the client, and Pro Recurring Series Management does the same for the Pro's own schedule; the cancellation rules govern what each management action does | Phase 2 (Explicit) |
| Cancel the whole series, or just one upcoming occurrence, independently | FEAT-21.SPEC-002, FEAT-21.SPEC-010, FEAT-21.SPEC-006 | Both cancel actions live on My Recurring Series (client) and on Pro Recurring Series Management (Pro); the cancellation rules spec defines their distinct effects and the contention resolution when the Client and the Pro act on the same series at the same time | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-21.SPEC-003 | Recurring Series Setup & Generation Limits | Phase 5 (Rule Discovery) | The Validation & Limits field's interval bound and booking-horizon ceiling are conditional rules referenced by both the setup screen and every later generation attempt -- past the standalone threshold |
| FEAT-21.SPEC-004 | Occurrence Generation & Conflict Handling | Phase 4 (Trigger-Response) + Phase 6 (Negative/Failure Analysis) | Occurrence generation is a time-based trigger nobody's Key Capability names directly; its Alternate ("the Pro changed hours... the client is asked to pick a new time for that occurrence only") and its Error state ("a failed occurrence generation is retried and... surfaces to the Pro as a flagged gap") are unhappy-path behavior discovered, not stated, by the feature description |
| FEAT-21.SPEC-005 | Occurrence Deposit Request & Release | Phase 4 (External Dependencies lens) | The Primary Flows field's "each occurrence still requires its own deposit... paid through a deposit link sent about a week before" and the Validation & Limits field's unpaid-release rule both cross the product boundary into payment collection -- an implied automation no Key Capability names |
| FEAT-21.SPEC-007 | Occurrence Generated Notification | Phase 4 (Notification surfacing) | The Communications field names this message directly: "a confirmation when a new occurrence is generated" -- it has a defined audience and trigger, so it needs its own Notification spec rather than staying a bare side-effect row |
| FEAT-21.SPEC-008 | Occurrence Time Change Advance Notice | Phase 4 (Notification surfacing) | The Communications field names this message directly: "an advance notice if an occurrence needs a new time" -- same disposition as SPEC-007 |
| FEAT-21.SPEC-009 | Occurrence Deposit Lifecycle Notification | Phase 4 (Notification surfacing) | Not named by the Communications field itself, but surfaced from the Primary Flows field ("paid through a deposit link sent... before") and the Validation & Limits field ("both the client and the Pro are told") on release -- both are real, channeled communications, so per the disposition rule they cannot stay inline even though the Communications field is silent on them (see Shared Context recorded reading) |
| FEAT-21.SPEC-010 | Pro Recurring Series Management | Phase 3 (Entity Lifecycle) + Phase 2 (Access field) | The Access field grants the Pro Full access to series tied to their own schedule, and the dependency map has Recurring Series created "by the client, or by the Pro at the chair"; no Key Capability names a Pro-side surface, and the Pro-side Create and manage operations previously deferred to FEAT-30, which does not build recurring appointments (Pass D gap MS-01). The Pro-side screen therefore lives in this feature |

## Entity-Lifecycle Coverage Matrix

**Entity: Recurring Series**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-21.SPEC-001, FEAT-21.SPEC-010 | Set Up Recurring Series screen -- client submits an interval (1-12 weeks) from an existing booking, validated by SPEC-003, creating the series in state Active and handing off to SPEC-004 for its first occurrence(s) | Also created by the Pro at the chair via FEAT-21.SPEC-010, using the same SPEC-003 validation (per the Access field) |
| Read (single) | FEAT-21.SPEC-002, FEAT-21.SPEC-010 | My Recurring Series (client) and Pro Recurring Series Management (Pro) each load one series' detail -- interval, originating service/time, state | -- |
| Read (list) | FEAT-21.SPEC-002, FEAT-21.SPEC-010 | My Recurring Series lists the client's series and their grouped upcoming occurrences; shows no extra UI at all when the client holds none | The Pro reads the same series data through FEAT-21.SPEC-010, scoped to series tied to their own schedule; Support's view-only read is surfaced through FEAT-19, never through this feature's screens |
| Update | FEAT-21.SPEC-004 | Per-occurrence changes only -- the conflict-handling path lets the client pick a new time for one occurrence, leaving the series' interval and state untouched | The dependency map's Recurring Series entity also lists a Paused state; no Key Capability, flow, or States-field line names a pause action, so this Brief does not implement it -- flagged as a discrepancy, not resolved (see Shared Context) |
| Delete/Archive | FEAT-21.SPEC-006 | Soft -- cancelling the whole series (from FEAT-21.SPEC-002 by the client or FEAT-21.SPEC-010 by the Pro) transitions it from Active to Ended; no hard delete. No restore path -- a client who wants standing appointments again sets up a fresh series through SPEC-001. Cascade: every not-yet-occurred generated occurrence's Booking is cancelled individually, its deposit outcome following FEAT-09's ordinary policy (XBR-09); already-completed occurrences are untouched. Retention: an Ended series is kept indefinitely as history, consistent with SC-22's retention of booking and financial history | -- |
| State Transition | FEAT-21.SPEC-004, FEAT-21.SPEC-006 | A newly created series holds only Active; SPEC-006 transitions it to Ended on whole-series cancellation; SPEC-004 never changes the series-level state, only an individual occurrence's assigned time | -- |

**Entity: Booking** (scoped to this feature's Connected Entities as "create -- generated per occurrence")

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-21.SPEC-004 | Occurrence Generation creates one Booking per occurrence, in Pending Payment state, sourced as "recurring occurrence," subject to the same slot validation as any booking (FEAT-03, XBR-01) | -- |
| Read (single) | FEAT-21.SPEC-002, FEAT-21.SPEC-010 | My Recurring Series and Pro Recurring Series Management read each occurrence's Booking to show it in its series group | Ordinary single-booking reads (receipts, full detail) remain FEAT-06's/FEAT-16's responsibility, not duplicated here |
| Read (list) | FEAT-21.SPEC-002, FEAT-21.SPEC-010 | Those screens list every upcoming occurrence Booking together, grouped by series | -- |
| Update | FEAT-21.SPEC-004, FEAT-21.SPEC-005, FEAT-21.SPEC-006 | SPEC-004 updates an occurrence's Booking with a new time when the usual slot conflicts; SPEC-005 updates its deposit-related state as the request goes out and, on lapse, releases it; SPEC-006 updates it to Cancelled on an occurrence-only or whole-series cancellation | Every other Booking state transition (Confirmed, Completed, No-Show, etc.) is owned by the features that already manage Booking's full lifecycle (FEAT-07, FEAT-10, FEAT-11, FEAT-12, FEAT-30) -- this feature only ever creates the occurrence and touches the few transitions named here |
| Delete/Archive | N/A | Bookings are kept for the life of the account (dependency map; SC-22); a cancelled or expired occurrence remains as history rather than being deleted | -- |
| State Transition | FEAT-21.SPEC-004, FEAT-21.SPEC-005, FEAT-21.SPEC-006 | Pending Payment (created here) -> Confirmed (deposit paid, via FEAT-07, cross-feature) -> Completed/No-Show (via FEAT-11/FEAT-12, cross-feature) or Cancelled/Expired (via this feature's SPEC-005/SPEC-006) | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Availability Rule (via FEAT-03) | FEAT-21.SPEC-003, FEAT-21.SPEC-004 | Consulted indirectly through FEAT-03's slot truth to confirm each occurrence still fits the Pro's booking horizon and open hours; this feature never reads or edits the rule directly |
| Deposit Transaction | FEAT-21.SPEC-005 | Read to confirm whether an occurrence's deposit was captured before its cancellation cut-off; the transaction itself is created and updated by FEAT-07/FEAT-09, never by this feature |
| Messaging Consent | FEAT-21.SPEC-007, FEAT-21.SPEC-008, FEAT-21.SPEC-009 | Read to determine text-vs-email channel for all three notifications (XBR-15); this feature never changes consent |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Client or Pro submits a recurring setup request | Validate the interval (1-12 weeks) and the booking-horizon ceiling | Standalone Logic/Rule | FEAT-21.SPEC-003 |
| Client or Pro submits a valid setup request | Create the Recurring Series in state Active and hand off to occurrence generation | Inline in triggering screen | FEAT-21.SPEC-001 (client) / FEAT-21.SPEC-010 (Pro) |
| Client or Pro submits an invalid setup request (interval out of range) | Show a plain reason and keep the submitter on the setup form | Inline in triggering screen | FEAT-21.SPEC-001 / FEAT-21.SPEC-010 |
| A series is Active and its booking horizon rolls forward | Generate the next occurrence's Booking, validated against the same slot check as any booking | Standalone Automation | FEAT-21.SPEC-004 |
| An occurrence generates successfully | Confirm the new occurrence to the client | Standalone Notification | FEAT-21.SPEC-007 |
| An occurrence's usual slot is no longer available (Pro changed hours) | Notify the client in advance and ask them to pick a new time for that occurrence only, leaving the rest of the series untouched | Standalone Automation + Standalone Notification | FEAT-21.SPEC-004 governs; FEAT-21.SPEC-008 notifies |
| Occurrence generation fails | Retry; if it keeps failing, flag the gap on the Pro's dashboard rather than silently missing the appointment | Standalone Automation, with the flag itself Cross-feature -- logged in touchpoints | FEAT-21.SPEC-004 governs; FEAT-12 responsibility for the flag |
| An occurrence approaches (about a week out) | Send that occurrence's own fresh deposit link | Standalone Automation + Standalone Notification | FEAT-21.SPEC-005 governs; FEAT-21.SPEC-009 notifies |
| An occurrence's deposit is unpaid by its cancellation cut-off | Release the occurrence and tell both the client and the Pro | Standalone Automation + Standalone Notification | FEAT-21.SPEC-005 governs; FEAT-21.SPEC-009 notifies |
| Client or Pro cancels one upcoming occurrence | Cancel that occurrence's Booking only; the series continues generating future ones normally | Standalone Logic/Rule + inline cancel action | FEAT-21.SPEC-006 governs; FEAT-21.SPEC-002 (client) or FEAT-21.SPEC-010 (Pro) executes |
| Client or Pro cancels the whole series | End the series and cancel every not-yet-occurred generated occurrence | Standalone Logic/Rule + inline cancel action | FEAT-21.SPEC-006 governs; FEAT-21.SPEC-002 (client) or FEAT-21.SPEC-010 (Pro) executes |
| The Pro acts on the same series or occurrence the Client is concurrently changing | Reject-with-refresh -- the first committed change wins and the other party sees the updated series before acting | Standalone Logic/Rule | FEAT-21.SPEC-006 governs; FEAT-21.SPEC-002 / FEAT-21.SPEC-010 show the refreshed series |
| Series or occurrence transitions occur (created, generated, time-changed, cancelled) | Emit the corresponding analytics signal | Inline in triggering spec | FEAT-21.SPEC-001 / SPEC-004 / SPEC-006 / SPEC-010 |
| Client attempts to set up or manage a series while offline or connectivity drops | Plain message that connectivity is required; nothing is submitted (States field: Offline-degraded N/A -- requires connectivity) | Inline in triggering screen | FEAT-21.SPEC-001 / SPEC-002 / SPEC-010 |

## Shared Context

**Shared Entities:**
- Recurring Series -- created by SPEC-001 (client) or SPEC-010 (Pro), read/listed by SPEC-002 and SPEC-010, updated (per-occurrence time only) by SPEC-004, ended by SPEC-006. Fields in scope here: interval (1-12 weeks), originating service and time, state (Active | Ended -- see recorded reading below on Paused), generated occurrences (within the booking horizon).
- Booking (occurrence) -- created by SPEC-004, read/listed by SPEC-002 and SPEC-010, updated by SPEC-004/SPEC-005/SPEC-006. Fields in scope here: source ("recurring occurrence"), the originating series reference, and the same fields any Booking carries (service, start_time, deposit_amount, state).
- Messaging Consent (read-only) -- read by SPEC-007, SPEC-008, and SPEC-009 to choose the notification channel.

**Shared UI Patterns:**
- Occurrence group list pattern -- SPEC-002 (client) and SPEC-010 (Pro) present every upcoming occurrence under its series with its own status (upcoming, awaiting deposit, needs new time) and its own cancel action, so cancelling one occurrence never reads as cancelling the series.
- Setup form pattern -- SPEC-001 uses a single interval field (every N weeks, 1-12), not a calendar picker, matching the "every 3 weeks" framing in the feature description; Spec Writers should describe it consistently.

**Shared Validation:**
- FEAT-21.SPEC-003 (Recurring Series Setup & Generation Limits) is referenced, not duplicated, by SPEC-001 and SPEC-010 at setup time and by SPEC-004 at every later generation attempt.
- FEAT-21.SPEC-006 (Series & Occurrence Cancellation Rules) is referenced, not duplicated, by SPEC-002's and SPEC-010's cancel actions; there is no separate cross-feature Pro-side cancel path.

**Recorded readings (decisions, not resolutions of ambiguity):**
- **Paused state is not implemented.** The dependency map's Recurring Series entity lists Active | Paused | Ended among its Fields, but no Key Capability, Primary Flow, Alternate, or States-field line in the feature entry describes a pause interaction -- only setting up, viewing/managing as a group, and cancelling (whole or one occurrence) are named. Per Phase 1 guidance that discrepancies between the feature entry and the dependency map are flagged, not resolved, this Brief models only Active and Ended and does not invent a pause capability beyond what Stage 2 decided; recorded again in Non-Goals below.
- **A third notification beyond the Communications field.** The Communications field names exactly two messages (occurrence-generated confirmation; advance notice for a time change). This Brief adds a third, Occurrence Deposit Lifecycle Notification (SPEC-009), surfaced by Phase 4's Notification-surfacing lens from the Primary Flows field's "deposit link sent about a week before" and the Validation & Limits field's "both the client and the Pro are told" on release. Both are Stage 2 decisions with a real channel, audience, and content, so per the standalone-Notification disposition rule they cannot stay bare side-effect rows even though the Communications field itself is silent on them.
- **ASMP-29's daytime-hours rule applies to all three of this feature's notifications.** Unlike a time-critical claim-window notification elsewhere in the product, none of this feature's messages depend on a short window for their value -- the generated confirmation, the time-change advance notice, and the deposit-request/release notices are all ordinary reminder-class communications, so all follow the roughly 8am-9pm daytime-hours rule.

## Internal Dependency Map

```
SPEC-001 (Set Up Recurring Series) -> [validates against] -> SPEC-003 (Recurring Series Setup & Generation Limits)
SPEC-001 (Set Up Recurring Series) -> [client submits a valid setup] -> Recurring Series created (Active) -> [hands off to] -> SPEC-004 (Occurrence Generation & Conflict Handling)
SPEC-010 (Pro Recurring Series Management) -> [validates against] -> SPEC-003 (Recurring Series Setup & Generation Limits)
SPEC-010 (Pro Recurring Series Management) -> [Pro submits a valid setup for a client] -> Recurring Series created (Active) -> [hands off to] -> SPEC-004 (Occurrence Generation & Conflict Handling)
SPEC-010 (Pro Recurring Series Management) -> [Pro cancels one occurrence or the whole series] -> [checked against] -> SPEC-006 (Series & Occurrence Cancellation Rules) -> [Booking(s) and/or series updated]
SPEC-004 (Occurrence Generation & Conflict Handling) -> [creates] -> Booking (occurrence) -> [appears on] -> SPEC-010 (Pro Recurring Series Management)
SPEC-006 (Series & Occurrence Cancellation Rules) -> [series or occurrence state changes] -> SPEC-010 (Pro Recurring Series Management) [list reflects the new status]
SPEC-004 (Occurrence Generation & Conflict Handling) -> [validates against] -> SPEC-003 (Recurring Series Setup & Generation Limits)
SPEC-004 (Occurrence Generation & Conflict Handling) -> [creates] -> Booking (occurrence) -> [appears on] -> SPEC-002 (My Recurring Series)
SPEC-004 (Occurrence Generation & Conflict Handling) -> [occurrence generated successfully] -> SPEC-007 (Occurrence Generated Notification)
SPEC-004 (Occurrence Generation & Conflict Handling) -> [usual slot no longer available] -> SPEC-008 (Occurrence Time Change Advance Notice)
SPEC-004 (Occurrence Generation & Conflict Handling) -> [occurrence approaches] -> SPEC-005 (Occurrence Deposit Request & Release)
SPEC-005 (Occurrence Deposit Request & Release) -> [deposit requested, or occurrence released unpaid] -> SPEC-009 (Occurrence Deposit Lifecycle Notification)
SPEC-002 (My Recurring Series) -> [client cancels one occurrence or the whole series] -> [checked against] -> SPEC-006 (Series & Occurrence Cancellation Rules) -> [Booking(s) and/or series updated]
SPEC-006 (Series & Occurrence Cancellation Rules) -> [series or occurrence state changes] -> SPEC-002 (My Recurring Series) [list reflects the new status]
```

**Default Entry:** This feature has no single default landing screen -- SPEC-001 (Set Up Recurring Series) is reached only from FEAT-05's post-booking confirmation outbound link, and SPEC-002 (My Recurring Series) is reached only from FEAT-06's own-bookings-view outbound link, and SPEC-010 (Pro Recurring Series Management) is reached only from FEAT-30's Pro booking detail outbound link; nobody navigates to this feature area directly.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-21.SPEC-001 | Inbound | FEAT-05 (Public Booking Page & Booking Flow) | Client opts into a recurring pattern from the post-booking confirmation | Client completes a booking and is offered the option to make it recurring |
| FEAT-21.SPEC-002 | Inbound | FEAT-06 (Client Booking Identity) | Client reaches My Recurring Series from their own bookings view | Client opens their own bookings view |
| FEAT-21.SPEC-010, FEAT-21.SPEC-003 | Inbound | FEAT-30 (Pro Booking Management) | Pro sets up a recurring series for a client at the chair on SPEC-010, subject to the same setup validation | Pro chooses "repeat this every N weeks" for a client at the chair |
| FEAT-21.SPEC-010, FEAT-21.SPEC-006 | Inbound | FEAT-30 (Pro Booking Management) | Pro reaches series management and cancel actions for a series or occurrence tied to their own schedule from a booking in Pro Booking Management | Pro opens a recurring booking's series from Pro Booking Management |
| FEAT-21.SPEC-004 | Outbound | FEAT-03 (Real-Time Slot Availability Engine) | Every generated occurrence is subject to the same live slot check and reserves the future slot (XBR-01, XBR-02) | Occurrence generation attempt |
| FEAT-21.SPEC-003, FEAT-21.SPEC-004 | Outbound | FEAT-02 (Availability & Working Hours Setup) | Generation never reaches further ahead than the Pro's configured booking horizon (XBR-03) | Occurrence generation attempt |
| FEAT-21.SPEC-005 | Outbound | FEAT-07 (Deposit Payment at Booking) | Each occurrence's own deposit is captured fresh through FEAT-07's deposit capture (XBR-05); no card is retained between occurrences | Deposit link sent about a week before an occurrence |
| FEAT-21.SPEC-006 | Outbound | FEAT-09 (Cancellation & No-Show Policy Engine) | The deposit outcome of a cancelled occurrence (refund vs. kept) follows the ordinary policy engine rule (XBR-09), not a rule this feature defines | Client or Pro cancels one occurrence or the whole series |
| FEAT-21.SPEC-004, FEAT-21.SPEC-006, FEAT-21.SPEC-010 | Outbound | FEAT-04 (Two-Way Calendar Sync) | Every generated, rescheduled, or cancelled occurrence is mirrored to the Pro's personal calendar (XBR-13) | Occurrence created, moved, or cancelled |
| FEAT-21.SPEC-007, FEAT-21.SPEC-008, FEAT-21.SPEC-009 | Outbound | FEAT-08 (Automated Booking Messaging) | All three notifications are delivered through the shared transactional text/email transports (FEAT-08.SPEC-012, FEAT-08.SPEC-013), not a spec of this feature's own | Any of this feature's notifications fires |
| FEAT-21.SPEC-007, FEAT-21.SPEC-008, FEAT-21.SPEC-009 | Outbound | FEAT-14 (Messaging Consent Management) | Channel choice (text vs. email) follows the client's current textability determination (XBR-15, FEAT-14.SPEC-007) | Before any notification is sent |
| FEAT-21.SPEC-004 | Outbound | FEAT-12 (Pro Daily Schedule Dashboard) | A repeatedly failing occurrence generation surfaces as a flagged gap on the Pro's dashboard rather than a separate screen here | Generation retries are exhausted |
| FEAT-21 (feature-wide) | Outbound | FEAT-16 (Booking & Payment Activity Record) | Occurrence lifecycle events (generated, time-changed, cancelled) land on the underlying Booking's activity timeline via FEAT-16, not a separate log this feature owns | An occurrence is generated, time-changed, or cancelled |
| FEAT-21 (feature-wide) | Inbound | FEAT-19 (Platform Support Read-Only Access) | Support's view-only access to Recurring Series is surfaced through FEAT-19's own screen, never through this feature's screens | Support opens a Pro's account after a help request |

## Non-Functional Notes

**Data volumes / growth:** Bounded and small by design -- occurrences are only ever generated within the Pro's booking horizon (1 week to 12 months, default 8 weeks, per the Availability Rule), so a series' working set of upcoming occurrences never exceeds that window; across a Pro's roughly 100-500 clients (ASMP-22) only some adopt a standing pattern at all, keeping total series volume small relative to ordinary bookings.

**Responsiveness:** Setup and management follow the same roughly-one-second, under-one-minute responsiveness expectation as any other client-facing screen (ASMP-21), though both are occasional actions rather than the core booking flow. Occurrence generation, deposit requests, and their notices are not time-critical the way a slot-claim window is -- ASMP-29's roughly 8am-9pm daytime-hours rule applies to all three of this feature's notifications (see Shared Context recorded reading).

**Data sensitivity / privacy:** A Recurring Series is personal data -- it reveals a client's standing appointment pattern with a specific Pro (dependency map, Data Sensitivity). It is Own-only for the Client, Full for the Pro on series tied to their own schedule, and view-only for Platform Operator (Support), surfaced through FEAT-19 (Access Matrix).

**Compliance flags:** ASMP-24 (US SMS-consent rules) governs all three notifications -- text only with active consent, otherwise email, per XBR-15. No health or financial regime applies; the occurrence deposit itself is a category-level payment-processing dependency (ASMP-31) owned by FEAT-07, not this feature.

**Signals:** recurring_series_created (SPEC-001, on successful setup), recurring_occurrence_generated (SPEC-004, on each successful generation), recurring_series_cancelled (SPEC-006, on the whole-series cancellation path) -- these three Stage 2 signals are fully covered. Stage 2 names no distinct signal for cancelling a single occurrence; that action reuses whatever generic booking-cancelled signal the owning cancellation feature already emits for any Booking, rather than a new series-specific signal invented here.

## Non-Goals

- **A dedicated pause action for a recurring series** -- Excluded per the recorded reading above: the dependency map's Recurring Series entity names a Paused state among its Fields, but no Key Capability, Primary Flow, Alternate, or States-field line in product-features.md describes a pause interaction. Only Active and Ended are exercised by this feature's actual decisions; introducing a pause capability would go beyond what Stage 2 defined.
- **Editing a series' interval after creation** -- Excluded because the feature's three Key Capabilities name only setup, group viewing/management, and cancellation (whole or one occurrence) -- changing the interval mid-series is not a named capability. A client who wants a different cadence cancels and sets up a fresh series through SPEC-001.
- **Partial refunds or a tiered cancellation schedule for a released or cancelled occurrence** -- Excluded per scope-boundaries.md SC-18: BRIEF.md's Business Context defines a binary deposit rule (refunded outside the window, kept inside it or on a no-show); this feature's occurrences follow that same binary rule (XBR-09) rather than inventing a proration scheme for standing appointments.
- **Retaining or reusing a client's card across occurrences** -- Excluded per scope-boundaries.md SC-11 and SC-13: card data is never stored or handled by the product, so each occurrence's deposit is paid fresh through its own link, exactly as the feature's own Primary Flows field states ("card details are never kept between occurrences").
- **Recurring-series management inside FEAT-30's own screens** -- Excluded: FEAT-30 keeps recurring appointments as a Non-Goal, so the Pro's Full access to series tied to their own schedule is satisfied by this feature's SPEC-010 (Pro Recurring Series Management), reached from FEAT-30 by an outbound link. Supersedes the earlier reading that routed Pro-side actions to a FEAT-30 screen (Pass D gap MS-01).

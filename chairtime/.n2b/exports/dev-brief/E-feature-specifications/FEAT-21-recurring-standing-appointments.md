# FEAT-21 — Recurring/Standing Appointments

This chapter covers Recurring/Standing Appointments (FEAT-21), a Nice-to-Have-tier feature. It carries 10 specifications carrying 146 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-21.SPEC-001 | Set Up Recurring Series | screen | 13 |
| FEAT-21.SPEC-002 | My Recurring Series | screen | 14 |
| FEAT-21.SPEC-003 | Recurring Series Setup & Generation Limits | logic-rule | 15 |
| FEAT-21.SPEC-004 | Occurrence Generation & Conflict Handling | automation | 14 |
| FEAT-21.SPEC-005 | Occurrence Deposit Request & Release | automation | 13 |
| FEAT-21.SPEC-006 | Series & Occurrence Cancellation Rules | logic-rule | 16 |
| FEAT-21.SPEC-007 | Occurrence Generated Notification | notification | 12 |
| FEAT-21.SPEC-008 | Occurrence Time Change Advance Notice | notification | 12 |
| FEAT-21.SPEC-009 | Occurrence Deposit Lifecycle Notification | notification | 15 |
| FEAT-21.SPEC-010 | Pro Recurring Series Management | screen | 22 |

The feature breakdown brief follows, then every specification in full.


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



# Screen Spec: Set Up Recurring Series

## Overview

**Name:** Set Up Recurring Series
**ID:** FEAT-21.SPEC-001
**Type:** Screen
**Purpose:** Lets Riley turn the booking she just made into a standing appointment by choosing how often it repeats, so future visits with Talia are generated automatically instead of booked one at a time.
**Parent Feature:** FEAT-21 -- Recurring/Standing Appointments

## Scope and Non-Goals

**In Scope:**
- Offering the recurring option immediately after a booking confirms, and capturing the repeat interval (every 1 to 12 weeks)
- Creating the Recurring Series in state Active from the just-confirmed Booking, and handing off to occurrence generation
- Showing a plain reason and keeping the client on the form when the chosen interval is out of range
- The offline/degraded behavior for this occasional, connectivity-dependent action

**Non-Goals:**
- Editing a series' interval after creation -- excluded per this Brief's Non-Goals: the feature's three Key Capabilities name only setup, group viewing/management, and cancellation; a client who wants a different cadence cancels and sets up a fresh series here again.
- Viewing or managing existing series and their occurrences -- owned by FEAT-21.SPEC-002 (My Recurring Series); this screen only ever creates a new series from a booking that was just confirmed.
- The Pro setting up a series for a client at the chair -- per the Access field's own wording, that path runs through FEAT-21.SPEC-010 (Pro Recurring Series Management, reached from FEAT-30's Pro booking detail), which invokes the same interval and horizon validation (FEAT-21.SPEC-003) from its own screen rather than this one.
- Validating the chosen interval and the booking-horizon ceiling -- owned by FEAT-21.SPEC-003 (Recurring Series Setup & Generation Limits); this screen only surfaces the result.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-05.SPEC-005 (Booking Confirmation) | Riley sees her booking's on-screen confirmation (FEAT-05.SPEC-005's footer action) and taps "Make this a standing appointment" | The just-confirmed Booking's reference, service, and start time; no separate sign-in step, since this follows directly from the confirmation the client is already viewing |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Full screen | Choose an interval and submit, for her own just-confirmed booking only | -- |
| The Pro (Talia) | No | No | Talia never reaches this screen; her equivalent action -- setting up a series for a client at the chair -- runs through FEAT-21.SPEC-010 (Pro Recurring Series Management), a distinct screen from this one (Non-Goals) |
| Platform Operator (Support) | No | No | This screen is client-facing and reached only from a client's own booking confirmation; Support's read-only view of a Recurring Series is surfaced through FEAT-19's own screen, never through this feature's screens (Capability Coverage Map) |
| Unauthenticated | Yes -- this screen carries no separate sign-in of its own | Yes, for the one booking just confirmed in the same session | There is no "unauthenticated" denial state on this screen: the client's identity for this one action is the booking session they are already in, not a signed-in account (consistent with the Client persona never holding a password-style account) |
| Expired session | Partial -- the confirmation context this screen depends on is time-bound | No, once the underlying confirmation context has lapsed | If Riley reaches this offer after the booking confirmation context has expired (for example, from a stale bookmark), she sees "This offer has expired. Open your booking to set up a recurring series." with a link to request access via FEAT-06 (Access Link Request), rather than a session sign-in prompt |

## Layout and Content

**Header:** Screen title "Make this a standing appointment?" with a back arrow (returns to FEAT-05.SPEC-005, Booking Confirmation, without setting up a series).

**Body:** A single-column form with the following elements, in order:
- Short explanatory line: "We'll book this automatically with {pro_display_name} every time it's due, using the same service and time."
- **Repeat interval** (numeric stepper input, required): a whole-number-of-weeks value, from 1 to 12, labeled "Repeat every ___ weeks." Defaults to no value pre-selected -- Riley must choose one.
- **Skip this** (secondary text link, below the stepper): declines the offer without creating a series.

**Footer:** A single primary "Set up recurring" action button, full width.

### Responsive Behavior

- **Compact size class:** Single-column form as described above, full width; the primary action stays pinned in the footer.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width (the design layer's decision) and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-05.SPEC-005 (Booking Confirmation) without creating a series | Screen closes | Animated transition back to the confirmation |
| Repeat interval stepper | Increment/decrement or type a value | Captures the chosen interval | Stepper shows the new value; "Set up recurring" becomes enabled once a value is chosen | Stepper shows the new value |
| Skip this | Tap | Declines the offer; no Recurring Series is created | Screen closes | Animated transition back to FEAT-05.SPEC-005 (Booking Confirmation), unchanged |
| Set up recurring button | Tap | 1. Validate the chosen interval via FEAT-21.SPEC-003. 2. If valid, create the Recurring Series in state Active from the just-confirmed Booking and hand off to FEAT-21.SPEC-004 for its first occurrence(s). | Button shows a loading state during submission | Success: toast "Recurring series set up -- every {interval} weeks" and navigate to FEAT-21.SPEC-002 (My Recurring Series). Failure: inline error banner or field-level message. |
| Set up recurring button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> explanatory line (read-only, not a focus stop) -> Repeat interval stepper -> Skip this -> Set up recurring.
- **Validation announcements:** When the interval is out of range, the error message is announced to assistive technology and programmatically associated with the stepper.
- **Success/failure announcements:** The "Recurring series set up" toast and any error banner are announced to assistive technology on appearance.
- **Keyboard alternatives:** The stepper's increment/decrement is reachable by keyboard (arrow keys or direct numeric entry); there are no pointer-only gestures on this screen.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (default) | Stepper unset, "Set up recurring" disabled | Screen first opens from the booking confirmation | Riley chooses an interval value |
| Filling | Stepper shows Riley's chosen value, "Set up recurring" enabled | Riley sets an interval | Riley taps "Set up recurring" or "Skip this," or navigates away |
| Validating/Submitting | "Set up recurring" shows a loading spinner, stepper disabled | Riley taps "Set up recurring" with a chosen value | Validation and creation complete or fail |
| Validation Error | Stepper shows an error state with the message below it; "Set up recurring" re-enabled | FEAT-21.SPEC-003 rejects the chosen interval | Riley corrects the value |
| Error | Error banner at the top of the form with a Retry option; the chosen value is preserved | Series creation fails after passing validation (e.g., a processing error) | Riley taps Retry or navigates away |
| Offline/Degraded | N/A -- requires connectivity, consistent with the rest of scheduling (per this Brief's States field); a submission attempted without connectivity shows the plain message "You'll need to be online to set this up. Please check your connection and try again." and nothing is submitted or queued | Connectivity lost while attempting to submit | Connectivity restored and Riley retries |

## Validation Rules

Validation governed by FEAT-21.SPEC-003 (Recurring Series Setup & Generation Limits). See that spec for the interval range rule and its exact error message. This screen applies validation on submit.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-05.SPEC-005 (Booking Confirmation) | FEAT-05 (Public Booking Page & Booking Flow) |
| Skip this tap | FEAT-05.SPEC-005 (Booking Confirmation) | FEAT-05 (Public Booking Page & Booking Flow) |
| Successful setup | FEAT-21.SPEC-002 (My Recurring Series) | -- |

## Data Model

**Creates:** Recurring Series -- interval (from the stepper), originating service and time (auto-populated from the just-confirmed Booking, not independently entered), state set to Active.
**Reads:** Booking -- service, start_time, from the just-confirmed booking this screen was reached from.
**Updates:** None.
**Deletes:** None.

## Business Rules

- Interval validation and the booking-horizon ceiling are governed entirely by FEAT-21.SPEC-003 -- this screen cannot save a series that spec would reject.
- Creating the series hands off immediately to FEAT-21.SPEC-004 (Occurrence Generation & Conflict Handling) for its first occurrence(s) within the Pro's current booking horizon; Riley does not wait on this screen for that generation to complete.
- XBR-01: the originating service and time are fixed at the moment the underlying Booking was confirmed; this screen never lets Riley pick a different service or time for the series -- it repeats the booking she just made.

## Edge Cases

- **Riley navigates away with a chosen interval but before submitting** -- No confirmation dialog is shown and no series is created; unlike a data-entry form, declining this optional offer carries no risk of lost work worth interrupting for.
- **Riley taps "Set up recurring" twice rapidly** -- The second tap is ignored while the first submission is in progress (button in loading state).
- **Network failure during submission** -- Error banner: "Could not set up your recurring series. Check your connection and try again." with a Retry button. The chosen interval is preserved.
- **Riley chooses an interval of 0 or 13+ weeks** -- FEAT-21.SPEC-003 rejects it; the stepper shows the error and the client stays on this screen with her attempted value visible.
- **The underlying booking is cancelled in the moments between confirmation and this screen loading** -- The offer is withdrawn: the screen shows "This booking is no longer active, so it can't be made recurring." with a single option returning to FEAT-06 (Client Booking Identity), since there is no confirmed booking left to repeat.
- **Riley reaches this screen a second time for the same booking (for example, via back navigation) after already setting up a series from it** -- The offer is not shown again; she is routed directly to FEAT-21.SPEC-002 (My Recurring Series) instead, since a booking can originate at most one series.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-05.SPEC-005 (Booking Confirmation) | Navigation (inbound/outbound) | Riley arrives here when she taps "Make this a standing appointment" on the confirmation; back arrow and "Skip this" return her there |
| FEAT-21.SPEC-003 (Recurring Series Setup & Generation Limits) | References (outbound) | Supplies the interval validation rule and error message |
| FEAT-21.SPEC-004 (Occurrence Generation & Conflict Handling) | Triggers (outbound) | Successful setup hands off to generate the series' first occurrence(s) |
| FEAT-21.SPEC-002 (My Recurring Series) | Navigation (outbound) | Successful setup navigates here |
| FEAT-06 (Client Booking Identity) | Navigation (outbound) | The withdrawn-offer edge case routes here when the underlying booking is no longer active |
| FEAT-21.SPEC-010 (Pro Recurring Series Management) | References (sibling) | The Pro-side counterpart that creates a series through the same FEAT-21.SPEC-003 validation and the same hand-off to FEAT-21.SPEC-004 |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| recurring_series_created | interval_weeks, entry source (client post-booking) | Successful submission creates the Recurring Series | supports success-metrics.md: "Self-Service Reschedule Rate" (a client managing their own standing cadence in-app, without contacting the Pro, is the same self-service pattern this metric measures) |
| recurring_setup_offer_declined | reason (skip / navigated away) | Riley taps "Skip this," or navigates away without submitting | N/A -- no Stage 2 metric measures decline rate for this optional offer; retained so adoption of the capability is observable |
| recurring_setup_validation_failed | attempted interval value | FEAT-21.SPEC-003 rejects the chosen interval | N/A -- no Stage 2 metric measures setup validation failures; retained so setup friction on this screen is observable |

## Acceptance Criteria

**FEAT-21.SPEC-001-AC-01:** Given Riley just saw her booking confirmed, when the confirmation offers to make it recurring and she taps it, then she lands on this screen with the interval stepper unset and "Set up recurring" disabled.

**FEAT-21.SPEC-001-AC-02:** Given Riley sets the interval to 3 weeks and taps "Set up recurring," when validation passes, then a Recurring Series is created in state Active from her booking, she sees the toast "Recurring series set up -- every 3 weeks," and she lands on FEAT-21.SPEC-002 (My Recurring Series).

**FEAT-21.SPEC-001-AC-03:** Given Riley sets the interval to 0 weeks and taps "Set up recurring," then the stepper shows FEAT-21.SPEC-003's error message and she remains on this screen with her attempted value visible.

**FEAT-21.SPEC-001-AC-04:** Given Riley sets the interval to 13 weeks and taps "Set up recurring," then the stepper shows FEAT-21.SPEC-003's error message and she remains on this screen.

**FEAT-21.SPEC-001-AC-05:** Given Riley taps "Skip this," then the screen closes without creating a series and she returns to her booking confirmation unchanged.

**FEAT-21.SPEC-001-AC-06:** Given Riley taps the back arrow with an interval already chosen but not submitted, then she returns to her booking confirmation and no series is created.

**FEAT-21.SPEC-001-AC-07:** Given Riley successfully creates a series, then FEAT-21.SPEC-004 is triggered to generate its first occurrence(s) within the Pro's current booking horizon.

**FEAT-21.SPEC-001-AC-08:** Given Riley taps "Set up recurring" and the operation fails due to a network error, then an error banner reads "Could not set up your recurring series. Check your connection and try again." with a Retry button, and her chosen interval is preserved.

**FEAT-21.SPEC-001-AC-09:** Given Riley loses connectivity while on this screen and attempts to submit, then she sees "You'll need to be online to set this up. Please check your connection and try again." and nothing is submitted.

**FEAT-21.SPEC-001-AC-10:** Given Riley's underlying booking is cancelled before she reaches this screen, when the screen loads, then she sees "This booking is no longer active, so it can't be made recurring." with a single option returning to FEAT-06.

**FEAT-21.SPEC-001-AC-11:** Given Riley already set up a series from this booking and navigates back to this offer, when the screen loads, then she is routed directly to FEAT-21.SPEC-002 instead of seeing the offer again.

**FEAT-21.SPEC-001-AC-12:** Given Talia wants to set up a standing appointment for a client at the chair, when she looks for that action, then it is not on this screen -- it is on FEAT-21.SPEC-010 (Pro Recurring Series Management), reached from FEAT-30's Pro booking detail.

**FEAT-21.SPEC-001-AC-13:** Given Riley reaches this offer through a stale link after the confirmation context has expired, then she sees "This offer has expired. Open your booking to set up a recurring series." with a link to FEAT-06 (Access Link Request), not a sign-in prompt.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 5 (empty, filling, validating, validation error, error) plus offline N/A | 6 |
| Business Rules | 3 | 3 |
| Edge Cases | 6 | 6 |



# Screen Spec: My Recurring Series

## Overview

**Name:** My Recurring Series
**ID:** FEAT-21.SPEC-002
**Type:** Screen
**Purpose:** Lets Riley see her standing appointment series and its upcoming occurrences grouped together, and cancel the whole series or just one occurrence, with no extra UI at all when she holds no series.
**Parent Feature:** FEAT-21 -- Recurring/Standing Appointments

## Scope and Non-Goals

**In Scope:**
- Listing every Recurring Series Riley holds with this Pro, each with its upcoming generated occurrences grouped underneath it
- Showing each occurrence's own status (upcoming, awaiting deposit, needs new time)
- Cancelling one occurrence, or the whole series, from this screen
- The fully-optional empty state when Riley holds no series

**Non-Goals:**
- Setting up a new series -- owned by FEAT-21.SPEC-001 (Set Up Recurring Series); this screen only manages series that already exist.
- Editing a series' interval -- excluded per this Brief's Non-Goals: a client who wants a different cadence cancels here and sets up a fresh series through FEAT-21.SPEC-001.
- Defining what cancellation actually does to the series and its occurrences -- owned by FEAT-21.SPEC-006 (Series & Occurrence Cancellation Rules); this screen only exposes the two cancel actions and shows their result.
- Viewing a single occurrence's full booking detail (receipt, deposit breakdown) -- remains FEAT-06's/FEAT-16's responsibility per the Entity-Lifecycle Coverage Matrix; this screen shows only what identifies an occurrence within its series group.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-06.SPEC-003 (My Bookings List) | Riley opens her own bookings view and taps the "Recurring appointments" row (shown by FEAT-06.SPEC-003 only when she holds a series) | None -- this screen loads all of Riley's own series with this Pro |
| FEAT-21.SPEC-001 (Set Up Recurring Series) | Riley completes setting up a new series | The just-created series, shown first in the list |
| FEAT-21.SPEC-007 (Occurrence Generated Notification) | Client taps "View my series" in an occurrence-generated notice | The client's series reference |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Full screen, her own series with this Pro only | Cancel one occurrence, or the whole series, for her own series only | -- |
| The Pro (Talia) | No | No | Talia never reaches this screen; her Full access to series tied to her own schedule is satisfied by FEAT-21.SPEC-010 (Pro Recurring Series Management), reached from FEAT-30's Pro booking detail |
| Platform Operator (Support) | No | No | This screen is client-facing; Support's View-only access to Recurring Series is surfaced through FEAT-19's own screen, never through this feature's screens (Capability Coverage Map) |
| Unauthenticated | No | No | Reached only via a client's own access link or manage-link session (FEAT-06); an unauthenticated visitor is shown the "request a new link" prompt rather than this screen |
| Expired session | No | No | If Riley's access link session has expired, she is shown "This link has expired. Request a new one to see your bookings." (FEAT-06.SPEC-001, Access Link Request) instead of this screen |

## Layout and Content

**Header:** Screen title "Recurring appointments" with a back arrow (returns to FEAT-06.SPEC-003, My Bookings List).

**Body:** One card per Recurring Series Riley holds with this Pro, each containing:
- Series summary line: "{service_name}, every {interval} weeks" and a "Cancel series" text action
- Below the summary, a list of the series' upcoming occurrences, each row showing: the occurrence's date and time, its status badge (Upcoming / Awaiting deposit / Needs new time), and a "Cancel this one" text action scoped to that single occurrence

Cards are ordered by the series' next upcoming occurrence date, soonest first.

**Footer:** None.

### Responsive Behavior

- **Compact size class:** Cards stack full width in a single column, as described above.
- **Medium size class and above:** Cards remain single-column, capped at a consistent platform-wide content width (the design layer's decision) and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-06.SPEC-003 (My Bookings List) | Screen closes | Animated transition back to the bookings list |
| "Cancel series" (per card) | Tap | Opens a confirmation dialog, then triggers FEAT-21.SPEC-006's whole-series cancellation rule if confirmed | Dialog appears; on confirm, the card is removed from the list once cancellation completes | Confirmation dialog: "Cancel this whole series? Your next {occurrence_count} upcoming appointments will be cancelled." with "Cancel Series" and "Keep Series" options; on success, toast "Series cancelled" |
| "Cancel this one" (per occurrence row) | Tap | Opens a confirmation dialog, then triggers FEAT-21.SPEC-006's single-occurrence cancellation rule if confirmed | Dialog appears; on confirm, the occurrence row is removed once cancellation completes, series card remains | Confirmation dialog: "Cancel this appointment on {occurrence_date}? The rest of your series will continue as usual." with "Cancel Appointment" and "Keep It" options; on success, toast "Appointment cancelled" |
| Occurrence status badge | Display only | No action -- read-only status indicator | None | Not interactive |

### Accessibility Notes

- **Focus order:** Back arrow -> each series card in list order -> within a card: series summary, "Cancel series", then each occurrence row's date/status and "Cancel this one" action, top to bottom.
- **Dynamic content announcements:** A confirmation dialog's text is announced to assistive technology on open; the "Series cancelled" and "Appointment cancelled" toasts are announced on success.
- **Keyboard alternatives:** Every cancel action and dialog choice is reachable by keyboard; there are no pointer-only gestures on this screen.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (default) | No extra UI at all -- the recurring-series section is not shown on FEAT-06.SPEC-003 and this screen is not reachable, since this Brief's States field defines the empty state as fully optional | Riley holds no Recurring Series with this Pro | Riley sets up a series through FEAT-21.SPEC-001 |
| Loaded | One or more series cards shown with their occurrence groups, as described in Layout and Content | Riley holds at least one series | Riley navigates away, or her last series is cancelled (returning to Empty) |
| Loading | A brief loading indicator in place of the card list | Screen first opens while series data is being fetched | Data loads (transition to Loaded) or fails (transition to Error) |
| Error | Error banner at the top with a Retry option; no card list shown | The series data fails to load | Riley taps Retry, or navigates away |
| Cancelling | The card or row being cancelled shows a brief loading indicator; its cancel action is disabled | Riley confirms a cancel dialog | Cancellation completes (row/card removed) or fails (Error) |
| Offline/Degraded | N/A -- requires connectivity for correctness, consistent with the rest of scheduling (per this Brief's States field); a cancel attempted without connectivity shows "You'll need to be online to do this. Please check your connection and try again." and nothing is submitted | Connectivity lost while attempting to cancel | Connectivity restored and Riley retries |

## Validation Rules

Validation governed by FEAT-21.SPEC-006 (Series & Occurrence Cancellation Rules). See that spec for the exact behavior of each cancel action, including the contention outcome when the Pro acts on the same series or occurrence at the same time.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-06.SPEC-003 (My Bookings List) | FEAT-06 (Client Booking Identity) |
| Successful series cancellation | This screen, with the cancelled card removed | -- |
| Successful occurrence cancellation | This screen, with the cancelled row removed | -- |

## Data Model

**Reads:** Recurring Series -- interval, originating service and time, state, generated occurrences (Riley's own series with this Pro only); Booking (occurrence) -- service, start_time, state, for each occurrence shown in a series group.
**Creates:** None.
**Updates:** Recurring Series -- state transitioned to Ended (via FEAT-21.SPEC-006, on whole-series cancel). Booking (occurrence) -- state transitioned to Cancelled (via FEAT-21.SPEC-006).
**Deletes:** None.

## Business Rules

- Both cancel actions are governed entirely by FEAT-21.SPEC-006 -- this screen never decides what cancellation does to a series or its occurrences, only exposes the two actions and reflects the result.
- The occurrence group list pattern (Brief's Shared UI Patterns) is followed exactly: cancelling one occurrence never reads as cancelling the series -- the two actions are visually and functionally distinct, with the series card remaining after a single-occurrence cancel.
- An occurrence's status badge reflects its Booking state as read from FEAT-21.SPEC-004 (generation) and FEAT-21.SPEC-005 (deposit lifecycle): Upcoming (Pending Payment, deposit not yet requested, or Confirmed), Awaiting deposit (deposit link sent, unpaid), Needs new time (usual slot unavailable, per FEAT-21.SPEC-004's conflict-handling path).

## Edge Cases

- **Riley has no upcoming occurrences left in a series (all generated occurrences have completed or been cancelled, but the series is still Active)** -- The series card still shows with an empty occurrence list and the note "Your next appointment will appear here once it's scheduled," since the series itself remains Active and will keep generating occurrences as the horizon rolls forward.
- **The Pro cancels the same series Riley is viewing, at the same time Riley taps "Cancel series" (concurrent-edit conflict)** -- Per the dependency map's Recurring Series Contention note, the resolution is reject-with-refresh: whichever cancellation commits first wins, and the other party's screen refreshes to show the already-cancelled state before their action completes; Riley sees "This series was just cancelled." instead of the usual success toast if the Pro's cancellation committed first.
- **Riley taps "Cancel series" or "Cancel this one" twice rapidly** -- The second tap is ignored while the first cancellation is in progress (action disabled during the Cancelling state).
- **Network failure during a cancel action** -- Error banner: "Could not complete that cancellation. Check your connection and try again." with a Retry button; the card/row remains in its pre-cancellation state.
- **An occurrence Riley tries to cancel was released unpaid moments earlier by FEAT-21.SPEC-005** -- The action is refused with refresh: "This appointment is no longer active." and the row updates to reflect the release, since there is nothing left for Riley's cancel action to act on.
- **Riley reopens this screen immediately after cancelling an occurrence or series** -- The list reflects the just-completed cancellation, not a stale cached view.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-003 (My Bookings List) | Navigation (inbound/outbound) | Riley reaches this screen from the "Recurring appointments" row on her own bookings view; the back arrow returns her there |
| FEAT-21.SPEC-001 (Set Up Recurring Series) | Navigation (inbound) | A newly created series lands Riley here |
| FEAT-21.SPEC-006 (Series & Occurrence Cancellation Rules) | References (outbound) | Governs the effect and contention outcome of both cancel actions |
| FEAT-21.SPEC-004 (Occurrence Generation & Conflict Handling) | References (inbound) | Supplies each occurrence's Upcoming/Needs-new-time status |
| FEAT-21.SPEC-005 (Occurrence Deposit Request & Release) | References (inbound) | Supplies each occurrence's Awaiting-deposit status |
| FEAT-21.SPEC-010 (Pro Recurring Series Management) | References (sibling) | The Pro-side counterpart showing the same series and occurrences; a Pro cancellation there refreshes this screen per the contention rule |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| recurring_occurrence_cancelled | series reference | Riley cancels a single occurrence from this screen | N/A -- Stage 2 names no distinct signal for cancelling a single occurrence; per this Brief's Signals note, this action reuses the generic booking-cancelled signal the owning cancellation feature already emits for any Booking, rather than a new series-specific signal invented here |
| recurring_series_cancelled | interval_weeks, occurrences_cancelled_count | Riley cancels the whole series from this screen | supports success-metrics.md: "Self-Service Reschedule Rate" |
| recurring_series_list_viewed | series_count | This screen loads with one or more series | N/A -- no Stage 2 metric measures view frequency for this screen; retained so usage of the management surface is observable |

## Acceptance Criteria

**FEAT-21.SPEC-002-AC-01:** Given Riley holds no Recurring Series with Talia, when she opens FEAT-06.SPEC-003 (My Bookings List), then no recurring-series section or extra UI appears at all.

**FEAT-21.SPEC-002-AC-02:** Given Riley holds one series with two upcoming occurrences, when she opens this screen, then she sees one card showing the series' interval and both occurrences listed underneath with their status badges.

**FEAT-21.SPEC-002-AC-03:** Given Riley taps "Cancel this one" on an occurrence, when she confirms "Cancel Appointment" in the dialog, then that occurrence's Booking is cancelled per FEAT-21.SPEC-006, its row is removed, the toast "Appointment cancelled" appears, and the series card remains with its other occurrences.

**FEAT-21.SPEC-002-AC-04:** Given Riley taps "Cancel series," when she confirms "Cancel Series" in the dialog, then the series is ended and every not-yet-occurred occurrence is cancelled per FEAT-21.SPEC-006, the card is removed, and the toast "Series cancelled" appears.

**FEAT-21.SPEC-002-AC-05:** Given Riley opens the "Cancel this one" dialog and taps "Keep It" instead, then the dialog closes and no cancellation occurs.

**FEAT-21.SPEC-002-AC-06:** Given an occurrence's deposit link has been sent and is unpaid, when Riley views this screen, then that occurrence shows the "Awaiting deposit" status badge.

**FEAT-21.SPEC-002-AC-07:** Given an occurrence's usual slot is no longer available per FEAT-21.SPEC-004, when Riley views this screen, then that occurrence shows the "Needs new time" status badge.

**FEAT-21.SPEC-002-AC-08:** Given Talia cancels the same series Riley is viewing at effectively the same moment Riley taps "Cancel series," when both commit, then the first to commit wins, and the other party sees the refreshed, already-cancelled state -- for example, Riley sees "This series was just cancelled." if Talia's action committed first.

**FEAT-21.SPEC-002-AC-09:** Given Riley taps "Cancel series" and the operation fails due to a network error, then an error banner reads "Could not complete that cancellation. Check your connection and try again." and the card remains in its pre-cancellation state.

**FEAT-21.SPEC-002-AC-10:** Given Riley loses connectivity and attempts a cancel action, then she sees "You'll need to be online to do this. Please check your connection and try again." and nothing is submitted.

**FEAT-21.SPEC-002-AC-11:** Given a series has no upcoming occurrences left but remains Active, when Riley views this screen, then the series card shows with the note "Your next appointment will appear here once it's scheduled."

**FEAT-21.SPEC-002-AC-12:** Given an occurrence Riley attempts to cancel was released unpaid moments earlier, when her cancel action is evaluated, then it is refused with "This appointment is no longer active." and the row refreshes to reflect the release.

**FEAT-21.SPEC-002-AC-13:** Given Riley cancels a single occurrence, when she reopens this screen, then the emitted event reuses the generic booking-cancelled signal, and no series-specific single-occurrence signal is recorded.

**FEAT-21.SPEC-002-AC-14:** Given Talia wants to view or manage her own schedule's series, when she looks for this screen, then it is not reachable to her -- her equivalent view is FEAT-21.SPEC-010 (Pro Recurring Series Management).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 3 | 3 |
| States | 5 (empty, loaded, loading, error, cancelling) plus offline N/A | 6 |
| Business Rules | 3 | 3 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Recurring Series Setup & Generation Limits

## Overview

**Name:** Recurring Series Setup & Generation Limits
**ID:** FEAT-21.SPEC-003
**Type:** Logic/Rule
**Purpose:** Defines the valid interval range for a Recurring Series, the booking-horizon ceiling that governs how far ahead occurrences may ever be generated, and who may create a series.
**Parent Feature:** FEAT-21 -- Recurring/Standing Appointments
**Governed Entity:** Recurring Series

## Scope and Non-Goals

**In Scope:**
- The interval field's valid range (every 1 to 12 weeks) and its error message
- The booking-horizon ceiling that bounds how far ahead FEAT-21.SPEC-004 may ever generate an occurrence
- Authorization for creating a Recurring Series
- Default values and derivations for the originating service and time fields

**Non-Goals:**
- Authorization for viewing, cancelling, or managing an existing series -- owned by FEAT-21.SPEC-006 (Series & Occurrence Cancellation Rules) for the cancellation actions, and by FEAT-21.SPEC-002 for the view action's own screen-level access rules.
- The per-occurrence slot validation performed at each generation attempt (duration, buffer, conflicts) -- owned by FEAT-03.SPEC-004 (Slot Validation & Timing Rules); this spec only supplies the horizon ceiling that FEAT-03's own check is bounded by.
- A dedicated pause action for a series -- excluded per this Brief's Non-Goals: no Key Capability, Primary Flow, Alternate, or States-field line describes a pause interaction, so only Active and Ended states are modeled.
- Editing a series' interval after creation -- excluded per this Brief's Non-Goals: the feature's Key Capabilities name only setup, group viewing/management, and cancellation; a client who wants a different cadence cancels and sets up a fresh series through FEAT-21.SPEC-001.

## Governed Entity

**Entity:** Recurring Series
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| interval | number (weeks) | How often the series repeats, in whole weeks |
| originating_service | reference (Service) | The Service the series repeats, captured from the booking the series was created from |
| originating_time | derived (time-of-day, day-of-week) | The time-of-day and day-of-week pattern the series repeats, captured from the originating Booking |
| state | enum (Active, Ended) | The series' lifecycle state |
| generated_occurrences | derived list (Booking references) | The occurrences this series has generated within the booking horizon |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-21.SPEC-001 | Set Up Recurring Series | On submit, when Riley chooses an interval |
| FEAT-21.SPEC-004 | Occurrence Generation & Conflict Handling | On every generation attempt, to bound how far ahead an occurrence may ever be created |
| FEAT-21.SPEC-010 | Pro Recurring Series Management | On submit, when Talia sets up a series for a client at the chair, subject to the same setup validation as FEAT-21.SPEC-001 |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| interval | Required; whole number of weeks, between 1 and 12 inclusive | Always | On submit | "Choose a repeat interval between 1 and 12 weeks." | Yes |
| originating_service | No validation beyond data type -- auto-populated from the originating Booking's Service at setup time; not independently entered | Always | -- | -- | -- |
| originating_time | No validation beyond data type -- auto-populated from the originating Booking's start_time (time-of-day and day-of-week) at setup time; not independently entered | Always | -- | -- | -- |
| state | No validation beyond data type on this spec's side; the Active -> Ended transition is governed by FEAT-21.SPEC-006, not by this spec | Always | -- | -- | -- |
| generated_occurrences | No validation beyond data type on this spec's side; each occurrence's own creation is governed by FEAT-21.SPEC-004 | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Generation horizon ceiling | interval (this spec), Availability Rule.booking_horizon (external, FEAT-02) | An occurrence is never generated further ahead than the Pro's currently configured booking_horizon, regardless of the series' interval; a series with a short interval simply accumulates more generated occurrences within the same horizon window than a series with a long interval | N/A -- this is a generation-time boundary enforced silently by FEAT-21.SPEC-004, not a form-submission error the client or Pro ever sees at setup time |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create Recurring Series (from own booking) | The Client (Riley) | Only from a Booking that is her own, currently active (not cancelled or expired), and has not already originated a series | Create controls (FEAT-21.SPEC-001's offer) are not shown for a booking that is cancelled, expired, or has already originated a series; a direct attempt shows the withdrawn-offer message defined in FEAT-21.SPEC-001's Edge Cases |
| Create Recurring Series (for a client at the chair) | The Pro (Talia) | Always, for any client and booking on her own schedule, through FEAT-21.SPEC-010 (Pro Recurring Series Management), reached from FEAT-30's Pro booking detail | -- |
| Create Recurring Series | Platform Operator (Support) | Never | The create action is not shown anywhere in Support's read-only view (FEAT-19); Support has no path to this action at all |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| originating_service | Set to the Service of the Booking the series is created from | On create only | No -- Riley and Talia both create a series from an existing booking, not by choosing a service independently |
| originating_time | Set to the time-of-day and day-of-week of the Booking the series is created from | On create only | No |
| state | Set to Active | On create only | No |
| generated_occurrences | Starts empty; populated by FEAT-21.SPEC-004 as occurrences are generated | On create (empty), then continuously by FEAT-21.SPEC-004 | No -- occurrences are system-generated, never manually added |

## Business Rules

- The interval range (1-12 weeks) and the booking-horizon ceiling are referenced, not duplicated, by FEAT-21.SPEC-001 at setup time and by FEAT-21.SPEC-004 at every later generation attempt, per this Brief's Shared Validation section.
- XBR-03: minimum booking notice and booking horizon limit every client-facing booking path, including occurrence generation; the Pro alone (through FEAT-21.SPEC-010, reached from FEAT-30) may set up or manage a series outside the ordinary notice window, consistent with her general booking-in exception.
- The booking-horizon ceiling is the Pro's currently configured Availability Rule.booking_horizon (FEAT-02) at the moment each generation attempt runs -- it is never a value this spec fixes independently, so a Pro who later widens or narrows her horizon immediately changes how far ahead future generation attempts may reach, without any action on the series itself.
- A series' interval, once set at creation, cannot be changed by any action this spec authorizes; the only way to change cadence is to cancel and create a fresh series (FEAT-21.SPEC-001).

## Edge Cases

- **Interval entered as exactly 1 week** -- Passes validation; the series generates as frequently as the booking horizon allows.
- **Interval entered as exactly 12 weeks** -- Passes validation, the upper boundary.
- **Interval entered as 0 or a negative number** -- Rejected with the standard error message; 0 and negative values are both out of range, not separately messaged.
- **Interval entered as a non-whole number (e.g., 2.5)** -- Rejected with the standard error message; only whole numbers of weeks are valid.
- **The Pro narrows her booking_horizon after a series already has generated occurrences** -- Already-generated occurrences are untouched (XBR-11: setup changes never silently cancel a confirmed booking); FEAT-21.SPEC-004 simply generates no further occurrences until the horizon rolls forward enough to reach the series' next due date again.
- **The Pro widens her booking_horizon while a series is Active** -- FEAT-21.SPEC-004 becomes able to generate further-ahead occurrences on its next run, up to the new, wider ceiling; no action on the series itself is required.
- **Riley attempts to create a second series from the same booking after the first has already been created and later cancelled** -- Denied per the Authorization Rules condition: a booking that has already originated a series (regardless of that series' current state) cannot originate a second one; Riley must build a new series from a different, later booking instead.

## Acceptance Criteria

**FEAT-21.SPEC-003-AC-01:** Given Riley chooses an interval of 3 weeks, when validation runs, then it passes.

**FEAT-21.SPEC-003-AC-02:** Given Riley chooses an interval of 1 week, when validation runs, then it passes.

**FEAT-21.SPEC-003-AC-03:** Given Riley chooses an interval of 12 weeks, when validation runs, then it passes.

**FEAT-21.SPEC-003-AC-04:** Given Riley chooses an interval of 0 weeks, when validation runs, then she sees "Choose a repeat interval between 1 and 12 weeks." and setup is blocked.

**FEAT-21.SPEC-003-AC-05:** Given Riley chooses an interval of 13 weeks, when validation runs, then she sees the same error message and setup is blocked.

**FEAT-21.SPEC-003-AC-06:** Given Riley chooses an interval of 2.5 weeks, when validation runs, then she sees the same error message and setup is blocked.

**FEAT-21.SPEC-003-AC-07:** Given a series is created, when its originating_service and originating_time are set, then they exactly match the Service and start_time of the Booking it was created from, with no independent input from Riley.

**FEAT-21.SPEC-003-AC-08:** Given Riley's booking is her own, active, and has not already originated a series, when she opens FEAT-21.SPEC-001, then the create offer is shown.

**FEAT-21.SPEC-003-AC-09:** Given Riley's booking has already originated a series, when she navigates back to FEAT-21.SPEC-001 for that same booking, then the create offer is not shown again, per FEAT-21.SPEC-001's own edge case.

**FEAT-21.SPEC-003-AC-10:** Given Talia (the Pro) sets up a series for a client at the chair through FEAT-21.SPEC-010, when she submits an interval within 1-12 weeks, then the same validation passes and the series is created.

**FEAT-21.SPEC-003-AC-11:** Given Platform Operator (Support) views a Pro's account, when they look for a create-series action, then none is shown anywhere in their read-only view.

**FEAT-21.SPEC-003-AC-12:** Given a series' next due occurrence falls further ahead than the Pro's currently configured booking_horizon, when FEAT-21.SPEC-004 evaluates generation, then no occurrence is generated until the horizon rolls forward far enough to reach it.

**FEAT-21.SPEC-003-AC-13:** Given the Pro narrows her booking_horizon after a series already has generated occurrences, when the change takes effect, then the already-generated occurrences are untouched and only future generation attempts are affected.

**FEAT-21.SPEC-003-AC-14:** Given the Pro widens her booking_horizon while a series is Active, when FEAT-21.SPEC-004 next runs, then it can generate occurrences up to the new, wider ceiling.

**FEAT-21.SPEC-003-AC-15:** Given a booking has already originated one series, when a second attempt is made to create a series from that same booking, then it is denied per the Authorization Rules condition.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 3 | 3 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |



# Automation Spec: Occurrence Generation & Conflict Handling

## Overview

**Name:** Occurrence Generation & Conflict Handling
**ID:** FEAT-21.SPEC-004
**Type:** Automation
**Purpose:** Generates each occurrence's Booking within the Pro's booking horizon as an Active series' due date arrives, subject to the same slot validation as any booking, and hands an occurrence whose usual time is no longer available to a client pick-a-new-time flow without breaking the rest of the series.
**Parent Feature:** FEAT-21 -- Recurring/Standing Appointments

## Scope and Non-Goals

**In Scope:**
- Generating the next occurrence's Booking for every Active series as the booking horizon rolls forward to reach its due date
- Re-validating each candidate occurrence against the same slot check as any booking (FEAT-03.SPEC-004, XBR-01)
- Handing an occurrence whose usual time is no longer available to a client pick-a-new-time flow for that occurrence only
- Retrying a failed generation attempt and flagging a persistently failing one to the Pro

**Non-Goals:**
- Deciding whether a candidate interval or horizon is valid in the first place -- owned by FEAT-21.SPEC-003 (Recurring Series Setup & Generation Limits); this spec applies that ceiling, it does not define it.
- Requesting or releasing an occurrence's deposit -- owned by FEAT-21.SPEC-005 (Occurrence Deposit Request & Release), which begins once this spec has created the occurrence's Booking.
- Ending a series or cancelling an occurrence -- owned by FEAT-21.SPEC-006 (Series & Occurrence Cancellation Rules); this spec only ever creates or time-shifts an occurrence, it never cancels one.
- The actual reservation and slot-hold mechanics themselves -- owned by FEAT-03 (Real-Time Slot Availability Engine); this spec triggers FEAT-03's ordinary validation and relies on the created Booking's own state to occupy the slot, per the mutual FEAT-03/FEAT-21 dependency this Brief's dependency map records.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A Recurring Series is newly created | FEAT-21.SPEC-001 (Set Up Recurring Series) / FEAT-21.SPEC-010 (Pro Recurring Series Management) | Fires once, immediately after series creation, to generate the series' first occurrence(s) within the current booking horizon | Recurring Series (interval, originating_service, originating_time), Availability Rule (booking_horizon) |
| An Active series' booking horizon rolls forward | Schedule-based (daily evaluation against each Active series' next due date and the Pro's current booking_horizon) | Fires whenever a series' next due occurrence date now falls within the Pro's currently configured booking_horizon and has not yet been generated | Recurring Series (interval, originating_service, originating_time, generated_occurrences), Availability Rule (booking_horizon) |
| The client picks a new time for an occurrence flagged as needing one | FEAT-21.SPEC-008 (Occurrence Time Change Advance Notice) | Fires when the client submits a replacement time for an occurrence this spec previously flagged as conflicted | The flagged occurrence's Booking reference, the client's chosen replacement time |

## Processing Logic

1. On the initial-creation trigger, or on each scheduled horizon-rollforward run, identify every Active Recurring Series whose next due occurrence date (the last generated occurrence's date plus the series' interval, or the originating Booking's date plus the interval if none has been generated yet) now falls within the Pro's currently configured booking_horizon (FEAT-21.SPEC-003).
2. For each such series, construct the candidate occurrence: the series' originating_service, and a start time on the due date at the series' originating time-of-day.
3. Confirm the originating_service is still Active (not Archived). If it has been archived, take the Service-archived path (Step 9) instead of continuing.
4. Re-validate the candidate slot against FEAT-03.SPEC-004's rules exactly as any other booking (full duration plus buffer inside an open window, no conflicting booking, block, other recurring occurrence, or calendar busy time), applying no Pro-only exception -- occurrence generation follows the ordinary client-facing notice and horizon rules (XBR-01, XBR-03), since it stands in for a booking the client would otherwise have made themselves.
5. If the candidate slot passes validation, create a Booking in Pending Payment state: service, start_time, duration, and price agreed from the current Service definition at generation time; client set to the series' client; source set to "recurring occurrence"; a reference to the originating Recurring Series.
6. Add the new Booking to the series' generated_occurrences list.
7. Trigger FEAT-21.SPEC-007 (Occurrence Generated Notification) for the newly created Booking.
8. Hand off the Booking to FEAT-21.SPEC-005 (Occurrence Deposit Request & Release) to begin its deposit-lifecycle monitoring.
9. **Service-archived path:** if the originating_service is Archived, generate no occurrence for this due date and take no further generation action for this series until the Pro either restores an equivalent active service or the client sets up a fresh series against a currently active service (FEAT-21.SPEC-001); flag the gap to the Pro on her dashboard (FEAT-12), since a standing client is due and nothing was booked for them.
10. **Conflict path:** if the candidate slot fails validation (the Pro changed her hours, added a block, or another commitment now occupies the time), still create the occurrence's Booking in Pending Payment state as in Step 5, but leave its start_time unset pending a replacement, and mark it as needing a new time.
11. Trigger FEAT-21.SPEC-008 (Occurrence Time Change Advance Notice) for the conflicted occurrence, asking the client to pick a new time for that occurrence only.
12. **Replacement-time path:** when the client submits a replacement time (via the flow FEAT-21.SPEC-008 links to), re-validate that specific candidate time against FEAT-03.SPEC-004 exactly as in Step 4. If it passes, set the occurrence Booking's start_time to the chosen time and proceed as a normally generated occurrence (Steps 6-8). If it fails, the client is shown the refreshed unavailable-time experience and asked to choose again -- the rest of the series is never affected by how many attempts this takes.
13. **Failure path:** if the generation attempt itself cannot complete (a processing error unrelated to slot validation), retry up to platform parameter: `occurrence-generation-retry-count` times. If it keeps failing after those retries, flag the gap on the Pro's dashboard (FEAT-12) as a missed occurrence needing her attention, rather than silently skipping the appointment.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Occurrence generated normally | Candidate slot passes validation | New Booking created (Pending Payment, source "recurring occurrence"); added to the series' generated_occurrences | Client receives the generated notification (FEAT-21.SPEC-007); occurrence appears Upcoming on FEAT-21.SPEC-002 | FEAT-21.SPEC-002, FEAT-21.SPEC-005, FEAT-21.SPEC-007 |
| Occurrence needs a new time | Candidate slot fails validation (usual time no longer available) | New Booking created (Pending Payment, start_time unset, flagged needing new time) | Client receives the advance-notice notification (FEAT-21.SPEC-008) and is asked to pick a new time; occurrence appears "Needs new time" on FEAT-21.SPEC-002 | FEAT-21.SPEC-002, FEAT-21.SPEC-008 |
| Replacement time accepted | Client's chosen replacement time passes validation | Booking's start_time set; occurrence proceeds as normally generated | Client receives the generated notification (FEAT-21.SPEC-007) for the now-scheduled occurrence | FEAT-21.SPEC-002, FEAT-21.SPEC-005, FEAT-21.SPEC-007 |
| Replacement time rejected | Client's chosen replacement time also fails validation | None | Client sees the refreshed unavailable-time experience and is asked to choose again | FEAT-21.SPEC-008 |
| Generation skipped -- horizon not yet reached | Series' next due date is still beyond the Pro's current booking_horizon | None | None -- this is silent, expected behavior each time the automation runs | -- |
| Generation skipped -- originating service archived | The series' originating_service is Archived at generation time | None (no Booking created for this due date) | No client-facing feedback; the Pro sees the gap flagged on her dashboard (FEAT-12) | FEAT-12 |
| Generation failure (after retries exhausted) | The generation attempt fails for reasons unrelated to slot validation, and retries are exhausted | None | No client-facing feedback; the Pro sees the gap flagged on her dashboard (FEAT-12) as a missed occurrence | FEAT-12 |

## Data Model

**Reads:** Recurring Series -- interval, originating_service, originating_time, state, generated_occurrences; Availability Rule -- booking_horizon (via FEAT-03); Service -- status, price, duration (at generation time); existing Slot Hold, Booking, Time Block, and Calendar Connection busy-time records (via FEAT-03.SPEC-004/FEAT-03.SPEC-006, for slot validation).
**Creates:** Booking -- service, start_time (or unset, if conflicted), duration, price_agreed, deposit_amount, client, source ("recurring occurrence"), Recurring Series reference, state Pending Payment.
**Updates:** Recurring Series -- generated_occurrences list (appended). Booking -- start_time (on replacement-time acceptance).
**Deletes:** None.

## Business Rules

- XBR-01: every generated occurrence is subject to the same live slot check as any other booking; the first commitment to a contested time wins, exactly as it would for a client-facing booking.
- XBR-02: a generated occurrence's slot is treated the same as any Pending Payment Booking for availability purposes -- it is never a separate reservation type, per the mutual FEAT-03/FEAT-21 dependency.
- XBR-03: occurrence generation follows the ordinary client-facing minimum-notice and booking-horizon rules; unlike a Pro-created deposit-request booking (FEAT-03.SPEC-007), no Pro-only exception applies here, since generation stands in for what the client would otherwise book herself.
- The booking-horizon ceiling this spec applies is always the Pro's currently configured Availability Rule.booking_horizon at the moment each generation attempt runs (FEAT-21.SPEC-003) -- never a value cached from series creation time.
- A conflicted occurrence's replacement-time flow never affects the series' other occurrences, its interval, or its state -- only that one occurrence's start_time is at stake (this Brief's Entity-Lifecycle Coverage Matrix, Update row).
- A repeatedly failing occurrence generation is retried (platform parameter: `occurrence-generation-retry-count`) and, if it keeps failing, surfaces to the Pro as a flagged gap on her dashboard rather than a silently missed appointment.

## Edge Cases

- **A series' due date arrives on a day the Pro has fully blocked with a Time Block** -- The candidate slot fails validation for the same reason any client-facing booking would; the occurrence takes the conflict path and the client is asked to pick a new time.
- **The originating service's price or duration has changed since the series was created** -- The generated occurrence uses the Service's current price and duration at generation time (Step 5), not the price agreed at the original booking, since each occurrence is a fresh booking in its own right, consistent with XBR-04 applying prospectively to each new booking rather than retroactively to a past one.
- **The originating service is archived and later a new, similarly named service is added** -- Generation for the existing series does not resume automatically against the new service, since the series' originating_service reference points to the specific archived Service, not a name; the client is left without occurrences until she sets up a fresh series (FEAT-21.SPEC-001) against the new service.
- **Two occurrences from different series happen to compete for the same slot on the same generation run** -- The first one processed in the run commits the slot; the second fails validation and takes the conflict path, per the same first-commit-wins rule that governs any two competing bookings (XBR-01).
- **Concurrent trigger firing (the initial-creation trigger and a scheduled horizon-rollforward run overlap for the same series)** -- A second generation attempt for a series already holding a due, ungenerated occurrence is a no-op: the series' generated_occurrences list is checked before creating a new Booking, preventing a duplicate occurrence for the same due date.
- **Trigger fires while a previous generation run is still in flight for the same series** -- Generation for a given series processes its due dates sequentially; a new trigger for that series queues behind the in-flight run rather than running concurrently against the same generated_occurrences list.
- **A client abandons the replacement-time flow without ever picking a new time** -- The occurrence's Booking remains in Pending Payment with start_time unset and status "Needs new time" indefinitely on FEAT-21.SPEC-002; it does not block the series from generating its next due occurrence, since the series' due-date calculation advances from the last successfully time-set occurrence, not from every attempted one -- this spec never auto-cancels an abandoned replacement flow, leaving that as the client's own choice via FEAT-21.SPEC-002's cancel action.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-21.SPEC-001 (Set Up Recurring Series) | Triggered by (inbound) | Series creation fires the first generation run |
| FEAT-21.SPEC-010 (Pro Recurring Series Management) | Triggered by (inbound) / Affects (outbound) | A Pro's series set-up fires the first generation run; generated and conflicted occurrences appear there |
| FEAT-21.SPEC-003 (Recurring Series Setup & Generation Limits) | References (outbound) | Supplies the booking-horizon ceiling this spec applies at every generation attempt |
| FEAT-03.SPEC-004 (Slot Validation & Timing Rules) | References (outbound) | Every candidate occurrence, and every replacement time, is validated against this spec's rules |
| FEAT-21.SPEC-002 (My Recurring Series) | Affects (outbound) | Newly generated and conflicted occurrences appear here |
| FEAT-21.SPEC-005 (Occurrence Deposit Request & Release) | Affects (outbound) | A successfully generated occurrence's Booking is handed off here for deposit-lifecycle monitoring |
| FEAT-21.SPEC-007 (Occurrence Generated Notification) | Triggers (outbound) | A successful generation (initial or after a replacement time is accepted) fires this notification |
| FEAT-21.SPEC-008 (Occurrence Time Change Advance Notice) | Triggers (outbound) / Triggered by (inbound) | A conflicted occurrence triggers this notification; the client's submitted replacement time re-enters this automation |
| FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) | A persistently failing generation, or an archived-service gap, is flagged here |

## Analytics and Success Signals

- **recurring_occurrence_generated** (series reference, generation_outcome: normal / conflicted) -- supports success-metrics.md: "Zero Double-Booking Confidence" (every generated occurrence passes the same live slot check any booking does, so this event's outcome split is direct evidence the engine never double-books a standing appointment)
- **recurring_occurrence_conflict_flagged** (series reference) -- N/A -- no Stage 2 metric measures occurrence conflict frequency directly; retained so how often the Pro's own hours changes disrupt standing clients is observable.
- **recurring_occurrence_replacement_time_set** (series reference, attempts_needed) -- N/A -- no Stage 2 metric measures the replacement-time flow directly; retained to observe how often and how easily a conflicted occurrence resolves.
- **recurring_occurrence_generation_failed** (series reference, reason: service_archived / processing_error) -- N/A -- no Stage 2 metric measures generation failure directly; retained so a silently-missed standing appointment never goes unobserved, consistent with this Brief's Error state commitment.

## Acceptance Criteria

**FEAT-21.SPEC-004-AC-01:** Given Riley's series is newly created with an interval of 3 weeks, when the initial-creation trigger fires, then the first occurrence due within the Pro's current booking_horizon is generated as a Booking in Pending Payment state.

**FEAT-21.SPEC-004-AC-02:** Given an Active series' next due date now falls within the Pro's booking_horizon, when the scheduled horizon-rollforward run evaluates it, then the next occurrence is generated.

**FEAT-21.SPEC-004-AC-03:** Given a series' next due date is still beyond the Pro's booking_horizon, when the scheduled run evaluates it, then no occurrence is generated and nothing is shown to Riley.

**FEAT-21.SPEC-004-AC-04:** Given Talia has added a Time Block over an occurrence's usual due time, when generation is attempted, then the candidate slot fails validation, the occurrence is created flagged "Needs new time," and FEAT-21.SPEC-008 is triggered.

**FEAT-21.SPEC-004-AC-05:** Given a conflicted occurrence flagged "Needs new time," when Riley submits a replacement time that passes validation, then the occurrence's start_time is set, it proceeds as a normally generated occurrence, and FEAT-21.SPEC-007 fires for it.

**FEAT-21.SPEC-004-AC-06:** Given a conflicted occurrence, when Riley submits a replacement time that also fails validation, then she sees the refreshed unavailable-time experience and is asked to choose again, with the rest of her series unaffected.

**FEAT-21.SPEC-004-AC-07:** Given the series' originating service has been archived by the time an occurrence is due, when generation is attempted, then no occurrence is created for that due date and the gap is flagged on Talia's dashboard.

**FEAT-21.SPEC-004-AC-08:** Given an occurrence's originating service's price has changed since the series was created, when the occurrence is generated, then it uses the Service's current price and duration, not the originally agreed ones.

**FEAT-21.SPEC-004-AC-09:** Given a generation attempt fails due to a processing error, when it is retried up to platform parameter: `occurrence-generation-retry-count` times and still fails, then the gap is flagged on Talia's dashboard as a missed occurrence.

**FEAT-21.SPEC-004-AC-10:** Given two different series each have a due occurrence competing for the same slot in the same generation run, when both are processed, then the first one processed commits the slot and the second takes the conflict path.

**FEAT-21.SPEC-004-AC-11:** Given a series already holds a due, ungenerated occurrence, when a second generation trigger fires for that same series before the first completes, then no duplicate occurrence is created.

**FEAT-21.SPEC-004-AC-12:** Given a generation run is already in flight for a series, when another trigger fires for that same series, then the new attempt queues behind the in-flight run rather than running concurrently.

**FEAT-21.SPEC-004-AC-13:** Given Riley abandons a conflicted occurrence's replacement-time flow, when the series' next due date is later evaluated, then the series still generates its next occurrence normally, unaffected by the abandoned one.

**FEAT-21.SPEC-004-AC-14:** Given a Booking is successfully generated for an occurrence, when the generation completes, then FEAT-21.SPEC-005 begins its deposit-lifecycle monitoring for that Booking.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 | 3 |
| Outcome Paths | 7 | 7 |
| Business Rules | 6 | 6 |
| Edge Cases | 7 | 7 |



# Automation Spec: Occurrence Deposit Request & Release

## Overview

**Name:** Occurrence Deposit Request & Release
**ID:** FEAT-21.SPEC-005
**Type:** Automation
**Purpose:** Sends each generated occurrence's own fresh deposit link about a week before it, and releases the occurrence if the deposit is never paid by its cancellation cut-off, without disturbing the rest of the series.
**Parent Feature:** FEAT-21 -- Recurring/Standing Appointments

## Scope and Non-Goals

**In Scope:**
- Sending the deposit-request link for each occurrence's Booking a fixed number of days before its appointment
- Monitoring an occurrence's Pending Payment Booking through to its cancellation cut-off
- Releasing (expiring) an occurrence whose deposit remains unpaid by that cut-off
- Handing off both the request and the release moments to FEAT-21.SPEC-009 for client- and Pro-facing notification

**Non-Goals:**
- Capturing the deposit payment itself, or authorizing the client's card -- owned by FEAT-07 (Deposit Payment at Booking); this spec only decides when to send the link and when to give up waiting for it.
- Composing or delivering the deposit-request or release notification content -- owned by FEAT-21.SPEC-009 (Occurrence Deposit Lifecycle Notification); this spec only triggers it at the right moments.
- Generating the occurrence's Booking in the first place -- owned by FEAT-21.SPEC-004 (Occurrence Generation & Conflict Handling), which hands the Booking to this spec once created.
- Retaining or reusing a client's card across occurrences -- excluded per this Brief's Non-Goals and scope-boundaries.md SC-11/SC-13: each occurrence's deposit is paid fresh through its own link, exactly as any other deposit payment.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| An occurrence's Booking is generated | FEAT-21.SPEC-004 (Occurrence Generation & Conflict Handling) | Fires once, immediately after a normally generated or replacement-time-accepted occurrence's Booking is created, to begin this spec's monitoring | Booking (start_time, deposit_amount, state), Recurring Series reference |
| An occurrence approaches its deposit-request point | Schedule-based (evaluated against each monitored occurrence's start_time) | Fires when the current time reaches the occurrence's start_time minus platform parameter: `recurring-occurrence-deposit-lead-days`, and no deposit-request link has yet been sent for it, and the Booking is still Pending Payment | Booking (service, start_time, deposit_amount) |
| An occurrence reaches its cancellation cut-off unpaid | Schedule-based (evaluated against each monitored occurrence's cancellation cut-off) | Fires when the current time reaches the occurrence's start_time minus the Pro's currently active Cancellation Policy window_hours (FEAT-09, XBR-08), and the Booking is still Pending Payment (deposit never captured) | Booking (service, start_time, state) |

## Processing Logic

1. On receiving a newly generated occurrence's Booking from FEAT-21.SPEC-004, begin monitoring it for its deposit-request point and its cancellation cut-off.
2. When the current time reaches the occurrence's start_time minus platform parameter: `recurring-occurrence-deposit-lead-days`, and the Booking remains Pending Payment with no deposit-request link yet sent, initiate the deposit-request send: hand off to FEAT-07's ordinary deposit-capture mechanism, scoped to this one occurrence's own fresh payment session (XBR-05 -- no card is retained between occurrences).
3. Record that the deposit-request link has been sent for this occurrence, so it is never sent a second time.
4. Trigger FEAT-21.SPEC-009 (Occurrence Deposit Lifecycle Notification), deposit-request variant, carrying the link.
5. Continue monitoring the occurrence. If the client completes the deposit payment at any point before the cancellation cut-off, FEAT-07 transitions the Booking to Confirmed; this spec detects the Confirmed state and stops monitoring that occurrence -- no release action is taken.
6. If the current time reaches the occurrence's cancellation cut-off (start_time minus the Pro's currently active Cancellation Policy window_hours) with the Booking still Pending Payment, release the occurrence: transition its Booking to Expired.
7. The freed time reappears in the availability engine's next slot-list computation, exactly as any other expired, unpaid booking.
8. Trigger FEAT-21.SPEC-009 (Occurrence Deposit Lifecycle Notification), release variant, to both the Client and the Pro.
9. Take no further action on this occurrence -- the series itself is untouched, and its next due occurrence continues to generate normally on its own schedule (FEAT-21.SPEC-004).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Deposit request sent | Current time reaches the deposit-request point and no link has been sent yet | Deposit-request-sent flag recorded on the Booking | Client receives the deposit-request notification (FEAT-21.SPEC-009); occurrence shows "Awaiting deposit" on FEAT-21.SPEC-002 | FEAT-21.SPEC-002, FEAT-21.SPEC-009 |
| Deposit paid before cut-off | Client completes payment via FEAT-07 while the Booking is still Pending Payment | Booking transitioned to Confirmed (by FEAT-07); monitoring for this occurrence stops | Client and Pro see the occurrence Confirmed; no release action occurs | FEAT-07, FEAT-21.SPEC-002 |
| Occurrence released unpaid | Cancellation cut-off is reached with the Booking still Pending Payment | Booking transitioned to Expired | Client and Pro both receive the release notification (FEAT-21.SPEC-009); occurrence disappears from Upcoming on FEAT-21.SPEC-002 | FEAT-21.SPEC-002, FEAT-21.SPEC-009 |
| Deposit-request send failure | The link cannot be sent at the scheduled moment (e.g., a processing error) | None -- the deposit-request-sent flag is not set | No client-facing feedback beyond the ordinary message-delivery retry/fallback FEAT-08 already provides; the occurrence continues toward its cancellation cut-off regardless | FEAT-08 (Automated Booking Messaging) |

## Data Model

**Reads:** Booking -- start_time, state, deposit_amount; Cancellation Policy -- the Pro's currently active version's window_hours (FEAT-09).
**Creates:** None.
**Updates:** Booking -- deposit-request-sent flag (set once, by this spec); state transitioned to Expired on release (this spec is the actor; FEAT-07 is the actor for the Confirmed transition).
**Deletes:** None.

## Business Rules

- The deposit-request lead time is fixed at platform parameter: `recurring-occurrence-deposit-lead-days`, matching this Brief's stated example of about a week before the occurrence.
- The cancellation cut-off this spec monitors is always the Pro's currently active Cancellation Policy window_hours (FEAT-09, XBR-08) measured back from the occurrence's start_time -- the same window that governs any booking's free-cancellation deadline, never a value this spec defines independently.
- XBR-05: each occurrence's deposit is computed once from the Service's rule and captured fresh, with no card retained between occurrences -- this spec never reuses a prior occurrence's payment method or authorization.
- The first committed action wins a race between a completing deposit payment and a reached cancellation cut-off (XBR-01, mirroring FEAT-03.SPEC-007's own race rule): if payment completes before this automation processes the release, the Booking is Confirmed, not Expired.
- Releasing an occurrence never affects its series: the series remains Active and its next due occurrence continues generating on its own schedule (FEAT-21.SPEC-004), per this Brief's Side-Effect Inventory.

## Edge Cases

- **The occurrence's appointment is scheduled sooner than platform parameter: `recurring-occurrence-deposit-lead-days` away at generation time** -- The deposit-request send fires immediately upon generation instead of waiting for the lead-time point, since that point has already passed; the occurrence is still monitored to its cancellation cut-off as usual.
- **The Pro changes her active Cancellation Policy window_hours while an occurrence's deposit is still pending** -- Per XBR-08, an occurrence's own policy_version is set when the client's deposit payment is acknowledged and captured, not at generation time; until that happens, this spec recalculates the cancellation cut-off against the Pro's currently active version each time it evaluates the occurrence, so a mid-flight policy change immediately reshapes an unpaid occurrence's release timing.
- **Deposit payment completes in the same instant the cancellation cut-off elapses** -- The completed-payment transition (Confirmed) takes precedence; this spec detects the already-Confirmed state before processing the release and takes no further action, mirroring FEAT-03.SPEC-007's own race resolution.
- **An occurrence is cancelled by the client or the Pro (FEAT-21.SPEC-006) before either the deposit-request point or the cancellation cut-off is reached** -- Monitoring stops immediately; no deposit-request link is sent and no release notification fires for a cancelled occurrence.
- **Concurrent trigger firing (two occurrences from different series reach their deposit-request point at effectively the same time)** -- Each occurrence's monitoring runs independently against its own start_time and deposit-request-sent flag; neither affects the other.
- **Trigger fires while a previous evaluation is still in flight for the same occurrence** -- The deposit-request-sent flag guards against a duplicate send even if two evaluations for the same occurrence overlap; the release action likewise checks the Booking's current state immediately before transitioning it, preventing a double-release.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-21.SPEC-004 (Occurrence Generation & Conflict Handling) | Triggered by (inbound) | Hands off each newly generated occurrence's Booking for deposit-lifecycle monitoring |
| FEAT-09 (Cancellation & No-Show Policy Engine) | References (outbound) | Supplies the currently active window_hours used to compute each occurrence's cancellation cut-off |
| FEAT-07 (Deposit Payment at Booking) | Triggers (outbound) / Triggered by (inbound, on payment completion) | Performs the actual deposit-request send and payment capture; its completion stops this spec's monitoring for that occurrence |
| FEAT-21.SPEC-009 (Occurrence Deposit Lifecycle Notification) | Triggers (outbound) | Both the deposit-request send and the release fire this notification, in its two variants |
| FEAT-21.SPEC-002 (My Recurring Series) | Affects (outbound) | An occurrence's "Awaiting deposit" status, and its removal on release, are reflected here |
| FEAT-21.SPEC-010 (Pro Recurring Series Management) | Affects (outbound) | An occurrence's "Awaiting deposit" status, and its removal on release, are reflected on the Pro's screen too |
| FEAT-21.SPEC-006 (Series & Occurrence Cancellation Rules) | References (inbound) | A cancelled occurrence stops this spec's monitoring immediately |

## Analytics and Success Signals

- **recurring_occurrence_deposit_requested** (series reference) -- supports success-metrics.md: "Deposit Capture Rate" (each occurrence's deposit request is an ordinary deposit-collection attempt this metric already tracks end to end)
- **recurring_occurrence_deposit_released** (series reference, days_unpaid) -- supports success-metrics.md: "Automatic Refund Correctness" (a release with no payment ever captured has nothing to refund, but the event is the direct evidence that an unpaid occurrence never silently lingers as a booking neither party can act on)
- **recurring_occurrence_deposit_request_send_failed** (series reference, reason category) -- N/A -- no Stage 2 metric measures deposit-request send failures directly for this feature; retained so a silently-unrequested occurrence deposit is observable rather than invisible.

## Acceptance Criteria

**FEAT-21.SPEC-005-AC-01:** Given an occurrence's Booking is generated 10 weeks out, when the current time reaches platform parameter: `recurring-occurrence-deposit-lead-days` before its start_time, then the deposit-request link is sent and FEAT-21.SPEC-009's request variant fires.

**FEAT-21.SPEC-005-AC-02:** Given an occurrence's appointment is sooner than platform parameter: `recurring-occurrence-deposit-lead-days` away at generation time, when the occurrence is generated, then the deposit-request send fires immediately.

**FEAT-21.SPEC-005-AC-03:** Given Riley receives an occurrence's deposit-request link, when she completes payment before the cancellation cut-off, then the Booking is transitioned to Confirmed and no release action ever occurs for that occurrence.

**FEAT-21.SPEC-005-AC-04:** Given an occurrence's Booking remains Pending Payment when the current time reaches its cancellation cut-off (start_time minus the Pro's active window_hours), when the release evaluation runs, then the Booking is transitioned to Expired and FEAT-21.SPEC-009's release variant fires to both Riley and Talia.

**FEAT-21.SPEC-005-AC-05:** Given an occurrence is released unpaid, when its series is next evaluated, then the series remains Active and its next due occurrence still generates on schedule.

**FEAT-21.SPEC-005-AC-06:** Given Talia changes her active Cancellation Policy window_hours while an occurrence's deposit is still pending, when this spec next evaluates that occurrence, then the cancellation cut-off is recalculated against the currently active window_hours.

**FEAT-21.SPEC-005-AC-07:** Given an occurrence's deposit payment completes at the same instant its cancellation cut-off is reached, when both are evaluated, then the completed payment takes precedence and the Booking is Confirmed, not Expired.

**FEAT-21.SPEC-005-AC-08:** Given an occurrence is cancelled before its deposit-request point is reached, when the cancellation completes, then no deposit-request link is ever sent for it.

**FEAT-21.SPEC-005-AC-09:** Given an occurrence is cancelled after its deposit-request link was already sent but before the cancellation cut-off, when the cancellation completes, then monitoring stops and no release notification fires.

**FEAT-21.SPEC-005-AC-10:** Given the deposit-request send fails due to a processing error, when the failure occurs, then no client-facing error appears from this spec, and the occurrence continues to be monitored toward its cancellation cut-off exactly as if the send had succeeded.

**FEAT-21.SPEC-005-AC-11:** Given two occurrences from different series each reach their deposit-request point at effectively the same time, when both are evaluated, then each is processed independently with no interference.

**FEAT-21.SPEC-005-AC-12:** Given an occurrence's deposit-request-sent flag is already set, when a second evaluation for that same occurrence runs before the first completes, then no duplicate deposit-request link is sent.

**FEAT-21.SPEC-005-AC-13:** Given a released occurrence's slot becomes free, when the availability engine next computes its slot list, then the freed time appears as open.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 | 3 |
| Outcome Paths | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Series & Occurrence Cancellation Rules

## Overview

**Name:** Series & Occurrence Cancellation Rules
**ID:** FEAT-21.SPEC-006
**Type:** Logic/Rule
**Purpose:** Governs what cancelling the whole series does to its not-yet-occurred occurrences versus cancelling a single occurrence, and how a concurrent Client/Pro change to the same series or occurrence resolves.
**Parent Feature:** FEAT-21 -- Recurring/Standing Appointments
**Governed Entity:** Recurring Series (and its generated occurrence Bookings)

## Scope and Non-Goals

**In Scope:**
- The state transition of a Recurring Series from Active to Ended on whole-series cancellation
- The cascade effect of whole-series cancellation on its not-yet-occurred generated occurrences
- The effect of cancelling a single occurrence, independent of its series
- Authorization for both cancel actions, per role
- The contention rule when the Client and the Pro act on the same series or occurrence at the same time

**Non-Goals:**
- The deposit refund-or-forfeit outcome of a cancelled occurrence -- owned by FEAT-09 (Cancellation & No-Show Policy Engine, XBR-09); this spec triggers that ordinary policy per occurrence, it does not define a separate rule for standing appointments.
- The screen-level presentation of the two cancel actions -- owned by FEAT-21.SPEC-002 (My Recurring Series) for the Client and FEAT-21.SPEC-010 (Pro Recurring Series Management) for the Pro, which enforce these rules but do not duplicate them.
- Interval and generation-limit validation -- owned by FEAT-21.SPEC-003; this spec begins only once a series already exists and is being cancelled, not created.
- A dedicated pause action as an alternative to cancellation -- excluded per this Brief's Non-Goals: no Key Capability, Primary Flow, Alternate, or States-field line describes a pause interaction, so this spec models only the Active-to-Ended transition, never a reversible pause.

## Governed Entity

**Entity:** Recurring Series (and its generated occurrence Bookings)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| state | enum (Active, Ended) | The series' lifecycle state; this spec owns the Active -> Ended transition |
| generated_occurrences | derived list (Booking references) | The occurrences cascaded by a whole-series cancellation |
| Booking.state (occurrence) | enum | The individual occurrence's state; this spec owns its transition to Cancelled by Client / Cancelled by Pro |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-21.SPEC-002 | My Recurring Series | On the "Cancel series" and "Cancel this one" actions, and on their confirmation dialogs |
| FEAT-21.SPEC-010 | Pro Recurring Series Management | On the Pro's "End series" and "Cancel this one" actions, and on their confirmation dialogs, for series tied to her own schedule |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| state | Must transition only Active -> Ended; no other transition is defined | On whole-series cancel | On cancel action | "This series has already been cancelled." (shown only if a second cancel attempt reaches an already-Ended series) | Yes |
| Booking.state (occurrence) | Must be a not-yet-occurred, not-already-cancelled occurrence to be eligible for cancellation | On occurrence cancel or cascade | On cancel action | "This appointment is no longer active." | Yes |
| generated_occurrences | No validation beyond data type -- this spec only reads generated_occurrences to drive the cascade defined under Cross-Field Rules | -- | -- | -- | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Whole-series cascade | Recurring Series.state, Booking.state (each generated occurrence) | When the series transitions to Ended, every generated occurrence whose Booking has not yet occurred and is not already Cancelled or Completed is itself transitioned to Cancelled | N/A -- the cascade is automatic and silent to the acting party beyond the confirmation dialog already shown on FEAT-21.SPEC-002 |
| Already-completed occurrences untouched | Booking.state (each generated occurrence) | A whole-series cancellation never changes an occurrence whose Booking is already Completed or No-Show | N/A -- these occurrences are simply excluded from the cascade |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Cancel one occurrence | The Client (Riley) | Only her own series' occurrence, and only while that occurrence has not yet occurred and is not already Cancelled | The "Cancel this one" action is not shown for an occurrence that has already occurred, is already Cancelled, or belongs to a series that is not her own; a direct attempt shows "This appointment is no longer active." |
| Cancel one occurrence | The Pro (Talia) | Any occurrence in a series tied to her own schedule, not yet occurred and not already Cancelled, through FEAT-21.SPEC-010 (Pro Recurring Series Management) | -- |
| Cancel one occurrence | Platform Operator (Support) | Never | The action is not shown anywhere in Support's read-only view (FEAT-19) |
| Cancel whole series | The Client (Riley) | Only her own series, while it is Active | The "Cancel series" action is not shown for a series that is already Ended; a direct attempt shows "This series has already been cancelled." |
| Cancel whole series | The Pro (Talia) | Any series tied to her own schedule, while it is Active, through FEAT-21.SPEC-010 (Pro Recurring Series Management) | -- |
| Cancel whole series | Platform Operator (Support) | Never | The action is not shown anywhere in Support's read-only view (FEAT-19) |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Series.state | Set to Ended | On whole-series cancel | No -- once cancelled, a series is never reactivated; a client who wants standing appointments again sets up a fresh series (FEAT-21.SPEC-001) |
| Booking.state (each cascaded occurrence) | Set to Cancelled by Client or Cancelled by Pro, matching whichever party's action triggered the whole-series cancellation | On whole-series cancel, for every not-yet-occurred, not-already-Cancelled occurrence | No |
| Booking.state (single-occurrence cancel) | Set to Cancelled by Client or Cancelled by Pro, matching the acting party | On single-occurrence cancel | No |

## Business Rules

- Whole-series cancellation is soft: the series transitions from Active to Ended, with no hard delete and no restore path (Entity-Lifecycle Coverage Matrix). A client who wants standing appointments again sets up a fresh series through FEAT-21.SPEC-001.
- Every cascaded or individually cancelled occurrence's deposit outcome follows FEAT-09's ordinary cancellation policy (XBR-09) exactly as any other booking's cancellation -- refunded outside the Pro's cancellation window, kept inside it or on a no-show; this spec never invents a separate, standing-appointment-specific deposit rule (scope-boundaries.md SC-18).
- Cancelling a single occurrence never changes the series' interval or state -- the series continues generating its future occurrences normally (this Brief's Side-Effect Inventory).
- Contention rule (dependency map, Recurring Series entity): when the Client and the Pro act on the same series or the same occurrence at effectively the same time, the resolution is reject-with-refresh -- the first committed change wins, and the other party's view refreshes to show the already-changed state before their own action completes, rather than either action being silently dropped or merged.
- An Ended series is kept indefinitely as history, consistent with scope-boundaries.md SC-22's retention of booking and financial history.

## Edge Cases

- **The Client cancels the whole series while an occurrence is mid-deposit-payment (client is on FEAT-07's payment screen for that occurrence)** -- Per the reject-with-refresh contention rule, whichever action commits first wins: if the whole-series cancellation commits first, the in-progress payment is refused with refresh and the client sees the booking is no longer active; if the payment captures first, that occurrence is Confirmed and is still cascaded to Cancelled by the series cancellation, with its deposit then following FEAT-09's ordinary refund rule for a Confirmed booking.
- **The Pro cancels the same series the Client is viewing on FEAT-21.SPEC-002 at the same time the Client taps "Cancel series"** -- The first commit wins; the other party's screen refreshes to the already-Ended state, per FEAT-21.SPEC-002's own edge case.
- **An occurrence reaches its unpaid cancellation cut-off (FEAT-21.SPEC-005) at the same moment the Client cancels it manually** -- Both actions arrive at the same outcome (the Booking becomes inactive); whichever transition commits first stands, and the other is a no-op against an already-inactive Booking -- no error is shown to the Client in either order, since the end state is the one she intended.
- **Cancelling the whole series when one occurrence is already Completed and another is Confirmed but not yet occurred** -- The Completed occurrence is untouched; the Confirmed, not-yet-occurred occurrence is cascaded to Cancelled and its deposit follows FEAT-09's ordinary rule for a Confirmed booking's cancellation.
- **A second whole-series cancellation attempt reaches an already-Ended series (e.g., a stale screen retried after the first cancellation already committed)** -- Rejected with "This series has already been cancelled." and no further cascade is attempted.
- **The Pro cancels one occurrence tied to her own schedule while the Client simultaneously attempts to cancel the same occurrence** -- The first commit wins, per the same reject-with-refresh contention rule; the other party sees the occurrence already cancelled (on FEAT-21.SPEC-002 for the Client, on FEAT-21.SPEC-010 for the Pro).

## Acceptance Criteria

**FEAT-21.SPEC-006-AC-01:** Given Riley cancels the whole series, when the cancellation commits, then the series transitions to Ended and every not-yet-occurred, not-already-Cancelled occurrence is cascaded to Cancelled by Client.

**FEAT-21.SPEC-006-AC-02:** Given a series being whole-cancelled has one Completed occurrence and one upcoming occurrence, when the cascade runs, then the Completed occurrence is untouched and only the upcoming one is cancelled.

**FEAT-21.SPEC-006-AC-03:** Given Riley cancels a single occurrence, when the cancellation commits, then only that occurrence's Booking is cancelled and the series' state and interval are unchanged.

**FEAT-21.SPEC-006-AC-04:** Given Talia cancels a series tied to her own schedule through FEAT-21.SPEC-010, when the cancellation commits, then the same Active-to-Ended transition and cascade occur, attributed to Cancelled by Pro.

**FEAT-21.SPEC-006-AC-05:** Given Platform Operator (Support) views a series, when they look for a cancel action, then none is shown anywhere in their read-only view.

**FEAT-21.SPEC-006-AC-06:** Given a cancelled occurrence's deposit was never captured, when FEAT-09's ordinary policy evaluates it, then no refund is due, since there is nothing to refund.

**FEAT-21.SPEC-006-AC-07:** Given a cancelled occurrence's deposit was already captured and the cancellation falls outside the Pro's cancellation window, when FEAT-09's ordinary policy evaluates it, then the client receives a full refund.

**FEAT-21.SPEC-006-AC-08:** Given a cancelled occurrence's deposit was already captured and the cancellation falls inside the Pro's cancellation window, when FEAT-09's ordinary policy evaluates it, then the deposit is kept.

**FEAT-21.SPEC-006-AC-09:** Given the Pro cancels the same series Riley is viewing at the same moment Riley taps "Cancel series," when both commit, then the first to commit wins and the other party's view refreshes to the already-Ended state.

**FEAT-21.SPEC-006-AC-10:** Given Riley cancels the whole series while an occurrence's deposit payment is in progress, when the series cancellation commits first, then the in-progress payment is refused with refresh.

**FEAT-21.SPEC-006-AC-11:** Given the same occurrence in-progress payment instead captures before the series cancellation commits, when the series cancellation then runs, then that now-Confirmed occurrence is still cascaded to Cancelled, and its deposit follows FEAT-09's Confirmed-booking refund rule.

**FEAT-21.SPEC-006-AC-12:** Given an occurrence reaches its unpaid cancellation cut-off (FEAT-21.SPEC-005) at the same moment Riley cancels it manually, when both transitions are evaluated, then whichever commits first stands and the other is a no-op, with no error shown to Riley.

**FEAT-21.SPEC-006-AC-13:** Given a series is already Ended, when a second whole-series cancellation attempt is made against it, then it is rejected with "This series has already been cancelled." and no further cascade runs.

**FEAT-21.SPEC-006-AC-14:** Given Talia cancels one occurrence tied to her own schedule at the same moment Riley attempts to cancel that same occurrence, when both commit, then the first to commit wins and the other sees the occurrence already cancelled.

**FEAT-21.SPEC-006-AC-15:** Given Riley cancels the whole series, when she reopens FEAT-21.SPEC-002 afterward, then the series and its occurrences no longer appear, consistent with the Ended state carrying no further active occurrences.

**FEAT-21.SPEC-006-AC-16:** Given Riley attempts to cancel an occurrence that has already occurred, when she looks for the "Cancel this one" action, then it is not shown for that occurrence.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 3 | 3 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 6 | 6 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Notification Spec: Occurrence Generated Notification

## Overview

**Name:** Occurrence Generated Notification
**ID:** FEAT-21.SPEC-007
**Type:** Notification
**Purpose:** Confirms to the client, each time a standing appointment's next occurrence is generated, exactly which appointment has just been scheduled from her series.
**Parent Feature:** FEAT-21 -- Recurring/Standing Appointments

## Scope and Non-Goals

**In Scope:**
- The confirmation sent when a new occurrence is successfully generated, on both of the product's client channels (text and email)
- The variant sent when a previously conflicted occurrence's replacement time is accepted and the occurrence proceeds
- Delivery, retry, and expiry behavior for this notification

**Non-Goals:**
- Notifying the client that an occurrence needs a new time in the first place -- owned by FEAT-21.SPEC-008 (Occurrence Time Change Advance Notice); this spec covers only a successfully scheduled occurrence.
- The deposit-request link and its own notice -- owned by FEAT-21.SPEC-009 (Occurrence Deposit Lifecycle Notification); this is a separate communication sent later, closer to the appointment.
- Deciding text vs. email for this send -- owned by FEAT-14.SPEC-007 (Textability Determination Rule), which this spec defers to before every send, per XBR-15.
- Notifying the Pro that an occurrence was generated -- product-features.md's Communications field for this feature names only a client-facing confirmation; the Pro sees generated occurrences on her own schedule view and on FEAT-21.SPEC-010 (Pro Recurring Series Management) without a separate push notification.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Text | The client has active Messaging Consent for texting (FEAT-14.SPEC-007) | This Brief's Communications field names this as an ordinary confirmation message, matching the product's default client channel for booking-related confirmations |
| Email | The client has not granted texting consent (FEAT-14.SPEC-007) | Every client who declines texting still supplies an email at booking, so the confirmation is never simply undeliverable |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| An occurrence's Booking is generated normally | FEAT-21.SPEC-004 (Occurrence Generation & Conflict Handling) | Fires once per occurrence, immediately when generation succeeds without a slot conflict | Booking (service, start_time), Pro Account (display_name, studio_address, timezone), Recurring Series (interval) |
| A previously conflicted occurrence's replacement time is accepted | FEAT-21.SPEC-004 (Occurrence Generation & Conflict Handling) | Fires once, when a client's submitted replacement time for a flagged occurrence passes validation | Booking (service, start_time -- the newly set replacement time), Pro Account (display_name, studio_address, timezone) |

## Audience and Preferences

**Recipients:** The Client tied to the occurrence's Booking (Access Matrix: Recurring Appointments = Own-only for the Client) -- the sole recipient, since this discloses that client's own standing-appointment schedule.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Texting consent (governs channel, not whether this message sends) | Granted / Revoked | Captured at booking | FEAT-06 (Client Booking Identity) at booking; changed via FEAT-14 (Messaging Consent Management) |

This confirmation carries no separate on/off toggle: it is the direct outcome of the client's own standing-appointment choice, not a discretionary reminder. A client cannot opt out of being told her next occurrence has been scheduled -- only the channel it arrives on varies, per FEAT-14.SPEC-007.

**Quiet Hours:** Per this Brief's Non-Functional Notes and its recorded reading, none of this feature's three notifications depend on a short window for their value, so all three follow the platform's roughly 8am-9pm daytime-hours rule (XBR-16, platform parameter: `reminder-window-start-hour` to platform parameter: `reminder-window-end-hour`, in the Pro's timezone): a generation event that occurs outside this window holds the notification until the window next opens.

## Content Definition

**Text:**
- **Body:** {pro_display_name} just scheduled your next standing appointment: {service_name} on {occurrence_date} at {occurrence_time} ({timezone}). Manage this series: {series_manage_link}
- **CTA:** {series_manage_link} -- deep-links to FEAT-21.SPEC-002 (My Recurring Series) for this client's series

**Email:**
- **Subject:** Your next appointment with {pro_display_name} is scheduled
- **Body:**
  Hi {client_first_name},

  Your standing appointment series (every {interval} weeks) has generated its next visit:

  Service: {service_name}
  Date & time: {occurrence_date} at {occurrence_time} ({timezone})
  Location: {studio_address}

  You'll get a separate deposit link closer to the date. To view or manage your series, use the link below.
- **CTA (button):** View my series -- deep-links to FEAT-21.SPEC-002 (My Recurring Series) for this client's series

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {pro_display_name} | Pro Account -- display_name | Talia | Never empty -- required (Pro Account entity) |
| {service_name} | Service -- name (at generation time) | Full Set Lashes | Never empty -- required at service creation (FEAT-01) |
| {occurrence_date} / {occurrence_time} | Booking (occurrence) -- start_time, rendered in the Pro's timezone | Nov 12, 2026 / 2:30 PM | Never empty -- start_time is set at successful generation |
| {timezone} | Pro Account -- timezone | Eastern Time | Never empty -- required per account (XBR-25) |
| {interval} | Recurring Series -- interval | 3 | Never empty -- required at series creation (FEAT-21.SPEC-003) |
| {studio_address} | Pro Account -- studio_address | 123 Main St, Suite 4, Austin, TX | Never empty -- required before go-live (XBR-26) |
| {series_manage_link} | Access Link -- scoped to this client's series, issued via FEAT-06's access-link mechanism | chairtime.app/m/9c3f2a | If link issuance fails, the notification is held and retried per FEAT-08.SPEC-009's failure handling -- it is never sent without its manage link |
| {client_first_name} | Client -- name (first token) | Riley | Renders the full name field if no separable first token exists |

## Delivery Rules

**Batching:** None -- exactly one notification is sent per occurrence generation event. If several occurrences from different series generate for the same client in the same run (an unusual case, since one client typically holds few series), each occurrence's confirmation is sent separately, since each names a distinct appointment the client needs to recognize individually.
**Deduplication:** At most one generated-notification per occurrence. FEAT-21.SPEC-004 guarantees at most one successful generation event per occurrence (a conflicted occurrence's eventual replacement-time acceptance is itself the one generation event for that occurrence), so this notification's trigger cannot re-fire for the same occurrence.
**Retry on failure:** Governed by FEAT-08.SPEC-009 (Message Delivery Retry & Fallback): a failed text is retried once, then falls back to email, and the delivery gap is flagged on the Pro's dashboard (XBR-17).
**Expiry:** None -- a generated-occurrence confirmation never becomes not worth sending; even a late-arriving confirmation (after a retry/fallback cycle) still correctly describes a real, upcoming appointment.

## Edge Cases

- **The occurrence's Booking is cancelled in the brief window between generation and this notification's send** -- The notification still sends (it reports what was true at the moment of successful generation); the client also promptly sees the updated state on FEAT-21.SPEC-002, so she is never left confused for long about a cancelled occurrence she was just told about.
- **Client has both texting consent and an email on file** -- Text is used; no duplicate confirmation is also sent by email.
- **Client's phone number changed since her last booking and fresh consent has not yet been captured** -- Per XBR-15, the notification routes to email, never to the old or unconsented number.
- **A generation event occurs at 11pm in the Pro's timezone** -- The notification is held until the daytime window opens the next morning (platform parameter: `reminder-window-start-hour`), consistent with this Brief's recorded reading that all three of this feature's notifications follow the daytime-hours rule.
- **The client's series manage link cannot be issued at send time** -- The send waits for the link and is treated as a delivery failure under FEAT-08.SPEC-009's retry/fallback path if issuance does not complete in time; the notification is never sent with a missing link.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-21.SPEC-004 (Occurrence Generation & Conflict Handling) | Triggered by (inbound) | A successful generation, or an accepted replacement time, fires this notification |
| FEAT-14.SPEC-007 (Textability Determination Rule) | References (inbound) | Decides text vs. email for this send |
| FEAT-08.SPEC-012 (Transactional Text Messaging Capability) | Triggers (outbound) | Performs the text send when text is the chosen channel |
| FEAT-08.SPEC-013 (Transactional Email Capability) | Triggers (outbound) | Performs the email send when email is chosen or is the fallback |
| FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | References (outbound) | Governs retry and fallback behavior if this send fails |
| FEAT-21.SPEC-002 (My Recurring Series) | Navigation (outbound) | The CTA deep-links here |

## Analytics and Success Signals

- **recurring_occurrence_generated_notification_sent** (channel: text / email; generation_outcome: normal / replacement_time_accepted) -- supports success-metrics.md: "Self-Service Reschedule Rate" (a client's standing series continuing to inform her without any contact to the Pro is the same self-service pattern this metric measures)
- **recurring_occurrence_generated_notification_cta_tapped** (channel) -- N/A -- no Stage 2 metric measures this notification's own click-through directly; retained so engagement with the series-management surface is observable.

## Acceptance Criteria

**FEAT-21.SPEC-007-AC-01:** Given Riley has active texting consent and her occurrence generates normally, when the generation completes, then she receives a text confirming the service, date/time with timezone, and a link to manage her series.

**FEAT-21.SPEC-007-AC-02:** Given Riley declined texting and provided an email, when her occurrence generates normally, then she receives the same content by email instead.

**FEAT-21.SPEC-007-AC-03:** Given a previously conflicted occurrence's replacement time is accepted, when the acceptance completes, then Riley receives this same notification for the newly scheduled time.

**FEAT-21.SPEC-007-AC-04:** Given Riley taps the manage-series link in this notification, when the tap registers, then it opens FEAT-21.SPEC-002 scoped to her series.

**FEAT-21.SPEC-007-AC-05:** Given Riley's occurrence is cancelled 30 seconds after generation, when the notification and cancellation both process, then Riley still receives the notification, reflecting the moment of successful generation, and separately sees the updated state on FEAT-21.SPEC-002.

**FEAT-21.SPEC-007-AC-06:** Given Riley has both texting consent and an email on file, when her notification sends, then it arrives once, by text, with no duplicate email also sent.

**FEAT-21.SPEC-007-AC-07:** Given Riley's phone number changed and fresh consent has not been captured, when her occurrence generates, then the notification is sent by email, never to the unconsented number.

**FEAT-21.SPEC-007-AC-08:** Given a text send to Riley fails to deliver, when FEAT-08.SPEC-009's retry-then-fallback runs, then Riley still receives the notification by email, and the delivery gap is flagged on Talia's dashboard.

**FEAT-21.SPEC-007-AC-09:** Given an occurrence generates at 11pm in the Pro's timezone, when the generation completes, then the notification is held and delivered when the daytime window next opens, not before.

**FEAT-21.SPEC-007-AC-10:** Given the series manage link cannot be issued at send time, when the send is attempted, then it waits for the link and is treated as a delivery failure under FEAT-08.SPEC-009 if issuance does not complete in time.

**FEAT-21.SPEC-007-AC-11:** Given FEAT-21.SPEC-004 guarantees at most one successful generation event per occurrence, when the notification trigger is evaluated, then at most one generated-notification is ever sent for that occurrence.

**FEAT-21.SPEC-007-AC-12:** Given Talia (the Pro) is not named as a recipient of this notification, when an occurrence generates, then no push or message is sent to her from this spec -- she sees the occurrence only through her own schedule view and FEAT-21.SPEC-010 (Pro Recurring Series Management).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (text, email) | 2 |
| Trigger Paths | 2 | 2 |
| Preference States | 2 (consent granted, consent revoked/declined) | 2 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |



# Notification Spec: Occurrence Time Change Advance Notice

## Overview

**Name:** Occurrence Time Change Advance Notice
**ID:** FEAT-21.SPEC-008
**Type:** Notification
**Purpose:** Gives the client advance notice, with a prompt to pick a new time, when a standing appointment's usual slot is no longer available for an upcoming occurrence -- without disturbing the rest of her series.
**Parent Feature:** FEAT-21 -- Recurring/Standing Appointments

## Scope and Non-Goals

**In Scope:**
- The advance notice sent when an occurrence's usual time fails validation at generation, on both of the product's client channels (text and email)
- The prompt and its link to the pick-a-new-time flow
- Delivery, retry, and expiry behavior for this notification

**Non-Goals:**
- Deciding whether the usual time is actually unavailable, and validating whichever replacement time the client picks -- owned by FEAT-21.SPEC-004 (Occurrence Generation & Conflict Handling); this spec only announces the outcome and links to the flow it hands the client to.
- Confirming the occurrence once a replacement time is accepted -- owned by FEAT-21.SPEC-007 (Occurrence Generated Notification), a separate communication sent once a time is actually set.
- Deciding text vs. email for this send -- owned by FEAT-14.SPEC-007 (Textability Determination Rule).
- Notifying the Pro that one of her clients needs a new time -- product-features.md's Communications field for this feature names only a client-facing advance notice; a repeatedly failing generation (this spec's occurrence-level trigger is distinct from that) surfaces to the Pro separately, per FEAT-21.SPEC-004's own dashboard-flag path.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Text | The client has active Messaging Consent for texting (FEAT-14.SPEC-007) | This Brief's Communications field names this as an advance notice the client needs to act on soon; text reaches her where she is most likely to respond promptly |
| Email | The client has not granted texting consent (FEAT-14.SPEC-007) | Every client who declines texting still supplies an email at booking, so the notice is never simply undeliverable |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| An occurrence's usual time fails slot validation at generation | FEAT-21.SPEC-004 (Occurrence Generation & Conflict Handling) | Fires once per conflicted occurrence, immediately when the candidate slot fails validation | Booking (service, the usual time the occurrence would have had), Pro Account (display_name, timezone), the pick-a-new-time flow's entry point |

## Audience and Preferences

**Recipients:** The Client tied to the conflicted occurrence's Booking (Access Matrix: Recurring Appointments = Own-only for the Client) -- the sole recipient, since only she can pick the replacement time.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Texting consent (governs channel, not whether this message sends) | Granted / Revoked | Captured at booking | FEAT-06 (Client Booking Identity) at booking; changed via FEAT-14 (Messaging Consent Management) |

This notice carries no separate on/off toggle: it requires the client's action to keep her standing appointment on schedule, so it is never a discretionary message she can silence -- only the channel it arrives on varies, per FEAT-14.SPEC-007.

**Quiet Hours:** Per this Brief's recorded reading, this notification is an ordinary reminder-class communication with no short-window urgency, so it follows the platform's roughly 8am-9pm daytime-hours rule (XBR-16, platform parameter: `reminder-window-start-hour` to platform parameter: `reminder-window-end-hour`, in the Pro's timezone).

## Content Definition

**Text:**
- **Body:** Heads up -- {pro_display_name}'s usual time for your standing appointment ({service_name}, normally {usual_day_time}) isn't available this time. Pick a new time for just this one visit: {pick_new_time_link}
- **CTA:** {pick_new_time_link} -- deep-links to FEAT-21.SPEC-004's replacement-time flow for this one occurrence

**Email:**
- **Subject:** We need a new time for your next appointment with {pro_display_name}
- **Body:**
  Hi {client_first_name},

  Your standing appointment's usual time ({service_name}, normally {usual_day_time}) isn't available for your next visit -- {pro_display_name}'s schedule changed.

  This affects only this one upcoming appointment; the rest of your series continues as usual. Pick a new time below.
- **CTA (button):** Pick a new time -- deep-links to FEAT-21.SPEC-004's replacement-time flow for this one occurrence

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {pro_display_name} | Pro Account -- display_name | Talia | Never empty -- required (Pro Account entity) |
| {service_name} | Service -- name | Full Set Lashes | Never empty -- required at service creation (FEAT-01) |
| {usual_day_time} | Recurring Series -- originating_time (day-of-week and time-of-day pattern) | Tuesdays at 2:30 PM | Never empty -- set at series creation (FEAT-21.SPEC-003) |
| {pick_new_time_link} | Access Link -- scoped to this one conflicted occurrence, issued via FEAT-06's access-link mechanism | chairtime.app/m/7b1d4e | If link issuance fails, the notification is held and retried per FEAT-08.SPEC-009's failure handling -- it is never sent without its link |
| {client_first_name} | Client -- name (first token) | Riley | Renders the full name field if no separable first token exists |

## Delivery Rules

**Batching:** None -- one notice per conflicted occurrence. Each conflict names a distinct occurrence the client must individually act on, so batching would obscure which appointment needs a new time.
**Deduplication:** At most one advance notice per conflicted occurrence. FEAT-21.SPEC-004 flags an occurrence as needing a new time at most once per generation attempt; a re-attempt against the same still-unresolved occurrence does not re-send this notice.
**Retry on failure:** Governed by FEAT-08.SPEC-009 (Message Delivery Retry & Fallback): a failed text is retried once, then falls back to email, and the delivery gap is flagged on the Pro's dashboard (XBR-17).
**Expiry:** None on the notice itself -- it does not expire, since a client can still pick a new time for the occurrence at any point before the occurrence's own cancellation cut-off; the occurrence's status simply remains "Needs new time" on FEAT-21.SPEC-002 until she acts, per FEAT-21.SPEC-004's edge case that an abandoned replacement flow does not block the rest of the series.

## Edge Cases

- **The conflicted occurrence is cancelled by the client (via FEAT-21.SPEC-002) before she acts on this notice** -- The pick-a-new-time link, if tapped afterward, shows "This appointment is no longer active." (FEAT-21.SPEC-006), since there is nothing left to reschedule.
- **The whole series is cancelled while a conflicted occurrence's notice is still unresolved** -- The link, if tapped afterward, shows the same "no longer active" experience; no further prompt is sent once the series is Ended.
- **Client has both texting consent and an email on file** -- Text is used; no duplicate notice is also sent by email.
- **A conflict is detected at 11pm in the Pro's timezone** -- The notice is held until the daytime window opens the next morning, consistent with this Brief's recorded reading.
- **The client's phone number changed since her last booking and fresh consent has not yet been captured** -- Per XBR-15, the notice routes to email, never to the old or unconsented number.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-21.SPEC-004 (Occurrence Generation & Conflict Handling) | Triggered by (inbound) / Navigation (outbound) | A slot-validation failure at generation fires this notice; the CTA deep-links into that spec's replacement-time flow |
| FEAT-14.SPEC-007 (Textability Determination Rule) | References (inbound) | Decides text vs. email for this send |
| FEAT-08.SPEC-012 (Transactional Text Messaging Capability) | Triggers (outbound) | Performs the text send when text is the chosen channel |
| FEAT-08.SPEC-013 (Transactional Email Capability) | Triggers (outbound) | Performs the email send when email is chosen or is the fallback |
| FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | References (outbound) | Governs retry and fallback behavior if this send fails |
| FEAT-21.SPEC-006 (Series & Occurrence Cancellation Rules) | References (outbound) | Governs the "no longer active" experience if the occurrence or series is cancelled before the client acts |

## Analytics and Success Signals

- **recurring_occurrence_time_change_notice_sent** (channel: text / email) -- N/A -- no Stage 2 metric measures this notice's delivery directly; retained so the frequency of schedule-driven conflicts reaching clients is observable, alongside FEAT-21.SPEC-004's own conflict-flagged event.
- **recurring_occurrence_time_change_cta_tapped** (channel) -- supports success-metrics.md: "Self-Service Reschedule Rate" (a client picking her own new time in-app, without contacting the Pro, is exactly the self-service pattern this metric measures)

## Acceptance Criteria

**FEAT-21.SPEC-008-AC-01:** Given Riley has active texting consent and her occurrence's usual time fails validation, when the conflict is detected, then she receives a text naming the affected appointment and a link to pick a new time.

**FEAT-21.SPEC-008-AC-02:** Given Riley declined texting and provided an email, when her occurrence's usual time fails validation, then she receives the same content by email instead.

**FEAT-21.SPEC-008-AC-03:** Given Riley taps the pick-a-new-time link, when the tap registers, then it opens FEAT-21.SPEC-004's replacement-time flow scoped to that one occurrence.

**FEAT-21.SPEC-008-AC-04:** Given Riley reads this notice, then it states that only this one upcoming appointment is affected and the rest of her series continues as usual.

**FEAT-21.SPEC-008-AC-05:** Given Riley cancels the conflicted occurrence before acting on this notice, when she later taps the link anyway, then she sees "This appointment is no longer active."

**FEAT-21.SPEC-008-AC-06:** Given Riley's whole series is cancelled while this notice is unresolved, when she later taps the link, then she sees the same "no longer active" experience.

**FEAT-21.SPEC-008-AC-07:** Given Riley has both texting consent and an email on file, when this notice sends, then it arrives once, by text, with no duplicate email also sent.

**FEAT-21.SPEC-008-AC-08:** Given Riley's phone number changed and fresh consent has not been captured, when a conflict is detected for her occurrence, then the notice is sent by email, never to the unconsented number.

**FEAT-21.SPEC-008-AC-09:** Given a text send to Riley fails to deliver, when FEAT-08.SPEC-009's retry-then-fallback runs, then Riley still receives the notice by email, and the delivery gap is flagged on Talia's dashboard.

**FEAT-21.SPEC-008-AC-10:** Given a conflict is detected at 11pm in the Pro's timezone, when the conflict occurs, then the notice is held and delivered when the daytime window next opens.

**FEAT-21.SPEC-008-AC-11:** Given Riley does not act on this notice at all, when she later opens FEAT-21.SPEC-002, then the occurrence still shows "Needs new time" and the rest of her series is unaffected.

**FEAT-21.SPEC-008-AC-12:** Given FEAT-21.SPEC-004 flags an occurrence as needing a new time at most once per generation attempt, when the notice trigger is evaluated, then at most one advance notice is sent for that occurrence's unresolved conflict.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (text, email) | 2 |
| Trigger Paths | 1 | 1 |
| Preference States | 2 (consent granted, consent revoked/declined) | 2 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |



# Notification Spec: Occurrence Deposit Lifecycle Notification

## Overview

**Name:** Occurrence Deposit Lifecycle Notification
**ID:** FEAT-21.SPEC-009
**Type:** Notification
**Purpose:** Sends the client her occurrence's own fresh deposit link about a week before it, and, separately, tells both the client and the Pro when an unpaid occurrence is released.
**Parent Feature:** FEAT-21 -- Recurring/Standing Appointments

## Scope and Non-Goals

**In Scope:**
- The deposit-request variant, sent to the client, with the fresh deposit link for one occurrence
- The release variant, sent to both the client and the Pro, when an occurrence's deposit is unpaid by its cancellation cut-off
- Delivery, retry, and expiry behavior for both variants

**Non-Goals:**
- Deciding when to send the request or when to release the occurrence -- owned by FEAT-21.SPEC-005 (Occurrence Deposit Request & Release), which this spec's two variants are triggered by.
- Capturing the deposit payment itself -- owned by FEAT-07 (Deposit Payment at Booking); this spec only carries the link, it never processes the payment.
- Confirming the occurrence is scheduled -- owned by FEAT-21.SPEC-007 (Occurrence Generated Notification), a separate, earlier communication.
- Deciding text vs. email for either variant -- owned by FEAT-14.SPEC-007 (Textability Determination Rule).

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Text | The recipient has active Messaging Consent for texting (FEAT-14.SPEC-007) | This Brief's Primary Flows and Validation & Limits fields name both messages directly ("paid through a deposit link sent about a week before," "both the client and the Pro are told"); text matches the product's default channel for money-related, time-sensitive messages |
| Email | The recipient has not granted texting consent (FEAT-14.SPEC-007) | Every client who declines texting still supplies an email at booking; the Pro Account always carries a sign-in email as a channel of record |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| An occurrence reaches its deposit-request point | FEAT-21.SPEC-005 (Occurrence Deposit Request & Release) | Fires once per occurrence, when the deposit-request send is initiated | Booking (service, start_time, deposit_amount), the deposit-request link, Pro Account (display_name) |
| An occurrence is released unpaid at its cancellation cut-off | FEAT-21.SPEC-005 (Occurrence Deposit Request & Release) | Fires once per occurrence, when the release transition completes | Booking (service, start_time), Pro Account (display_name) |

## Audience and Preferences

**Recipients:** The deposit-request variant goes to the Client tied to the occurrence's Booking only (Access Matrix: Recurring Appointments = Own-only for the Client), since only she can pay it. The release variant goes to both the Client and the Pro (Recurring Appointments = Own-only for the Client, Full for the Pro on series tied to her own schedule), per this Brief's Validation & Limits field: "both the client and the Pro are told."

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Client texting consent (governs the client's channel) | Granted / Revoked | Captured at booking | FEAT-06 (Client Booking Identity) at booking; changed via FEAT-14 (Messaging Consent Management) |
| Pro notification preferences (governs the Pro's channel for the release variant) | In-app / text / email, per FEAT-27's notification_preferences field | Set during onboarding | FEAT-27 (Pro Profile & Booking Page Settings) |

Neither variant carries an on/off toggle for the recipients named above: the deposit-request is the mechanism by which the occurrence's own money is collected, and the release notice reports money that was never collected -- both are treated the same as any other transactional payment communication in this product, never a discretionary reminder.

**Quiet Hours:** Per this Brief's recorded reading, both variants are ordinary reminder-class communications with no short-window urgency, so both follow the platform's roughly 8am-9pm daytime-hours rule (XBR-16, platform parameter: `reminder-window-start-hour` to platform parameter: `reminder-window-end-hour`, in the Pro's timezone) for the client's copy; the Pro's release notice follows her own in-app/text/email preference and is not additionally gated by the daytime window, consistent with how other Pro attention alerts (FEAT-08.SPEC-006) are delivered.

## Content Definition

**Deposit-request variant -- Text (to Client):**
- **Body:** Your next appointment with {pro_display_name} ({service_name}, {occurrence_date} at {occurrence_time}) needs its deposit. Pay here: {deposit_link}
- **CTA:** {deposit_link} -- deep-links to FEAT-07 (Deposit Payment at Booking) for this one occurrence

**Deposit-request variant -- Email (to Client):**
- **Subject:** Deposit needed for your upcoming appointment with {pro_display_name}
- **Body:**
  Hi {client_first_name},

  Your next standing appointment is coming up:

  Service: {service_name}
  Date & time: {occurrence_date} at {occurrence_time} ({timezone})
  Deposit due: {deposit_amount}

  Pay your deposit using the link below to hold this time.
- **CTA (button):** Pay my deposit -- deep-links to FEAT-07 (Deposit Payment at Booking) for this one occurrence

**Release variant -- Text (to Client):**
- **Body:** Your appointment with {pro_display_name} on {occurrence_date} wasn't held because the deposit wasn't paid in time. Your series continues -- {pro_display_name} will reach out or you can rebook.
- **CTA:** None -- this is a status report, not an action the client can take on this occurrence; her next occurrence generates normally on the series' own schedule.

**Release variant -- Email (to Client):**
- **Subject:** Your {occurrence_date} appointment with {pro_display_name} was released
- **Body:**
  Hi {client_first_name},

  Your {occurrence_date} appointment with {pro_display_name} wasn't held because the deposit wasn't paid before the cancellation cut-off. Your standing series continues as normal, and your next visit will be scheduled in its usual turn.
- **CTA (button):** None.

**Release variant -- Text/Email (to Pro):**
- **Body:** {client_name}'s {occurrence_date} standing appointment ({service_name}) was released -- the deposit wasn't paid in time.
- **CTA:** View series -- deep-links to FEAT-21.SPEC-010 (Pro Recurring Series Management) for this client's series

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {pro_display_name} | Pro Account -- display_name | Talia | Never empty -- required (Pro Account entity) |
| {service_name} | Service -- name | Full Set Lashes | Never empty -- required at service creation (FEAT-01) |
| {occurrence_date} / {occurrence_time} | Booking (occurrence) -- start_time, rendered in the Pro's timezone | Nov 12, 2026 / 2:30 PM | Never empty -- set at generation (FEAT-21.SPEC-004) |
| {timezone} | Pro Account -- timezone | Eastern Time | Never empty -- required per account (XBR-25) |
| {deposit_amount} | Booking (occurrence) -- deposit_amount | $40.00 | Never empty -- computed at generation from the Service's rule (XBR-05) |
| {deposit_link} | Access Link -- scoped to this occurrence's deposit payment, issued via FEAT-07's deposit-request mechanism | chairtime.app/m/4d8a1f | If link issuance fails, the send is held and retried per FEAT-08.SPEC-009's failure handling -- it is never sent without its link |
| {client_first_name} / {client_name} | Client -- name (first token) / full name | Riley / Riley Chen | Renders the full name field if no separable first token exists |

## Delivery Rules

**Batching:** None -- each variant reports on exactly one occurrence, and each occurrence generates at most one deposit-request send and, separately, at most one release. Multiple occurrences from the same client's series never arrive at this point together, since occurrences generate and resolve on the series' own successive schedule.
**Deduplication:** At most one deposit-request send per occurrence, guarded by the deposit-request-sent flag FEAT-21.SPEC-005 records; at most one release notice per occurrence, since a Booking transitions to Expired at most once and cannot be released a second time.
**Retry on failure:** Governed by FEAT-08.SPEC-009 (Message Delivery Retry & Fallback): a failed text is retried once, then falls back to email, and the delivery gap is flagged on the Pro's dashboard (XBR-17) for the client-facing sends; the Pro's own release notice follows the same retry-then-fallback discipline across her configured channels.
**Expiry:** The deposit-request variant does not expire on its own -- it remains valid and payable up to the occurrence's cancellation cut-off, at which point FEAT-21.SPEC-005's release action supersedes it and this spec's release variant fires instead. The release variant, once sent, never expires -- it is a permanent record of what happened.

## Edge Cases

- **The client pays the deposit in the moments after the request notification was sent but before she opens it** -- No further action is taken by this spec; she simply proceeds to FEAT-07's confirmation, and no release notice is ever sent for that occurrence.
- **The occurrence is cancelled (FEAT-21.SPEC-006) after the deposit-request notice was sent but before the cancellation cut-off** -- No release notice is sent, since FEAT-21.SPEC-005's monitoring stops on cancellation; the client's earlier deposit-request notice simply becomes moot.
- **The client has both texting consent and an email on file** -- Text is used for her copy of either variant; no duplicate email is also sent.
- **The Pro's release notice arrives while she is mid-appointment with another client** -- It is delivered on whatever channel her notification_preferences specify and waits in-app until she next checks, exactly like any other Pro attention alert; it is never re-sent solely because she has not yet seen it.
- **A deposit-request send occurs at 11pm in the Pro's timezone** -- The client's copy is held until the daytime window next opens; the release variant, when it later fires, is likewise held on the client's side but not additionally gated for the Pro.
- **The client's phone number changed since her last booking and fresh consent has not yet been captured** -- Per XBR-15, her copy of either variant routes to email, never to the old or unconsented number.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-21.SPEC-005 (Occurrence Deposit Request & Release) | Triggered by (inbound) | The request and release moments each fire this notification's matching variant |
| FEAT-07 (Deposit Payment at Booking) | Navigation (outbound) | The deposit-request variant's CTA deep-links here |
| FEAT-14.SPEC-007 (Textability Determination Rule) | References (inbound) | Decides text vs. email for the client's copy of either variant |
| FEAT-27 (Pro Profile & Booking Page Settings) | References (inbound) | Supplies the Pro's notification_preferences for the release variant's Pro copy |
| FEAT-08.SPEC-012 (Transactional Text Messaging Capability) | Triggers (outbound) | Performs the text send when text is the chosen channel |
| FEAT-08.SPEC-013 (Transactional Email Capability) | Triggers (outbound) | Performs the email send when email is chosen or is the fallback |
| FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | References (outbound) | Governs retry and fallback behavior if either send fails |
| FEAT-21.SPEC-010 (Pro Recurring Series Management) | Navigation (outbound) | The release variant's Pro-facing CTA deep-links here |

## Analytics and Success Signals

- **recurring_occurrence_deposit_request_notification_sent** (channel: text / email) -- supports success-metrics.md: "Deposit Capture Rate" (this notice is the mechanism by which the client is invited to complete the capture this metric measures)
- **recurring_occurrence_release_notification_sent** (recipient: client / pro; channel) -- supports success-metrics.md: "Automatic Refund Correctness" (both parties being told, without either having to chase the outcome, is exactly the "completes without either person having to chase it" standard this metric names)
- **recurring_occurrence_deposit_request_cta_tapped** (channel) -- supports success-metrics.md: "Deposit Capture Rate"

## Acceptance Criteria

**FEAT-21.SPEC-009-AC-01:** Given Riley's occurrence reaches its deposit-request point and she has active texting consent, when the request fires, then she receives a text with the deposit amount and a payment link.

**FEAT-21.SPEC-009-AC-02:** Given Riley declined texting and provided an email, when her occurrence's deposit-request fires, then she receives the same content by email instead.

**FEAT-21.SPEC-009-AC-03:** Given Riley taps her deposit link, when the tap registers, then it opens FEAT-07's deposit payment flow for that one occurrence.

**FEAT-21.SPEC-009-AC-04:** Given Riley's occurrence is released unpaid at its cancellation cut-off, when the release completes, then both Riley and Talia are notified.

**FEAT-21.SPEC-009-AC-05:** Given Riley receives the release notice, then it states that her series continues and that her next visit will be scheduled in its usual turn.

**FEAT-21.SPEC-009-AC-06:** Given Talia receives the release notice, then it names the client and the occurrence date and links to her own series view (FEAT-21.SPEC-010).

**FEAT-21.SPEC-009-AC-07:** Given Riley pays her deposit before opening the request notification, when payment completes, then no release notice is ever sent for that occurrence.

**FEAT-21.SPEC-009-AC-08:** Given Riley's occurrence is cancelled after the deposit-request notice was sent but before the cancellation cut-off, when the cancellation completes, then no release notice is sent.

**FEAT-21.SPEC-009-AC-09:** Given Riley has both texting consent and an email on file, when either variant sends to her, then it arrives once, by text, with no duplicate email also sent.

**FEAT-21.SPEC-009-AC-10:** Given Talia's notification_preferences specify email for Pro notifications, when the release variant fires for her, then she receives it by email, not text or in-app alone.

**FEAT-21.SPEC-009-AC-11:** Given a deposit-request send occurs at 11pm in the Pro's timezone, when the send is attempted, then Riley's copy is held until the daytime window next opens.

**FEAT-21.SPEC-009-AC-12:** Given Riley's phone number changed and fresh consent has not been captured, when either variant fires for her, then it is sent by email, never to the unconsented number.

**FEAT-21.SPEC-009-AC-13:** Given a text send to Riley fails to deliver, when FEAT-08.SPEC-009's retry-then-fallback runs, then she still receives the notification by email, and the delivery gap is flagged on Talia's dashboard.

**FEAT-21.SPEC-009-AC-14:** Given a Booking can transition to Expired at most once, when the release trigger is evaluated for an occurrence, then at most one release notice is ever sent for it.

**FEAT-21.SPEC-009-AC-15:** Given Riley's deposit-request-sent flag is already set for an occurrence (FEAT-21.SPEC-005), when a second evaluation runs before the first send completes, then no duplicate deposit-request notification is sent.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (text, email) | 2 |
| Trigger Paths | 2 | 2 |
| Preference States | 3 (client consent granted, client consent revoked/declined, Pro channel preference) | 3 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 6 | 6 |



# Screen Spec: Pro Recurring Series Management

## Overview

**Name:** Pro Recurring Series Management
**ID:** FEAT-21.SPEC-010
**Type:** Screen
**Purpose:** Lets Talia set up a standing appointment for a client while the client is at the chair, then see that client's series and upcoming occurrences on her own schedule, and cancel one occurrence or end the whole series.
**Parent Feature:** FEAT-21 -- Recurring/Standing Appointments

## Scope and Non-Goals

**In Scope:**
- Setting up a Recurring Series for a client from one of that client's existing bookings on Talia's schedule, by choosing a repeat interval of every 1 to 12 weeks
- Listing the client's Active series on Talia's schedule, each with its upcoming generated occurrences grouped underneath and each occurrence's own status
- Cancelling one upcoming occurrence, or ending the whole series, on the client's behalf
- Showing the refreshed series when the client changes the same series or occurrence at the same moment
- The empty, loading, error, and offline/degraded behavior of this connectivity-dependent screen

**Non-Goals:**
- Validating the interval and the booking-horizon ceiling -- owned by FEAT-21.SPEC-003 (Recurring Series Setup & Generation Limits); this screen only surfaces its result and error message.
- Defining what cancelling one occurrence or the whole series does, and how a simultaneous client and Pro change resolves -- owned by FEAT-21.SPEC-006 (Series & Occurrence Cancellation Rules); this screen only exposes the two actions and reflects their result.
- Generating occurrences, resolving a conflicted occurrence time, or requesting and releasing occurrence deposits -- owned by FEAT-21.SPEC-004 and FEAT-21.SPEC-005; this screen shows their results as occurrence statuses and never picks a replacement time on the client's behalf, because the pick-a-new-time flow belongs to the client (FEAT-21.SPEC-008).
- Editing a series' interval after creation -- excluded per this Brief's Non-Goals: the feature names only setup, group viewing/management, and cancellation; a different cadence means ending the series here and setting up a fresh one from a later booking.
- A pause action for a series -- excluded per this Brief's Non-Goals: no Stage 2 flow describes pausing, so only Active and Ended states exist.
- Recurring-series management inside FEAT-30's own screens -- excluded per FEAT-30's Non-Goals; FEAT-30 links to this screen instead.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-30.SPEC-001 (Cancel Booking (Pro-Initiated)) -- within FEAT-30 (Pro Booking Management), Pro booking detail outbound link | Talia opens a client's booking on her schedule and taps the "Recurring series" link ("Repeat this booking" when the booking has no series, "Manage recurring series" when it belongs to one) | Booking reference and client reference; the screen opens on the set-up form when that booking is eligible to originate a series, otherwise on the client's series |
| FEAT-21.SPEC-009 (Occurrence Deposit Lifecycle Notification) | Talia taps "View series" in the release notice for an unpaid occurrence | The released occurrence's series reference |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen, for series and bookings tied to her own schedule only | Set up a series for a client, cancel one occurrence, end the whole series | A reference to a series or booking that is not on her schedule shows "This series isn't on your schedule." with a single "Back" option; nothing about the other Pro's client is shown |
| The Client (Riley) | No | No | No control on any client-facing surface reaches this screen; Riley's own equivalent screens are FEAT-21.SPEC-001 (set up) and FEAT-21.SPEC-002 (view and cancel), which are reached through her own booking confirmation and bookings view |
| Platform Operator (Support) | No | No | This screen is Pro-facing and offers write actions; Support's view-only access to Recurring Series is surfaced through FEAT-19's own screen, never through this feature's screens |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29); no client or series data is shown, and after signing in Talia lands on FEAT-12 (Pro Daily Schedule Dashboard) |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- a chosen but unsubmitted interval is preserved and restored once she re-authenticates; no cancellation is ever completed on an expired session |

## Layout and Content

**Header:** Back arrow (returns to where Talia came from) and the title "Standing appointments" with the client's name ("{client_name}") beneath it.

**Body:** Two regions, top to bottom.

1. **Set-up region** (shown only when the entry booking is eligible, see Business Rules):
   - Booking summary line, read-only: "{service_name}, {booking_date} at {booking_time}" -- the service and time the series will repeat.
   - **Repeat interval** (numeric stepper, required): whole weeks from 1 to 12, labeled "Repeat every ___ weeks". No value is pre-selected.
   - Preview line, read-only, shown once an interval is chosen: "First repeat: {first_due_date}. It is booked automatically once that date is inside your booking window."
   - Primary "Set up standing appointment" button, disabled until an interval is chosen.
2. **Series region:** one card per Active Recurring Series that the client holds on Talia's schedule (normally zero or one). Each card contains:
   - Series summary line: "{service_name}, every {interval} weeks" and an "End series" text action.
   - A list of the series' upcoming occurrences, ordered by date, soonest first. Each row shows the occurrence's date and time, a status badge (Upcoming / Awaiting deposit / Needs new time), and a "Cancel this one" text action scoped to that single occurrence.
   - When a series has no upcoming occurrences yet, the note "The next appointment will appear here once it's scheduled."

When both regions apply, the set-up region sits above the series region. When the entry booking is not eligible and the client holds no series, the body shows the empty state (see States).

**Footer:** None.

### Responsive Behavior

- **Compact size class:** Both regions stack full width in a single column; the primary set-up button sits at the bottom of the set-up region.
- **Medium size class and above:** The column stays single, capped at a consistent platform-wide content width (the design layer's decision) and horizontally centered; no structural change beyond width capping.
- **Occurrence rows:** The date/status pair and the "Cancel this one" action share a row at every size class; at compact width the action wraps beneath the date/status pair.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to the originating screen: FEAT-30.SPEC-001 (Cancel Booking (Pro-Initiated)) booking detail, or FEAT-12 (Pro Daily Schedule Dashboard) when Talia arrived from a FEAT-21.SPEC-009 notice | Screen closes | Animated transition back |
| Repeat interval stepper | Increment/decrement or type a value | Captures the chosen interval | Stepper shows the value; the preview line appears; "Set up standing appointment" becomes enabled | Stepper and preview show the new values |
| "Set up standing appointment" button | Tap | 1. Validate the interval via FEAT-21.SPEC-003. 2. If valid, create the Recurring Series in state Active from the entry booking, with originating service and time copied from that booking. 3. Hand off to FEAT-21.SPEC-004 to generate the first occurrence(s). | Button shows a loading state; on success the set-up region is replaced by the new series card | Success: toast "Standing appointment set up -- every {interval} weeks". Failure: field-level or banner message (see States) |
| "Set up standing appointment" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |
| "End series" (per card) | Tap | Opens a confirmation dialog; on confirm, triggers FEAT-21.SPEC-006's whole-series cancellation as The Pro | Dialog appears; on confirm the card is removed once the cancellation completes | Dialog: "End this whole series? {occurrence_count} upcoming appointments for {client_name} will be cancelled and their deposits refunded in full." with "End Series" and "Keep Series". Success toast: "Series ended" |
| "Cancel this one" (per occurrence row) | Tap | Opens a confirmation dialog; on confirm, triggers FEAT-21.SPEC-006's single-occurrence cancellation as The Pro | Dialog appears; on confirm the row is removed once the cancellation completes; the series card remains | Dialog: "Cancel the {occurrence_date} appointment for {client_name}? The rest of the series continues and any deposit is refunded in full." with "Cancel Appointment" and "Keep It". Success toast: "Appointment cancelled" |
| Occurrence status badge | Display only | No action -- read-only status indicator | None | Not interactive |
| Preview line, booking summary line | Display only | No action | None | Not interactive |

### Accessibility Notes

- **Focus order:** Back arrow -> Repeat interval stepper -> "Set up standing appointment" -> each series card in order: series summary, "End series", then each occurrence row's date/status and "Cancel this one", top to bottom.
- **Dynamic content announcements:** The validation message, the set-up toast, both cancellation toasts, the refreshed-series notices, and every confirmation dialog's text are announced to assistive technology on appearance; a validation message is programmatically associated with the stepper.
- **Focus management:** After a dialog closes, focus returns to the control that opened it; after a successful set-up, focus moves to the new series card's summary line.
- **Keyboard alternatives:** The stepper is operable by arrow keys or direct numeric entry; every action and dialog choice is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | A loading indicator in place of both regions | Screen opens while the booking and the client's series are being fetched | Data loads (Set-up ready, Loaded, or Empty) or fails (Error) |
| Set-up ready | Set-up region with the stepper unset and the button disabled; series region shown below if the client also holds a series | The entry booking is eligible and the client holds no series, or holds one from another booking | Talia chooses an interval, or leaves |
| Filling | Stepper shows the chosen value, preview line shown, button enabled | Talia sets an interval | Talia submits or leaves |
| Submitting | Button shows a loading spinner; stepper disabled | Talia taps "Set up standing appointment" | Validation and creation complete or fail |
| Validation Error | Stepper error state with "Choose a repeat interval between 1 and 12 weeks." below it; button re-enabled; attempted value stays visible | FEAT-21.SPEC-003 rejects the interval | Talia corrects the value |
| Loaded | Series region with one or more cards and their occurrence groups | The client holds at least one Active series on Talia's schedule | Talia leaves, or the last series ends (returning to Empty) |
| Empty | Text "No standing appointments for {client_name}." and, when the entry booking is not eligible, one line stating why (for example "This booking has already started a standing appointment or was cancelled.") | The entry booking is not eligible and the client holds no Active series | Talia leaves; the state is never blocking |
| Cancelling | The card or row being cancelled shows a loading indicator and its action is disabled | Talia confirms a cancel dialog | Cancellation completes (row/card removed) or fails (Error) |
| Error | Error banner at the top with a Retry option; the chosen interval, if any, is preserved; regions that failed to load are not shown | A data load, series creation, or cancellation fails after passing validation | Talia taps Retry or leaves |
| Offline/Degraded | N/A -- requires connectivity for correctness, consistent with the rest of scheduling (this Brief's States field); an attempted set-up or cancellation shows "You'll need to be online to do this. Please check your connection and try again." and nothing is submitted or queued | Connectivity lost while attempting an action | Connectivity restored and Talia retries |

## Validation Rules

Validation of the repeat interval and the booking-horizon ceiling is governed by FEAT-21.SPEC-003 (Recurring Series Setup & Generation Limits). See that spec for the exact rule and its error message, "Choose a repeat interval between 1 and 12 weeks." This screen applies the rule on submit. The effect and eligibility of each cancel action is governed by FEAT-21.SPEC-006 (Series & Occurrence Cancellation Rules); its refusal messages ("This series has already been cancelled." and "This appointment is no longer active.") are shown here exactly as that spec defines them.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap (arrived from a booking) | FEAT-30.SPEC-001 (Cancel Booking (Pro-Initiated)) booking detail | FEAT-30 (Pro Booking Management) |
| Back arrow tap (arrived from a release notice) | FEAT-12 (Pro Daily Schedule Dashboard) | FEAT-12 (Pro Daily Schedule Dashboard) |
| Successful set-up | This screen, with the new series card shown | -- |
| Successful occurrence or series cancellation | This screen, with the cancelled row or card removed | -- |

## Data Model

**Creates:** Recurring Series -- interval (from the stepper), originating service and time (copied from the entry booking, never entered here), state set to Active.
**Reads:** Recurring Series -- interval, originating service and time, state, generated occurrences (series on Talia's schedule only); Booking (occurrence and entry booking) -- service, start_time, state, source, series reference; Client -- name (for display only).
**Updates:** Recurring Series -- state transitioned to Ended (via FEAT-21.SPEC-006, on "End series"). Booking (occurrence) -- state transitioned to Cancelled by Pro (via FEAT-21.SPEC-006).
**Deletes:** None.

## Business Rules

- Set-up is offered only when the entry booking is on Talia's schedule, is in Pending Payment, Confirmed, or Completed state, is not itself a generated occurrence of a series, and has never originated a series (FEAT-21.SPEC-003's one-series-per-booking rule, whatever that series' current state). Otherwise the set-up region is not shown.
- Talia may set up a series for any client on her schedule (FEAT-21.SPEC-003 Authorization Rules); the interval and horizon rules are the same as on FEAT-21.SPEC-001. Occurrence generation afterwards follows the ordinary client-facing notice and horizon rules (FEAT-21.SPEC-004, XBR-01, XBR-03), because it stands in for a booking the client would otherwise make.
- A successful set-up hands off immediately to FEAT-21.SPEC-004 for the first occurrence(s); Talia does not wait on this screen for generation, and the new series card shows "The next appointment will appear here once it's scheduled." until it completes.
- Both cancel actions are governed entirely by FEAT-21.SPEC-006. Every cancelled occurrence's deposit is refunded in full because any Pro cancellation refunds in full (XBR-09, applied by FEAT-09); the client is told through FEAT-08.SPEC-004 (Booking Change & Refund Notice), and the Pro's personal calendar mirrors each cancelled occurrence (XBR-13, FEAT-04).
- The occurrence group list pattern (this Brief's Shared UI Patterns) is followed exactly: cancelling one occurrence never reads as ending the series, the two actions are separate controls, and the series card remains after a single-occurrence cancellation.
- An occurrence's status badge follows FEAT-21.SPEC-002's mapping: Upcoming (Pending Payment before the deposit request, or Confirmed), Awaiting deposit (deposit link sent, unpaid), Needs new time (usual slot unavailable, per FEAT-21.SPEC-004).
- Contention (dependency map, Recurring Series): the Client and the Pro can both change a series or an occurrence; the resolution is reject-with-refresh, first committed change wins (FEAT-21.SPEC-006).

## Edge Cases

- **Riley cancels the same series or occurrence Talia is viewing, at the same moment Talia confirms a cancel (concurrent-edit conflict)** -- Reject-with-refresh, per FEAT-21.SPEC-006 and the dependency map's Contention note: the first committed change wins, Talia's screen refreshes to the updated series, and she sees "Riley already cancelled this. Your view has been updated." instead of the success toast.
- **The client's booking is cancelled or expires between Talia opening the screen and submitting the set-up** -- Submission is refused with refresh: "This booking is no longer active, so it can't be made recurring." and the set-up region disappears.
- **Talia opens the screen for a booking that already originated a series** -- The set-up region is not shown; the client's series (if still Active) is shown instead, or the empty state explains that the booking already started a standing appointment.
- **Talia taps a set-up or cancel control twice rapidly** -- The second tap is ignored while the first is in progress (control disabled in Submitting/Cancelling).
- **Talia chooses 0, 13 or more, or a fractional number of weeks** -- FEAT-21.SPEC-003 rejects it; the stepper shows the error and her attempted value stays visible.
- **Network failure during set-up or cancellation** -- Error banner: "Could not complete that. Check your connection and try again." with Retry; the interval stays entered and the card or row stays in its pre-action state.
- **An occurrence Talia tries to cancel was released unpaid moments earlier by FEAT-21.SPEC-005, or was already cancelled** -- Refused with refresh: "This appointment is no longer active." and the row updates to its current state.
- **Talia taps "End series" on a series that already ended** -- Refused with "This series has already been cancelled." and the card is removed on refresh.
- **The series' service was archived and generation has paused** -- The series card still shows; the note "New appointments aren't being scheduled because this service is no longer active." appears under the summary line, matching the gap FEAT-21.SPEC-004 flags on FEAT-12.
- **Talia reopens the screen right after a set-up or cancellation** -- The screen loads the current data, not a stale cached view.
- **The client holds a very long list of upcoming occurrences (a 1-week interval across a wide booking horizon)** -- The occurrence list scrolls inside the card; nothing is truncated or paginated, since the list is bounded by the booking horizon.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-30.SPEC-001 (Cancel Booking (Pro-Initiated)) -- within FEAT-30 (Pro Booking Management) | Navigation (inbound/outbound) | Talia arrives from the "Recurring series" link on a client's booking detail and returns there with the back arrow |
| FEAT-21.SPEC-003 (Recurring Series Setup & Generation Limits) | References (outbound) | Validates the interval and horizon at set-up; authorizes Talia to create a series for a client |
| FEAT-21.SPEC-004 (Occurrence Generation & Conflict Handling) | Triggers (outbound) / References (inbound) | A successful set-up hands off to it; the occurrences and statuses it produces appear here |
| FEAT-21.SPEC-005 (Occurrence Deposit Request & Release) | References (inbound) | Supplies the Awaiting deposit status and the removal of a released occurrence |
| FEAT-21.SPEC-006 (Series & Occurrence Cancellation Rules) | References (outbound) | Governs the effect, authorization, and contention outcome of both cancel actions |
| FEAT-21.SPEC-009 (Occurrence Deposit Lifecycle Notification) | Navigation (inbound) | The release notice's "View series" call to action opens this screen |
| FEAT-21.SPEC-002 (My Recurring Series) | References (inbound) | The client-facing counterpart; the same series and occurrences, shown to Riley |
| FEAT-09 (Cancellation & No-Show Policy Engine) | References (outbound) | Applies the full-refund outcome for each Pro-cancelled occurrence (XBR-09) |
| FEAT-08.SPEC-004 (Booking Change & Refund Notice) | Triggers (outbound) | Tells the client about each occurrence the Pro cancels |
| FEAT-04 (Two-Way Calendar Sync) | Triggers (outbound) | Mirrors created, moved, and cancelled occurrences to Talia's personal calendar (XBR-13) |
| FEAT-12 (Pro Daily Schedule Dashboard) | Navigation (outbound) | Back destination when Talia arrived from a release notice; also where generation gaps are flagged |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| recurring_series_created | interval_weeks, entry source (Pro at the chair) | Talia's set-up submission creates the Recurring Series | N/A -- "Self-Service Reschedule Rate" measures clients acting without the Pro, so a Pro-created series does not feed it; the event keeps the Stage 2 signal name so Pro-created and client-created series are countable together |
| recurring_occurrence_cancelled | series reference, acting role (Pro) | Talia cancels a single occurrence from this screen | N/A -- Stage 2 names no distinct signal for cancelling a single occurrence; per this Brief's Signals note, this action reuses the generic booking-cancelled signal the owning cancellation feature emits for any Booking |
| recurring_series_cancelled | interval_weeks, occurrences_cancelled_count, acting role (Pro) | Talia ends the whole series from this screen | N/A -- "Self-Service Reschedule Rate" measures client-initiated changes; a Pro-initiated series end is recorded under the Stage 2 signal name for completeness but feeds no client self-service metric |
| recurring_series_management_viewed | series_count, entry source (booking detail / release notice) | The screen loads with one or more series | N/A -- no Stage 2 metric measures view frequency for this screen; retained so usage of the Pro-side surface is observable |

## Acceptance Criteria

**FEAT-21.SPEC-010-AC-01:** Given Talia opens Riley's eligible booking in FEAT-30.SPEC-001 and taps "Repeat this booking", when this screen opens, then the set-up region shows the booking summary, an unset interval stepper, and a disabled "Set up standing appointment" button.

**FEAT-21.SPEC-010-AC-02:** Given Talia sets the interval to 3 weeks, when the value is entered, then the button becomes enabled and the preview line shows the first repeat date and that it is booked once inside her booking window.

**FEAT-21.SPEC-010-AC-03:** Given Talia submits an interval of 3 weeks, when FEAT-21.SPEC-003 validation passes, then a Recurring Series is created in state Active with the originating service and time copied from Riley's booking, the toast "Standing appointment set up -- every 3 weeks" appears, and FEAT-21.SPEC-004 is triggered to generate its first occurrence(s).

**FEAT-21.SPEC-010-AC-04:** Given Talia submits an interval of 0, 13, or 2.5 weeks, when validation runs, then the stepper shows "Choose a repeat interval between 1 and 12 weeks.", no series is created, and her attempted value stays visible.

**FEAT-21.SPEC-010-AC-05:** Given Riley's booking has already originated a series, when Talia opens this screen from that booking, then the set-up region is not shown.

**FEAT-21.SPEC-010-AC-06:** Given Riley's booking is Cancelled or Expired, when Talia opens this screen from it and Riley holds no Active series, then the Empty state shows "No standing appointments for Riley Chen." with the reason line.

**FEAT-21.SPEC-010-AC-07:** Given Riley holds one Active series with two upcoming occurrences, when Talia opens this screen, then one card shows the interval and both occurrences with their status badges, soonest first.

**FEAT-21.SPEC-010-AC-08:** Given Talia taps "Cancel this one" and confirms "Cancel Appointment", when FEAT-21.SPEC-006 commits the cancellation, then only that occurrence's Booking becomes Cancelled by Pro, its row is removed, the toast "Appointment cancelled" appears, and the series card remains.

**FEAT-21.SPEC-010-AC-09:** Given Talia taps "End series" and confirms "End Series", when FEAT-21.SPEC-006 commits, then the series becomes Ended, every not-yet-occurred occurrence becomes Cancelled by Pro, the card is removed, and the toast "Series ended" appears.

**FEAT-21.SPEC-010-AC-10:** Given Talia opens either cancel dialog and taps "Keep It" or "Keep Series", then the dialog closes and nothing is cancelled.

**FEAT-21.SPEC-010-AC-11:** Given a cancelled occurrence had a captured deposit, when Talia's cancellation commits, then the deposit is refunded in full under FEAT-09 regardless of timing, and Riley is notified through FEAT-08.SPEC-004.

**FEAT-21.SPEC-010-AC-12:** Given an occurrence's deposit link was sent and is unpaid, when Talia views the screen, then the occurrence shows the "Awaiting deposit" badge; and given its usual slot is unavailable per FEAT-21.SPEC-004, it shows "Needs new time" with no control for Talia to pick a time.

**FEAT-21.SPEC-010-AC-13:** Given Riley taps "Cancel series" on FEAT-21.SPEC-002 at the same moment Talia confirms "End series", when both reach commit, then the first commit wins and the other party's screen refreshes; if Riley's committed first, Talia sees "Riley already cancelled this. Your view has been updated."

**FEAT-21.SPEC-010-AC-14:** Given an occurrence was released unpaid by FEAT-21.SPEC-005 moments before Talia confirms cancelling it, when her action is evaluated, then it is refused with "This appointment is no longer active." and the row refreshes.

**FEAT-21.SPEC-010-AC-15:** Given a series already ended, when Talia's stale screen submits "End series", then it is refused with "This series has already been cancelled." and the card is removed on refresh.

**FEAT-21.SPEC-010-AC-16:** Given Talia loses connectivity, when she attempts a set-up or a cancellation, then she sees "You'll need to be online to do this. Please check your connection and try again." and nothing is submitted or queued.

**FEAT-21.SPEC-010-AC-17:** Given a network failure occurs during a cancellation, when the request fails, then an error banner reads "Could not complete that. Check your connection and try again." with Retry, and the row or card stays in its pre-cancellation state.

**FEAT-21.SPEC-010-AC-18:** Given the series data is loading, when Talia opens the screen, then a loading indicator replaces both regions; and given the load fails, an error banner with Retry replaces them.

**FEAT-21.SPEC-010-AC-19:** Given Talia taps "Set up standing appointment" or a confirm button twice rapidly, then the second tap is ignored while the first is in progress.

**FEAT-21.SPEC-010-AC-20:** Given Riley (the Client) or a Platform Operator (Support) looks for this screen, when they search their own surfaces, then no control reaches it -- Riley uses FEAT-21.SPEC-001 and FEAT-21.SPEC-002, and Support uses FEAT-19's view-only screen.

**FEAT-21.SPEC-010-AC-21:** Given Talia is signed out or her session has expired, when she opens this screen, then she is sent to the Pro sign-in screen or sees "Your session has expired. Sign in to continue." with a chosen interval preserved after re-authentication.

**FEAT-21.SPEC-010-AC-22:** Given Talia sets up a series, cancels one occurrence, and ends a series, then recurring_series_created, recurring_occurrence_cancelled (reusing the generic booking-cancelled signal), and recurring_series_cancelled are emitted with acting role Pro.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 9 (loading, set-up ready, filling, submitting, validation error, loaded, empty, cancelling, error) plus offline N/A | 10 |
| Business Rules | 7 | 7 |
| Edge Cases | 11 | 11 |


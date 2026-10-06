---
document_type: spec
spec_type: automation
spec_id: FEAT-06.SPEC-002
spec_name: Access Link Validation & Redemption
spec_slug: access-link-validation-redemption
parent_feature: FEAT-06
parent_feature_name: Client Booking Identity
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Automation Spec: Access Link Validation & Redemption

## Overview

**Name:** Access Link Validation & Redemption
**ID:** FEAT-06.SPEC-002
**Type:** Automation
**Purpose:** System validates a tapped access link -- on-demand or booking-specific -- checks its scope, expiry, and used/unused state, writes the resulting state, and routes the client to the right screen.
**Parent Feature:** FEAT-06 -- Client Booking Identity

## Scope and Non-Goals

**In Scope:**
- Looking up the token carried by any tapped access link (on-demand or booking-specific)
- Checking the link's expiry, single-use state, and scope against FEAT-06.SPEC-007's lifecycle rules
- Marking the link Used on first successful redemption
- Routing the client to FEAT-06.SPEC-003 (on-demand, valid) or FEAT-06.SPEC-004 (booking-specific, valid), or back to FEAT-06.SPEC-001 with a re-request prompt (invalid, expired, or already used)

**Non-Goals:**
- Generating the access link -- on-demand links are created by FEAT-06.SPEC-001, booking-specific links by FEAT-08 and FEAT-30; this spec only consumes an already-issued link
- Displaying the client's bookings once routed -- owned by FEAT-06.SPEC-003 and FEAT-06.SPEC-004
- Enforcing the rate limit on how often a link can be requested -- owned by FEAT-06.SPEC-007; this spec enforces expiry and single-use state, not request frequency
- Any password or code-entry fallback if the link fails -- excluded per BRIEF.md's Target Users & Roles: the only recovery path is requesting a new link (FEAT-06.SPEC-001), never a separate reset flow to learn

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Client taps a delivered on-demand access link | FEAT-06.SPEC-006 (Access Link Delivery) | Always, whenever a delivered link is tapped, regardless of its current validity | The tapped link's token |
| Client taps a booking-specific manage link from a confirmation or reminder | FEAT-08.SPEC-012 (text delivery) or FEAT-08.SPEC-013 (email delivery) | Always, whenever a booking-specific link embedded in a confirmation or reminder message is tapped | The tapped link's token |
| Client taps a fresh booking-specific manage link issued after a Pro reschedule | FEAT-30 (Pro Booking Management) | Fires when the client taps the replacement link issued after the Pro reschedules the booking | The tapped link's token |

## Processing Logic

1. Extract the token from the tapped link.
2. Look up the Access Link record by that token.
3. If no Access Link record matches the token, treat it as invalid (Outcome: Invalid Token).
4. If a record is found, read its scope (all bookings with this Pro, or one booking), expiry, and current state (Issued | Used | Expired).
5. If the state is already Used, treat it as invalid (Outcome: Already Used).
6. If the state is Issued, check expiry per FEAT-06.SPEC-007: an on-demand link is expired once 30 minutes have elapsed since creation; a booking-specific link is expired once the referenced Booking's appointment start time has passed.
7. If expired by either measure, mark the link's state Expired and treat the redemption as invalid (Outcome: Expired).
8. If the link is Issued and not expired, mark its state Used and record the redemption timestamp -- this is the point at which "first tap wins" is decided.
9. Confirm the link's scope still resolves to exactly one Client record with this Pro (and, for a booking-specific link, exactly one Booking belonging to that Client), per FEAT-06.SPEC-008.
10. Route the client based on scope: an on-demand link routes to FEAT-06.SPEC-003 (My Bookings List); a booking-specific link routes directly to FEAT-06.SPEC-004 (Booking Detail via Manage Link) for that one booking, with no code or password.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Valid, on-demand | Link found, Issued, not expired, scope is "all bookings with this Pro" | Access Link state set to Used, redemption timestamp recorded | Client is routed straight into their bookings list | FEAT-06.SPEC-003 |
| Valid, booking-specific | Link found, Issued, not expired, scope is one Booking | Access Link state set to Used, redemption timestamp recorded | Client is routed straight into that one booking's detail, with no code or password | FEAT-06.SPEC-004 |
| Invalid Token | No Access Link record matches the tapped token | None | Client is routed to FEAT-06.SPEC-001 showing the "request a new link" prompt; the message never distinguishes this from any other invalid outcome | FEAT-06.SPEC-001 |
| Already Used | Link found but its state is already Used (a second device opening a single-use link after first use) | None -- the Used state and its original redemption timestamp are left unchanged | Client is routed to FEAT-06.SPEC-001 showing the identical "request a new link" prompt | FEAT-06.SPEC-001 |
| Expired | Link found, Issued, but its expiry condition (30 minutes for on-demand, appointment passed for booking-specific) has been reached | Access Link state set to Expired | Client is routed to FEAT-06.SPEC-001 showing the identical "request a new link" prompt | FEAT-06.SPEC-001 |
| Scope resolution failure | Link is otherwise valid but its scoped Client or Booking record can no longer be resolved (e.g., the referenced booking record no longer exists in a state this feature can show) | None | Client is routed to FEAT-06.SPEC-001 showing the identical "request a new link" prompt -- never a technical error, per the "no hint" pattern | FEAT-06.SPEC-001 |

## Data Model

**Reads:** Access Link -- token, scope, expiry, state. Client -- matched via the Access Link's scope, for this Pro only, governed by FEAT-06.SPEC-008. Booking -- for a booking-specific link, the referenced Booking's start time (to evaluate expiry) and its owning Client (to confirm scope).

**Booking reader:** This spec is a Booking reader, as the feature-overview.md Referenced Entities table records (FEAT-06.SPEC-002, FEAT-06.SPEC-003, FEAT-06.SPEC-004): it reads the referenced Booking's start time (expiry evaluation at Steps 6-7) and owning Client (scope confirmation at Step 9).
**Creates:** None -- this automation never creates a new Access Link.
**Updates:** Access Link -- state set to Used (with redemption timestamp) on a valid redemption, or to Expired when an Issued link is found past its expiry condition at redemption time.
**Deletes:** None -- links are never deleted, only transitioned to Used or Expired (scope-boundaries.md: expired and used links are retained as an audit trail, with no automatic purge).

## Business Rules

- XBR-18: Access links open only that one client's bookings with that one Pro; on-demand links are single-use for 30 minutes; booking-specific links stop working once the appointment passes; a Pro reschedule issues a fresh manage link.
- Expiry and single-use enforcement follow FEAT-06.SPEC-007 exactly -- this automation applies those rules, it does not redefine them.
- Phone-to-Client matching and the privacy isolation boundary follow FEAT-06.SPEC-008 -- a link can never resolve to a Client or Booking outside its own stated scope.
- "First tap wins": for a single-use on-demand link, the tap that reaches Step 8 first is the one that succeeds; every subsequent tap of the same token is Already Used, regardless of which device made it.
- The re-request message is identical across Invalid Token, Already Used, Expired, and Scope Resolution Failure -- the client is never told which of these occurred, per the "no hint" empty/error pattern shared with FEAT-06.SPEC-001.

## Edge Cases

- **A single-use link is opened on a second device after first use** -- The first device's tap reaches Step 8 first and succeeds; the second device's tap finds the link already Used and is routed to the identical re-request prompt. First use wins (dependency map, Access Link Contention).
- **Client taps a booking-specific link after the appointment has passed** -- The expiry check at Step 6 finds the appointment start time already past; the link is marked Expired and the client sees the same re-request pattern as any other invalid link.
- **Pro reschedules a booking while the client is mid-tap on the old manage link** -- If the old link's tap reaches Step 8 before the reschedule replaces it, the redemption is honored for the booking's prior time (the client is not left in an inconsistent state); if the reschedule commits first, the old link is superseded by the fresh one FEAT-30 issues, and the old link's tap resolves as Expired via its now-stale scope (Scope Resolution Failure), so the client is not shown outdated appointment details.
- **Concurrent trigger firing (the same token tapped from two devices at effectively the same time)** -- Only one tap can reach Step 8 first; that tap alone transitions the link to Used. The other tap, whichever arrives second, always evaluates the state as already Used (Already Used outcome) -- there is no scenario where both taps succeed.
- **Trigger fires while a previous run for the same token is still in flight** -- A second tap for the same token that arrives before the first run has finished writing the Used state is held until the first run completes, then evaluates the now-updated state and resolves to Already Used. Runs for different tokens proceed independently and never block one another.
- **Client's phone number was changed by the Pro (FEAT-13) between link issuance and tap** -- Per XBR-18, a Pro-initiated phone number change invalidates the client's existing access links; the tapped link resolves as Scope Resolution Failure and the client sees the standard re-request prompt.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-006 (Access Link Delivery) | Triggered by (inbound) | On-demand links delivered by this spec are what the client taps to fire this automation |
| FEAT-08.SPEC-012 (Text send and delivery) / FEAT-08.SPEC-013 (Email send and delivery) | Triggered by (inbound, external-event source) | These are the integration sources of the booking-specific manage links embedded in confirmations and reminders; a tap on such a link fires this automation |
| FEAT-30 (Pro Booking Management) | Triggered by (inbound) | A fresh booking-specific manage link issued after a Pro reschedule fires this automation when tapped |
| FEAT-06.SPEC-007 (Access Link Lifecycle & Scope Rules) | References (outbound) | Expiry and single-use rules applied at Steps 6-8 |
| FEAT-06.SPEC-008 (Client Identity & Privacy Isolation Rule) | References (outbound) | Scope resolution and the privacy isolation boundary applied at Step 9 |
| FEAT-06.SPEC-001 (Access Link Request) | Affects (outbound) | Every invalid outcome routes here with the re-request prompt |
| FEAT-06.SPEC-003 (My Bookings List) | Affects (outbound) | A valid on-demand redemption routes here |
| FEAT-06.SPEC-004 (Booking Detail via Manage Link) | Affects (outbound) | A valid booking-specific redemption routes here |

**Note:** This spec is a Booking reader (see Data Model above); it never writes a Booking.

## Analytics and Success Signals

- **access_link_used** (scope: on_demand / booking_specific) -- supports success-metrics.md: "Self-Service Access Success"
- **access_link_expired_unused** (scope: on_demand / booking_specific) -- supports success-metrics.md: "Self-Service Access Success"
- **access_link_redemption_failed** (reason: invalid_token / already_used / scope_resolution_failure) -- N/A -- no Stage 2 metric distinguishes these sub-reasons from a plain expiry; retained so a silent-seeming failure mode is still observable internally rather than invisible.

## Acceptance Criteria

**FEAT-06.SPEC-002-AC-01:** Given Riley taps a valid, unused, unexpired on-demand access link within 30 minutes of it being issued, when the automation validates it, then the link is marked Used and Riley is routed to her My Bookings List (FEAT-06.SPEC-003).

**FEAT-06.SPEC-002-AC-02:** Given Riley taps a valid, unused, unexpired booking-specific manage link from a reminder text, when the automation validates it, then the link is marked Used and Riley is routed straight into that booking's detail (FEAT-06.SPEC-004), with no code or password required.

**FEAT-06.SPEC-002-AC-03:** Given Riley taps a link whose token matches no Access Link record, when the automation looks it up, then Riley is routed to FEAT-06.SPEC-001 showing the "request a new link" prompt.

**FEAT-06.SPEC-002-AC-04:** Given Riley's on-demand link was already used from another device, when Riley taps the same link, then the automation finds it already Used and routes Riley to FEAT-06.SPEC-001 showing the identical "request a new link" prompt.

**FEAT-06.SPEC-002-AC-05:** Given more than 30 minutes have passed since Riley's on-demand link was issued, when Riley taps it, then the automation marks it Expired and routes her to FEAT-06.SPEC-001 showing the "request a new link" prompt.

**FEAT-06.SPEC-002-AC-06:** Given Riley's booking-specific manage link points to an appointment whose start time has already passed, when Riley taps it, then the automation marks it Expired and routes her to the "request a new link" prompt.

**FEAT-06.SPEC-002-AC-07:** Given Riley's link is otherwise valid but Talia changed Riley's phone number since the link was issued (FEAT-13, XBR-18), when Riley taps the link, then the automation resolves it as Scope Resolution Failure and routes her to the "request a new link" prompt.

**FEAT-06.SPEC-002-AC-08:** Given Riley's single-use on-demand link is tapped from two devices at effectively the same time, when the automation processes both taps, then exactly one succeeds (marks the link Used and routes to FEAT-06.SPEC-003) and the other resolves as already used, routing to FEAT-06.SPEC-001.

**FEAT-06.SPEC-002-AC-09:** Given a second tap of the same token arrives while the first tap's redemption is still being processed, when the automation handles the second tap, then it waits for the first run to complete and then evaluates the link as already Used.

**FEAT-06.SPEC-002-AC-10:** Given Talia reschedules Riley's booking after issuing a manage link but before Riley taps the old one, when Riley taps the old link, then it resolves as Scope Resolution Failure and Riley is routed to the "request a new link" prompt rather than seeing outdated appointment details.

**FEAT-06.SPEC-002-AC-11:** Given Riley taps a booking-specific link issued fresh by FEAT-30 after a Pro reschedule, when the automation validates it, then it is treated exactly like any other valid booking-specific link and routes Riley to FEAT-06.SPEC-004 for the rescheduled appointment.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 | 3 |
| Outcome Paths | 6 | 6 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-06.SPEC-007
spec_name: Access Link Lifecycle & Scope Rules
spec_slug: access-link-lifecycle-scope-rules
parent_feature: FEAT-06
parent_feature_name: Client Booking Identity
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 17
acceptance_criteria_count: 13
---

# Logic/Rule Spec: Access Link Lifecycle & Scope Rules

## Overview

**Name:** Access Link Lifecycle & Scope Rules
**ID:** FEAT-06.SPEC-007
**Type:** Logic/Rule
**Purpose:** Governs the Access Link entity's expiry (30-minute single-use vs. booking-specific until the appointment passes), single-use enforcement, scope, and the per-phone-number request rate limit.
**Parent Feature:** FEAT-06 -- Client Booking Identity
**Governed Entity:** Access Link

## Scope and Non-Goals

**In Scope:**
- Expiry rules for on-demand and booking-specific Access Links
- Single-use enforcement ("first tap wins")
- Scope definition (all bookings with one Pro, vs. one booking)
- The per-phone-number request rate limit enforced at issuance
- Authorization over who may create and redeem an Access Link

**Non-Goals:**
- Phone-to-Client matching and the cross-client/cross-Pro privacy boundary -- owned by FEAT-06.SPEC-008 (Client Identity & Privacy Isolation Rule); this spec governs the link's own lifecycle, not identity matching
- The actual redemption processing steps (lookup, state write, routing) -- owned by FEAT-06.SPEC-002 (Access Link Validation & Redemption), which applies these rules
- The content and channel of the delivered message -- owned by FEAT-06.SPEC-006 (Access Link Delivery)
- Automatic purge or deletion of expired or used links -- explicitly recorded as a non-goal in feature-overview.md's Non-Goals: Stage 2 states only that links "expire automatically," with no purge window specified; links are retained as an audit trail with no automatic deletion, consistent with scope-boundaries SC-22

## Governed Entity

**Entity:** Access Link
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| scope | enum | All of this client's bookings with one Pro (on-demand), or one specific Booking (booking-specific) |
| expiry | derived | The timestamp after which the link can no longer be redeemed |
| state | enum | Issued \| Used \| Expired |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-06.SPEC-001 | Access Link Request | Rate-limit check before creating a new on-demand link; sets scope and initial expiry at creation |
| FEAT-06.SPEC-002 | Access Link Validation & Redemption | Expiry check, single-use check, and scope resolution at the moment of redemption; writes the resulting Used or Expired state |

## Field Validation Rules

{Access Link has no client-facing input form -- its fields are entirely system-derived at creation and system-managed thereafter. Every field is addressed below.}

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| scope | No validation beyond data type -- system-set at creation to exactly one of the two defined values, never user-entered | Always | -- | -- | -- |
| expiry | Must be computed exactly per the Defaults and Derivations below; never a value the client can set or extend | Always | On creation (set) and on redemption (checked) | -- (this is a system rule, not a user-facing validation) | Yes (redemption is blocked once expiry is reached) |
| state | Must follow the transition path Issued -> Used or Issued -> Expired only; no other transition is valid, and no role can set it directly | Always | On redemption | -- (this is a system rule, not a user-facing validation) | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Expiry depends on scope | scope, expiry | An on-demand link's expiry is 30 minutes from creation; a booking-specific link's expiry is the referenced Booking's appointment start time | N/A -- enforced at creation, not user-facing |
| State transition is single-use | state, scope | A link already in state Used can never be redeemed again regardless of scope; the second attempt always resolves as already used (FEAT-06.SPEC-002) | N/A -- surfaced to the client as the shared "request a new link" prompt (FEAT-06.SPEC-001), not a specific error text |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Request (create) an on-demand Access Link | The Client (Riley) | Their phone number matches a Client record for this Pro (FEAT-06.SPEC-008), and the per-phone-number request rate limit has not been exceeded | Rate limit exceeded: FEAT-06.SPEC-001 shows a plain message that no more links can be requested for this number right now; no new link is created |
| Request (create) an on-demand Access Link | The Pro (Talia) | Never -- the Pro does not use this mechanism | This action is not shown or reachable from the Pro's own dashboard; the Pro has no path to create a client-facing Access Link |
| Request (create) an on-demand Access Link | Platform Operator (Support) | Never | Support access never uses or bypasses client identity (scope-boundaries SC-05); no route to this action exists for Support |
| Redeem (tap) an Access Link | The Client (Riley) | The link is Issued, unexpired, and scoped to a Client/Booking this same client identity resolves to | Denied: FEAT-06.SPEC-002 routes to the shared "request a new link" prompt on FEAT-06.SPEC-001, never revealing the specific reason |
| Redeem (tap) an Access Link | The Pro (Talia) | Never -- the Pro reaches bookings only through Pro sign-in (FEAT-29), never through a client's Access Link | A Pro who somehow taps a client's link is treated identically to any other client viewer; the link resolves against whatever Client/Booking it is scoped to, never against the Pro's own account |
| Redeem (tap) an Access Link | Platform Operator (Support) | Never -- support never uses or bypasses a client's access link, even for troubleshooting (scope-boundaries SC-05) | Support has no operational path that redeems a client's link; if a link were ever tapped from a support context it would be treated the same as any other viewer, never as a support bypass |
| Directly set an Access Link's state (Used or Expired) | No role -- state transitions are automation-driven only (FEAT-06.SPEC-002) | Always denied to every role | There is no control, for any role, that writes this field directly; all transitions happen inside FEAT-06.SPEC-002's processing |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| state | Issued | On creation | No |
| expiry (on-demand scope) | Creation timestamp + 30 minutes | On creation | No |
| expiry (booking-specific scope) | The referenced Booking's appointment start_time | On creation (by FEAT-08 or FEAT-30, which create booking-specific links) | No |
| scope | "All bookings with this Pro" for links created by FEAT-06.SPEC-001; "one Booking" for links created by FEAT-08 or FEAT-30 | On creation | No |

## Business Rules

- XBR-18: Client identity is a phone number with one Pro only; access links open only that client's bookings with that Pro; on-demand links are single-use for 30 minutes; booking-specific links stop working once the appointment passes; a Pro reschedule issues a fresh manage link.
- Per-phone-number request rate limit: at most 5 on-demand Access Link requests per phone number within any rolling 60-minute window, enforced by FEAT-06.SPEC-001 before a new link is created. This limit exists to prevent message flooding (product-features.md, Validation & Limits) and applies regardless of whether prior requests succeeded, failed to match, or were themselves rate-limited.
- The rate limit counts requests, not delivered messages -- a request that finds no matching Client record (FEAT-06.SPEC-008) still counts toward the limit, so the limit cannot be used to probe which phone numbers exist.
- A link's expiry, once set at creation, is never extended or renewed by a later action (including a delivery retry, per FEAT-06.SPEC-006) -- only a brand-new request creates a brand-new expiry.
- Expired and Used links are retained indefinitely as an audit trail of issuance and use; scope-boundaries SC-22 governs history retention, and no purge policy exists for this entity.

## Edge Cases

- **Rate limit counted exactly at the boundary (the 5th request within the window)** -- The 5th request within the rolling 60-minute window succeeds normally; the 6th request within that same window is rejected with the rate-limited message.
- **Rate limit window rolls forward mid-use** -- The window is rolling, not fixed to the clock hour: a request made 61 minutes after an earlier one no longer counts that earlier request against the limit, even if requests in between still do.
- **On-demand link redeemed at exactly 30 minutes and 0 seconds after creation** -- Treated as expired; the 30-minute window is inclusive of everything strictly before the boundary, not at or after it.
- **Booking-specific link redeemed at the exact appointment start_time** -- Treated as expired; "until the appointment passes" means strictly before start_time, not at or after it.
- **A booking-specific link's Booking is rescheduled to a new time before the link is tapped** -- Per XBR-18, the reschedule issues a fresh manage link with a new expiry tied to the new start_time; the old link's own expiry is not recalculated -- it is instead superseded, and FEAT-06.SPEC-002 resolves a tap on the old link as a scope resolution failure rather than checking it against the new time.
- **Client requests a new on-demand link while a previously issued one is still Issued and unexpired** -- Both links independently follow this spec's rules; the earlier link remains redeemable until it is used or reaches its own 30-minute expiry, and creating the new one does not expire or invalidate the earlier one.

## Acceptance Criteria

**FEAT-06.SPEC-007-AC-01:** Given Riley requests her first on-demand access link this hour, when FEAT-06.SPEC-001 checks the rate limit, then the request is allowed and a new Access Link is created with state Issued.

**FEAT-06.SPEC-007-AC-02:** Given Riley has already made 5 access-link requests for her phone number within the current rolling 60-minute window, when she requests a 6th, then it is rejected with the rate-limited message and no new link is created.

**FEAT-06.SPEC-007-AC-03:** Given Riley's 6th request was rejected at minute 10 of the window, when 61 minutes have passed since her 1st request, then a new request from her no longer counts that 1st request against the limit.

**FEAT-06.SPEC-007-AC-04:** Given an on-demand Access Link was created at 2:00pm, when Riley taps it at 2:29pm, then it is treated as unexpired and redemption proceeds.

**FEAT-06.SPEC-007-AC-05:** Given an on-demand Access Link was created at 2:00pm, when Riley taps it at exactly 2:30pm or later, then it is treated as expired.

**FEAT-06.SPEC-007-AC-06:** Given a booking-specific Access Link scoped to an appointment starting at 4:00pm, when Riley taps it at 3:59pm, then it is treated as unexpired.

**FEAT-06.SPEC-007-AC-07:** Given a booking-specific Access Link scoped to an appointment starting at 4:00pm, when Riley taps it at exactly 4:00pm or later, then it is treated as expired.

**FEAT-06.SPEC-007-AC-08:** Given Talia reschedules the booking a booking-specific link points to, when FEAT-30 issues a fresh manage link for the new time, then the old link's redemption is resolved as a scope resolution failure rather than being checked against the new appointment time.

**FEAT-06.SPEC-007-AC-09:** Given a prior on-demand link Riley received is still Issued and unexpired, when she requests a new one, then the new link is created independently and the prior link remains separately redeemable until it is used or expires on its own.

**FEAT-06.SPEC-007-AC-10:** Given the Pro (Talia) has no path in her own dashboard to request or redeem a client's Access Link, when she looks for one, then no such control exists -- this mechanism is not the Pro's sign-in path.

**FEAT-06.SPEC-007-AC-11:** Given Platform Operator (Support) is assisting a Pro with a help request, when Support attempts to use or bypass a client's Access Link, then no operational path exists for doing so.

**FEAT-06.SPEC-007-AC-12:** Given no role or control can set an Access Link's state directly, when a link's state changes, then it always changes only as a result of FEAT-06.SPEC-002's redemption processing.

**FEAT-06.SPEC-007-AC-13:** Given Riley's request finds no matching Client record for this Pro, when the rate limit is evaluated for her next request within the same window, then the earlier, unmatched request still counts toward the limit.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 3 | 3 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 7 | 7 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

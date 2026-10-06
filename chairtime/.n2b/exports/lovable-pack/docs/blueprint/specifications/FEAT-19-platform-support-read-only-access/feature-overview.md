---
document_type: feature-overview
feature_number: FEAT-19
feature_name: Platform Support Read-Only Access
feature_slug: platform-support-read-only-access
priority_tier: Important
feature_type: Platform
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 4
screen_count: 2
automation_count: 1
logic_rule_count: 1
integration_count: 0
notification_count: 0
---

# Feature Breakdown Brief: Platform Support Read-Only Access

## Summary

**Feature:** Platform Support Read-Only Access
**ID:** FEAT-19
**Description:** The founder, in a support capacity, can open a read-only view into a specific Pro's account -- services, schedule, bookings, and billing status -- to help troubleshoot a reported problem, with no ability to edit anything and no client-facing access of any kind.
**Priority:** Important
**Phase:** MVP
**Type:** Platform
**Rationale:** BRIEF.md's Target Users & Roles names this directly: "the founder needs only a read-only support view of a pro's account to help them... It is minimal admin access." Ranked Important rather than Core because it serves the business's operational need rather than either product role's own value; still needed at MVP because support requests will arrive from day one with real, paying pros. [RESEARCH-INFORMED: slow, email-only customer support and unanswered payout questions are frequently mentioned complaints about three competitors, so a support view that lets the founder diagnose a problem without a screen-share is a trust asset, from BBB complaint records and aggregator reviews (MEDIUM confidence)]

**Key Capabilities:**
- Look up a specific Pro's account by request
- View their services, schedule, bookings, and billing status read-only
- View booking timelines (FEAT-16) to help resolve a dispute
- Every support view is recorded in the Pro's account activity, which the Pro can see [AUDIT-ADDED: 4 -- Audit Logging concern: access to a Pro's client data by anyone other than the Pro needed a "who looked, when" trail consistent with BRIEF.md's privacy posture]

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-19.SPEC-001 | Pro Account Lookup & Support Session Entry | Screen | Platform Operator (Support) | Support looks up one specific Pro by request, records a reason/ticket reference, and opens a single-account, structurally read-only session that hands into the already-validated read-only surfaces of other features for services, schedule, bookings, billing status, and disputed booking timelines |
| FEAT-19.SPEC-002 | Support View Logging | Automation | Platform Operator (Support), The Pro | On session open and on each disputed-booking timeline opened within it, assembles the support-view event (actor, time, reason/ticket reference) and hands it to FEAT-16.SPEC-002 to write as the Pro Account's Activity Event -- this feature never writes a second event store |
| FEAT-19.SPEC-003 | Support Access Log | Screen | The Pro, Platform Operator (Support) | The Pro's (and Support's own) account-level list of every past support view -- who, when, and the reason/ticket reference -- distinct from FEAT-16.SPEC-001's per-booking timeline |
| FEAT-19.SPEC-004 | Support Session Scope & Access Rules | Logic/Rule | Platform Operator (Support) | Governs the structural no-write-path rule, one-Pro-account-at-a-time scoping (opening a new lookup ends the prior session), the "only after a Pro's help request" precondition, and the field-level exclusions (private client notes, bank/identity details, sign-in codes) that ground XBR-24 |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Look up a specific Pro's account by request | FEAT-19.SPEC-001 | The lookup form and session-open action are the screen's primary purpose | Phase 2 (Explicit) |
| View their services, schedule, bookings, and billing status read-only | FEAT-19.SPEC-001 | The session hands into the already-validated read-only surfaces of other features (FEAT-01, FEAT-12, FEAT-13, FEAT-17, FEAT-18, FEAT-28) rather than duplicating mirror screens; this feature owns only the entry and scoping | Phase 2 (Explicit) |
| View booking timelines (FEAT-16) to help resolve a dispute | FEAT-19.SPEC-001, FEAT-19.SPEC-002 | The session hands into FEAT-16.SPEC-001's timeline (excluding private notes per FEAT-16.SPEC-005); each such view is itself a loggable support view | Phase 2 (Explicit) |
| Every support view is recorded in the Pro's account activity, which the Pro can see | FEAT-19.SPEC-002, FEAT-19.SPEC-003 | The automation constructs and hands off the event to FEAT-16.SPEC-002 for the actual write; the screen is where the Pro (and Support) read the resulting log | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 4-5:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-19.SPEC-002 | Support View Logging | Phase 4 (Trigger-Response) | The Key Capability only states that a view "is recorded" -- the mechanics of assembling the event on session open, and on each subsequent disputed-timeline view within the same session, are a cross-feature, hand-off automation (writing through FEAT-16.SPEC-002 per the Requirements Architect's coordination note) rather than a bare inline consequence of the lookup screen |
| FEAT-19.SPEC-004 | Support Session Scope & Access Rules | Phase 5 (Rule-Constraint Discovery) | The Access field states four interacting authorization conditions (product-wide View scope, one-account-at-a-time, only-after-a-help-request precondition, and three named field exclusions) plus the Validation & Limits field's structural "no write path exists at all" rule -- conditional, cross-spec rules referenced by SPEC-001, SPEC-002, and SPEC-003 alike, past the inline-validation threshold |

## Entity-Lifecycle Coverage Matrix

**Entity: Activity Event (support-view entries only; this feature is a second creator, not the sole owner)**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-19.SPEC-002 (triggers) / FEAT-16.SPEC-002 (writes) | This feature assembles the support-view event (actor, time, reason/ticket reference, and, when applicable, the booking timeline viewed) and hands it to FEAT-16.SPEC-002, the validated single append-only writer, per the Requirements Architect's coordination note; FEAT-19 never defines a second event store | A support-view entry belongs to the Pro Account, not to a Booking, per the Activity Event Relationships line in the dependency map -- even a view of a specific booking's timeline logs against the Pro Account |
| Read (single) | N/A | A support-view entry is never opened individually; it is always read as one row in the account-level log | Consistent with FEAT-16.SPEC-005's treatment of Activity Event as read-as-a-list, never single-record |
| Read (list) | FEAT-19.SPEC-003 | Renders the full, time-ordered list of support-view entries for one Pro Account | This is the account-level "who looked, when" log; FEAT-16.SPEC-001 remains the separate per-booking timeline |
| Update | N/A | No spec ever updates an Activity Event -- enforced by FEAT-16.SPEC-005 (append-only, immutable once written); this feature's writes are a one-time creation only | Same immutability guarantee that makes the log trustworthy to the Pro |
| Delete/Archive | N/A | Governed entirely by FEAT-16.SPEC-002/SPEC-005's lifecycle for Activity Event (retained for the life of the account; de-identified only on client deletion, which does not apply to a Pro-Account-level support-view entry); this feature introduces no separate retention or purge policy | Recorded as an explicit non-goal below rather than a silent omission |
| State Transition | N/A | A support-view entry carries no state beyond its fixed fields at creation -- a point-in-time fact, consistent with FEAT-16's treatment of every Activity Event | -- |

**Referenced Entities (read-only, via the reused screens this feature hands into -- FEAT-19 itself neither creates, updates, nor deletes any of these):**

| Entity | Read By | Context |
|--------|---------|---------|
| Pro Account | FEAT-19.SPEC-001 (session subject); via hand-off, FEAT-15.SPEC-004 (setup-progress state) | Support looks up the account by request; status (active/paused/closing) is read but never edited (SC-05) |
| Service | Via hand-off, FEAT-01.SPEC-003 (Edit Service, Support views read-only) | Support reviews a Pro's services and pricing while troubleshooting |
| Booking | Via hand-off, FEAT-12.SPEC-001/002/003 (schedule, attention list, past bookings), FEAT-16.SPEC-001 (booking timeline) | Support views schedule and booking history, and a booking's full activity timeline for a dispute |
| Client | Via hand-off, FEAT-13 (all specs; private note excluded by FEAT-13.SPEC-005) | Support may need client context for a booking dispute, excluding the Pro's private note |
| Subscription | Via hand-off, FEAT-18.SPEC-002 (Billing & Subscription Management Screen) | Support answers billing status questions |
| Payout Account | Via hand-off, FEAT-28.SPEC-002 (Payout Status & Money Dashboard, status and money list only) | Support checks payout status without ever seeing bank or identity details |
| Message | Via hand-off, FEAT-08's message and notification screens ("Support listed" per the dependency map's Roles Touched) | Support reviews message delivery history relevant to a reported problem |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Support submits a lookup for a specific Pro with a reason/ticket reference | Open a scoped, read-only support session for that one Pro Account; end any prior session first | Inline in triggering screen, governed by | FEAT-19.SPEC-001, FEAT-19.SPEC-004 |
| Support session opens | Assemble and hand off a support_view_opened event | Standalone Automation | FEAT-19.SPEC-002 |
| Support opens a disputed booking's timeline within an open session | Assemble and hand off a second support-view event for that view | Standalone Automation | FEAT-19.SPEC-002 |
| Support-view event is assembled | Written as an immutable Activity Event | Cross-feature -- owned by FEAT-16.SPEC-002 | FEAT-16.SPEC-002 responsibility |
| Pro (or Support) opens the account's support access log | Render the time-ordered list of past support views; fire support_view_log_viewed_by_pro when the Pro is the viewer | Inline in triggering screen | FEAT-19.SPEC-003 |
| Support attempts any write action inside the session | Refused outright -- no edit control exists anywhere in this view by design | Standalone Logic/Rule | FEAT-19.SPEC-004 |
| Support looks up a second Pro while a session is already open | The prior session ends before the new one opens -- never two concurrent sessions | Standalone Logic/Rule | FEAT-19.SPEC-004 |
| Support's session reaches a screen that would show a Pro's private client notes, bank/identity details, or sign-in codes | Those fields are excluded from the read-only rendering | Standalone Logic/Rule, enforced jointly with the owning feature's own visibility rule (e.g., FEAT-13.SPEC-005, FEAT-16.SPEC-005, FEAT-28.SPEC-002) | FEAT-19.SPEC-004 |
| Support submits a lookup that matches no Pro (mistyped or closed account) | Show a plain no-match message; no session opens | Inline in triggering screen | FEAT-19.SPEC-001 |

## Shared Context

**Shared Entities:**
- Activity Event (support-view entries) -- assembled by SPEC-002, written by FEAT-16.SPEC-002 (the single append-only writer this feature never duplicates), read as a list by SPEC-003. Fields: actor (support), time, reason/ticket reference, and, when applicable, which booking's timeline was viewed. Always belongs to the Pro Account, never to a Booking.
- Pro Account -- the subject of every session; looked up by SPEC-001, referenced by every support-view event's "belongs to" relationship, read (status only) via hand-off to FEAT-15.SPEC-004.

**Shared UI Patterns:**
- Read-only session hand-off -- SPEC-001 is the single entry point into every other feature's already-validated Support-facing read-only rendering (FEAT-01.SPEC-003, FEAT-12.SPEC-001/002/003/008, FEAT-13, FEAT-15.SPEC-004, FEAT-16.SPEC-001, FEAT-17.SPEC-002, FEAT-18.SPEC-002, FEAT-28.SPEC-002, FEAT-05.SPEC-001-005, FEAT-08's message screens); this feature does not re-specify those screens' read-only rendering, only the mechanics of entering and scoping the session. Spec Writers for those other features' screens remain the source of truth for how Support's masked view looks; this feature's Spec Writer should describe only the lookup, hand-off, and session-boundary behavior.

**Shared Validation:**
- SPEC-004 defines the structural read-only rule, one-account-at-a-time scoping, the help-request precondition, and the three named field exclusions once. SPEC-001 (session entry), SPEC-002 (event assembly), and SPEC-003 (log rendering) all reference it rather than restating the rules.

**Flagged discrepancy (not resolved, per the Requirements Architect's coordination note 6):** Stage 2's Interactions field names only FEAT-15, FEAT-12, FEAT-16, FEAT-18, while Connected Entities and the Access Matrix also require reading Client (FEAT-13), Payout Account (FEAT-28), Service (FEAT-01), and Message (FEAT-08). This Brief treats all of those entity reads as in scope (reflected in the Referenced Entities table and Cross-Feature Touchpoints below) and flags the Interactions field's narrower wording as a Stage 2 gap for the Requirements Architect, rather than narrowing this feature's actual coverage to match it.

## Internal Dependency Map

```
FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) -> [Support submits a reason/ticket reference for a specific Pro] -> support session opens
FEAT-19.SPEC-001 -> [session opens] -> FEAT-19.SPEC-002 (Support View Logging)
FEAT-19.SPEC-001 -> [Support views services/schedule/bookings/billing] -> FEAT-01.SPEC-003, FEAT-12.SPEC-001/002/003, FEAT-13 (all specs), FEAT-15.SPEC-004, FEAT-17.SPEC-002, FEAT-18.SPEC-002, FEAT-28.SPEC-002, FEAT-05.SPEC-001-005, FEAT-08 (message screens)
FEAT-19.SPEC-001 -> [Support opens a disputed booking's timeline] -> FEAT-16.SPEC-001 (Booking Activity Timeline) -> [each such view] -> FEAT-19.SPEC-002
FEAT-19.SPEC-002 -> [hands off the assembled event] -> FEAT-16.SPEC-002 (Activity Event Recording)
FEAT-19.SPEC-002 -> [event exists] -> FEAT-19.SPEC-003 (Support Access Log)
FEAT-19.SPEC-001 -> [governed by] -> FEAT-19.SPEC-004 (Support Session Scope & Access Rules)
FEAT-19.SPEC-002 -> [governed by] -> FEAT-19.SPEC-004
FEAT-19.SPEC-003 -> [governed by] -> FEAT-19.SPEC-004
```

**Default Entry:** FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) -- the only entry point into this feature, reached solely by Platform Operator (Support), never by a Pro or a Client, and never by navigation from any client- or Pro-facing screen.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-19.SPEC-001 | Outbound | FEAT-01 (Service & Pricing Management) | Hands into Edit Service (FEAT-01.SPEC-003) for Support's read-only view | Support views a Pro's services |
| FEAT-19.SPEC-001 | Outbound | FEAT-12 (Pro Daily Schedule Dashboard) | Hands into Today's & Upcoming Schedule, Attention List, and Past Bookings Browse, gated by FEAT-12.SPEC-008's masked read-only authorization | Support views a Pro's schedule and bookings |
| FEAT-19.SPEC-001 | Outbound | FEAT-13 (Client Record Management) | Hands into the Pro's client records, private note excluded by FEAT-13.SPEC-005 | Support needs client context for a reported issue |
| FEAT-19.SPEC-001 | Outbound | FEAT-15 (Pro Onboarding & Setup Wizard) | Reads the Pro Account's setup-progress state (FEAT-15.SPEC-004) read-only; Support can never act on a step | Support checks how far a Pro's setup has progressed |
| FEAT-19.SPEC-001 / FEAT-19.SPEC-002 | Outbound | FEAT-16 (Booking & Payment Activity Record) | Hands into the Booking Activity Timeline (FEAT-16.SPEC-001) to help resolve a dispute; every write of a support-view event goes through FEAT-16.SPEC-002, the sole Activity Event writer | Support opens a disputed booking's timeline |
| FEAT-19.SPEC-001 | Outbound | FEAT-17 (Manual Time Blocking) | Hands into Manage Time Blocks (FEAT-17.SPEC-002), view-only for Support | Support checks a Pro's blocked time |
| FEAT-19.SPEC-001 | Outbound | FEAT-18 (Pro Subscription Billing & Account Management) | Hands into the Billing & Subscription Management Screen (FEAT-18.SPEC-002) for billing status questions | Support answers a billing question |
| FEAT-19.SPEC-001 | Outbound | FEAT-28 (Payout Account Connection & Payout Visibility) | Hands into the Payout Status & Money Dashboard (FEAT-28.SPEC-002), status and money list only | Support answers a payout status question |
| FEAT-19.SPEC-001 | Outbound | FEAT-05 (Public Booking Page & Booking Flow) | Hands into the Pro's public booking-page screens in Support's read-only mode | Support reviews how a Pro's booking page appears |
| FEAT-19.SPEC-001 | Outbound | FEAT-08 (Automated Booking Messaging) | Hands into message and notification history in Support's read-only mode | Support reviews message delivery relevant to a reported issue |
| FEAT-19.SPEC-001 | Inbound | FEAT-27 (Pro Profile & Booking Page Settings) | The Pro-side "send a help request" action (owned by FEAT-27, analysed in this same batch) is the entry trigger into this feature; FEAT-27 owns no support-side view | A Pro sends a help request |
| FEAT-19.SPEC-001 | Inbound | FEAT-29 (Pro Sign-In & Account Lifecycle) | Reads Pro Account status (active/paused/closing), which FEAT-29 (analysed in this same batch) writes on closing/closed transitions -- Support never sees sign-in codes and can never sign in as the Pro (SC-05) | Support views account status during a session |
| FEAT-19.SPEC-002 | Outbound | FEAT-16 (Booking & Payment Activity Record) | Every assembled support-view event is written through FEAT-16.SPEC-002, the single append-only writer named by the dependency map's Lifecycle line for Activity Event | Support session opens, or a disputed timeline is viewed within it |

## Non-Functional Notes

**Data volumes / growth:** This feature's own dataset is small and bounded -- one Pro's account looked up at a time, and the support-view log accumulates only as many entries as there are actual help requests, not per ordinary use (Stage 2 Data Notes: "small per-account dataset"); it never approaches the volume of the full booking or activity history it reads read-only.

**Responsiveness:** The feature's States field states this view "loads instantly" given its small per-account dataset -- no dedicated loading state beyond an instantaneous render is expected for the lookup, the hand-off screens, or the support access log.

**Data sensitivity / privacy:** This feature exists specifically to bound access to already-sensitive data: it grants read-only, one-account-at-a-time visibility into personal and financial records (ASMP-23) while structurally excluding the Pro's private client notes, bank or identity details, and sign-in codes (ASMP-30, Access field). The Support Access Log itself is visible to the Pro so that the Pro can always see who looked and when -- the trust mechanism the audit-added Key Capability introduced.

**Compliance flags:** N/A -- assumptions-constraints.md names no compliance regime (health, payment-card, or otherwise) specific to this feature beyond the general privacy posture (ASMP-23, ASMP-30) already carried in the Data Sensitivity line above; SC-11 (card data never stored or handled by the product) is inherited from the Payout Account and Subscription entities this feature reads, not created by this feature itself.

## Non-Goals

- **Support performing any write, edit, refund, or sign-in-as action from within this view** -- Excluded per SC-05: "support cannot edit, refund, sign in as a Pro, or change a Pro's sign-in details." Enforced structurally by FEAT-19.SPEC-004 with no override path anywhere in this feature; the Validation & Limits field states the constraint is structural, not a permission check that could be bypassed.
- **Any administrative, manager, or staff role beyond the founder's own read-only support access** -- Excluded per SC-02 and SC-01: user-persona.md confirms exactly two product roles plus one narrow support function, with no multi-staff or salon-scale role ever introduced.
- **Cross-pro or cross-client visibility, or any comparative view across accounts** -- Excluded per SC-03: this feature is scoped to one Pro Account at a time (FEAT-19.SPEC-004's one-account-at-a-time rule); no shared or side-by-side view across accounts exists anywhere in this feature.
- **Support viewing a Pro's private client notes, bank/identity details, or sign-in codes** -- Excluded per the Access field and ASMP-23/ASMP-30; enforced by FEAT-19.SPEC-004 jointly with the owning feature's own visibility rule at each hand-off point (e.g., FEAT-13.SPEC-005, FEAT-16.SPEC-005, FEAT-28.SPEC-002).
- **A standalone Notification spec for support-view activity** -- Excluded per this feature's own Communications field ("N/A -- this is an internal tool with no client- or pro-facing messages of its own") and the Requirements Architect's coordination note 7; the Pro learns of support views through the Support Access Log screen (FEAT-19.SPEC-003), never through a pushed message.
- **A standalone Integration spec for this feature** -- Excluded because the context package's Dependencies slice names no category-level external capability this feature itself relies on ("None directly"); every entity it touches is read through another feature's already-specified surface.
- **Chairtime resolving the underlying pro-client dispute** -- Excluded per SC-17: this feature only lets Support view the same trustworthy record (FEAT-16) the Pro already has; who is right is never decided by the product or by Support.
- **Loading, empty, error, or offline states for the read-only view itself, beyond the plain no-match message on lookup** -- Per Stage 2's own States field, every state is explicitly N/A for this view ("this view only exists once a specific Pro Account is looked up... small per-account dataset... no write path to fail... used only in a connected context"); the one negative state this feature does introduce -- a mistyped or closed-account lookup finding no match -- stays inline in FEAT-19.SPEC-001 rather than becoming a standalone spec, since it is a single-step consequence with no further failure modes.

# FEAT-19 — Platform Support Read-Only Access

This chapter covers Platform Support Read-Only Access (FEAT-19), a Important-tier feature. It carries 4 specifications carrying 76 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-19.SPEC-001 | Pro Account Lookup & Support Session Entry | screen | 20 |
| FEAT-19.SPEC-002 | Support View Logging | automation | 15 |
| FEAT-19.SPEC-003 | Support Access Log | screen | 14 |
| FEAT-19.SPEC-004 | Support Session Scope & Access Rules | logic-rule | 27 |

The feature breakdown brief follows, then every specification in full.


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



# Screen Spec: Pro Account Lookup & Support Session Entry

## Overview

**Name:** Pro Account Lookup & Support Session Entry
**ID:** FEAT-19.SPEC-001
**Type:** Screen
**Purpose:** Support looks up one specific Pro by request, records a reason or ticket reference, and opens a single-account, structurally read-only session that hands into the already-validated read-only surfaces of other features for services, schedule, bookings, billing status, and disputed booking timelines.
**Parent Feature:** FEAT-19 -- Platform Support Read-Only Access

## Scope and Non-Goals

**In Scope:**
- The lookup form: entering a Pro identifier and a reason/ticket reference, and opening a scoped support session
- The no-match outcome when the lookup finds no Pro
- The session hub: once a session is open, the single set of navigation entries into every other feature's already-validated Support-facing read-only rendering (services, schedule/bookings, client records, billing, payouts, setup progress, time blocks, booking page preview, messages) and into the Support Access Log (FEAT-19.SPEC-003)
- Ending the current session (explicitly, or automatically when a new lookup is submitted)

**Non-Goals:**
- The detailed rendering of services, schedule, bookings, client records, billing, payouts, setup progress, time blocks, the booking page, or messages themselves -- each is owned by its own feature's Spec Writer (FEAT-01.SPEC-003, FEAT-12.SPEC-001/002/003/008, FEAT-13 (all specs), FEAT-15.SPEC-004, FEAT-16.SPEC-001, FEAT-17.SPEC-002, FEAT-18.SPEC-002, FEAT-28.SPEC-002, FEAT-05.SPEC-001-005, FEAT-08's message screens); this spec owns only the lookup, hand-off, and session-boundary behavior, per feature-overview.md's Shared UI Patterns.
- Recording the support-view event -- owned by FEAT-19.SPEC-002 (Support View Logging), which this screen's session-open and disputed-timeline-view actions trigger but do not themselves write.
- The structural no-write rule, one-account-at-a-time scoping, the help-request precondition, and the field-level exclusions -- owned by FEAT-19.SPEC-004 (Support Session Scope & Access Rules), which this screen enforces but does not restate.
- Any write, edit, refund, or sign-in-as action -- excluded per SC-05: no such control exists anywhere in this screen or any screen it hands into for Support.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| External -- Support's own initiative, prompted by a Pro's help request sent through FEAT-27 (Pro Profile & Booking Page Settings) | Support decides to look up an account after receiving a Pro's help request | None -- FEAT-27 owns no support-side view or direct link into this screen; Support opens this screen on their own and enters the lookup manually |
| This screen (re-entry) | Support submits a new lookup while a session is already open | The prior session's Pro Account reference is discarded as the new lookup replaces it (per FEAT-19.SPEC-004's one-account-at-a-time rule) |

This is the only entry point into FEAT-19 (feature-overview.md's Internal Dependency Map, Default Entry), reached solely by Platform Operator (Support), never by the Pro or a Client, and never by navigation from any client- or Pro-facing screen.

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Platform Operator (Support) | Full screen -- lookup form and, once a session is open, the session hub | Submit a lookup, open a hand-off screen, view the Support Access Log, end the session | -- |
| The Pro (Talia) | No | No | No navigation path anywhere in the product reaches this screen for the Pro; this screen exists outside the Pro's own navigation entirely |
| The Client (Riley) | No | No | No navigation path anywhere in the product reaches this screen for a Client; this screen is never linked from any client-facing surface |
| Unauthenticated | No | No | This screen requires an authenticated operator; an unauthenticated visitor sees a plain "this page isn't available" experience, since no client- or Pro-facing sign-in screen applies to it and no operator sign-in screen is itself part of this feature's scope |
| Expired session (operator's own access) | No | No | The screen (and any open support session) closes immediately; nothing is lost because no draftable input exists beyond the lookup form's two fields, which are discarded |

## Layout and Content

**Header:** Screen title "Support: Pro Account Lookup." Once a session is open, the title changes to "Support Session: {Pro Account display_name}" with an "End Session" action (right-aligned).

**Body -- Lookup state (default, no session open):**
- A single-column form with two fields, in order:
  - Pro Account Lookup (text input, required) -- accepts the Pro's booking_link_name, sign_in_email, sign_in_mobile, or display_name, whichever detail the help request included
  - Reason / Ticket Reference (text input, required) -- a short free-text note Support enters describing the help request
- "Open Support Session" action button, below the form
- If a session is already open when this state is reached again (Support returned to submit a fresh lookup), an informational banner above the form: "Opening a new lookup will end your current session with {display_name}."

**Body -- Session Active state (hub, once a lookup succeeds):**
- A Pro Account summary card at the top: display_name, status (Active | Paused | Closing | Closed), and the reason/ticket reference entered for this session
- Below the card, a list of navigation entries, each handing into that feature's already-validated Support-facing read-only rendering:
  - Services -- FEAT-01.SPEC-003
  - Schedule & Bookings -- FEAT-12.SPEC-001, FEAT-12.SPEC-002, FEAT-12.SPEC-003 (gated by FEAT-12.SPEC-008's masked read-only authorization)
  - Client Records -- FEAT-13 (all specs, private note excluded by FEAT-13.SPEC-005)
  - Setup Progress -- FEAT-15.SPEC-004
  - Time Blocks -- FEAT-17.SPEC-002
  - Billing & Subscription -- FEAT-18.SPEC-002
  - Payout Status -- FEAT-28.SPEC-002 (status and money list only)
  - Booking Page Preview -- FEAT-05.SPEC-001 through FEAT-05.SPEC-005
  - Messages -- FEAT-08's message and notification screens
  - Support Access Log -- FEAT-19.SPEC-003
- No edit control, save control, or action button of any kind appears anywhere on this list or the summary card, by design (SC-05)

**Body -- No Match state:** A single plain message in place of the form's result area: "No Pro account matches that lookup." The form's two fields remain filled with the values Support entered, and no session opens.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Single-column form (lookup state) or single-column summary card and stacked navigation list (session state), full width.
- **Medium size class and above:** Form and summary card remain single-column, capped at a consistent platform-wide content width and horizontally centered; the navigation list becomes a two-column grid of entries rather than a single stacked column, with no other structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Pro Account Lookup input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Pro Account Lookup input | Blur (empty) | Triggers field validation via FEAT-19.SPEC-004 | Error state on field | "A Pro account identifier is required." below field |
| Reason / Ticket Reference input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Reason / Ticket Reference input | Blur (empty or whitespace-only) | Triggers field validation via FEAT-19.SPEC-004 | Error state on field | "A reason or ticket reference is required to open a support session." below field |
| Open Support Session button | Tap | 1. Validate both fields via FEAT-19.SPEC-004. 2. If valid, resolve the Pro Account lookup. 3. If a match is found, end any prior session (FEAT-19.SPEC-004) and open a new one, triggering FEAT-19.SPEC-002 (support_view_opened). 4. If no match, show the No Match state. | Button shows loading state during resolution | Success: screen transitions to Session Active state. No match: plain message shown, form fields retained. |
| Open Support Session button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |
| Navigation entry (e.g., Services, Schedule & Bookings) | Tap | Navigate to the corresponding feature's Support-facing read-only screen, carrying the current session's Pro Account reference | Screen transitions to the destination spec | Standard navigation transition |
| Navigation entry -- disputed booking timeline (reached via Schedule & Bookings or Client Records) | Tap | Navigate to FEAT-16.SPEC-001 (Booking Activity Timeline); this view triggers FEAT-19.SPEC-002 a second time for this specific timeline view | Screen transitions to FEAT-16.SPEC-001 | Standard navigation transition |
| Support Access Log entry | Tap | Navigate to FEAT-19.SPEC-003, scoped to the current session's Pro Account | Screen transitions to FEAT-19.SPEC-003 | Standard navigation transition |
| End Session action | Tap | Ends the current session (FEAT-19.SPEC-004) | Screen returns to the Lookup state, fields empty | Screen transitions back to the default lookup form |

### Accessibility Notes

- **Focus order (Lookup state):** Pro Account Lookup input -> Reason/Ticket Reference input -> Open Support Session button.
- **Focus order (Session Active state):** End Session action -> Pro Account summary card -> navigation entries in the order listed -> Support Access Log entry.
- **Validation announcements:** When a field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Session transition announcement:** On a successful lookup, the transition to the session hub is announced ("Support session opened for {display_name}"); on a no-match result, the message "No Pro account matches that lookup" is announced.
- **Keyboard alternatives:** Every action on this screen (including all navigation entries and End Session) is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Lookup (default) | Empty two-field form, Open Support Session button enabled | Screen first opens, or a session was just ended | Support submits a lookup |
| Resolving | Open Support Session button shows a brief loading indicator | Support taps Open Support Session with both fields valid | Lookup resolves (match or no match) -- feature-overview.md's Non-Functional Notes state this view "loads instantly" given its small per-account dataset, so this state is momentary |
| No Match | Plain message "No Pro account matches that lookup." shown below the retained form fields | Lookup resolves to zero matching Pro Accounts | Support edits the lookup field and resubmits |
| Session Active | Pro Account summary card and navigation list rendered | Lookup resolves to exactly one Pro Account | Support taps End Session, or submits a new lookup (which ends this session automatically) |
| Offline/Degraded | N/A -- feature-overview.md's Non-Goals state every state beyond the plain no-match message is explicitly N/A for this feature; it is "used only in a connected context" with no offline exposure defined anywhere in its Stage 2 source | -- | -- |

## Validation Rules

Validation governed by FEAT-19.SPEC-004 (Support Session Scope & Access Rules). See that spec for the Pro Account Lookup and Reason/Ticket Reference field rules, and for the one-account-at-a-time and help-request-precondition rules this screen enforces on open.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Services entry tap | FEAT-01.SPEC-003 (Edit Service, Support read-only) | FEAT-01 |
| Schedule & Bookings entry tap | FEAT-12.SPEC-001 / FEAT-12.SPEC-002 / FEAT-12.SPEC-003 (gated by FEAT-12.SPEC-008) | FEAT-12 |
| Disputed booking timeline tap | FEAT-16.SPEC-001 (Booking Activity Timeline) | FEAT-16 |
| Client Records entry tap | FEAT-13 (all specs, private note excluded by FEAT-13.SPEC-005) | FEAT-13 |
| Setup Progress entry tap | FEAT-15.SPEC-004 | FEAT-15 |
| Time Blocks entry tap | FEAT-17.SPEC-002 (Manage Time Blocks, view-only) | FEAT-17 |
| Billing & Subscription entry tap | FEAT-18.SPEC-002 (Billing & Subscription Management Screen) | FEAT-18 |
| Payout Status entry tap | FEAT-28.SPEC-002 (Payout Status & Money Dashboard) | FEAT-28 |
| Booking Page Preview entry tap | FEAT-05.SPEC-001 through FEAT-05.SPEC-005 | FEAT-05 |
| Messages entry tap | FEAT-08's message and notification screens | FEAT-08 |
| Support Access Log entry tap | FEAT-19.SPEC-003 (Support Access Log) | -- |
| End Session tap | This screen, Lookup state | -- |

## Data Model

**Creates:** None -- a support session is an ephemeral scoping context (per FEAT-19.SPEC-004's Governed Entity), not a persisted entity in the Domain Entity Inventory; it exists only as this screen's own state for the duration of the visit.
**Reads:** Pro Account -- display_name and status, to resolve the lookup and populate the session hub's summary card; the reference is carried into every hand-off spec listed in Navigation Out, each of which reads its own additional fields under its own Access rules.
**Updates:** None -- this screen never writes to the Pro Account or any entity it hands into (SC-05, structural read-only).
**Deletes:** None.

## Business Rules

- XBR-24: Support access is read-only, one account at a time, used only after a Pro's help request, never shows private client notes, bank or identity details or sign-in codes, and every view is logged in the Pro's visible account activity.
- FEAT-19.SPEC-004 governs the structural no-write rule, one-account-at-a-time scoping (opening a new lookup ends the prior session before the new one opens), and the help-request precondition -- this screen enforces those rules but does not restate them.
- FEAT-19.SPEC-002 is triggered on every session open and on every disputed-timeline view reached from this session -- this screen's own success path never waits for that logging to complete (consistent with FEAT-19.SPEC-002's non-blocking design).
- No edit, save, refund, or sign-in-as control exists anywhere in this screen's session hub or in any screen it hands into for Support (SC-05) -- this is a structural absence of capability, not a permission check.

## Edge Cases

- **Support submits a lookup that matches no Pro (mistyped identifier or a closed account)** -- Shows the plain "No Pro account matches that lookup." message; no session opens; a Closed Pro Account (per XBR-20) is treated identically to a non-existent one, since a closed account no longer has an active surface for Support to view.
- **Support looks up a second Pro while a session is already open** -- The prior session ends automatically before the new one opens (FEAT-19.SPEC-004); Support never sees two sessions at once, and the informational banner in the Lookup state's Layout warns of this before submission.
- **Support taps Open Support Session twice rapidly** -- Second tap is ignored while the first resolution is in progress (button in loading state).
- **The looked-up Pro's account status changes while a session is open (e.g., FEAT-29 transitions it to Paused or Closing)** -- The summary card reflects the current status the next time the session hub is opened or refreshed; this screen is a snapshot per view, not live-updating, and since Support never writes to the account, no conflict exists to resolve.
- **Support navigates directly to a hand-off screen's own address without an active session** -- Refused by that screen's own authorization gate (e.g., FEAT-12.SPEC-008), consistent with XBR-24's "used only after" condition; this spec's own Access and Visibility governs only the lookup and hub, and relies on each hand-off spec's own Access rules for its own surface.
- **Support's operator access itself expires while a session is open** -- The session ends immediately per the Access and Visibility table's Expired session row; nothing is lost because this screen holds no unsaved input beyond the two lookup fields, which are simply discarded.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-19.SPEC-002 (Support View Logging) | Triggers (outbound) | Session open, and each disputed-timeline view reached from this session, trigger the logging automation |
| FEAT-19.SPEC-003 (Support Access Log) | Navigation (outbound) | The session hub's log entry navigates here, scoped to the current session's Pro Account |
| FEAT-19.SPEC-004 (Support Session Scope & Access Rules) | References (inbound) | Governs the lookup form's field validation, the one-account-at-a-time rule, and the structural no-write rule this screen enforces |
| FEAT-01.SPEC-003, FEAT-12.SPEC-001/002/003/008, FEAT-13 (all specs), FEAT-15.SPEC-004, FEAT-16.SPEC-001, FEAT-17.SPEC-002, FEAT-18.SPEC-002, FEAT-28.SPEC-002, FEAT-05.SPEC-001-005, FEAT-08's message screens | Navigation (outbound) | Every read-only surface this session hands into; each owns its own rendering and Access rules |
| FEAT-27 (Pro Profile & Booking Page Settings) | References (inbound, external trigger) | The Pro's "send a help request" action is the real-world prompt for Support to open this screen; FEAT-27 carries no direct link into it |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| support_lookup_submitted | reason/ticket reference present (boolean) | Support taps Open Support Session with both fields valid | N/A -- no success-metrics.md metric is connected to FEAT-19; retained for operational observability of how often Support opens the tool relative to help-request volume |
| support_lookup_no_match | -- | The lookup resolves to zero matching Pro Accounts | N/A -- no connected success-metrics.md metric; retained to observe how often Support mistypes or looks up a closed account, informing whether the lookup field needs clearer guidance |
| support_session_opened | Pro Account reference | A lookup resolves to exactly one Pro Account and the session hub renders | N/A -- no connected success-metrics.md metric; retained as the operational counterpart to FEAT-19.SPEC-002's logging, confirming every opened session is also recorded |
| support_session_ended | duration, ended via (explicit End Session / superseded by new lookup / operator access expired) | The session hub is closed by any of the three listed causes | N/A -- no connected success-metrics.md metric; retained to observe typical support-session duration for operational tuning |

## Acceptance Criteria

**FEAT-19.SPEC-001-AC-01:** Given Support is on the lookup form, when they leave the Pro Account Lookup field empty and move to the next field, then the field shows an error state with "A Pro account identifier is required."

**FEAT-19.SPEC-001-AC-02:** Given Support is on the lookup form, when they leave the Reason/Ticket Reference field empty and move away, then the field shows an error state with "A reason or ticket reference is required to open a support session."

**FEAT-19.SPEC-001-AC-03:** Given Support enters a valid Pro Account identifier and a reason, when they tap Open Support Session and the lookup matches exactly one Pro Account, then the screen transitions to the Session Active state showing that Pro's display_name and status.

**FEAT-19.SPEC-001-AC-04:** Given Support enters a mistyped identifier, when they tap Open Support Session and no Pro Account matches, then the plain message "No Pro account matches that lookup." appears and no session opens.

**FEAT-19.SPEC-001-AC-05:** Given Support enters the booking_link_name of a Pro Account whose status is Closed, when they submit the lookup, then the result is the No Match state, identical to a non-existent account.

**FEAT-19.SPEC-001-AC-06:** Given Support has an active session with one Pro, when they submit a fresh lookup for a different Pro, then the prior session ends automatically and the new session opens, without ever showing both at once.

**FEAT-19.SPEC-001-AC-07:** Given Support taps Open Support Session, when the tap registers a second time before resolution completes, then the second tap has no effect and the button remains in its loading state.

**FEAT-19.SPEC-001-AC-08:** Given Support is viewing the session hub, when they tap the Services entry, then they are navigated to FEAT-01.SPEC-003's Support-facing read-only rendering for that Pro's services.

**FEAT-19.SPEC-001-AC-09:** Given Support is viewing a disputed booking within the session hub's hand-off, when they open its timeline, then FEAT-19.SPEC-002 is triggered a second time to log that specific view.

**FEAT-19.SPEC-001-AC-10:** Given Support is viewing the session hub, when they tap the Support Access Log entry, then they are navigated to FEAT-19.SPEC-003 scoped to the current session's Pro Account.

**FEAT-19.SPEC-001-AC-11:** Given Support is viewing the session hub, when they look for any edit, save, refund, or sign-in-as control anywhere on the screen or its navigation targets, then none exists.

**FEAT-19.SPEC-001-AC-12:** Given Support taps End Session, when the action completes, then the screen returns to the empty Lookup state.

**FEAT-19.SPEC-001-AC-13:** Given Support's operator access expires while a session is open, when the expiry occurs, then the session ends immediately with no data loss, since no unsaved input exists beyond the discarded lookup fields.

**FEAT-19.SPEC-001-AC-14:** Given the Pro (Talia) or a Client (Riley) attempts to reach this screen, when the attempt is made through any product navigation, then no path exists anywhere in the product that leads them here.

**FEAT-19.SPEC-001-AC-15:** Given a looked-up Pro's account status changes to Paused while Support's session is open, when Support reopens or refreshes the session hub, then the summary card reflects the current status.

**FEAT-19.SPEC-001-AC-16:** Given Support submits a valid lookup, when the session opens, then FEAT-19.SPEC-002 is triggered to log the support_view_opened event.

**FEAT-19.SPEC-001-AC-17:** Given Support attempts to navigate directly to a hand-off screen's own address without an active session, when the attempt reaches that screen, then it is refused by that screen's own authorization gate (e.g., FEAT-12.SPEC-008).

**FEAT-19.SPEC-001-AC-18:** Given Support is on the lookup form with a session already open, when they view the form before submitting a new lookup, then the informational banner "Opening a new lookup will end your current session with {display_name}." is shown.

**FEAT-19.SPEC-001-AC-19:** Given Support is viewing the session hub, when they tap the Booking Page Preview entry, then they are navigated into the Pro's public booking-page screens (FEAT-05.SPEC-001 through FEAT-05.SPEC-005) in Support's read-only mode.

**FEAT-19.SPEC-001-AC-20:** Given Support is viewing the session hub, when they tap the Payout Status entry, then they are navigated to FEAT-28.SPEC-002 showing status and the money list only, never bank or identity details.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 10 | 10 |
| States | 5 (lookup, resolving, no match, session active, offline/degraded N/A) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Automation Spec: Support View Logging

## Overview

**Name:** Support View Logging
**ID:** FEAT-19.SPEC-002
**Type:** Automation
**Purpose:** On session open, and on each disputed-booking timeline opened within it, assembles the support-view event (actor, time, reason/ticket reference) and hands it to FEAT-16.SPEC-002 to write as the Pro Account's Activity Event -- this feature never writes a second event store.
**Parent Feature:** FEAT-19 -- Platform Support Read-Only Access

## Scope and Non-Goals

**In Scope:**
- Assembling the support-view event's data (actor, time, reason/ticket reference, and, when applicable, which booking's timeline was viewed) from each of its two triggers
- Handing the assembled event off to FEAT-16.SPEC-002, the sole Activity Event writer, for every trigger
- Firing once per qualifying trigger, with no deduplication across repeated views within the same session

**Non-Goals:**
- Writing the Activity Event record itself -- owned by FEAT-16.SPEC-002 (Activity Event Recording), the single append-only writer this feature hands off to and never duplicates, per feature-overview.md's Entity-Lifecycle Coverage Matrix.
- Rendering the resulting log -- owned by FEAT-19.SPEC-003 (Support Access Log), which reads what this automation's hand-offs eventually produce.
- Determining whether a session was opened for a legitimate reason -- excluded per scope-boundaries.md SC-05 and the Requirements Architect's own note that no Help Request entity exists in the Domain Entity Inventory for this automation to check against; this spec logs every session and timeline view that occurs, regardless of the reason's substance.
- Deciding when a session may open at all (the one-account-at-a-time and help-request-precondition rules) -- owned by FEAT-19.SPEC-004 (Support Session Scope & Access Rules); this automation assumes a session has already legitimately opened by the time it fires.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Support session opens | FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) | A lookup resolves to exactly one Pro Account and the session hub renders | Pro Account reference, reason/ticket reference, timestamp |
| Support opens a disputed booking's timeline within an open session | FEAT-16.SPEC-001 (Booking Activity Timeline), reached via FEAT-19.SPEC-001's hand-off | The timeline view opens while a support session is Active for that Pro Account | Pro Account reference, Booking reference, reason/ticket reference (carried from the session context), timestamp |

## Processing Logic

1. Receive the triggering event's data: the actor (a support view), a Pro Account reference, the reason/ticket reference carried by the current session, a timestamp, and (for the timeline trigger only) a Booking reference.
2. Determine the event_type for the entry: "support_view_opened" for the session-open trigger, or "support_view_booking_timeline" for the timeline-view trigger.
3. Assemble the details field: the reason/ticket reference always, and the Booking reference when the trigger is a timeline view.
4. Set the entry's actor to "a support view" (the fourth actor value in FEAT-16.SPEC-002's Processing Logic vocabulary, alongside Client, Pro, and the product automatically).
5. Hand off the assembled event -- event_type, time, actor, details, and the Pro Account it belongs to -- to FEAT-16.SPEC-002 for writing. This automation performs no write of its own; FEAT-16.SPEC-002 (the recording automation for support-view logging, which accepts this hand-off as an inbound trigger) is the entry's only writer.
6. Confirm the hand-off completed before returning control to the triggering spec; this automation's own completion never blocks or delays Support's view (consistent with FEAT-16.SPEC-002's non-blocking recording design).
7. If the disputed timeline is opened more than once within the same session, repeat steps 1--6 once per view -- no view is deduplicated or merged with a prior one, per feature-overview.md's Key Capability ("every support view is recorded").

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Session-open event handed off | A support session opens successfully | One Activity Event is written by FEAT-16.SPEC-002, belonging to the Pro Account | None directly -- the entry becomes visible the next time FEAT-19.SPEC-003 is opened, by the Pro or by Support | FEAT-19.SPEC-003, FEAT-16.SPEC-002 |
| Timeline-view event handed off | A disputed booking's timeline is opened within an open session | One Activity Event is written by FEAT-16.SPEC-002, belonging to the Pro Account (not the Booking, per feature-overview.md's Shared Entities note) | None directly | FEAT-19.SPEC-003, FEAT-16.SPEC-002 |
| Hand-off failure | FEAT-16.SPEC-002's own write cannot complete (e.g., the referenced Pro Account cannot be found) | No entry is created for this occurrence | Non-blocking -- Support's session or timeline view proceeds unaffected; the gap is retried automatically per FEAT-16.SPEC-002's own Write failure outcome | FEAT-19.SPEC-003 (shows a gap until the retry succeeds) |
| No-op (nothing to record) | N/A -- both triggers in the table above always correspond to a qualifying, recordable action; there is no trigger path in this automation that produces nothing worth logging | -- | -- | -- |

## Data Model

**Reads:** Pro Account (reference), Booking (reference, when the trigger is a timeline view) -- read only to identify what the assembled event belongs to and, where applicable, which timeline was viewed; never independently re-queried beyond what the trigger provides.
**Creates:** None directly -- this automation assembles and hands off event data; FEAT-16.SPEC-002 is the sole creator of the resulting Activity Event, per the Entity-Lifecycle Coverage Matrix.
**Updates:** None.
**Deletes:** None.

## Business Rules

- XBR-24: every view is logged in the Pro's visible account activity -- this automation is the mechanism that fulfills that obligation for FEAT-19.
- XBR-21: every booking, payment, messaging, and support-view event is written to an append-only, immutable activity record that no role can edit -- this automation hands off into that same mechanism rather than defining a second one.
- Every qualifying trigger produces exactly one hand-off; no batching, delay, or deduplication across repeated views in the same session (a booking's timeline opened three times in one session produces three separate entries).
- This automation never blocks or delays the triggering screen's own completion -- logging is a side effect that runs alongside, not a gate Support's view must pass through, consistent with FEAT-16.SPEC-002's own non-blocking rule.
- This automation assumes the session it is logging has already satisfied FEAT-19.SPEC-004's opening rules; it logs every session that reaches it, without re-validating those rules itself.

## Edge Cases

- **Support opens a session and immediately opens a disputed booking's timeline within it** -- Each trigger produces its own independent hand-off; the second write proceeds without waiting for or merging with the first, per FEAT-16.SPEC-002's own append-only concurrency handling.
- **Concurrent trigger firing (two views logged at effectively the same moment, e.g., session open and an immediate timeline view)** -- Each writes its own independent entry, ordered by its own recorded time; neither write depends on or is blocked by the other.
- **Trigger fires while a previous hand-off for the same session is still in flight** -- The second hand-off proceeds independently and additively; append-only writes never need to wait for, merge with, or overwrite an in-flight one.
- **Support opens the same disputed booking's timeline twice within one session** -- Two separate Activity Events are logged, with no deduplication; the Support Access Log (FEAT-19.SPEC-003) later shows both as distinct rows.
- **The hand-off to FEAT-16.SPEC-002 fails (e.g., the Pro Account reference cannot be resolved at that moment)** -- Per the Hand-off failure outcome, Support's own session or timeline view is unaffected, and the gap is retried automatically without blocking any part of the read-only experience.
- **A session opens and ends again within moments (Support immediately realizes the wrong Pro was looked up)** -- The support_view_opened event handed off at session open was already recorded and is never retracted, consistent with XBR-21's immutability guarantee, even though the session itself was very short.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) | Triggered by (inbound) | Session open fires this automation's first trigger |
| FEAT-16.SPEC-001 (Booking Activity Timeline) | Triggered by (inbound) | A disputed-timeline view reached from the open session fires this automation's second trigger |
| FEAT-16.SPEC-002 (Activity Event Recording) | Affects (outbound) | The automation that records support-view logging: FEAT-16.SPEC-002 lists FEAT-19.SPEC-002 as a writer source (actor "a support view") and receives every assembled event for the actual write; this automation is the source of that trigger |
| FEAT-19.SPEC-003 (Support Access Log) | Affects (outbound) | The entries this automation produces (via FEAT-16.SPEC-002) are what that screen renders |
| FEAT-16.SPEC-005 (Activity Record Immutability & Visibility Rules) | References (inbound) | Governs the append-only immutability of the entries this automation's hand-offs produce |
| FEAT-19.SPEC-004 (Support Session Scope & Access Rules) | References (inbound) | Governs the precondition for a session existing at all; this automation assumes that precondition was already satisfied |

## Analytics and Success Signals

- **support_view_logged** (event_type: support_view_opened / support_view_booking_timeline) -- N/A -- no success-metrics.md metric is connected to FEAT-19; retained to observe how reliably every support view is recorded, since XBR-24's trust guarantee to the Pro depends on this log's completeness.
- **support_view_log_handoff_failed** (trigger source, reason) -- N/A -- no connected success-metrics.md metric; retained to observe how often a hand-off gap occurs before FEAT-16.SPEC-002's automatic retry closes it, since an unrecorded support view would undermine the audit trail feature-overview.md's Key Capabilities promise to the Pro.

## Acceptance Criteria

**FEAT-19.SPEC-002-AC-01:** Given Support submits a valid lookup for Talia's account, when FEAT-19.SPEC-001 opens the session, then a "support_view_opened" event is assembled and handed off with the Pro Account reference, reason/ticket reference, and timestamp.

**FEAT-19.SPEC-002-AC-02:** Given Support's session for Talia's account is open, when Support opens a disputed booking's timeline, then a "support_view_booking_timeline" event is assembled and handed off with the Booking reference in addition to the session's reason/ticket reference.

**FEAT-19.SPEC-002-AC-03:** Given a support-view event is assembled, when it is handed off, then its actor field reads "a support view," never Client, Pro, or "the product automatically."

**FEAT-19.SPEC-002-AC-04:** Given a support-view event's hand-off completes, when FEAT-16.SPEC-002 writes it, then the resulting entry belongs to Talia's Pro Account, never to the specific Booking, even when the trigger was a timeline view of that booking.

**FEAT-19.SPEC-002-AC-05:** Given Support opens the same disputed booking's timeline twice within one session, when both views occur, then two separate Activity Events are logged, with neither view overwriting or merging into the other.

**FEAT-19.SPEC-002-AC-06:** Given Support opens a session and immediately opens a disputed timeline within it, when both triggers fire in close succession, then each produces its own independent hand-off, correctly ordered by its own recorded time.

**FEAT-19.SPEC-002-AC-07:** Given a hand-off to FEAT-16.SPEC-002 cannot complete because the Pro Account reference cannot be resolved, when the failure occurs, then Support's own session or timeline view proceeds unaffected, and the gap is retried automatically.

**FEAT-19.SPEC-002-AC-08:** Given a support session opens and is ended again within moments, when the session closes early, then the originally handed-off support_view_opened event is never retracted or removed.

**FEAT-19.SPEC-002-AC-09:** Given two support-view triggers fire for the same session at effectively the same time, when both automations run, then neither write waits for or is blocked by the other.

**FEAT-19.SPEC-002-AC-10:** Given a hand-off for one trigger is still in flight, when a second, unrelated trigger fires for the same session, then the second hand-off proceeds independently and does not queue behind the first.

**FEAT-19.SPEC-002-AC-11:** Given this automation completes its hand-off, when Talia later views her Support Access Log (FEAT-19.SPEC-003), then the entry appears with the actor, time, and reason/ticket reference this automation assembled.

**FEAT-19.SPEC-002-AC-12:** Given this automation's hand-off is in progress, when Support's own session-open or timeline-view action completes on the screen, then that screen's own completion is never blocked or delayed by whether this automation's hand-off has finished.

**FEAT-19.SPEC-002-AC-13:** Given a support session's reason/ticket reference is carried into a disputed-timeline-view trigger, when the second event is assembled, then it carries the same reason/ticket reference as the session's opening event.

**FEAT-19.SPEC-002-AC-14:** Given FEAT-19.SPEC-004's opening rules were satisfied before this automation's trigger fired, when this automation processes the trigger, then it logs the view without re-validating those opening rules itself.

**FEAT-19.SPEC-002-AC-15:** Given a hand-off failure occurs and is retried automatically, when the retry succeeds, then the resulting Activity Event carries the same event_type, actor, and details that were originally assembled, with only the completion timing delayed.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 | 2 |
| Outcome Paths | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Screen Spec: Support Access Log

## Overview

**Name:** Support Access Log
**ID:** FEAT-19.SPEC-003
**Type:** Screen
**Purpose:** Renders the Pro's (and Support's own) account-level list of every past support view -- who, when, and the reason/ticket reference -- distinct from FEAT-16.SPEC-001's per-booking timeline.
**Parent Feature:** FEAT-19 -- Platform Support Read-Only Access

## Scope and Non-Goals

**In Scope:**
- Rendering the full, time-ordered list of support-view entries for one Pro Account
- The Pro's own entry point into this screen from her account settings
- Support's entry point into this screen from within an open session, scoped to that same account
- Firing support_view_log_viewed_by_pro when the Pro is the viewer

**Non-Goals:**
- Writing the entries this screen displays -- owned by FEAT-19.SPEC-002 (Support View Logging), which assembles them, and FEAT-16.SPEC-002 (Activity Event Recording), which is their sole writer.
- The per-booking activity timeline -- owned by FEAT-16.SPEC-001 (Booking Activity Timeline); this screen is the separate account-level "who looked, when" log, not a booking's own history.
- Any edit, delete, or dismiss control on any entry -- excluded per FEAT-16.SPEC-005's append-only, immutable guarantee, which this entity inherits in full; no such control exists anywhere for any role, including the Pro.
- Retention or purge policy for these entries -- excluded per feature-overview.md's Entity-Lifecycle Coverage Matrix, which states this feature introduces no separate retention or purge policy beyond FEAT-16.SPEC-002/FEAT-16.SPEC-005's own lifecycle for Activity Event.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-27 (Pro Profile & Booking Page Settings) | The Pro navigates to "Support Access Log" from her own account settings | Her own Pro Account reference (implicit -- she can only ever view her own log) |
| FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) | Support taps the Support Access Log entry from within an open session | The current session's Pro Account reference, scoping the log to that one account only |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full list, for her own account only | None -- this is a read-only log with no action controls for any role | -- |
| Platform Operator (Support) | Full list, for the one Pro Account currently under an active session only | None | Attempting to view this log for any account other than the one under an active session is refused -- no path renders it, since this screen is only ever reached scoped to the current session's account |
| The Client (Riley) | No | No | No navigation path anywhere in the product reaches this screen for a Client; it is never linked from any client-facing surface |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen when arriving via the Pro's own settings path (XBR-29); a plain "this page isn't available" experience when arriving via the Support path, since no operator sign-in screen is in scope for this feature |
| Expired session | No | No | The Pro's expired session redirects to sign-in per XBR-29, with no unsaved input to preserve (this is a read-only screen); Support's expired operator access ends the entire support session per FEAT-19.SPEC-004, closing this screen along with it |

## Layout and Content

**Header:** Screen title "Support Access Log" with a back action -- returns the Pro to FEAT-27's account settings, or returns Support to FEAT-19.SPEC-001's session hub, depending on entry point.

**Body:** A single-column, time-ordered list of entries, most recent first. Each entry shows:
- The reviewer label -- always "Chairtime Support," since Platform Operator (Support) is this product's only reviewer role
- The date and time of the view
- The reason/ticket reference recorded for that view
- When the entry represents a disputed-booking-timeline view, a reference to which booking's timeline was viewed, alongside the entry's other details

No entry ever includes an edit, delete, or dismiss control (FEAT-16.SPEC-005).

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Single-column list, full width; each entry's date/time, reason/ticket reference, and (where applicable) booking reference stack vertically within the entry.
- **Medium size class and above:** The list remains single-column, capped at a consistent platform-wide content width and horizontally centered; each entry's date/time, reason/ticket reference, and booking reference lay out in a single row rather than stacking, with no other structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back action | Tap | Navigate to FEAT-27 (Pro entry) or FEAT-19.SPEC-001 (Support entry) | Screen closes | Standard navigation transition |
| Entry with a referenced booking | Tap | Navigate to FEAT-16.SPEC-001 (Booking Activity Timeline) for that booking | Screen transitions to FEAT-16.SPEC-001 | Standard navigation transition |
| Entry with no referenced booking (a plain session-open entry) | Tap | No action -- display-only | None | None |
| List (more entries than fit on screen) | Scroll | Loads further entries | Additional entries appear below | Standard scroll behavior |

### Accessibility Notes

- **Focus order:** Back action -> each list entry in displayed (most-recent-first) order.
- **Dynamic content announcements:** When the list finishes its initial load, the entry count is announced (e.g., "12 support views recorded"); when further entries load on scroll, no additional announcement interrupts reading.
- **Keyboard alternatives:** Scrolling and opening a referenced booking's timeline are both reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Populated (default) | Full time-ordered list of entries | The account has at least one recorded support view | -- (remains the resting state while entries exist) |
| Empty | Plain message "No support views recorded for this account yet." in place of the list | The account has zero recorded support views (a Pro who has never had a help request looked into) | An entry is recorded and the screen is next opened or refreshed |
| Loading | N/A -- feature-overview.md's Non-Goals state this feature's per-account dataset is small and "loads instantly"; no dedicated loading state is defined beyond an instantaneous render | -- | -- |
| Error | N/A -- feature-overview.md's Non-Goals state every state beyond the plain no-match message (owned by FEAT-19.SPEC-001) is explicitly N/A for this feature; this read-only log has no failure mode distinct from the Empty state already covering the no-data condition | -- | -- |
| Offline/Degraded | N/A -- per the same Non-Goals statement, this feature is "used only in a connected context" with no offline exposure defined anywhere in its Stage 2 source | -- | -- |

## Validation Rules

N/A -- this screen has no user-entered fields; it is a pure read-only list with nothing to validate.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back action (Pro entry) | FEAT-27 (account settings) | FEAT-27 |
| Back action (Support entry) | FEAT-19.SPEC-001 (session hub) | -- |
| Entry with a referenced booking tap | FEAT-16.SPEC-001 (Booking Activity Timeline) | FEAT-16 |

## Data Model

**Creates:** None -- this screen only reads existing entries.
**Reads:** Activity Event (support-view entries only) -- actor, time, reason/ticket reference, and (when applicable) the referenced booking, filtered to entries belonging to the current Pro Account; assembled by FEAT-19.SPEC-002, written by FEAT-16.SPEC-002 (per the Feature Dependency Map's Activity Event lifecycle).
**Updates:** None -- consistent with FEAT-16.SPEC-005's append-only, immutable guarantee; no role, including the Pro, can alter an entry.
**Deletes:** None.

## Business Rules

- XBR-24: every support view is logged in the Pro's visible account activity -- this screen is the Pro-facing (and Support-facing) surface that fulfills that visibility obligation.
- FEAT-16.SPEC-005 governs the underlying Activity Event's append-only immutability and its private-notes exclusion for Support's own rendering elsewhere; a support-view entry itself never contains a client's private note, so no exclusion rendering applies within this screen's own entries.
- FEAT-19.SPEC-004 governs Support's access scope: this screen is reachable for Support only while a session is Active for the account being viewed, never for any other account.
- This is an account-level log, distinct from FEAT-16.SPEC-001's per-booking timeline -- a support-view entry belongs to the Pro Account even when it records a view of a specific booking's timeline (feature-overview.md's Entity-Lifecycle Coverage Matrix).

## Edge Cases

- **Support attempts to view this log for a Pro account with no active session** -- Refused; this screen has no independent entry point of its own outside an active session (per FEAT-19.SPEC-004), so no such attempt can reach it.
- **The Pro views her own log while a Support session for her account happens to be open at that same moment** -- The list reflects whatever entries existed at the time this screen was opened; it is a snapshot per view, not live-updating, so an entry logged moments earlier may not appear until the Pro reopens or refreshes the screen.
- **A Pro Account accumulates a very large number of entries over years of use** -- feature-overview.md's Non-Functional Notes describe this dataset as small and bounded (it grows only with actual help requests, not ordinary use); the list scrolls to show further entries rather than requiring a dedicated pagination control.
- **Multiple entries are logged within the same session (a session-open entry plus one or more timeline-view entries)** -- Each appears as its own separate row, in the order they were recorded, with no merging.
- **A support-view entry references a booking that has since been deleted or archived** -- Bookings are never hard-deleted (per the Feature Dependency Map's Booking lifecycle: "kept for the life of the account"), so this scenario does not occur; the referenced booking's timeline remains reachable for the life of the account.
- **The account has zero recorded support views** -- The Empty state's plain message is shown; this is a normal, non-error condition for a Pro who has never had a help request looked into.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-19.SPEC-002 (Support View Logging) | References (inbound) | Assembles the events this screen ultimately renders |
| FEAT-16.SPEC-002 (Activity Event Recording) | References (inbound) | The sole writer of the entries this screen reads |
| FEAT-16.SPEC-005 (Activity Record Immutability & Visibility Rules) | References (inbound) | Governs the append-only, immutable guarantee these entries carry |
| FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) | Navigation (inbound) | Support arrives here from the open session's hub |
| FEAT-27 (Pro Profile & Booking Page Settings) | Navigation (inbound) | The Pro arrives here from her own account settings |
| FEAT-16.SPEC-001 (Booking Activity Timeline) | Navigation (outbound) | Tapping an entry with a referenced booking opens that booking's own timeline |
| FEAT-19.SPEC-004 (Support Session Scope & Access Rules) | References (inbound) | Governs Support's access scope to this screen |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| support_view_log_viewed_by_pro | Pro Account reference | The Pro opens this screen for her own account | N/A -- no success-metrics.md metric is connected to FEAT-19; retained per feature-overview.md's Key Capabilities, since the Pro's ability to see who looked and when is itself the trust mechanism the audit-added capability introduced, and observing how often she checks it informs whether that trust mechanism is actually being used |
| support_view_log_viewed_by_support | Pro Account reference | Support opens this screen from within an active session | N/A -- no connected success-metrics.md metric; retained for operational observability of how often Support reviews the log during their own session |

## Acceptance Criteria

**FEAT-19.SPEC-003-AC-01:** Given Talia opens her Support Access Log from account settings, when the screen loads and at least one support view has been recorded, then she sees the full, time-ordered list of entries, most recent first.

**FEAT-19.SPEC-003-AC-02:** Given Talia's account has never had a support view, when she opens this screen, then she sees the plain message "No support views recorded for this account yet."

**FEAT-19.SPEC-003-AC-03:** Given Support has an active session with Talia's account, when they tap the Support Access Log entry, then they see the list scoped to that same account only.

**FEAT-19.SPEC-003-AC-04:** Given an entry represents a disputed-booking-timeline view, when Talia or Support taps it, then they are navigated to FEAT-16.SPEC-001 for that specific booking.

**FEAT-19.SPEC-003-AC-05:** Given an entry represents a plain session-open view with no referenced booking, when Talia or Support taps it, then nothing happens -- the entry is display-only.

**FEAT-19.SPEC-003-AC-06:** Given Talia opens this screen, when the reviewer label is examined for any entry, then it reads "Chairtime Support," never a specific individual's name.

**FEAT-19.SPEC-003-AC-07:** Given Talia or Support views any entry on this screen, when they look for an edit, delete, or dismiss control, then none exists anywhere on the screen.

**FEAT-19.SPEC-003-AC-08:** Given Talia opens her Support Access Log, when the screen finishes loading, then the support_view_log_viewed_by_pro event fires with her Pro Account reference.

**FEAT-19.SPEC-003-AC-09:** Given Support opens the log from within an active session, when the screen loads, then the support_view_log_viewed_by_support event fires, and support_view_log_viewed_by_pro does not.

**FEAT-19.SPEC-003-AC-10:** Given a Pro Account has accumulated more entries than fit on one screen, when Talia scrolls, then further entries load below the visible list.

**FEAT-19.SPEC-003-AC-11:** Given one support session produced both a session-open entry and a disputed-timeline-view entry, when Talia views her log, then both appear as separate rows in the order they were recorded.

**FEAT-19.SPEC-003-AC-12:** Given a support view was logged moments before Talia opens this screen, when the screen renders, then it reflects the entries that existed at the moment the screen was opened, without live-updating thereafter.

**FEAT-19.SPEC-003-AC-13:** Given a Client (Riley) attempts to reach this screen, when the attempt is made through any product navigation, then no path exists anywhere in the product that leads them here.

**FEAT-19.SPEC-003-AC-14:** Given Support attempts to view this log for a Pro account with no currently active session, when the attempt is made, then it is refused, since this screen has no independent entry point outside an active session.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 4 | 4 |
| States | 5 (populated, empty, loading N/A, error N/A, offline/degraded N/A) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Support Session Scope & Access Rules

## Overview

**Name:** Support Session Scope & Access Rules
**ID:** FEAT-19.SPEC-004
**Type:** Logic/Rule
**Purpose:** Governs the structural no-write-path rule, one-Pro-account-at-a-time scoping (opening a new lookup ends the prior session), the "only after a Pro's help request" precondition, and the field-level exclusions (private client notes, bank/identity details, sign-in codes) that ground XBR-24.
**Parent Feature:** FEAT-19 -- Platform Support Read-Only Access
**Governed Entity:** Support Session

## Scope and Non-Goals

**In Scope:**
- Field-level rules for the Support Session (the ephemeral scoping context this feature defines)
- Authorization rules for every action this feature and its hand-off points define with respect to Support: opening a session, viewing within it, viewing the Support Access Log, ending a session, attempting a write, and viewing an excluded field
- Default values and derivations for the Support Session's own fields
- The one-account-at-a-time and help-request-precondition business rules that XBR-24 depends on
- The field-level exclusions Support's read-only rendering must honor at every hand-off point (private client notes, bank/identity details, sign-in codes)

**Non-Goals:**
- The lookup form's own layout, states, and navigation -- owned by FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry), which enforces this spec's field validation and business rules but does not restate them.
- The support-view logging mechanics -- owned by FEAT-19.SPEC-002 (Support View Logging), which this spec's rules assume already fires correctly once a session legitimately opens.
- The Support Access Log's own rendering -- owned by FEAT-19.SPEC-003 (Support Access Log), which enforces this spec's Support-scoping rule but does not restate it.
- Each hand-off entity's own full authorization matrix (e.g., every action on Client, Booking, Payout Account) -- owned by that entity's own Logic/Rule spec (FEAT-13.SPEC-005, FEAT-16.SPEC-005, FEAT-28.SPEC-002's own rules, and equivalents); this spec governs only the Support-specific exclusions and read-only scoping layered on top of those specs' own rules, not their full rule sets.

## Governed Entity

**Entity:** Support Session
**Source:** Feature Dependency Map (feature-overview.md's Shared Context and Entity-Lifecycle Coverage Matrix) -- an ephemeral scoping context this feature defines; it is not one of the 18 entities in the Domain Entity Inventory, since it is never persisted beyond the duration of one lookup-to-end-session visit.

| Field | Data Type | Description |
|-------|-----------|--------------|
| pro_account_reference | reference | The one Pro Account this session is scoped to for its entire duration |
| reason_ticket_reference | text | The reason or ticket reference Support entered when opening the session; carried into every support-view event logged during it |
| opened_at | date (with time) | The moment the session became Active |
| ended_at | date (with time) or null | The moment the session became Ended; null while Active |
| status | enum | Active \| Ended |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-19.SPEC-001 | Pro Account Lookup & Support Session Entry | On lookup submit: field validation for pro_account_reference and reason_ticket_reference; on every submit: the one-account-at-a-time rule (ending any prior session); no edit/write control ever rendered anywhere in the session hub |
| FEAT-19.SPEC-002 | Support View Logging | On every trigger: assumes this spec's opening rules were already satisfied; carries reason_ticket_reference and pro_account_reference into every assembled event |
| FEAT-19.SPEC-003 | Support Access Log | On screen entry: Support-scoping rule (log reachable only for the account under an active session) |
| FEAT-13.SPEC-005, FEAT-16.SPEC-005, FEAT-28.SPEC-002 (and every other hand-off spec's own rules) | Client Field Validation & Access Rules, Activity Record Immutability & Visibility Rules, Payout Status & Money Dashboard (and equivalents) | On their own screen render: the private-client-notes, bank/identity-details, and sign-in-code exclusions this spec defines are enforced jointly with each owning spec's own visibility rule |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| pro_account_reference | Required; must resolve to exactly one existing Pro Account | Always | On lookup submit | "A Pro account identifier is required." (empty field) / "No Pro account matches that lookup." (no resolution) | Yes |
| reason_ticket_reference | Required, non-empty after trimming whitespace | Always | On lookup submit | "A reason or ticket reference is required to open a support session." | Yes |
| opened_at | Not user-entered; set automatically to the moment the session becomes Active | Always | On session open | N/A -- not user-entered | Yes |
| ended_at | Not user-entered; set automatically to the moment the session becomes Ended | Always | On session end | N/A -- not user-entered | Yes |
| status | Not user-entered; derived automatically (Active on open, Ended on close) | Always | On open and on end | N/A -- not user-entered | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| One active session at a time | status, pro_account_reference | Opening a session with a new pro_account_reference while another session's status is Active automatically sets the prior session's status to Ended (with ended_at set to that moment) before the new session's status becomes Active | N/A -- this is an automatic transition Support is warned of in advance (FEAT-19.SPEC-001's informational banner), not a validation failure |
| ended_at consistency | status, ended_at | ended_at is null whenever status is Active, and set whenever status is Ended; a session is never in a state where these two fields disagree | N/A -- structural guarantee of the automatic status transition, not a user-facing validation |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Open a support session | Platform Operator (Support) | Both governed fields (pro_account_reference, reason_ticket_reference) pass Field Validation Rules; process expectation per XBR-24 is that this occurs only in response to a Pro's own help request, though no Help Request entity exists for the system to check this against -- see Business Rules | -- |
| Open a support session | The Pro (Talia) | Never | No path exists anywhere in the product for the Pro to reach the lookup screen -- FEAT-19.SPEC-001's own Access and Visibility renders it for Support only |
| Open a support session | The Client (Riley) | Never | No path exists anywhere in the product for a Client to reach the lookup screen |
| View a Pro Account's read-only surfaces through an open support session | Platform Operator (Support) | Only while this session's status is Active, and only for its own pro_account_reference | No path renders a second Pro's data inside one session; opening a new lookup ends the current session first (see Cross-Field Rules), so two accounts are never viewable at once |
| View a Pro Account's read-only surfaces through an open support session | The Pro (Talia) | Never (this specific action) | The Pro views her own services, schedule, bookings, and billing through her own Full access elsewhere in the product, never through this feature's session mechanism |
| View a Pro Account's read-only surfaces through an open support session | The Client (Riley) | Never | No client-facing access of any kind exists in this feature (SC-05) |
| View the Support Access Log for this account | Platform Operator (Support) | Only while this session's status is Active, for its own pro_account_reference | Attempting to reach the log for any other account is impossible -- FEAT-19.SPEC-003 has no entry point independent of an active session |
| View the Support Access Log for her own account | The Pro (Talia) | Always | -- |
| View the Support Access Log | The Client (Riley) | Never | No navigation path anywhere in the product reaches this screen for a Client |
| View a Pro's private client notes through a support session | Platform Operator (Support) | Never | The private note field is simply absent from the rendering at every hand-off point (e.g., FEAT-13.SPEC-005); no "hidden" placeholder appears -- the rest of the record renders normally |
| View bank or identity details on the Payout Account through a support session | Platform Operator (Support) | Never | Absent from the rendering (FEAT-28.SPEC-002); Support sees status and the money list only |
| View sign-in codes, or sign in as the Pro, through a support session | Platform Operator (Support) | Never | No such control or field is ever surfaced to Support anywhere in the product (FEAT-29 defines no Support-facing sign-in view at all) |
| Perform any write, edit, refund, or sign-in-as action within an open support session | Platform Operator (Support) | Never (SC-05) | No edit, save, refund, or sign-in-as control is ever rendered on any screen this session hands into for Support; this is a structural absence of capability, not a permission check that could be bypassed |
| End a support session | Platform Operator (Support) | Always -- explicitly via the End Session action, or automatically when a new lookup is submitted, or automatically when the operator's own authenticated access ends | -- |
| End a support session | The Pro (Talia) | Never | No control exists anywhere for the Pro to end a support session; only Support's own action or the automatic causes end it |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|---------------------|
| status | "Active" | On session open | No |
| status | "Ended" | On explicit End Session, on a new lookup being submitted (superseding this session), or on the operator's own access expiring | No |
| opened_at | The exact moment the lookup resolves to exactly one Pro Account | On session open only | No |
| ended_at | The exact moment status transitions to Ended | On session end only | No |
| reason_ticket_reference | The exact text Support entered on the lookup form | On session open only; carried unchanged into every event this session's activity logs (FEAT-19.SPEC-002) | No -- the value is fixed for the life of the session |

## Business Rules

- XBR-24: Support access is read-only, one account at a time, used only after a Pro's help request, never shows private client notes, bank or identity details or sign-in codes, and every view is logged in the Pro's visible account activity.
- Structural no-write rule: no edit, save, refund, or sign-in-as control exists anywhere in the product for Support (SC-05). This is a structural absence of capability designed into every hand-off screen, not a permission check that could be bypassed by a determined actor -- there is no path to attempt the action in the first place.
- One-Pro-account-at-a-time: enforced by the Cross-Field Rules' automatic status transition; two sessions are never simultaneously Active.
- Help-request precondition: a session should be opened only in response to a Pro's own help request, per XBR-24. This precondition is a process expectation for Support to follow, not a system-checkable rule -- the Domain Entity Inventory defines no Help Request entity for the lookup screen to validate against. The reason_ticket_reference field is where this process is recorded, and the visible Support Access Log (FEAT-19.SPEC-003) is the trust mechanism that makes every session accountable after the fact (feature-overview.md's Key Capabilities), rather than a gate that blocks a session from opening.
- Field-level exclusions apply jointly at each hand-off point, enforced by the owning feature's own Logic/Rule spec: private client notes (FEAT-13.SPEC-005), bank/identity details (FEAT-28.SPEC-002's own scoping), and sign-in codes (never surfaced to Support by any spec in the first place, since FEAT-29 defines no Support-facing sign-in-code view).
- FEAT-16.SPEC-005 governs the append-only immutability of the Activity Event entries this session's activity produces via FEAT-19.SPEC-002; this spec never redefines that guarantee, only triggers it.
- This product defines exactly one Platform Operator (Support) identity (the founder, per user-persona.md); no multi-operator concurrency rule is needed beyond the one-account-at-a-time rule already covering a single operator's own successive lookups.

## Edge Cases

- **Support enters a Pro identifier that resolves, but the Pro Account's status is Closed (per XBR-20)** -- Treated identically to no match: the lookup returns the No Match outcome, since a closed account no longer has an active surface for Support to view.
- **Support's own authenticated access expires while a session's status is Active** -- The session's status transitions to Ended immediately and automatically, with ended_at set to that moment; nothing is lost, since this session holds no unsaved input beyond its two opening fields.
- **The reason_ticket_reference field is filled with only whitespace** -- Fails the "non-empty after trimming" rule identically to an empty field; the same blocking error message is shown.
- **Two field-level exclusions apply to different entries within the same hand-off screen (e.g., one booking's Activity Event entry references a private note, another does not)** -- Each entry's exclusion is evaluated independently, per FEAT-16.SPEC-005's own per-entry rule; excluding one entry's note content never affects another entry's own rendering.
- **Support views the Support Access Log immediately after a prior session (for a different Pro) has just ended** -- Access is strictly scoped to the currently Active session's pro_account_reference; the ended session's account is not concurrently viewable, consistent with the one-account-at-a-time rule.
- **A session's status transitions to Ended (via a new lookup) at the same moment FEAT-19.SPEC-002 is mid-hand-off for that session's prior activity** -- The in-flight hand-off completes independently and is unaffected by the session ending, since FEAT-16.SPEC-002's own append-only write does not depend on the session that produced it remaining Active.

## Acceptance Criteria

**FEAT-19.SPEC-004-AC-01:** Given Support leaves the pro_account_reference (lookup) field empty, when they attempt to open a session, then the error "A Pro account identifier is required." is shown and no session opens.

**FEAT-19.SPEC-004-AC-02:** Given Support enters a lookup value that matches no Pro Account, when they submit it, then the error "No Pro account matches that lookup." is shown and no session opens.

**FEAT-19.SPEC-004-AC-03:** Given Support leaves the reason_ticket_reference field empty, when they attempt to open a session, then the error "A reason or ticket reference is required to open a support session." is shown and no session opens.

**FEAT-19.SPEC-004-AC-04:** Given Support enters only whitespace in the reason_ticket_reference field, when they attempt to open a session, then the same required-field error is shown as for an empty field.

**FEAT-19.SPEC-004-AC-05:** Given Support enters a valid lookup value and a non-empty reason, when they open a session, then status is set to Active, opened_at is set to that moment, and ended_at remains null.

**FEAT-19.SPEC-004-AC-06:** Given Support has an Active session for Talia's account, when they submit a new valid lookup for a different Pro, then Talia's session's status transitions to Ended (with ended_at set) before the new session's status becomes Active.

**FEAT-19.SPEC-004-AC-07:** Given a session's status is Active, when its pro_account_reference and ended_at fields are examined, then ended_at is null; given its status is Ended, ended_at holds the exact transition moment.

**FEAT-19.SPEC-004-AC-08:** Given Support has an Active session for one Pro Account, when they attempt to view any surface belonging to a different Pro Account, then no path in the product renders it -- a second lookup is required, which ends the current session first.

**FEAT-19.SPEC-004-AC-09:** Given Talia opens her own Support Access Log, when the screen renders, then she sees it regardless of whether any support session is currently open for her account.

**FEAT-19.SPEC-004-AC-10:** Given Support has no Active session, when they attempt to reach the Support Access Log for any account, then no such path exists -- the log is reachable only from within an Active session.

**FEAT-19.SPEC-004-AC-11:** Given a Client (Riley) attempts to view the Support Access Log, when the attempt is made, then no navigation path anywhere in the product reaches it.

**FEAT-19.SPEC-004-AC-12:** Given Support is viewing a Pro's client records within an Active session, when a client record's private note is examined, then that field is entirely absent from the rendering, with no "hidden" placeholder in its place.

**FEAT-19.SPEC-004-AC-13:** Given Support is viewing a Pro's Payout Status within an Active session, when the screen renders, then only status and the money list appear, never bank or identity details.

**FEAT-19.SPEC-004-AC-14:** Given Support is viewing any surface within an Active session, when they look for any control that would allow signing in as the Pro or viewing a sign-in code, then none exists anywhere in the product.

**FEAT-19.SPEC-004-AC-15:** Given Support is viewing any surface within an Active session, when they look for any write, edit, or refund control, then none exists anywhere on that surface.

**FEAT-19.SPEC-004-AC-16:** Given Support's own authenticated access expires while a session's status is Active, when the expiry occurs, then status transitions to Ended automatically, with no data lost.

**FEAT-19.SPEC-004-AC-17:** Given Support taps End Session on an Active session, when the action completes, then status transitions to Ended and ended_at is set to that moment.

**FEAT-19.SPEC-004-AC-18:** Given a session's reason_ticket_reference was set at open, when subsequent activity within that session is logged (FEAT-19.SPEC-002), then every logged event carries that same reason_ticket_reference, unchanged.

**FEAT-19.SPEC-004-AC-19:** Given Support looks up a Pro Account whose status is Closed, when the lookup is submitted, then the result is identical to a no-match lookup -- no session opens.

**FEAT-19.SPEC-004-AC-20:** Given there is no Help Request entity anywhere in the product for the system to validate against, when Support opens a session for any reason, then the system permits the open based solely on the two Field Validation Rules -- the help-request precondition is a process expectation, not a system-enforced gate.

**FEAT-19.SPEC-004-AC-21:** Given one booking's Activity Event entry within a hand-off screen references a private client note and another entry does not, when Support views both, then only the note-bearing entry has its note content excluded -- the other entry renders in full.

**FEAT-19.SPEC-004-AC-22:** Given a prior session for a different Pro has just ended, when Support views the Support Access Log immediately afterward, then only the currently Active session's account is shown -- the ended session's account is not concurrently accessible.

**FEAT-19.SPEC-004-AC-23:** Given this product defines exactly one Platform Operator (Support) identity, when the one-account-at-a-time rule is evaluated, then no scenario of two distinct operators holding simultaneous sessions occurs.

**FEAT-19.SPEC-004-AC-24:** Given a session's status transitions to Ended via a new lookup while FEAT-19.SPEC-002 is still handing off that session's prior activity, when the hand-off completes, then it completes successfully and independently of the session's own status transition.

**FEAT-19.SPEC-004-AC-25:** Given Talia looks anywhere in the product for a way to open a support session, when she searches her own account screens and navigation, then no such entry point exists for her.

**FEAT-19.SPEC-004-AC-26:** Given Riley (the Client) attempts to view any Pro Account's surfaces through the support-session mechanism, when the attempt is made, then no such path exists anywhere in the product.

**FEAT-19.SPEC-004-AC-27:** Given Talia looks for a control to end an active support session on her own account, when she examines her own account screens, then no such control exists for her -- only Support's own action or an automatic cause ends it.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 15 | 15 |
| Defaults/Derivations | 5 | 5 |
| Business Rules | 7 | 7 |
| Edge Cases | 6 | 6 |


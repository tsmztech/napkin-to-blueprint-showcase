---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-19.SPEC-004
spec_name: Support Session Scope & Access Rules
spec_slug: support-session-scope-access-rules
parent_feature: FEAT-19
parent_feature_name: Platform Support Read-Only Access
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 34
acceptance_criteria_count: 27
---

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

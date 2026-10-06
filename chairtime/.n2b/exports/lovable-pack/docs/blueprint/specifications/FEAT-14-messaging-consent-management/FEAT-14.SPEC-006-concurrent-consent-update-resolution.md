---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-14.SPEC-006
spec_name: Concurrent Consent Update Resolution
spec_slug: concurrent-consent-update-resolution
parent_feature: FEAT-14
parent_feature_name: Messaging Consent Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 12
acceptance_criteria_count: 11
---

# Logic/Rule Spec: Concurrent Consent Update Resolution

## Overview

**Name:** Concurrent Consent Update Resolution
**ID:** FEAT-14.SPEC-006
**Type:** Logic/Rule
**Purpose:** Resolves a STOP reply and an in-app re-grant (or any two consent-changing writes) arriving close together for the same client-Pro relationship, by most-recent-explicit-action timestamp, defaulting to the no-text state when the outcome is uncertain.
**Parent Feature:** FEAT-14 -- Messaging Consent Management
**Governed Entity:** Messaging Consent

## Scope and Non-Goals

**In Scope:**
- The precedence rule between two consent-state-changing writes for the same relationship arriving close together
- The fail-safe default (no-text) applied when precedence cannot be determined
- Which writes are "explicit client actions" subject to this rule, and which are not

**Non-Goals:**
- Performing the writes themselves -- owned by FEAT-14.SPEC-004 (revoke) and FEAT-14.SPEC-005 (re-grant); this spec governs which of their writes is the one that persists, not how each write is made.
- The booking-time creation or update path -- owned by FEAT-14.SPEC-003; a booking submission is always a single, sequential write with no concurrent counterpart in the same instant, so it never enters this spec's conflict scenario.
- Deriving the client-visible textability status from the resolved state -- owned by FEAT-14.SPEC-007, which reads whatever final state this spec's resolution produces.

## Governed Entity

**Entity:** Messaging Consent
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| channel | enum | The consent's channel (text; WhatsApp reserved for a later phase) |
| state | enum | Granted \| Revoked \| Re-granted |
| timestamp | date | When the current state was set |
| consent_wording | text | The exact wording shown to the client when consent was given, kept as evidence |
| phone_number | text | The phone number the consent applies to |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-14.SPEC-004 | Opt-Out / STOP Processing | Defers to this spec for the final persisted state whenever its revoke write could race a concurrent re-grant |
| FEAT-14.SPEC-005 | Consent Re-Grant Action | Defers to this spec for the final persisted state whenever its re-grant write could race a concurrent revoke |
| FEAT-08.SPEC-011 | Messaging Consent & Channel Selection Rule | Reads the state this spec's resolution produces before every client-directed send |
| FEAT-14.SPEC-007 | Textability Determination Rule | Reads the state this spec's resolution produces to compute textability |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| channel | No validation beyond data type -- unaffected by conflict resolution | Always | -- | -- | -- |
| state | Must resolve to exactly one of Granted / Revoked / Re-granted after any concurrent-write scenario -- never left ambiguous or partially applied | Always | On every detected concurrent write | N/A -- this is a system-resolved value with no user-facing entry; resolution is automatic | Yes (the record is never left with two pending, unresolved writes) |
| timestamp | Must reflect the winning write's own timestamp, not the moment resolution itself runs | Always | On every detected concurrent write | N/A -- system-set | Yes |
| consent_wording | No validation beyond data type -- never altered by conflict resolution, per FEAT-14.SPEC-004 and FEAT-14.SPEC-005's shared rule that this field is untouched by any state-only write | Always | -- | -- | -- |
| phone_number | No validation beyond data type -- never altered by conflict resolution | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Most-recent-explicit-action precedence | state, timestamp | When two writes (one from FEAT-14.SPEC-004, one from FEAT-14.SPEC-005) target the same Messaging Consent record within a window where both are "in flight" at once, the write with the later timestamp is the one whose state and timestamp persist; the earlier write's state does not persist, even though its own write attempt completed | N/A -- resolved automatically; no user-facing error, since both actions were genuinely taken and one legitimately supersedes the other |
| Fail-safe default on genuine uncertainty | state, timestamp | If the two competing timestamps are indistinguishable (recorded at the same instant, or the order in which they were received cannot be established), the record resolves to Revoked (the no-text state), regardless of which write technically committed last in storage | N/A -- resolved automatically; this is the feature's named Error-state discipline (product-features.md: "if in doubt, the system defaults to the safer (no-text) state") |
| STOP reply always acknowledged regardless of resolution outcome | state | Even when a STOP reply's write is the one that does NOT persist (because a later re-grant superseded it), FEAT-14.SPEC-004's confirmation-notification rule is unaffected by this spec -- the confirmation was already correct at the moment it was sent, describing the state as it stood then | N/A -- notification behavior is owned by FEAT-14.SPEC-004/009, not restated here |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Trigger a competing write (revoke or re-grant) that this rule may need to resolve | The Client (Riley) | Own-only -- only the Client whose consent it is (Access Matrix: Messaging & Consent = Own-only for the Client) | -- |
| Trigger a competing write (revoke or re-grant) that this rule may need to resolve | The Pro (Talia) | Never -- the Pro has no path to change a client's consent state (Access Matrix: "the Pro can see but never override") | No control exists anywhere for the Pro to write consent state, so the Pro can never be a party to a conflict this rule resolves |
| Trigger a competing write that this rule may need to resolve | Platform Operator (Support) | Never | Support's View-only access (Access Matrix) never includes a write path, so Support can never be a party to a conflict this rule resolves |
| Read the resolved final state | The Client (Riley) | Own-only, via FEAT-14.SPEC-001 | -- |
| Read the resolved final state | The Pro (Talia) | View-only, for planning, via FEAT-12's dashboard reading FEAT-14.SPEC-007's output | -- |
| Read the resolved final state | Platform Operator (Support) | View-only, for troubleshooting | -- |
| Override or manually force a resolution outcome | Any role | Never -- resolution is fully automatic for every role, including the Pro and Support | No control exists for any role to override the resolved state; the rule applies uniformly regardless of who is asking |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Final persisted state after a detected write conflict | The state and timestamp of whichever of the two competing writes carries the later timestamp; Revoked if the two timestamps are indistinguishable | Whenever FEAT-14.SPEC-004 and FEAT-14.SPEC-005 both target the same record within the same resolution window | No -- fully automatic, no override for any role |

## Business Rules

- XBR-15 and this feature's own Error-state discipline both require the same posture: when a client's current consent is uncertain, the safe default is no-text, never a guess that could send a text without valid consent.
- A conflict this spec resolves is detected only between FEAT-14.SPEC-004's and FEAT-14.SPEC-005's writes -- the two automations that can change an existing record's state after creation. FEAT-14.SPEC-003's booking-time writes are excluded per the Non-Goals above, since they are never concurrent with another write by construction (one client, one sequential booking submission).
- The resolution is silent to the client and the Pro -- neither is shown a "your action was overridden" message; the Client sees only the resulting state on their next view of FEAT-14.SPEC-001, and the Pro sees only the resulting state via FEAT-14.SPEC-007's output on FEAT-12.
- This rule never changes consent_wording or phone_number -- only state and timestamp are subject to resolution, consistent with FEAT-14.SPEC-004 and FEAT-14.SPEC-005 both leaving those fields untouched on their own writes.

## Edge Cases

- **A STOP reply's write and an in-app re-grant's write are both recorded with the exact same timestamp value (down to the finest precision the product records)** -- The two timestamps are treated as indistinguishable; the fail-safe default applies and the record resolves to Revoked, per the Cross-Field Rules' second entry.
- **The re-grant's write technically commits to storage a fraction of a second before the STOP reply's write, but the STOP reply's own timestamp (when the client sent it) is earlier than the re-grant's timestamp (when the client tapped the button)** -- Precedence follows the timestamp of the client's explicit action, not the order in which the writes happened to commit in storage; the re-grant's later action-timestamp wins, even though its write committed second.
- **Only one of the two writes actually occurs (e.g., a STOP reply arrives with no concurrent re-grant at all)** -- There is no conflict to resolve; the single write's state and timestamp simply persist as written by FEAT-14.SPEC-004, and this spec's rule is never invoked.
- **A third write (e.g., a later booking's choice via FEAT-14.SPEC-003) arrives after this spec has already resolved a two-way conflict** -- The prior resolution is now history; FEAT-14.SPEC-003's own write (Business Rules, Non-Goals above) simply updates the record in the normal sequential way, since by the time a new booking is submitted there is no longer a live conflict in progress.
- **The Pro views a client's textability on FEAT-12 at the exact moment a conflict is being resolved** -- The Pro's view reflects whatever state is currently persisted at the moment of the read; if the read happens to land between the two writes, the Pro may briefly see the earlier of the two states, but the very next read after resolution completes shows the final resolved state -- the Pro's read is never itself blocked or delayed waiting for resolution.

## Acceptance Criteria

**FEAT-14.SPEC-006-AC-01:** Given a STOP reply for Riley's relationship with Talia is written with an earlier timestamp than a concurrent in-app re-grant, when both writes are evaluated, then the record resolves to Re-granted with the re-grant's timestamp.

**FEAT-14.SPEC-006-AC-02:** Given an in-app re-grant is written with an earlier timestamp than a concurrent STOP reply for the same relationship, when both writes are evaluated, then the record resolves to Revoked with the STOP reply's timestamp.

**FEAT-14.SPEC-006-AC-03:** Given the two competing writes' timestamps are indistinguishable, when resolution runs, then the record resolves to Revoked regardless of which write committed to storage last.

**FEAT-14.SPEC-006-AC-04:** Given only a STOP reply arrives with no concurrent re-grant, when it is processed, then no conflict resolution is invoked and the STOP reply's write persists as-is.

**FEAT-14.SPEC-006-AC-05:** Given a re-grant's write commits to storage before a STOP reply's write, but the STOP reply's own action-timestamp is later, when resolution runs, then the STOP reply's later action-timestamp wins and the record resolves to Revoked.

**FEAT-14.SPEC-006-AC-06:** Given Riley (the Client) is the only role that can trigger either competing write, when Talia (the Pro) looks for any way to influence the outcome, then no such control exists for her.

**FEAT-14.SPEC-006-AC-07:** Given Support views a resolved consent record, when they look for an override control, then none exists -- Support sees the resolved state read-only.

**FEAT-14.SPEC-006-AC-08:** Given a conflict resolves to Revoked, when Riley next views FEAT-14.SPEC-001, then she sees "Texting: off" with no explanation that her re-grant tap was overridden.

**FEAT-14.SPEC-006-AC-09:** Given a conflict is resolved, when the record's consent_wording and phone_number are inspected, then both remain exactly as they were before the conflicting writes -- only state and timestamp changed.

**FEAT-14.SPEC-006-AC-10:** Given a resolution favors the re-grant, when FEAT-08.SPEC-011 next evaluates the channel for a message to Riley, then it selects text, consistent with the resolved Re-granted state.

**FEAT-14.SPEC-006-AC-11:** Given Talia's FEAT-12 dashboard reads Riley's textability at the exact moment a conflict is mid-resolution, when her next read occurs immediately after, then it reflects the final resolved state, with the earlier read never blocked or delayed by the resolution itself.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 7 | 7 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |

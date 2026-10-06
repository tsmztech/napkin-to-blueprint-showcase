---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-14.SPEC-007
spec_name: Textability Determination Rule
spec_slug: textability-determination-rule
parent_feature: FEAT-14
parent_feature_name: Messaging Consent Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 11
acceptance_criteria_count: 10
---

# Logic/Rule Spec: Textability Determination Rule

## Overview

**Name:** Textability Determination Rule
**ID:** FEAT-14.SPEC-007
**Type:** Logic/Rule
**Purpose:** Computes, as a single authoritative answer, whether a given client is currently textable for a given Pro relationship -- the value every other feature reads instead of re-deriving consent logic itself.
**Parent Feature:** FEAT-14 -- Messaging Consent Management
**Governed Entity:** Messaging Consent

## Scope and Non-Goals

**In Scope:**
- The textable / not-textable determination for a single Client-Pro relationship, evaluated fresh on every read
- The conditions under which the determination is true (active consent, matching phone number) or false (everything else)
- Making this determination the single source of truth every consuming spec reads rather than re-implements

**Non-Goals:**
- Deciding what a consumer does with the result (choosing text vs. email for a send, or displaying a status label) -- owned by each consuming spec: FEAT-08.SPEC-011 for channel selection, FEAT-12 for the Pro's planning display, FEAT-14.SPEC-001 for the client's own status view.
- Writing or changing the Messaging Consent record -- owned by FEAT-14.SPEC-003, FEAT-14.SPEC-004, and FEAT-14.SPEC-005; this spec only reads the record, never modifies it.
- Resolving a race between two writes to the record -- owned by FEAT-14.SPEC-006; this spec always reads whatever state that resolution (or the absence of any conflict) has already settled on.

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

**Referenced (read-only):** Client -- phone, to compare against the consent record's phone_number.

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-14.SPEC-001 | Consent & Preferences | Reads this determination to render the client's own status line |
| FEAT-14.SPEC-002 | Opt-Out Link Landing | Reads this determination indirectly through FEAT-14.SPEC-004's outcome to know whether the revoke was a no-op |
| FEAT-08.SPEC-011 | Messaging Consent & Channel Selection Rule | Reads this determination before every client-directed send in FEAT-08 to choose text or email |
| FEAT-12 | Pro Daily Schedule Dashboard (Pro's planning view) | Reads this determination, View-only, so the Pro can see whether a client can currently be texted |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| channel | Must equal "text" for this determination to consider consent applicable at all | Always | On every determination | N/A -- read-only evaluation, no user-facing error; a non-text channel simply evaluates as not-textable-by-text | No |
| state | Must be Granted or Re-granted for the determination to resolve true; Revoked resolves false | Always | On every determination | N/A -- read-only evaluation | No |
| timestamp | No validation beyond data type -- not itself part of the true/false determination, only relevant to FEAT-14.SPEC-006's prior resolution of which state is current | Always | -- | -- | -- |
| consent_wording | No validation beyond data type -- irrelevant to the determination itself | Always | -- | -- | -- |
| phone_number | Must match the Client's current phone field for the determination to resolve true; a mismatch resolves false regardless of state | Always | On every determination | N/A -- read-only evaluation | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Textability gate | state, phone_number, channel | Textable = true only when state is Granted or Re-granted, AND phone_number matches the Client's current phone, AND channel is text; otherwise Textable = false | N/A -- a computed boolean, never a user-facing error |
| No-record default | (absence of a Messaging Consent record) | If no Messaging Consent record exists yet for the relationship (a state that should not occur once FEAT-14.SPEC-003 has run at first booking, but is defined for completeness), Textable = false | N/A -- defaults to the safe no-text answer |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Read the textability determination for a relationship | The Pro (Talia) | Read-only, own clients only, for planning (Access Matrix: Messaging & Consent = View for the Pro) | -- |
| Read the textability determination for a relationship | The Client (Riley) | Own-only, via FEAT-14.SPEC-001's status display | -- |
| Read the textability determination for a relationship | Platform Operator (Support) | View-only, for troubleshooting | -- |
| Read the textability determination for a relationship | Any other feature's automation or notification spec (e.g., FEAT-08.SPEC-011) | Always -- this determination is the shared read surface every client-directed send consults | -- |
| Override the computed determination for a single send or view | The Pro (Talia) | Never -- the Pro cannot force a "textable" answer against a client's revoked consent, even for her own client | No override control exists anywhere in the product; the determination is fully automatic and non-negotiable |
| Override the computed determination for a single send or view | Platform Operator (Support) | Never | No override control exists for Support under any circumstance |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Textable (boolean) | Derived: true when Messaging Consent.state is Granted or Re-granted AND Messaging Consent.phone_number matches the Client's current phone AND channel is text; false in every other case, including when no record exists | Evaluated fresh on every read -- never cached from a prior evaluation or from booking time | No -- fully automatic, no override for any role |

## Business Rules

- XBR-15 governs this determination entirely: no text is sent without active consent for that client and Pro; otherwise email is used; a revoke is honored on the very next message; a changed phone number requires fresh consent -- every one of these is a direct restatement of the Textability gate above.
- This spec is the single source of truth every other spec and feature (FEAT-08, FEAT-12) reads rather than re-deriving the state itself, per the Brief's Shared Validation section; a consuming spec that computed its own version of this logic independently would risk drifting from this rule over time.
- The determination is evaluated fresh at the moment of each read, never cached -- this is the mechanism by which a revoke is honored on the very next message: there is no stale "textable" answer left over from before a revoke.
- FEAT-14.SPEC-008 (Phone Number Change Consent Invalidation Rule) lists this rule as an enforcer, and this rule in turn enforces FEAT-14.SPEC-008: the phone-number-match condition here is how an invalidated consent immediately evaluates as not textable, with no separate write required.
- Re-granted is treated identically to Granted -- textability is a two-state answer (yes/no), not a three-tier reflection of the underlying three-state consent model.

## Edge Cases

- **A client has Granted consent but no phone number on file (a data inconsistency that should not occur given FEAT-05's capture flow, which requires a phone number to create any Client record)** -- The phone_number match condition cannot be satisfied against an empty Client.phone, so Textable resolves false; the client is never left in an undefined state, and the consuming spec (FEAT-08.SPEC-011) routes to email.
- **A client's phone number changes mid-session while a determination is being read for a message already queued to send** -- The determination re-evaluates at the moment of the actual read (send time), not at queue time, so a number change is reflected correctly even for an in-flight queued message.
- **FEAT-14.SPEC-006 is mid-resolution of a conflicting pair of writes at the exact moment this rule is evaluated** -- The determination reads whatever state is currently persisted at that instant; if the read lands between the two writes, it may reflect the earlier of the two states, but the very next read after resolution completes reflects the final resolved state -- this spec does not itself wait for or participate in that resolution.
- **The consent record's channel is a value other than text (a future WhatsApp-consented record, per the entity's reserved field)** -- Textable (for the text channel this feature governs) resolves false, since the Textability gate requires channel to be text; a WhatsApp-specific determination is a distinct concern the entity's channel field reserves for FEAT-26's later phase, not something this spec computes.
- **No Messaging Consent record exists at all for the relationship being queried (an integration error elsewhere, since FEAT-14.SPEC-003 should always create one at first booking)** -- Per the No-record default cross-field rule, Textable resolves false; a missing record is treated identically to a Revoked one rather than causing an error or an assumed-true default.

## Acceptance Criteria

**FEAT-14.SPEC-007-AC-01:** Given Riley's Messaging Consent with Talia is Granted and her phone number matches the record, when this rule is evaluated, then it returns Textable = true.

**FEAT-14.SPEC-007-AC-02:** Given Riley's Messaging Consent with Talia is Revoked, when this rule is evaluated, then it returns Textable = false.

**FEAT-14.SPEC-007-AC-03:** Given Riley's Messaging Consent state is Re-granted, when this rule is evaluated, then it returns Textable = true, identical to a Granted state.

**FEAT-14.SPEC-007-AC-04:** Given Riley's phone number no longer matches her consent record's phone_number (a recent change), when this rule is evaluated, then it returns Textable = false regardless of the state field.

**FEAT-14.SPEC-007-AC-05:** Given no Messaging Consent record exists at all for a queried relationship, when this rule is evaluated, then it returns Textable = false.

**FEAT-14.SPEC-007-AC-06:** Given Talia (the Pro) views her dashboard, when FEAT-12 reads this rule's output for one of her clients, then she sees the current textability with no control to override it.

**FEAT-14.SPEC-007-AC-07:** Given FEAT-08.SPEC-011 is about to select a channel for a message to Riley, when it reads this rule, then it receives a freshly evaluated result, never a cached value from an earlier point in the session.

**FEAT-14.SPEC-007-AC-08:** Given Riley's consent record's channel field holds a non-text value, when this rule is evaluated for the text channel, then it returns Textable = false for texting purposes.

**FEAT-14.SPEC-007-AC-09:** Given a read of this rule happens to land between two writes that FEAT-14.SPEC-006 is resolving, when the read completes, then it reflects whichever state is currently persisted at that instant, without waiting for resolution to finish.

**FEAT-14.SPEC-007-AC-10:** Given Support views a client's textability for troubleshooting, when they look for a way to change the underlying determination, then no such control exists -- the view is read-only.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 6 | 6 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |

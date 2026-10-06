---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-13.SPEC-005
spec_name: Activity Trail Access & Visibility Rules
spec_slug: activity-trail-access-visibility-rules
parent_feature: FEAT-13
parent_feature_name: Immutable Activity & Audit Trail
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 11
acceptance_criteria_count: 15
---

# Logic/Rule Spec: Activity Trail Access & Visibility Rules

## Overview

**Name:** Activity Trail Access & Visibility Rules
**ID:** FEAT-13.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs who may see the cross-event activity trail and its entries -- Nadia Full, Dana View (session-scoped), Owen and Priya None -- versus who sees only the outcome of their own actions within their own scoped views elsewhere in the product.
**Parent Feature:** FEAT-13 -- Immutable Activity & Audit Trail
**Governed Entity:** Activity Log Entry (read/visibility dimension)

## Scope and Non-Goals

**In Scope:**
- The authoritative role-by-role visibility rule for the cross-event activity trail (FEAT-13.SPEC-001) and the printable record copy (FEAT-13.SPEC-002)
- What each role experiences when they lack access to this capability
- How Dana's session-scoped, read-only access is bounded
- The distinction between "seeing the cross-event trail" (this spec) and "seeing the outcome of one's own action within one's own scoped view" (owned by each individual feature's own screens, e.g., FEAT-03's acceptance confirmation)

**Non-Goals:**
- Field-level content and immutability rules for entries -- owned by FEAT-13.SPEC-004 (Entry Immutability, Content & Attribution Rules); this spec governs only who may see an entry, never what it must contain.
- Retention and account-deletion purge behavior -- owned by FEAT-13.SPEC-006 (Retention & Account-Deletion Purge Rule).
- Defining what a Primary or Reviewer contact may do on their own company's proposals, milestones, or invoices -- owned by each of those features' own access rules (FEAT-03, FEAT-07, FEAT-08, FEAT-09, FEAT-10); this spec only governs the cross-event trail, never those features' own scoped views.

## Governed Entity

**Entity:** Activity Log Entry
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| event_type | enum | The kind of record-worthy event this entry describes |
| actor | text | The freelancer, a named client contact, the operator, or "Automatic" |
| occurred_at | date (timestamp) | The exact date and time the event occurred |
| affected_record | text (reference) | A reference to the specific record the event concerns |
| project | text (reference) | The project the event belongs to, where it has one |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-13.SPEC-001 | Activity Trail | On screen entry -- determines whether the screen is reachable at all for the current role/session, and what it renders when it is |
| FEAT-13.SPEC-002 | Printable Record Copy | On screen entry -- determines whether the screen is reachable at all for the current role |
| FEAT-13.SPEC-003 | Activity Entry Recording | Relies on this spec to know that entries it writes for Dana's own support sessions will be correctly scoped to Dana's view and Nadia's view once written |

## Field Validation Rules

No field-level validation applies in this spec -- content and format rules for every field are owned entirely by FEAT-13.SPEC-004. Visibility in this spec operates at the whole-entry level: an entry a role can see is shown with every field intact; there is no partial-field redaction within a visible entry.

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| event_type | No field-level visibility rule -- visibility is entry-level, per Authorization Rules below | Always | -- | -- | -- |
| actor | No field-level visibility rule -- see above | Always | -- | -- | -- |
| occurred_at | No field-level visibility rule -- see above | Always | -- | -- | -- |
| affected_record | No field-level visibility rule -- see above | Always | -- | -- | -- |
| project | No field-level visibility rule -- see above | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Whole-entry visibility, no partial redaction | event_type, actor, occurred_at, affected_record, project | If a role can view an entry at all (per Authorization Rules), every field on that entry is shown in full -- there is no rule that hides one field of a visible entry while showing the rest | N/A -- this is a display-completeness rule, not a validation failure; there is nothing for a user to correct |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View the cross-event activity trail for a project (FEAT-13.SPEC-001) | Nadia (Freelancer) | Always, her own projects only | -- |
| View the cross-event activity trail for a project (FEAT-13.SPEC-001) | Dana (Support Operator) | Only inside an open support session on the freelancer's account (FEAT-31); read-only | Outside an open session, the screen is unreachable -- no navigation path, tab, or link exists, identical to the unauthenticated experience. |
| View the cross-event activity trail for a project (FEAT-13.SPEC-001) | Owen (Client Primary Contact) | Never | The capability is not shown at all -- no tab, link, or navigation path from Owen's portal view reaches this screen. He sees only the outcome of his own actions within his own scoped views (e.g., his acceptance confirmation in FEAT-03, his approval confirmation in FEAT-08, his invoice detail in FEAT-09). |
| View the cross-event activity trail for a project (FEAT-13.SPEC-001) | Priya (Client Reviewer Contact) | Never | Same as Owen -- not shown at all. She sees only the outcome of her own comment activity within her own scoped views (e.g., FEAT-07). |
| View a single entry's detail (e.g., a linked entry, or the printable copy's single-entry scope) | Nadia (Freelancer) | Always, her own projects only | -- |
| View a single entry's detail | Dana (Support Operator) | Only inside an open support session | Same as above -- unreachable outside a session. |
| View a single entry's detail | Owen, Priya | Never | Same as above -- not shown at all. |
| Produce a printable/shareable copy of the trail or a single entry (FEAT-13.SPEC-002) | Nadia (Freelancer) | Always, her own projects only | -- |
| Produce a printable/shareable copy of the trail or a single entry | Owen, Priya, Dana | Never | Not shown at all -- no control anywhere in their respective sessions reaches this capability. |

## Defaults and Derivations

N/A -- this spec defines no default or derived field values of its own. Every default and derivation for Activity Log Entry's fields is owned entirely by FEAT-13.SPEC-004 (Entry Immutability, Content & Attribution Rules); this spec governs visibility only.

## Business Rules

- XBR-08: role entitlements follow the Access Matrix (user-persona.md) everywhere, including this feature -- Nadia Full (read and share; entries are never editable by anyone), Owen None, Priya None, Dana View, exactly as this spec's Authorization Rules state.
- The Access Matrix's closing definition applies literally: "'None' means the capability is not shown at all" -- Owen's and Priya's denial is never a visible-but-disabled control, an error message, or an explanation screen; the capability has no footprint anywhere in their portal experience.
- XBR-29: Dana's support sessions are always logged and announced to Nadia, and always listed in her trail -- so the very session that grants Dana read-only access also becomes, once closed, an entry Nadia herself can see in the same trail Dana was viewing.
- Nadia's "Full" access is bounded to her own projects by construction -- every project in her account belongs to her, so no further ownership check beyond authentication is needed for her to see the whole trail for any of her own projects.
- ASMP-23 (client isolation): even though Owen and Priya have no access to this trail at all, the same strict per-client isolation that governs every other feature also applies here in principle -- were a future product change ever to grant any client-side trail access, it would still be scoped to that client's own company only, never another client's.

## Edge Cases

- **Owen or Priya attempts to reach the trail through a stale or guessed direct link (e.g., a URL copied from Nadia's own session)** -- Denied exactly as any other unauthorized attempt: no route exists for their role's application surface, so the request resolves the same way an invalid destination would inside their own portal -- they land on their own portal home (FEAT-05), never on trail content.
- **Dana has no open support session and attempts to navigate directly to a trail URL from an earlier session** -- Unreachable, identical to the unauthenticated experience; a closed session grants no residual access.
- **A person is a contact for several freelancers (a Primary contact for one company, a Reviewer for another) and is signed into one freelancer's portal** -- Their access to that freelancer's trail remains exactly None regardless of their role at the other freelancer, since portal scoping (FEAT-05, XBR-09) isolates each freelancer's portal session independently; there is no cross-account bleed-through to check.
- **Dana's support session is open, but the freelancer account she is viewing has multiple projects** -- Her View access extends across every project on the account she is currently sessioned into, not just one project, consistent with FEAT-31's account-level session scope; she cannot use this access to reach a different freelancer's account.
- **Nadia's account has only one project (a new freelancer with a single client)** -- Her Full access applies identically; there is no threshold of project count that changes her access level.

## Acceptance Criteria

**FEAT-13.SPEC-005-AC-01:** Given Nadia opens the activity trail for any of her own projects, when the screen loads, then she sees the full cross-event trail with every field of every entry intact.

**FEAT-13.SPEC-005-AC-02:** Given Owen is signed into his company's portal, when he looks for any way to reach the cross-event activity trail, then no tab, link, or navigation path to it exists anywhere in his portal view.

**FEAT-13.SPEC-005-AC-03:** Given Priya is signed into her company's portal, when she looks for any way to reach the cross-event activity trail, then no tab, link, or navigation path to it exists anywhere in her portal view.

**FEAT-13.SPEC-005-AC-04:** Given Owen just accepted a proposal, when he looks at his own portal, then he sees the outcome of that action within FEAT-03's own confirmation, never the cross-event trail.

**FEAT-13.SPEC-005-AC-05:** Given Dana has an open support session on Nadia's account, when she opens the trail, then she sees the full trail read-only, with every field intact and no editable or actionable controls.

**FEAT-13.SPEC-005-AC-06:** Given Dana has no open support session, when she attempts to navigate directly to a trail URL, then the screen is unreachable, identical to the unauthenticated experience.

**FEAT-13.SPEC-005-AC-07:** Given Dana's support session closes, when she attempts to continue viewing the trail, then access ends immediately.

**FEAT-13.SPEC-005-AC-08:** Given Nadia is the only role permitted to produce a printable copy, when Dana looks for an equivalent control during her session, then none is rendered.

**FEAT-13.SPEC-005-AC-09:** Given Owen or Priya attempts to reach a trail entry through a stale or guessed direct link, when the request is made, then they land on their own portal home rather than any trail content.

**FEAT-13.SPEC-005-AC-10:** Given a person is a Primary contact for one freelancer and a Reviewer contact for a different freelancer, when they are signed into the first freelancer's portal, then their access to that freelancer's trail remains None, unaffected by their role at the other freelancer.

**FEAT-13.SPEC-005-AC-11:** Given Dana's support session opens and later closes, when Nadia opens her trail afterward, then she sees both the opened and closed entries for that session, each attributed to Dana.

**FEAT-13.SPEC-005-AC-12:** Given Nadia views an entry she has access to, when she reads it, then every field (event description, actor, timestamp, affected-record reference) is shown -- no field is hidden from a role that can see the entry at all.

**FEAT-13.SPEC-005-AC-13:** Given Dana's open session spans a freelancer account with several projects, when she opens the trail for any of those projects, then her View access applies identically across all of them.

**FEAT-13.SPEC-005-AC-14:** Given Nadia's account has only a single project, when she opens the trail, then her Full access applies exactly as it would for an account with many projects.

**FEAT-13.SPEC-005-AC-15:** Given Owen attempts to reach the printable record copy screen directly, when the request is made, then the screen is unreachable to him, identical to his experience with the trail itself.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 9 | 9 |
| Defaults/Derivations | 0 (N/A -- owned by FEAT-13.SPEC-004) | 0 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |

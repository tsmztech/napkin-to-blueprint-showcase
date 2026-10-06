---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-05.SPEC-007
spec_name: Portal Access & Isolation Rules
spec_slug: portal-access-isolation-rules
parent_feature: FEAT-05
parent_feature_name: Client Portal Access (Magic-Link Login)
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 24
acceptance_criteria_count: 12
---

# Logic/Rule Spec: Portal Access & Isolation Rules

## Overview

**Name:** Portal Access & Isolation Rules
**ID:** FEAT-05.SPEC-007
**Type:** Logic/Rule
**Purpose:** Governs client isolation, role-scoped portal display, and multi-freelancer separation for a contact who serves several freelancers.
**Parent Feature:** FEAT-05 -- Client Portal Access (Magic-Link Login)
**Governed Entity:** Client Contact (portal session scope)

## Scope and Non-Goals

**In Scope:**
- Scoping a verified session to exactly one Client Contact's freelancer and client company
- Role-based display rules for Owen (Primary) versus Priya (Reviewer) across the portal
- Multi-freelancer separation: a person who is a contact for several freelancers
- The out-of-scope experience: what happens when a link or session resolves outside the requesting contact's own scope

**Non-Goals:**
- Token lifecycle (single-use, expiry, invalidation) -- owned by FEAT-05.SPEC-006 (Link Validity & Recognition Rules), enforced independently of scope.
- The specific screens that display scoped content -- owned by FEAT-05.SPEC-002 and FEAT-05.SPEC-003, which enforce this spec's rules but do not define them.
- Role entitlements for actions inside other features (accepting a proposal, approving a milestone) -- owned by each of those features' own Logic/Rule specs (e.g., FEAT-03, FEAT-08); this spec governs only what is visible and reachable from within the portal shell FEAT-05 owns, not the authorization rules of the destination features themselves.
- Client-side roles beyond Primary and Reviewer -- excluded per scope-boundaries.md SC-02: no further client-side tier is modeled in this spec's role-action matrix.

## Governed Entity

**Entity:** Client Contact (portal session scope)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| email | text | Sign-in and notification address; unique within the client company |
| role | enum | Primary or Reviewer -- drives the scoping and display rules in this spec |
| status | enum | Invited, Active, or Removed |
| client company (relationship) | reference | The single Client this contact belongs to |
| freelancer account (relationship, via client company) | reference | The single Freelancer Account that owns the client company |
| last_sign_in | date (timestamp) | Written by FEAT-05.SPEC-005; not itself a scoping field |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-05.SPEC-005 | Magic Link Verification | Applies scope when creating the session, at the point of successful verification |
| FEAT-05.SPEC-002 | Link Verification Landing | Displays the shared out-of-scope explanation, identical to an expired link, when a resolved scope does not match the requesting browser's expected context |
| FEAT-05.SPEC-003 | Portal Home | Applies role-based display rules to the project list and "Waiting on you" region on every load |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| email | No validation beyond data type within this spec's scope -- format and recognition are governed by FEAT-05.SPEC-006; this spec only uses the field to resolve which client company and freelancer a verified session belongs to | Always | At session creation | N/A -- governed elsewhere | N/A |
| role | Must be exactly Primary or Reviewer -- no other value is recognized by this spec's display rules | Always | On every session-scoped render | N/A -- role is set by FEAT-18, not entered here; a value outside these two is a data-integrity condition owned by FEAT-18, not this spec | N/A |
| status | No validation beyond data type within this spec's scope -- whether a contact may sign in at all (Active vs. Removed) is governed by FEAT-05.SPEC-006's Use-a-token authorization row; this spec only consumes an already-Active contact's `role` and relationships to derive scope | Always | At session creation | N/A -- governed elsewhere | N/A |
| client company (relationship) | A session is scoped to exactly one client company -- the one belonging to the verified Client Contact | Always | At session creation (FEAT-05.SPEC-005) | N/A -- internal scoping, not user input | Yes |
| freelancer account (relationship) | A session is scoped to exactly one freelancer account, derived from the client company | Always | At session creation | N/A -- internal scoping | Yes |
| last_sign_in | No validation beyond data type -- written by FEAT-05.SPEC-005, not read or scoped by this spec's rules | Always | N/A | N/A -- not a scoping field | N/A |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Scope consistency | client company, freelancer account | The freelancer account bound to a session is always the one derived from the session's client company -- the two can never disagree, since a Client belongs to exactly one Freelancer Account | N/A -- structurally guaranteed by the Client entity's Relationships, not a checkable user-facing rule |
| Role gates action visibility | role, (waiting items surfaced by FEAT-05.SPEC-003) | A Reviewer's session never surfaces a proposal-accept or invoice-pay waiting item, regardless of what exists in the underlying data | N/A -- enforced as a display filter, not a user-facing error |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View own client company's project list and stages | Owen, Priya | Always, scoped to their own client company only | -- |
| View own client company's project list and stages | Nadia (Freelancer) | Never through this portal shell -- Nadia has her own freelancer-side project view (FEAT-01), not this one | The portal shell has no entry path that authenticates Nadia; she cannot reach FEAT-05.SPEC-003 at all |
| View own client company's project list and stages | Dana (Support Operator) | Never -- Dana has no portal access (SC-04, Access Matrix: Client Portal Access -- None) | Dana has no entry path into this portal shell; her read-only support session (FEAT-31) is a separate surface entirely |
| View a proposal, invoice, or milestone-approval waiting item | Owen | Always, for items within his own client company's projects | -- |
| View a proposal or invoice waiting item | Priya | Never -- Reviewer contacts cannot see proposal or invoice content (Access Matrix; XBR-08) | The item is never surfaced in Priya's "Waiting on you" region or project detail; there is no disabled control to encounter |
| View a deliverable-review waiting item | Owen, Priya | Always, for items within their own client company's projects | -- |
| Accept, approve, or pay from a waiting item's destination | Owen | Always (the destination feature's own Authorization Rules apply the final check) | -- |
| Accept, approve, or pay from a waiting item's destination | Priya | Never -- no such item is ever shown to her, per the row above | Not reachable, since the waiting item itself is never surfaced |
| Invite a Reviewer colleague from the portal | Owen | Always, for his own client company | -- |
| Invite a Reviewer colleague from the portal | Priya | Never -- inviting is a Primary-only action (Access Matrix: Client Contact Management -- Own-only for Owen, None for Priya) | The "Invite a colleague" control is not shown to Priya |
| Reach any project, milestone, or record belonging to a client company other than the requesting session's own | Nobody -- no role | Never | The shared out-of-scope explanation on FEAT-05.SPEC-002, identical to an expired link, never the other company's data or even a hint of its existence (XBR-09) |
| Reach a second freelancer's portal using a session scoped to the first | Nobody -- no role | Never | Each freelancer's portal requires its own separately issued and verified token (FEAT-05.SPEC-006); a session scoped to one freelancer carries no access to another, even for the same person |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|---------------------|
| Session's client company scope | Derived from the verified Client Contact's own client company relationship | On session creation (FEAT-05.SPEC-005) | No |
| Session's freelancer account scope | Derived from the session's client company | On session creation | No |
| Displayed action set (Portal Home) | Derived from the session's `role` field -- Primary sees accept/approve/pay/review items, Reviewer sees review-only items | On every Portal Home load | No |

## Business Rules

- XBR-09: client isolation holds throughout this feature -- a contact reaches only their own company's projects under one freelancer; a contact for several freelancers sees each portal separately; an out-of-scope or expired link shows a plain explanation and a fresh-link option, never another company's data. This spec is the rule's owning authority per the dependency map.
- XBR-08: role entitlements follow the Access Matrix everywhere -- only Primary contacts accept proposals, request changes, approve milestones, and see, pay, and download invoices; Reviewer contacts view and comment only and never see proposal or invoice content.
- A person holding Client Contact records for several freelancers has one independently scoped session per freelancer; entering one freelancer's portal never carries any access, visibility, or navigation path into another's, and each portal is shown under its own freelancer's Branding Profile (XBR-31).
- Scope is fixed for the lifetime of a session -- it is never widened by an in-portal action, and it is re-derived fresh every time a new session is created by FEAT-05.SPEC-005, never inherited or cached from a prior session.

## Edge Cases

- **Owen is a Primary contact for Freelancer A and a Reviewer contact for Freelancer B** -- His two Client Contact records are entirely independent; his session in Freelancer A's portal shows Primary-level access, and his separately verified session in Freelancer B's portal shows Reviewer-only access, each under that freelancer's own branding, with no cross-navigation between them.
- **A deep link (e.g., from a deliverable-ready email) points to a record that has since moved to a different client company (data correction)** -- The link is re-evaluated against the current session's scope at open time; if the record's current client company no longer matches, the shared out-of-scope explanation is shown rather than the record.
- **Priya's role is changed from Reviewer to Primary by Nadia while Priya's portal session is open** -- Per the dependency map's Contention note on Client Contact ("a role change applies to future actions only"), Priya's already-open session continues to reflect Reviewer-level display until she next verifies a new session (a fresh sign-in); the role change takes effect on her next Portal Home load driven by a new session, not retroactively inside the open one.
- **A contact's client company is archived while their session is open** -- The session remains scoped to that client company; Portal Home reflects the archived project state normally (per FEAT-05.SPEC-003's Edge Cases) rather than this spec treating archival as an isolation violation.
- **An out-of-scope attempt and an expired-token attempt produce the exact same user-visible outcome** -- This is intentional, not an omission: distinguishing them would let an attacker learn whether a guessed or reused token structure was merely expired versus scoped to someone else's data, which XBR-09 explicitly forbids.
- **Dana (Support Operator) opens a support session on a freelancer's account and that freelancer's portal happens to be reachable at a public address** -- Dana's support session (FEAT-31) is a wholly separate authenticated surface from this feature's client-facing sessions; reaching the public portal address without a valid client token still requires a valid, unused token per FEAT-05.SPEC-006, which Dana's support credentials never satisfy.

## Acceptance Criteria

**FEAT-05.SPEC-007-AC-01:** Given Owen verifies a link addressed to his Client Contact record, when the session is created, then it is scoped to exactly his own client company and its owning freelancer.

**FEAT-05.SPEC-007-AC-02:** Given Priya's session is scoped to her client company, when she views Portal Home, then her "Waiting on you" region never lists a proposal or invoice item.

**FEAT-05.SPEC-007-AC-03:** Given Owen's session is scoped to his client company, when he views Portal Home, then he sees proposal, deliverable, milestone-approval, and invoice waiting items for that company.

**FEAT-05.SPEC-007-AC-04:** Given Owen is a Client Contact for two different freelancers, when he signs into each portal separately, then each session shows only that freelancer's data, under that freelancer's own branding, with no path from one into the other.

**FEAT-05.SPEC-007-AC-05:** Given a verified session resolves to a client company different from the one a deep link expected (a scope mismatch), when the mismatch is detected, then the contact sees the same "not valid anymore" explanation as an expired token, never the other company's data.

**FEAT-05.SPEC-007-AC-06:** Given Priya is on Portal Home, when she looks for an "Invite a colleague" control, then it is not shown, since inviting is Primary-only.

**FEAT-05.SPEC-007-AC-07:** Given Owen is on Portal Home, when he looks for an "Invite a colleague" control, then it is shown and opens FEAT-18's invite flow for his own client company.

**FEAT-05.SPEC-007-AC-08:** Given Nadia (Freelancer) attempts to reach the client portal shell, when she does so, then there is no entry path that authenticates her into it, since she holds no Client Contact record.

**FEAT-05.SPEC-007-AC-09:** Given Dana (Support Operator) has an open, valid support session on a freelancer's account, when she attempts to reach that freelancer's client-facing portal, then she has no valid client token and cannot enter it (SC-04).

**FEAT-05.SPEC-007-AC-10:** Given Nadia changes Priya's role from Reviewer to Primary while Priya's portal session is already open, when Priya continues browsing that open session, then it continues to reflect Reviewer-level display until she signs in again with a fresh session.

**FEAT-05.SPEC-007-AC-11:** Given a contact's client company is archived while their portal session is open, when they reload Portal Home, then the archived project's state is shown normally rather than being treated as an isolation violation.

**FEAT-05.SPEC-007-AC-12:** Given Owen holds separate Client Contact records for two freelancers, when Nadia (Freelancer A) views her own client roster, then she has no visibility into Owen's relationship with Freelancer B, and neither freelancer's session can ever expose the other's client data.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 6 | 6 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 12 | 12 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |

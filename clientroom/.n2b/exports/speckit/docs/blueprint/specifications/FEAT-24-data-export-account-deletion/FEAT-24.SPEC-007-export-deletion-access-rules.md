---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-24.SPEC-007
spec_name: Export & Deletion Access Rules
spec_slug: export-deletion-access-rules
parent_feature: FEAT-24
parent_feature_name: Data Export & Account Deletion
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 12
acceptance_criteria_count: 12
---

# Logic/Rule Spec: Export & Deletion Access Rules

## Overview

**Name:** Export & Deletion Access Rules
**ID:** FEAT-24.SPEC-007
**Type:** Logic/Rule
**Purpose:** Restricts every export and deletion action in this feature to Nadia alone, with no access of any kind for client contacts or the Support Operator.
**Parent Feature:** FEAT-24 -- Data Export & Account Deletion
**Governed Entity:** Data Export Archive and Freelancer Account (export & deletion authority) -- the access boundary applied to every screen and automation in this feature

## Scope and Non-Goals

**In Scope:**
- The single, feature-wide authorization gate applied to every screen and automation in FEAT-24
- The exact denied experience for each role that cannot reach this feature
- The total exclusion of the Support Operator, unlike her View access to almost every other feature

**Non-Goals:**
- Determining what content is shown once access is granted (open-item warnings, archive status, retention classification) -- owned by FEAT-24.SPEC-006 and the individual screen and automation specs; this spec governs only whether the action is reachable at all.
- Any ownership-based restriction within the feature -- excluded because there is exactly one governed account per Nadia's own session; the Access Matrix establishes no finer-grained ownership split within this feature than "her own account."
- Client-side account or portal-level permissions in general -- owned by FEAT-18 (Client Contact Management & Roles); this spec addresses only this one feature's total exclusion of client contacts.
- Operator support-session mechanics generally -- owned by FEAT-31 (Operator Support Access); this spec states only the FEAT-24-specific exclusion that XBR-29 requires of every support session.

## Governed Entity

**Entity:** Data Export Archive and Freelancer Account (export & deletion authority)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| Data Export Archive: requested-at | date | When the current export was requested |
| Data Export Archive: status | enum | Requested, Ready, Downloaded, Expired |
| Data Export Archive: download link/window | derived | Scoped, time-limited download link |
| Freelancer Account: deletion state | enum | Active, Deletion Requested, Deletion Confirmed, Deleted |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-24.SPEC-001 | Data Export Screen | On screen entry (route guard) and on every action (Request Export, Download) |
| FEAT-24.SPEC-002 | Account Deletion Screen | On screen entry (route guard) and on the confirmation action |
| FEAT-24.SPEC-003 | Data Export Archive Generation | On trigger receipt, before any aggregation begins |
| FEAT-24.SPEC-004 | Account Deletion Processing | On trigger receipt, before any cascade step begins |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| Data Export Archive: requested-at | No validation beyond data type -- system-set, not directly editable by any role | Always | -- | -- | -- |
| Data Export Archive: status | No validation beyond data type -- system-managed enum, transitions driven entirely by FEAT-24.SPEC-003 | Always | -- | -- | -- |
| Data Export Archive: download link/window | No validation beyond data type -- system-generated | Always | -- | -- | -- |
| Freelancer Account: deletion state | No validation beyond data type -- system-managed enum, transitions driven entirely by FEAT-24.SPEC-004 | Always | -- | -- | -- |

## Cross-Field Rules

No cross-field rules apply. Both governed fields (Data Export Archive's lifecycle fields and the Freelancer Account's deletion state) are each managed independently by their own automation (FEAT-24.SPEC-003 and FEAT-24.SPEC-004 respectively); this spec governs the authorization boundary around them, not any interaction between them.

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Request a data export | Nadia (Freelancer) | Always, her own account only | -- |
| View export status / download the completed archive | Nadia (Freelancer) | Always, her own account only | -- |
| View the Account Deletion Screen | Nadia (Freelancer) | Always, her own account only | -- |
| Give explicit deletion confirmation | Nadia (Freelancer) | Always, her own account only | -- |
| Request a data export, view export status, download an archive, view the Account Deletion Screen, or give deletion confirmation | Owen (Client Primary Contact) | Never | No navigation path in the client portal reaches any FEAT-24 screen; a direct link shows the client portal's standard out-of-scope explanation (XBR-09), never this feature's content |
| Request a data export, view export status, download an archive, view the Account Deletion Screen, or give deletion confirmation | Priya (Client Reviewer Contact) | Never | Same as Owen -- no client-portal surface exists for this feature |
| Request a data export, view export status, download an archive, view the Account Deletion Screen, give deletion confirmation, or view any output of this feature during a support session | Dana (Support Operator) | Never | Every FEAT-24 screen and automation output is excluded entirely from every read-only support session (XBR-29); a direct link during a session shows "This isn't available during a support session," not a read-only view -- unlike Dana's View access to nearly every other feature |

## Defaults and Derivations

N/A -- this spec governs access authorization only. Default values and derivations for the Data Export Archive and the Freelancer Account's deletion state are owned by FEAT-24.SPEC-003 and FEAT-24.SPEC-004 respectively, which create and transition those fields as part of their own processing logic.

## Business Rules

- XBR-29: the operator's support sessions exclude file downloads and data/accounting exports, and exclude account-lifecycle actions entirely -- this spec is FEAT-24's own enforcement of that exclusion, applied to every screen and automation in the feature rather than left to be inferred from omission.
- product-features.md's Access field: "Nadia only (Full, her own account); no client contact can export or delete the freelancer's account... Dana (Support Operator) has no access: she cannot request an export or delete an account on a freelancer's behalf."
- There is no administrative override anywhere in the product for any party other than Nadia to trigger an export or a deletion on her account.
- This exclusion is total and feature-wide: unlike other features where Dana holds View access and client contacts hold Own-only access to some capability groups, this feature grants neither role any access of any kind (Access Matrix, Subscription & Account Data column, as narrowed by feature-overview.md's Access field).

## Edge Cases

- **Dana attempts to reach a FEAT-24 screen via a direct link while an active support session is open on Nadia's account** -- Denied with "This isn't available during a support session," regardless of how the link was reached (bookmark, typed URL, or a link surfaced elsewhere in the product).
- **Owen or Priya attempts to reach a FEAT-24 screen via a direct link, without any support session involved** -- Denied with the client portal's standard out-of-scope explanation (XBR-09); there is no state in which a client contact reaches this feature's content.
- **Nadia's own session is authenticated but her account happens to be mid-deletion (Deletion Confirmed) when she (implausibly) attempts a fresh export request** -- Not reachable in practice, since FEAT-24.SPEC-002 disables all controls during Processing and Nadia is signed out once deletion completes; this spec's authorization gate is not the mechanism that prevents this scenario, FEAT-24.SPEC-002's own state handling is.
- **A support session is opened on Nadia's account by Dana after Nadia has already started (but not confirmed) an account deletion** -- Dana still has no access to FEAT-24.SPEC-002 or any indication of Nadia's in-progress deletion attempt; the total exclusion applies regardless of what state Nadia's own deletion flow is in.

## Acceptance Criteria

**FEAT-24.SPEC-007-AC-01:** Given Nadia (Freelancer) opens FEAT-24.SPEC-001, when the screen loads, then she can view and act on it in full.

**FEAT-24.SPEC-007-AC-02:** Given Nadia (Freelancer) opens FEAT-24.SPEC-002, when the screen loads, then she can view and act on it in full.

**FEAT-24.SPEC-007-AC-03:** Given Nadia triggers FEAT-24.SPEC-003 by requesting an export, when the trigger is received, then authorization passes since it is her own account.

**FEAT-24.SPEC-007-AC-04:** Given Nadia triggers FEAT-24.SPEC-004 by confirming deletion, when the trigger is received, then authorization passes since it is her own account.

**FEAT-24.SPEC-007-AC-05:** Given Owen (Client Primary Contact) attempts to reach FEAT-24.SPEC-001 via a direct link, when he does, then he sees the client portal's standard out-of-scope explanation, never this screen's content.

**FEAT-24.SPEC-007-AC-06:** Given Priya (Client Reviewer Contact) attempts to reach FEAT-24.SPEC-002 via a direct link, when she does, then she sees the same out-of-scope explanation as Owen.

**FEAT-24.SPEC-007-AC-07:** Given Dana (Support Operator) is in an active read-only support session on Nadia's account, when she attempts to open FEAT-24.SPEC-001 or FEAT-24.SPEC-002 directly, then she sees "This isn't available during a support session."

**FEAT-24.SPEC-007-AC-08:** Given Dana is in an active support session, when Nadia's export or deletion actions occur during that session, then Dana receives no indication of them through her support-session view, since this feature is excluded from it entirely.

**FEAT-24.SPEC-007-AC-09:** Given no client contact has any account-level surface to reach this feature from, when Owen or Priya browse their portal, then no navigation element anywhere leads to FEAT-24.

**FEAT-24.SPEC-007-AC-10:** Given Dana's support session grants View access to nearly every other feature, when she looks for an equivalent read-only view of this feature, then none exists -- her access here is None, not View.

**FEAT-24.SPEC-007-AC-11:** Given Nadia's own account is the only account she can act on, when she requests an export or confirms deletion, then the action is always scoped to her own account and never any other freelancer's.

**FEAT-24.SPEC-007-AC-12:** Given a support session is opened on Nadia's account while she has an in-progress (unconfirmed) deletion attempt, when Dana views her session, then Dana still has no access to any FEAT-24 screen or indication of that attempt.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 1 (N/A, stated) | 1 |
| Authorization Rules | 7 | 7 |
| Defaults/Derivations | 1 (N/A, stated) | 1 |
| Business Rules | 4 | 4 |
| Edge Cases | 4 | 4 |

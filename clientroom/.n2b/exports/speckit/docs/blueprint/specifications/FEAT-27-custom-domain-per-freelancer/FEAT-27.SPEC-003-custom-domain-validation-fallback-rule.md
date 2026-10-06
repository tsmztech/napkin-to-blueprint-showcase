---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-27.SPEC-003
spec_name: Custom Domain Validation & Fallback Rule
spec_slug: custom-domain-validation-fallback-rule
parent_feature: FEAT-27
parent_feature_name: Custom Domain per Freelancer
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 10
acceptance_criteria_count: 14
---

# Logic/Rule Spec: Custom Domain Validation & Fallback Rule

## Overview

**Name:** Custom Domain Validation & Fallback Rule
**ID:** FEAT-27.SPEC-003
**Type:** Logic/Rule
**Purpose:** Enforces the domain-format rule and the one-domain-per-account limit on the Custom Domain Record, and guarantees the shared default portal address always remains reachable as a fallback.
**Parent Feature:** FEAT-27 -- Custom Domain per Freelancer
**Governed Entity:** Custom Domain Record

## Scope and Non-Goals

**In Scope:**
- Field validation for domain_name (format, required)
- The one-domain-per-account cardinality limit, and how Add versus Replace is resolved against it
- Authorization rules for every action on the Custom Domain Record, per role
- Default values and derivations for verification_state
- The fallback guarantee: the shared default portal address always remains reachable, regardless of the Custom Domain Record's state

**Non-Goals:**
- Actually verifying that Nadia controls the submitted domain, or serving the portal securely once verified -- owned by FEAT-27.SPEC-002 (Domain Verification & Secure Serving); this spec defines the entry conditions and the fallback guarantee, not the verification mechanism itself
- The Custom Domain Settings screen's own layout, buttons, and confirmation dialogs -- owned by FEAT-27.SPEC-001; this spec's rules are applied there by reference
- Resolving which address (custom or shared default) FEAT-05's portal and client-facing links actually render at -- owned by FEAT-05 (Client Portal Access), which consumes this spec's verification_state and fallback guarantee per XBR-35, but implements the resolution itself
- Multiple domains, or per-team-member subdomains -- excluded per the Validation & Limits field ("one custom domain per freelancer account," product-features.md) and SC-01: Clientroom has no internal-staff seat model, so there is no second user whose work would need a distinct subdomain

## Governed Entity

**Entity:** Custom Domain Record
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|--------------|
| domain_name | text | The domain Nadia has configured to serve her portal; required, and limited to one per Freelancer Account |
| verification_state | enum | Added, Verifying, Verified, or Verification Failed (carrying a specific failure reason when Failed, and the verification instructions when Verifying; both are system-written detail from FEAT-27.SPEC-002). Added is set here and on FEAT-27.SPEC-001's create and replace; Verifying, Verified, and Verification Failed are written only by FEAT-27.SPEC-002 |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|---------------------|
| FEAT-27.SPEC-001 | Custom Domain Settings | domain_name format validated on Add and Replace-save submit; the one-domain-per-account limit governs whether "Add" or "Replace" applies; authorization on screen entry (which controls are shown at all) and on every action attempt |
| FEAT-27.SPEC-002 | Domain Verification & Secure Serving | Relies on the one-domain-per-account limit to know there is at most one record to verify per freelancer; relies on the fallback guarantee to keep the shared default address serving throughout verification, failure, and re-check |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|---------------|-----------------|-----------|
| domain_name | Required, non-empty | Always | On submit (Add and Replace-save) | "Enter your domain to continue." | Yes |
| domain_name | Must be a recognizable domain name: letters, numbers, and hyphens grouped into labels separated by dots, with no spaces, no protocol prefix (e.g. no "http://" or "https://"), and no path or query text | Always | On submit (Add and Replace-save) | "Enter a valid domain name (e.g., myportal.com) -- without \"http://\" or any extra text." | Yes |
| verification_state | No validation beyond data type -- this field is never set directly from user input; it is set to Added on creation and thereafter only by FEAT-27.SPEC-002's reported events -- instructions issued (Verifying), succeeded, or failed (see Defaults and Derivations) | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|------------------|
| Replacing the domain resets verification | domain_name, verification_state | When domain_name is changed on an existing Custom Domain Record (Replace), verification_state is reset to Added in the same operation, and verification is re-triggered via FEAT-27.SPEC-002 for the new value | N/A -- this is an automatic reset, not a rejected input; no error message applies |
| One record per account | domain_name (record cardinality) | At most one Custom Domain Record may exist per Freelancer Account (XBR-35). Submitting a domain when a record already exists is never a second create -- it is resolved as a Replace of the existing record's domain_name, never a second, parallel record | N/A -- there is no invalid-state error here; FEAT-27.SPEC-001 always presents this as "Replace domain" once a record exists, so the ambiguity never reaches the user as an error |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|-------------------------------------------------|
| Add domain (create the Custom Domain Record) | Nadia (Freelancer) | Only when no Custom Domain Record currently exists for her account | -- |
| View domain and verification status | Nadia (Freelancer) | Always | -- |
| View domain and verification status | Dana (Support Operator) | Only inside an active, logged support session opened through FEAT-31 (ASMP-18, XBR-29) | Outside a support session, Dana has no route to this data at all -- the screen and the data it would show are simply not reachable |
| Replace domain (update domain_name on an existing record) | Nadia (Freelancer) | Always, when a record exists | -- |
| Request re-check on a failed verification | Nadia (Freelancer) | Only when verification_state is Verification Failed | The "Re-check" control is not shown for any other verification_state, including Verifying (even when FEAT-27.SPEC-002 reports the verification is slow); Nadia waits, replaces, or removes, and FEAT-27.SPEC-002 resolves the record to Verified or Verification Failed |
| Remove domain (delete the Custom Domain Record) | Nadia (Freelancer) | Always, when a record exists | -- |
| Add domain | Owen (Client Primary Contact) | Never | No route to this screen or control exists from the client portal; client contacts have no custom-domain surface at all |
| Replace domain | Owen (Client Primary Contact) | Never | Same as Add domain -- no route exists |
| Request re-check | Owen (Client Primary Contact) | Never | Same as Add domain -- no route exists |
| Remove domain | Owen (Client Primary Contact) | Never | Same as Add domain -- no route exists |
| View domain and verification status | Owen (Client Primary Contact) | Never | Owen has no visibility into this configuration surface; he only experiences whichever domain FEAT-27.SPEC-003's fallback resolves as active (feature-overview.md) |
| Add domain | Priya (Client Reviewer Contact) | Never | Same as Owen -- no route exists |
| Replace domain | Priya (Client Reviewer Contact) | Never | Same as Owen -- no route exists |
| Request re-check | Priya (Client Reviewer Contact) | Never | Same as Owen -- no route exists |
| Remove domain | Priya (Client Reviewer Contact) | Never | Same as Owen -- no route exists |
| View domain and verification status | Priya (Client Reviewer Contact) | Never | Same as Owen -- no visibility into this configuration surface |
| Add domain | Dana (Support Operator) | Never | Support sessions are strictly read-only in every feature (XBR-29, ASMP-18): no domain-configuration control is ever shown to Dana, inside or outside a support session |
| Replace domain | Dana (Support Operator) | Never | Same as Add domain -- read-only boundary (XBR-29, ASMP-18) |
| Request re-check | Dana (Support Operator) | Never | Same as Add domain -- read-only boundary (XBR-29, ASMP-18) |
| Remove domain | Dana (Support Operator) | Never | Same as Add domain -- read-only boundary (XBR-29, ASMP-18) |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-----------------------|-----------------|--------------------|
| verification_state | Set to Added | On create (a new Custom Domain Record) | No -- this is the fixed starting state; it is never user-settable |
| verification_state | Reset to Added | On update, whenever domain_name changes (Replace) | No -- the reset is automatic, per the Replacing the domain resets verification cross-field rule |
| verification_state | Set to Verifying, with the verification instructions stored as system-written detail | Whenever FEAT-27.SPEC-002 reports its "verification instructions issued" event after accepting a submission (Add, Replace-save, or Re-check); FEAT-27.SPEC-002 is the only writer of Verifying | No -- Nadia can only trigger a new attempt, never set the state directly |
| verification_state | Set to Verified, or Verification Failed (with a specific failure reason when Failed) | Whenever FEAT-27.SPEC-002 reports a verification succeeded, verification failed, or re-check result | No -- this field is entirely system-derived from FEAT-27.SPEC-002's reports; Nadia can only trigger a new attempt (Add, Replace, Re-check), never set the resulting state directly |

## Business Rules

- XBR-35: With a verified custom domain, the portal and client-facing links use it, and the shared default address always remains available as a fallback -- this fallback holds in every verification_state, including Added, Verifying, and Verification Failed, so the portal is never unreachable while a custom domain is unverified, failed, being replaced, or being removed.
- Removing the Custom Domain Record immediately reverts portal and client-facing links to the shared default address (FEAT-27.SPEC-001, per XBR-35). There is no restore path: re-adding the exact same domain_name later starts verification from Added, with no memory of the prior verification outcome.
- No cascade: nothing else in the product references the Custom Domain Record directly (dependency map, Relationships), so a Remove or a Replace has no downstream records to clean up beyond the fallback reversion itself.
- The Custom Domain Record is also deleted as part of a full account deletion, owned by FEAT-24 (Data Export & Account Deletion) -- that deletion path is FEAT-24's own cascade, not an action this spec's Authorization Rules govern.
- No retention or purge window applies to a removed Custom Domain Record: it carries no personal data (dependency map, Data Sensitivity: None), so deletion is immediate and complete, unlike the legal-retention exception that applies to financial records elsewhere in the product.

## Edge Cases

- **domain_name submitted with a leading or trailing "http://" or "https://"** -- Rejected by the format rule with "Enter a valid domain name (e.g., myportal.com) -- without \"http://\" or any extra text." even though the underlying domain portion may otherwise be valid; the rule does not attempt to strip and salvage a protocol prefix.
- **domain_name submitted with only whitespace** -- Treated as empty; rejected with "Enter your domain to continue."
- **domain_name at the shortest valid form (two labels separated by one dot, e.g. "ab.co")** -- Passes validation; the format rule sets no minimum label count beyond requiring at least one dot separating two non-empty labels.
- **Nadia submits the domain she already has configured, unchanged, via "Replace domain"** -- Treated as a normal Replace: verification_state resets to Added and verification re-triggers, even though domain_name did not actually change in value; the reset is keyed to the Replace action, not to whether the value differs.
- **Nadia attempts to add a domain while one already exists (e.g. by returning to a stale "Add domain" view)** -- The one-record-per-account rule resolves this as a Replace of the existing record, never a second, parallel Custom Domain Record; FEAT-27.SPEC-001 never actually presents an "Add" control once a record exists, so this scenario can only arise from a stale or replayed submission, and it is handled identically to a normal Replace.
- **Dana's support session closes while she is viewing this data** -- Access is revoked immediately; any further attempt to view or act on the Custom Domain Record is denied per the View row's condition (no active session) with no route to the data at all, consistent with FEAT-31's session-close behavior.
- **A Remove and a Replace are both attempted from Nadia's own two open sessions at nearly the same moment** -- There is at most one Custom Domain Record and no other human writer (dependency map, Contention: "None -- only Nadia configures it"), so whichever action reaches the record first completes normally, and the second session's subsequent action operates on the record's new state (e.g., a Replace submitted just after a Remove completed creates a fresh record from Added, rather than updating a record that no longer exists).

## Acceptance Criteria

**FEAT-27.SPEC-003-AC-01:** Given Nadia has no Custom Domain Record, when she submits "myportal.com" via Add domain, then the domain-format rule passes and a Custom Domain Record is created with verification_state Added.

**FEAT-27.SPEC-003-AC-02:** Given Nadia submits an empty domain field, when she taps Add domain, then she sees "Enter your domain to continue." and no record is created.

**FEAT-27.SPEC-003-AC-03:** Given Nadia submits "http://myportal.com", when she taps Add domain, then she sees "Enter a valid domain name (e.g., myportal.com) -- without \"http://\" or any extra text." and no record is created.

**FEAT-27.SPEC-003-AC-04:** Given Nadia already has a Custom Domain Record for "myportal.com", when she submits "newdomain.com" via Replace domain, then domain_name updates to "newdomain.com" and verification_state resets to Added, re-triggering verification.

**FEAT-27.SPEC-003-AC-05:** Given Nadia's Custom Domain Record is in Verification Failed, when she taps Re-check, then the request is allowed and verification is re-triggered for the same domain_name.

**FEAT-27.SPEC-003-AC-06:** Given Nadia's Custom Domain Record is Verified or Verifying (including a slow verification), when she looks for a Re-check control, then none is shown -- Re-check is available only from Verification Failed.

**FEAT-27.SPEC-003-AC-07:** Given Nadia (Freelancer), when she attempts to add, replace, request a re-check on, or remove her domain, then every action is allowed without further condition beyond the record-existence and verification-state conditions stated above.

**FEAT-27.SPEC-003-AC-08:** Given Owen or Priya, when either looks for any control to add, replace, re-check, or remove a custom domain, then no such control or route exists anywhere in the client portal.

**FEAT-27.SPEC-003-AC-09:** Given Dana is inside an active, logged support session opened via FEAT-31, when she views FEAT-27.SPEC-001, then she can see the domain and its verification state but has no Add, Replace, Re-check, or Remove control available to her.

**FEAT-27.SPEC-003-AC-10:** Given Dana's support session has closed, when she attempts to view the Custom Domain Record again, then she has no route to it at all.

**FEAT-27.SPEC-003-AC-11:** Given a domain is verified and reachable, when a client contact opens a portal link, then it resolves to the verified custom domain while the shared default address remains reachable too, per XBR-35.

**FEAT-27.SPEC-003-AC-12:** Given Nadia's domain is in Verification Failed, when a client contact opens a portal link during that time, then the portal is still fully reachable at the shared default address, per the fallback guarantee.

**FEAT-27.SPEC-003-AC-13:** Given Nadia removes her verified domain, when the removal is confirmed, then verification_state and domain_name are cleared (the record is deleted), and portal links immediately resolve to the shared default address again.

**FEAT-27.SPEC-003-AC-14:** Given Nadia re-adds the exact domain she just removed, when the new record is created, then verification_state starts at Added with no memory of the prior verification outcome.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 3 | 3 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 20 | 20 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |

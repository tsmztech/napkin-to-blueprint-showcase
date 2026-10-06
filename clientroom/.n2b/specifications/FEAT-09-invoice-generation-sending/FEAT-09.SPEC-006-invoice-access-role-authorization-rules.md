---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-09.SPEC-006
spec_name: Invoice Access & Role Authorization Rules
spec_slug: invoice-access-role-authorization-rules
parent_feature: FEAT-09
parent_feature_name: Invoice Generation & Sending
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 17
acceptance_criteria_count: 15
---

# Logic/Rule Spec: Invoice Access & Role Authorization Rules

## Overview

**Name:** Invoice Access & Role Authorization Rules
**ID:** FEAT-09.SPEC-006
**Type:** Logic/Rule
**Purpose:** Governs who may view, issue, follow the pay link on, or download an invoice, including Priya's total exclusion and Dana's read-only support-session visibility.
**Parent Feature:** FEAT-09 -- Invoice Generation & Sending
**Governed Entity:** Invoice (access and action authorization, not content)

## Scope and Non-Goals

**In Scope:**
- Authorization rules for every action defined on the Invoice entity by this feature: view (list and detail), issue (ad-hoc/credit note), follow the pay link, download a copy
- The exact denied experience for each role on each action, including Priya's total exclusion from invoice content
- Dana's read-only, session-scoped visibility per Operator Support Access (FEAT-31)

**Non-Goals:**
- Field-level content and validation rules (numbering, amount, due date) -- owned by FEAT-09.SPEC-007
- Immutability and correction behavior -- owned by FEAT-09.SPEC-008
- Pay-link wording and availability -- owned by FEAT-09.SPEC-009; this spec governs only whether a role may follow the link at all, not what it says
- Actions on Payment or Reminder Log records -- owned respectively by FEAT-10 and FEAT-11; this spec governs the Invoice entity only

## Governed Entity

**Entity:** Invoice
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| invoice_number | text (derived) | Unique sequential number per freelancer -- not directly access-relevant, addressed by FEAT-09.SPEC-007 |
| project, triggering_event | reference / enum | Owning project and generation source -- not directly access-relevant |
| amount, currency, tax_label, tax_rate, total | number / text | Financial content -- access-relevant: entirely hidden from Priya |
| business/billing details | text | Access-relevant: entirely hidden from Priya |
| issue_date, due_date | date | Access-relevant: entirely hidden from Priya |
| status | enum | Access-relevant: visibility of status is gated the same as the rest of the invoice's content |
| pay_link availability | derived | Access-relevant: Owen alone may act on it |
| Reminder Log `pause_state` (read-only here; not an Invoice field) | enum (Active, Paused by freelancer, Paused while bank transfer pending) | Access-relevant: the invoice's reminder pause is read from the Reminder Log entity's `pause_state`, which is written only by FEAT-11.SPEC-003 (Nadia's pause/resume) and FEAT-11.SPEC-005 (automatic bank-transfer-pending pause/resume); FEAT-09 never writes it. Visible to Nadia only, not to Owen or Priya |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-09.SPEC-001 | Invoice List | Screen entry (navigation hidden/redirected) and per-row rendering |
| FEAT-09.SPEC-002 | Invoice Detail | Screen entry and per-action rendering |
| FEAT-09.SPEC-003 | Manual Invoice & Credit Note Issuance | Screen entry (Nadia-only) |
| FEAT-09.SPEC-004, FEAT-09.SPEC-005 | Automatic / Manual Generation and Recording | Recipient entitlement check before any notification is composed |
| FEAT-11.SPEC-003 | Invoice Reminder Panel | Owns the Nadia-only display and write of the Reminder Log `pause_state` shown on FEAT-09.SPEC-002; FEAT-11.SPEC-005 writes it automatically |
| FEAT-31 | Operator Support Access | Session-scoped read-only rendering for Dana |

## Field Validation Rules

Not applicable to this spec -- field-level validation is owned by FEAT-09.SPEC-007. Every field above is addressed only for its access relevance, per the Governed Entity table.

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Content-or-nothing for Priya | All Invoice fields | If the viewing role is Priya, no field of any Invoice is ever rendered, individually or in aggregate (no partial summary, no status-only view) | Not applicable -- no error is shown; the entry point simply does not exist for Priya (see Authorization Rules) |
| Session-scoped visibility for Dana | All Invoice fields | Every field is readable by Dana only while a Support Access Session (FEAT-31) is Opened for the account being viewed; once the session closes, none of this feature's screens are reachable by her | Not applicable -- outside an open session, none of FEAT-09's screens render for Dana at all |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View invoice list (FEAT-09.SPEC-001) | Nadia (Freelancer) | Always, for any project or client she owns | -- |
| View invoice list (FEAT-09.SPEC-001) | Owen (Client Primary Contact) | Own-only -- his own company's invoices across all its projects with this freelancer | -- |
| View invoice list (FEAT-09.SPEC-001) | Priya (Client Reviewer Contact) | Never | The "Invoices" entry point is not shown anywhere in Priya's portal navigation; a direct link redirects to her portal home with no error message |
| View invoice list (FEAT-09.SPEC-001) | Dana (Support Operator) | Only inside a logged, Opened Support Access Session (FEAT-31), read-only | Outside an open session, this screen is not reachable by Dana at all |
| View invoice detail (FEAT-09.SPEC-002) | Nadia (Freelancer) | Always, for any of her invoices | -- |
| View invoice detail (FEAT-09.SPEC-002) | Owen (Client Primary Contact) | Own-only -- his own company's invoices | -- |
| View invoice detail (FEAT-09.SPEC-002) | Priya (Client Reviewer Contact) | Never | A direct link to this screen redirects to her portal home with no invoice content ever rendered |
| View invoice detail (FEAT-09.SPEC-002) | Dana (Support Operator) | Only inside a logged, Opened Support Access Session, read-only | Every action control (download, pay, credit note, off-platform payment, refund) is not rendered; content is visible, actions are not |
| Issue an ad-hoc invoice or credit note (FEAT-09.SPEC-003) | Nadia (Freelancer) | Always | -- |
| Issue an ad-hoc invoice or credit note (FEAT-09.SPEC-003) | Owen, Priya | Never | This screen has no entry point in either contact's portal; a direct link redirects to portal home |
| Issue an ad-hoc invoice or credit note (FEAT-09.SPEC-003) | Dana (Support Operator) | Never | This screen is never rendered inside a support session -- issuance is excluded from read-only support access entirely (XBR-29) |
| Follow the pay link | Owen (Client Primary Contact) | Own-only, and only when FEAT-09.SPEC-009's availability rule shows the "ready to pay" state | Rendered as a disabled or absent action in the two non-ready states, per FEAT-09.SPEC-009's own wording -- never a bare "denied" |
| Follow the pay link | Nadia, Priya, Dana | Never | Not shown; the pay link is Owen's action alone |
| Download a copy of an invoice, credit note, or receipt | Nadia (Freelancer), Owen (Client Primary Contact) | Always, for any invoice each is otherwise entitled to view | -- |
| Download a copy of an invoice, credit note, or receipt | Priya, Dana | Never | Not rendered for Priya (no access at all); not rendered for Dana (read-only support access excludes downloads, per XBR-29) |

## Defaults and Derivations

Not applicable to this spec -- no field defaults or derivations are governed here; see FEAT-09.SPEC-007.

## Business Rules

- XBR-08: role entitlements follow the Access Matrix everywhere -- only Primary contacts see, pay, and download invoices; Reviewer contacts never see invoice content. This spec is the FEAT-09-specific instantiation of that rule for the Invoice entity.
- XBR-09: client isolation applies to every invoice screen -- Owen never sees another client company's invoices, and an out-of-scope link shows a plain explanation rather than another company's data.
- XBR-29: Dana's support sessions are read-only in every feature, exclude downloads and any send/pay/issue action, are always announced to Nadia by email, and are always listed in her trail -- this spec's Dana rows are the FEAT-09 instantiation of that rule.
- Every denied action across this spec's table states the exact experience (control hidden, redirect, or absent action) -- "access denied" alone is never shown to any role.

## Edge Cases

- **Priya is given a direct deep link to a specific invoice by someone who has one** -- The link resolves to a redirect to her portal home; no invoice content, not even the invoice number, is ever rendered in the process.
- **Owen's role is changed from Primary to Reviewer (a hypothetical the product does not define, since role changes are Nadia-only per FEAT-18) while he has an invoice detail screen open** -- Not applicable to this product: FEAT-18's Authorization Rules gate role changes to Nadia only, and a contact's own role change takes effect for future actions only; this spec assumes the role in effect at the moment of each access check.
- **A Support Access Session closes while Dana has an invoice detail screen open** -- The screen becomes unreachable immediately; any further interaction attempt is treated as unauthenticated for that session, consistent with FEAT-31's session-closing behavior.
- **Owen requests to download a copy of an invoice that has since been corrected (status: Corrected)** -- Allowed: the download control remains available on the original invoice's detail screen even after correction, since the original record itself is never deleted or hidden, only marked Corrected (FEAT-09.SPEC-008).
- **Dana attempts to reach FEAT-09.SPEC-003 by a direct link during an open support session** -- The screen is never rendered for her regardless of the link; issuance has no support-session-accessible form at all.

## Acceptance Criteria

**FEAT-09.SPEC-006-AC-01:** Given Nadia is signed in, when she opens any of her projects' invoice lists, then she sees every invoice regardless of status.

**FEAT-09.SPEC-006-AC-02:** Given Owen is signed into his portal, when he opens "Invoices", then he sees only his own company's invoices across all of its projects.

**FEAT-09.SPEC-006-AC-03:** Given Priya is signed into her portal, when she looks for any invoice-related entry point, then none exists anywhere in her navigation.

**FEAT-09.SPEC-006-AC-04:** Given Priya is given a direct link to a specific invoice, when the link resolves, then she is redirected to her portal home with no invoice content rendered.

**FEAT-09.SPEC-006-AC-05:** Given Dana opens a Support Access Session for a freelancer's account, when she navigates to that account's invoices, then she sees full content read-only with no action controls.

**FEAT-09.SPEC-006-AC-06:** Given Dana has no open Support Access Session, when she attempts to reach any invoice screen, then none is reachable.

**FEAT-09.SPEC-006-AC-07:** Given Nadia opens FEAT-09.SPEC-003, when the screen loads, then she can issue an ad-hoc invoice or credit note.

**FEAT-09.SPEC-006-AC-08:** Given Owen attempts to reach FEAT-09.SPEC-003 by a direct link, when the link resolves, then he is redirected to his portal home.

**FEAT-09.SPEC-006-AC-09:** Given Dana is inside an open support session, when she attempts to reach FEAT-09.SPEC-003 by any means, then it is never rendered.

**FEAT-09.SPEC-006-AC-10:** Given Owen's invoice has a ready, connected payment account, when he views its detail, then he can follow the pay link.

**FEAT-09.SPEC-006-AC-11:** Given Owen's invoice does not yet have a ready payment account, when he views its detail, then no active pay-link action is offered to him.

**FEAT-09.SPEC-006-AC-12:** Given Nadia or Owen views an invoice each is entitled to see, when they tap Download, then the copy is produced.

**FEAT-09.SPEC-006-AC-13:** Given Dana is inside an open support session, when she looks for a Download control on any invoice, then none is rendered.

**FEAT-09.SPEC-006-AC-14:** Given a Support Access Session closes while Dana has an invoice screen open, when she attempts any further interaction, then the screen is no longer reachable.

**FEAT-09.SPEC-006-AC-15:** Given an invoice has been corrected (status: Corrected), when Owen or Nadia attempts to download the original, then the download still succeeds, since the original record is never hidden by correction.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 0 (N/A -- owned by FEAT-09.SPEC-007) | 0 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 15 | 15 |
| Defaults/Derivations | 0 (N/A -- owned by FEAT-09.SPEC-007) | 0 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |

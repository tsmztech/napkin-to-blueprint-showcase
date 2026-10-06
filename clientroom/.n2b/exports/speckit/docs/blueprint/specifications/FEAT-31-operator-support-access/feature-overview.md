---
document_type: feature-overview
feature_number: FEAT-31
feature_name: Operator Support Access
feature_slug: operator-support-access
priority_tier: Important
feature_type: Platform
produced_by: feature-analyst
status: final
created: 2026-09-27
spec_count: 7
screen_count: 2
automation_count: 2
logic_rule_count: 1
integration_count: 0
notification_count: 2
---

# Feature Breakdown Brief: Operator Support Access

## Summary

**Feature:** Operator Support Access
**ID:** FEAT-31
**Description:** When a freelancer asks for help, the Clientroom operator can open a read-only view of that freelancer's account to diagnose the problem. Every such session is shown to the freelancer in her activity trail. The freelancer can contact support from inside the product.
**Priority:** Important
**Phase:** MVP
**Type:** Platform
**Rationale:** BRIEF.md, Target Users & Roles: "The founder, as operator, needs read-only support access to a freelancer's account, nothing more." The draft Access Matrix gave the Support Operator View across the product, but no feature defined how that access starts, how it stays read-only, or how the freelancer can see it. Research shows responsive support is a durable differentiator in this category and its absence is Moxie's top complaint (Capterra and G2 reviews, HIGH). Important rather than Core because the client-facing loop works without it; MVP because a solo founder must be able to support the first paying freelancers from launch (BRIEF.md, Constraints). [AUDIT-ADDED: 3 -- role coverage: the Support Operator row in the Access Matrix had no feature granting, bounding, or recording its access] [RESEARCH-INFORMED: support responsiveness as a loyalty driver, from Capterra and G2 reviews of Dubsado and Moxie]

**Key Capabilities:**
- Contact support -- Nadia sends a support request describing the problem from inside the product
- Open a read-only support session -- the operator sees the named freelancer's account as she sees it, with every edit, send, approve, and pay control unavailable
- See who looked -- every support session appears in the freelancer's activity trail with when it started and ended

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-31.SPEC-001 | Contact Support Screen | Screen | Nadia (Freelancer) | Nadia describes her problem and sends a support request from inside the product, and gets an on-screen and emailed confirmation that it was received |
| FEAT-31.SPEC-002 | Operator Support Session Console | Screen | Dana (Support Operator) | Dana sees the queue of open support requests, opens a read-only session on one named freelancer's account, works through that freelancer's own screens under a permanent read-only banner, and closes the session |
| FEAT-31.SPEC-003 | Support Session Open & Read-Only Enforcement | Automation | Dana (Support Operator), Nadia (Freelancer) | Opens the Support Access Session record the instant Dana starts a session, and enforces that every edit, send, approve, pay, file-download, and export-generation control is unavailable for the session's duration |
| FEAT-31.SPEC-004 | Support Session Auto-Close on Inactivity | Automation | Dana (Support Operator), Nadia (Freelancer) | Closes an open Support Access Session automatically after a period of inactivity, recording the close time and returning Dana to the queue |
| FEAT-31.SPEC-005 | Support Access Authorization & Read-Only Rules | Logic/Rule | Dana (Support Operator), Nadia (Freelancer), Owen (Client Primary Contact), Priya (Client Reviewer Contact) | Governs who may open, view, or never see a support session, the one-account-at-a-time and unconditional read-only constraints, the excluded actions (file downloads, data/accounting export generation, signing in as a client contact), and Nadia's standing right to see every session on her account |
| FEAT-31.SPEC-006 | Support Request Confirmation | Notification | Nadia (Freelancer) | Sends Nadia a confirmation email the moment her support request is received |
| FEAT-31.SPEC-007 | Support Session Opened Notice | Notification | Nadia (Freelancer) | Sends Nadia an email notice whenever a support session opens on her account |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Contact support -- Nadia sends a support request describing the problem from inside the product | FEAT-31.SPEC-001, FEAT-31.SPEC-006 | The screen captures the request text and creates the Support Access Session record in its unopened state; the notification confirms receipt by email | Phase 2 (Explicit) |
| Open a read-only support session -- the operator sees the named freelancer's account as she sees it, with every edit, send, approve, and pay control unavailable | FEAT-31.SPEC-002, FEAT-31.SPEC-003, FEAT-31.SPEC-005, FEAT-31.SPEC-007 | The console lets Dana pick a pending request and mirrors the freelancer's own screens under a permanent banner; the automation enforces read-only across every control the instant the session opens; the rules spec defines exactly what "read-only" excludes; the notification tells Nadia it started | Phase 2 (Explicit) |
| See who looked -- every support session appears in the freelancer's activity trail with when it started and ended | FEAT-31.SPEC-003, FEAT-31.SPEC-004 | The open automation timestamps the start of the session and the auto-close automation timestamps its end; both write the append-only trail entry that Immutable Activity & Audit Trail (FEAT-13) displays to Nadia | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-31.SPEC-004 | Support Session Auto-Close on Inactivity | Phase 3 (Entity-Lifecycle Analysis) / Phase 4 (Trigger-Response) | The Support Access Session's State Transition cell (Opened → Closed) is required by product-features.md's Validation & Limits field ("A session ends automatically after a period of inactivity") but is distinct from the operator-initiated open in SPEC-003, since it fires on a time-based trigger rather than Dana's own action |
| FEAT-31.SPEC-005 | Support Access Authorization & Read-Only Rules | Phase 5 (Rule-Constraint Discovery) | The Access, Validation & Limits, and Communications fields together produce 5+ interacting rules (Dana-only opening, one-account-at-a-time, unconditional read-only with no exception, the excluded actions, Nadia's standing view right, and Owen/Priya's total exclusion) shared across SPEC-001 through SPEC-004 -- past the inline-validation threshold |
| FEAT-31.SPEC-006 | Support Request Confirmation | Phase 4 (Notification surfacing) | The Communications field names a confirmation email to Nadia with a defined trigger and audience -- not a same-screen toast with no delivery rules, so it needs a standalone Notification spec |
| FEAT-31.SPEC-007 | Support Session Opened Notice | Phase 4 (Notification surfacing) | The Communications field names a distinct email, triggered by session opening rather than request submission, with its own audience and content -- a second Notification spec rather than folding it into SPEC-006 |

## Entity-Lifecycle Coverage Matrix

**Entity: Support Access Session**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-31.SPEC-001 | Nadia's support-request submission creates the record with `freelancer_account` and `request_text` populated; `operator`, `opened_at`, and `closed_at` remain unset until Dana opens it | Matches the dependency map's lifecycle: "Created by FEAT-31 (support request, then session)" |
| Read (single) | FEAT-31.SPEC-002 | Dana opens one queued request to work it; the mirrored read-only view is scoped to that one Freelancer Account for the session's duration | Also read by Nadia through her activity trail, which is FEAT-13's display, not a screen of this feature |
| Read (list) | FEAT-31.SPEC-002 | The Operator Support Session Console lists every request not yet opened, oldest first, so Dana can pick her next session | Validation & Limits scopes each session to one freelancer account at a time, so the list is Dana's queue across accounts, never a merged multi-account view |
| Update | FEAT-31.SPEC-003 (sets `operator` and `opened_at`), FEAT-31.SPEC-004 (sets `closed_at`) | Opening fills in who is looking and when; auto-close fills in when it ended | The dependency map states the record is "Never edited after closing" -- no update path exists once `closed_at` is set |
| Delete/Archive | N/A -- no delete or archive path inside this feature | The dependency map states the entity is "Deleted by FEAT-24" only, as part of account deletion; a closed session is permanent evidence for the life of the account, with no independent purge, soft-delete, or restore behavior of its own | See Non-Goals for the explicit rationale |
| State Transition | FEAT-31.SPEC-003 (Requested → Opened), FEAT-31.SPEC-004 (Opened → Closed, automatic on inactivity) | -- | No manual close exists in the Key Capabilities or Validation & Limits fields -- every closure is either the inactivity timeout or, per SPEC-003, Dana ending her own diagnosis, both handled as the same Opened → Closed transition |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Freelancer Account | FEAT-31.SPEC-002 | The mirrored read-only view during an open session reads the named freelancer's data across FEAT-01 through FEAT-25, exactly as she would see it, with every write control disabled |
| Activity Log Entry | FEAT-31.SPEC-002 (indirectly, via FEAT-13's display) | Support session entries a freelancer sees in her own trail are written by FEAT-13 on this feature's behalf; this feature never reads or renders the trail itself |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Nadia submits a support request | Create the Support Access Session record with her request text | Inline in SPEC-001 (simple, single-step data write) | SPEC-001 |
| A support request is created | Send Nadia a confirmation email | Standalone Notification | SPEC-006 |
| Dana selects a queued request and opens it | Fill in `operator` and `opened_at`, and enforce read-only across every screen and control for that freelancer account | Standalone Automation | SPEC-003 |
| A session opens | Send Nadia an email notice that a support session has started | Standalone Notification | SPEC-007 |
| A session opens or closes | Write an append-only Activity Log Entry with the session's start or end time | Cross-feature -- owned by Immutable Activity & Audit Trail (FEAT-13) | FEAT-13 responsibility |
| A session sits idle past the inactivity period | Close the session automatically, set `closed_at`, and return Dana to the queue | Standalone Automation | SPEC-004 |
| Dana attempts an edit, send, approve, pay, file-download, or export action during an open session | Block the action; the control is not shown as available in the first place | Standalone Logic/Rule, enforced inline by SPEC-002/SPEC-003 | SPEC-005 |
| A session cannot open (the account's data cannot be loaded read-only) | Nothing about the account changes; Dana sees why it failed | Inline in SPEC-002 (Error state) | SPEC-002 |
| Dana diagnoses the problem | Dana replies to Nadia by email from outside the product | Non-goal -- a personal reply, not a system-triggered notification (see Non-Goals) | -- |
| A fix requires a change to Nadia's records | Dana tells Nadia what to change; Nadia makes the change herself in her own session | Cross-feature -- the change itself is recorded under Nadia's own actor identity by whichever feature owns that record | Feature-specific responsibility |
| Nadia's account is deleted | Every Support Access Session belonging to it is removed with the account | Cross-feature -- owned by Data Export & Account Deletion (FEAT-24) | FEAT-24 responsibility |

## Shared Context

**Shared Entities:**
- Support Access Session -- created by SPEC-001 (request text and freelancer account), updated by SPEC-003 (opened) and SPEC-004 (closed), listed and read by SPEC-002, and referenced by SPEC-005 for the one-account-at-a-time and read-only constraints. Fields: `freelancer_account`, `request_text`, `operator`, `opened_at`, `closed_at`.
- Freelancer Account -- read-only across SPEC-002's mirrored view; this feature never writes any Freelancer Account field.

**Shared UI Patterns:**
- Permanent read-only banner -- SPEC-002's mirrored screens all carry the same "Read-only support session" banner and disabled-control treatment, driven by SPEC-003's enforcement, so the read-only state is one implementation pattern reused across every screen Dana views rather than a separate spec per underlying feature.
- Reason-first error display -- both the request-confirmation failure (SPEC-001) and the session-open failure (SPEC-002) show the specific reason nothing changed, never a generic error, consistent with the feature's States field.

**Shared Validation:**
- SPEC-005 defines the Dana-only opening rule, the one-account-at-a-time scope, the unconditional read-only rule and its excluded actions, Nadia's standing view right, and Owen/Priya's total exclusion. SPEC-001 through SPEC-004 all reference SPEC-005 rather than restating these rules.

## Internal Dependency Map

```
SPEC-001 (Contact Support Screen) -> [Nadia submits a request] -> SPEC-006 (Support Request Confirmation)
SPEC-001 (Contact Support Screen) -> [request created] -> SPEC-002 (Operator Support Session Console queue)
SPEC-002 (Operator Support Session Console) -> [Dana selects a queued request] -> SPEC-005 (Support Access Authorization & Read-Only Rules) -> [pass] -> SPEC-003 (Support Session Open & Read-Only Enforcement)
SPEC-003 (Support Session Open & Read-Only Enforcement) -> [session opened] -> SPEC-007 (Support Session Opened Notice)
SPEC-003 (Support Session Open & Read-Only Enforcement) -> [session opened] -> SPEC-002 (Operator Support Session Console shows the mirrored read-only view)
SPEC-002 (Operator Support Session Console) -> [session idle past the inactivity period] -> SPEC-004 (Support Session Auto-Close on Inactivity)
SPEC-004 (Support Session Auto-Close on Inactivity) -> [session closed] -> SPEC-002 (Operator Support Session Console returns to the queue)
SPEC-003 (Support Session Open & Read-Only Enforcement), SPEC-004 (Support Session Auto-Close on Inactivity) -> [opened / closed] -> FEAT-13 (Immutable Activity & Audit Trail)
```

**Default Entry:** SPEC-001 (Contact Support Screen) for Nadia, reached from Notifications & Help; SPEC-002 (Operator Support Session Console) for Dana, her sole entry point into this feature and, during an open session, into every other feature's screens in read-only form.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-31.SPEC-003, FEAT-31.SPEC-004 | Outbound | FEAT-13 (Immutable Activity & Audit Trail) | Every session open and close writes an append-only trail entry that FEAT-13 displays in Nadia's activity trail (XBR-05) | A session opens or closes |
| FEAT-31.SPEC-006, FEAT-31.SPEC-007 | Outbound | FEAT-14 (Notifications (Email)) | Both emails are sent through the transactional email delivery capability owned by FEAT-14; this feature carries no Integration spec of its own for that capability | A support request is received, or a session opens |
| FEAT-31.SPEC-002 | Inbound | FEAT-01 through FEAT-25 | The mirrored read-only view reads each feature's own screens and data exactly as Nadia would see them, with every write control disabled | Dana opens a session |
| FEAT-31.SPEC-002 | Inbound | FEAT-06 (Deliverable Upload & Sharing) | The read-only view never offers a file-download control, per SPEC-005 | Dana views a deliverable inside a session |
| FEAT-31.SPEC-002 | Inbound | FEAT-16 (Large File Handling & Storage), FEAT-22 (Accounting Export) | The read-only view never offers data or accounting export generation, per SPEC-005 | Dana views a project or the financial dashboard inside a session |
| FEAT-31.SPEC-002 | Inbound | FEAT-32 (Payment Account Connection) | Dana sees connection status only, never the processor account reference or credentials | Dana views payment settings inside a session |
| FEAT-31.SPEC-005 | Outbound | FEAT-24 (Data Export & Account Deletion) | Support Access Sessions are removed only as part of account deletion; this feature defines no independent delete path | Nadia deletes her account |

## Non-Functional Notes

**Data volumes / growth:** Volume scales with support demand rather than with the freelancer roster -- a solo founder-operator (BRIEF.md, Constraints) opens sessions one at a time, so this feature carries no growth concern of its own beyond the append-only trail entries FEAT-13 retains. This feature emits `support_request_sent`, `support_session_opened`, and `support_session_closed` signals (product-features.md, Signals field); SPEC-001 fires the first, SPEC-003 the second, and SPEC-004 the third.

**Responsiveness:** No offline or degraded mode applies -- product-features.md's States field marks Offline-degraded "N/A — support sessions are an operator-side, connectivity-required action," so SPEC-001 and SPEC-002 assume connectivity throughout. Dana's mirrored view loads like Nadia's own screens (product-features.md, States field), so it inherits each underlying feature's own loading behavior rather than defining a new one.

**Data sensitivity / privacy:** Support Access Session records hold the freelancer's support-request text and the operator's identity, both personal data (assumptions-constraints.md, ASMP-24), and the session itself grants read-only visibility of the account's data, so it is always logged and announced (feature-dependency-map.md, Support Access Session entity; ASMP-18, ASMP-23). Card data is never visible in a session because the product never holds it at all (BRIEF.md, Constraints; product-features.md, Validation & Limits).

**Compliance flags:** ASMP-23 requires that any operator access be read-only and visible to the freelancer -- this feature is the sole mechanism that satisfies that posture for the whole product. ASMP-27 requires the mirrored read-only screens to remain usable with a screen reader and keyboard and to never rely on colour alone to signal the read-only state, so the "Read-only support session" banner must carry text, not just a colour treatment.

## Non-Goals

- **The operator changing, sending, approving, paying, downloading deliverable files, or generating data or accounting exports during a session** -- Excluded per product-features.md's Validation & Limits field and scope-boundaries.md (SC-04): sessions are read-only without exception, and the operator diagnoses only, never acts.
- **The operator signing in as a client contact to diagnose a client-side problem** -- Excluded per product-features.md's Primary Flows & Alternates ("Dana never signs in as a client contact") and scope-boundaries.md (SC-04); client-side sign-in or email trouble is diagnosed from the freelancer's own side (contact list, delivery warnings) instead.
- **A manual or on-demand close control for Dana** -- Not established by any Stage 2 field; the Validation & Limits field states only that "a session ends automatically after a period of inactivity," so this feature defines a single automatic close path (SPEC-004) rather than inventing a manual one.
- **Independent deletion, archival, or purge of a closed Support Access Session** -- Excluded per the dependency map's lifecycle statement that the entity is "Never edited after closing" and "Deleted by FEAT-24" only; a closed session is permanent evidence for the life of the account, removed solely as part of account deletion.
- **System-templated delivery of the operator's diagnostic reply** -- The Primary Flows & Alternates field states Dana "replies by email," and the Communications field lists only the request-confirmation and session-opened emails as product-sent notices; the reply itself is Dana's own personal email, outside this feature's Notification specs.
- **Owen or Priya seeing, opening, or being notified about a support session** -- Excluded per the Access Matrix (user-persona.md): both carry "None" for Support Access, and product-features.md's Access field states plainly that "Owen and Priya have no access and never see support sessions."
- **Scoped or delegated support-operator seats for additional staff** -- Excluded per scope-boundaries.md (SC-01): the product models solo freelancer accounts with no internal-staff seat model, and BRIEF.md, Target Users & Roles describes exactly one operator identity (the founder, as Dana).

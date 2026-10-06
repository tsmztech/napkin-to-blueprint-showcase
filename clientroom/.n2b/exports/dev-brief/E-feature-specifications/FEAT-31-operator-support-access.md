# FEAT-31 — Operator Support Access

This chapter covers Operator Support Access, a Important-tier feature. It contains the feature breakdown brief followed by every specification in full: 7 specifications carrying 82 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-31.SPEC-001 | Contact Support Screen | screen | 9 |
| FEAT-31.SPEC-002 | Operator Support Session Console | screen | 15 |
| FEAT-31.SPEC-003 | Support Session Open & Read-Only Enforcement | automation | 9 |
| FEAT-31.SPEC-004 | Support Session Auto-Close on Inactivity | automation | 9 |
| FEAT-31.SPEC-005 | Support Access Authorization & Read-Only Rules | logic-rule | 24 |
| FEAT-31.SPEC-006 | Support Request Confirmation | notification | 8 |
| FEAT-31.SPEC-007 | Support Session Opened Notice | notification | 8 |

The feature breakdown brief follows, then every specification in full.


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



# Screen Spec: Contact Support Screen

## Overview

**Name:** Contact Support Screen
**ID:** FEAT-31.SPEC-001
**Type:** Screen
**Purpose:** Nadia describes a problem she is having and sends a support request from inside the product, and receives an on-screen and emailed confirmation that it was received.
**Parent Feature:** FEAT-31 -- Operator Support Access

## Scope and Non-Goals

**In Scope:**
- Capturing Nadia's free-text description of her problem and submitting it as a support request
- Creating the Support Access Session record in its unopened state (`freelancer_account` and `request_text` populated; `operator`, `opened_at`, `closed_at` unset)
- An on-screen confirmation that the request was received, immediately on submit
- Triggering the emailed confirmation (FEAT-31.SPEC-006)

**Non-Goals:**
- Editing or withdrawing a request after it is sent -- no such capability is established by product-features.md's Key Capabilities or Validation & Limits fields for this feature; a request is a one-way message to support, like the email thread it replaces (BRIEF.md, Problem Statement).
- Any view of the operator's diagnosis, reply, or session activity from this screen -- Dana's reply is sent by email outside the product (product-features.md, Primary Flows & Alternates), and the session itself is visible only in Nadia's activity trail (FEAT-13), never here.
- A live chat or real-time conversation with Dana -- product-features.md's Communications field names only the confirmation and session-opened emails; this feature defines no synchronous channel, consistent with the founder being a solo operator (BRIEF.md, Constraints).

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| Notifications & Help (product-wide entry) | Nadia selects "Contact support" | None -- form starts empty |
| FEAT-32.SPEC-001 (Payment Connection Screen) | Nadia taps "Contact support" from the Needs attention state | None -- form starts empty |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Submit a support request describing her own problem | -- |
| Owen (Client Primary Contact) | No | No | The "Contact support" entry is not shown anywhere in his portal view; the Access Matrix gives client contacts no Support Access |
| Priya (Client Reviewer Contact) | No | No | Same as Owen -- not shown in her portal view |
| Dana (Support Operator) | No | No | Dana works from her own console (FEAT-31.SPEC-002), never from this screen; it is a freelancer-facing entry point only |
| Unauthenticated | No | No | Redirected to sign-in; after signing in, the user lands on her own dashboard, not this screen directly |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- any typed but unsent problem description is preserved locally and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Contact Support" with a back arrow (returns to wherever Nadia entered from) and a "Send" action button (right-aligned).

**Body:** A single-column form with one field:
- Problem description (multi-line text input, required) -- placeholder text: "Describe what's going wrong. We'll get back to you by email."

**Footer:** None -- Send is in the header.

### Responsive Behavior

- **Compact breakpoint:** Single-column form, full width; Send remains in the header.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.
- **Problem description field:** Grows from 4 visible lines (compact) to 8 visible lines (medium and above).

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to the screen Nadia entered from | Screen closes | Standard transition back |
| Problem description input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Problem description input | Blur (empty) | Triggers field validation via FEAT-31.SPEC-005 | Error state on field | "Describe your problem before sending." below field |
| Send button | Tap | 1. Validate the field via FEAT-31.SPEC-005. 2. If valid, create the Support Access Session record with `freelancer_account` and `request_text`. 3. Trigger FEAT-31.SPEC-006 (Support Request Confirmation). | Button shows loading state during send | Success: on-screen confirmation "We received your request -- Dana will get back to you by email." and the form clears. Failure: inline error message, entered text preserved. |
| Send button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> Problem description input -> Send.
- **Validation announcements:** When the field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Send feedback:** The on-screen confirmation is announced on success; on validation failure, focus moves to the problem description field.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (default) | Problem description empty, Send enabled | Screen first opens | Nadia begins typing |
| Filling | Field contains typed text, Send enabled | Nadia types in the field | Nadia taps Send or navigates away |
| Sending | Send button shows loading spinner, field disabled | Nadia taps Send with valid text | Send completes or fails |
| Confirmation | On-screen message "We received your request -- Dana will get back to you by email." replaces the form | Send completes successfully | Nadia navigates away (confirmation is not persisted as a screen state to return to) |
| Error | Error banner at top of form: "Could not send your request. Check your connection and try again." with a Retry option | Send operation fails | Nadia taps Retry or navigates away |
| Offline/Degraded | N/A -- product-features.md's States field marks this screen's Offline-degraded state "N/A -- support sessions are an operator-side, connectivity-required action," and this screen assumes connectivity throughout (Non-Functional Notes, Responsiveness) | -- | -- |

## Validation Rules

**Option A -- Reference Logic/Rule spec:**
Validation governed by FEAT-31.SPEC-005 (Support Access Authorization & Read-Only Rules). See that spec for the `request_text` field rule. This screen checks it on field blur and on form submit.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | Wherever Nadia entered from | -- |
| Successful send | Same screen, showing the Confirmation state | -- |

## Data Model

**Creates:** Support Access Session -- `freelancer_account` (Nadia's own account) and `request_text` (the submitted description) are set; `operator`, `opened_at`, and `closed_at` remain unset until Dana opens a session (FEAT-31.SPEC-003).
**Reads:** None (this is a submission screen -- no existing Support Access Session data is loaded).
**Updates:** None.
**Deletes:** None.

## Business Rules

- Field validation and authorization for the Support Access Session record are governed by FEAT-31.SPEC-005 -- this screen enforces them but does not restate them.
- Submitting a request creates exactly one Support Access Session record; there is no draft state -- the request is sent the moment Send succeeds.
- XBR-29: this request is the record's origin; every downstream open and close of the resulting session is read-only and always announced to Nadia, per the rule this feature owns.

## Edge Cases

- **Nadia navigates away with unsent text in the field** -- Confirmation dialog: "You have an unsent message. Discard?" with "Discard" and "Keep Editing" options.
- **Nadia taps Send twice rapidly** -- Second tap is ignored while the first send is in progress (button in loading state).
- **Network failure during send** -- Error banner: "Could not send your request. Check your connection and try again." with a Retry button. Entered text preserved.
- **Nadia submits a second request while an earlier one is still unopened** -- Both requests are created as separate Support Access Session records and both appear independently in Dana's queue (FEAT-31.SPEC-002); this screen places no limit on how many pending requests Nadia may have.
- **Concurrent-edit conflict** -- Not applicable: the dependency map's Contention note for Support Access Session states "None -- only Dana opens and closes a session, one account at a time, and the record is never edited after it closes; Nadia only reads it." This screen only creates a new record each time; it never loads or resaves an existing one, so no load-then-save race exists to resolve.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-31.SPEC-005 (Support Access Authorization & Read-Only Rules) | References (inbound) | Field validation for `request_text` |
| FEAT-31.SPEC-006 (Support Request Confirmation) | Triggers (outbound) | Successful send triggers the emailed confirmation |
| FEAT-31.SPEC-002 (Operator Support Session Console) | Navigation (outbound, indirect) | The created request appears in Dana's queue; Nadia never navigates there herself |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| support_request_sent | none beyond the event itself | Send completes successfully | N/A -- no metric in success-metrics.md is connected to Operator Support Access or names this behavior; retained per product-features.md's Signals field so support activity stays observable |
| support_request_send_failed | reason (network / validation) | Send operation fails | N/A -- same reason as above |

## Acceptance Criteria

**FEAT-31.SPEC-001-AC-01:** Given Nadia is on the Contact Support screen, when she types a description of her problem and taps Send, then the Support Access Session record is created with her account and description, an on-screen confirmation "We received your request -- Dana will get back to you by email." appears, and the confirmation email (FEAT-31.SPEC-006) is triggered.

**FEAT-31.SPEC-001-AC-02:** Given Nadia is on the Contact Support screen, when she taps Send with the problem description empty, then the field shows the error "Describe your problem before sending." and the send does not proceed.

**FEAT-31.SPEC-001-AC-03:** Given Nadia has typed an unsent description, when she taps the back arrow, then a confirmation dialog appears asking "You have an unsent message. Discard?" with "Discard" and "Keep Editing" options.

**FEAT-31.SPEC-001-AC-04:** Given Nadia loses connectivity while filling the form, when she taps Send, then the error banner "Could not send your request. Check your connection and try again." appears and her entered text is preserved.

**FEAT-31.SPEC-001-AC-05:** Given Owen or Priya is signed in to their portal, when they look for a way to contact support, then no such entry exists anywhere in their view.

**FEAT-31.SPEC-001-AC-06:** Given Nadia already has one unopened support request pending, when she submits a second, different request, then both appear as separate entries in Dana's queue (FEAT-31.SPEC-002).

**FEAT-31.SPEC-001-AC-07:** Given Nadia's session has expired while she was typing a problem description, when the expiry is detected, then the dialog "Your session has expired. Sign in to continue." appears and her typed text is restored after she signs back in.

**FEAT-31.SPEC-001-AC-08:** Given Nadia taps Send twice in rapid succession, when the first send is still in progress, then the second tap has no effect and the button remains in its loading state.

**FEAT-31.SPEC-001-AC-09:** Given Nadia's send request fails on the server, when the failure is returned, then the error banner "Could not send your request. Check your connection and try again." appears with a Retry option and her entered text remains in the field.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 6 (empty, filling, sending, confirmation, error, offline/degraded N/A) | 6 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



# Screen Spec: Operator Support Session Console

## Overview

**Name:** Operator Support Session Console
**ID:** FEAT-31.SPEC-002
**Type:** Screen
**Purpose:** Dana sees the queue of open support requests, opens a read-only session on one named freelancer's account, works through that freelancer's own screens under a permanent read-only banner, and steps away when done -- the session then ends on its own after inactivity.
**Parent Feature:** FEAT-31 -- Operator Support Access

## Scope and Non-Goals

**In Scope:**
- The queue of pending (unopened) support requests, oldest first, across every freelancer account
- Opening a read-only session on one named freelancer account at a time
- Hosting the mirrored, read-only view of that account's own screens (FEAT-01 through FEAT-25) under the permanent "Read-only support session" banner
- Returning Dana to the queue when a session closes, whether by inactivity (FEAT-31.SPEC-004) or by her own navigation away

**Non-Goals:**
- A manual "End Session" or "Close" control -- not established by any Stage 2 field; product-features.md's Validation & Limits states only that "a session ends automatically after a period of inactivity," so this screen defines no on-demand close action of its own. Dana ends her diagnosis simply by stepping away; the only mechanism that actually closes the record is the automatic inactivity close (FEAT-31.SPEC-004).
- The content and controls of the mirrored screens themselves -- each underlying feature (FEAT-01 through FEAT-25) owns its own screen's layout and content; this spec owns only the queue, the session-open action, the permanent banner, and the read-only enforcement surface, per FEAT-31.SPEC-003.
- File downloads and data or accounting export generation from inside a session -- excluded per scope-boundaries.md (SC-04) and FEAT-31.SPEC-005; these controls are never offered on any mirrored screen during a session.
- Signing in as a client contact to diagnose a client-side problem -- excluded per product-features.md's Primary Flows & Alternates ("Dana never signs in as a client contact") and scope-boundaries.md (SC-04).

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| External (Dana's own operator access, outside the freelancer- and client-facing product) | Dana opens the console | None -- queue loads fresh |
| FEAT-31.SPEC-004 (Support Session Auto-Close on Inactivity) | An open session closes automatically | Notice naming the freelancer whose session just closed |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Dana (Support Operator) | Full screen | Open a session on one queued request at a time; navigate the mirrored read-only view; return to the queue | -- |
| Nadia (Freelancer) | No | No | This screen does not exist anywhere in her product; she never reaches it |
| Owen (Client Primary Contact) | No | No | Same -- not reachable from the client portal |
| Priya (Client Reviewer Contact) | No | No | Same -- not reachable from the client portal |
| Unauthenticated | No | No | Redirected to Dana's own sign-in; this console is never reachable without her operator credentials |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- if a support session was open, it remains open in the background (subject to its own inactivity clock, FEAT-31.SPEC-004) and the mirrored view resumes once Dana signs back in |

## Layout and Content

**Header:** Screen title "Support Session Console." When no session is open: no further header content. When a session is open: a full-width, persistent banner reading "Read-only support session -- {freelancer_account name}" replaces the default header treatment and remains visible on every mirrored screen for the duration of the session.

**Body (queue view, no session open):** A list of pending Support Access Session requests, oldest first, each row showing: the freelancer's account name, a one-line preview of the request text, and the time the request was submitted. Each row carries an "Open Session" action. While the queue is being fetched, the body shows the text "Loading support requests..." in place of the list. If the fetch fails, the body shows the text "Support requests could not be loaded right now." with a "Retry" button beneath it.

**Body (session open):** The mirrored screen for whatever feature Dana is currently viewing (FEAT-01 through FEAT-25), rendered exactly as Nadia would see it, with every edit, send, approve, pay, file-download, and export-generation control disabled or not shown (FEAT-31.SPEC-003, FEAT-31.SPEC-005). A "Return to queue" link sits below the banner at all times.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Queue list is single-column, full width; the read-only banner spans the full width above the mirrored content and stays fixed at the top on scroll.
- **Medium size class and above:** Queue list remains single-column, capped at a consistent platform-wide list width; the banner remains full-width and fixed at the top; mirrored screens inherit each underlying feature's own responsive behavior unchanged.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Queue row "Open Session" | Tap | 1. Check authorization via FEAT-31.SPEC-005 (Dana has no other session open). 2. If authorized, trigger FEAT-31.SPEC-003 (Support Session Open & Read-Only Enforcement). | Button shows loading state; on success the screen transitions from queue to mirrored view with the banner | Success: banner appears, mirrored view loads. Failure: exact denied or error message shown inline (see Edge Cases) |
| Queue row "Open Session" (while another session is already open) | Tap | No action taken -- blocked before FEAT-31.SPEC-003 is triggered | Control remains present but the attempt is refused; the open session stays open | Inline message on the row: "Your support session on {freelancer_account name} is still open and ends automatically after inactivity. Tap Resume to return to it." The persistent "Session open on {freelancer_account name} -- Resume" notice remains shown above the queue. Meanwhile Dana can tap Resume to keep working in the open session, or step away and wait for the automatic close, after which every row's "Open Session" works again |
| Queue "Retry" button (queue load-error state) | Tap | Re-fetches the queue of pending requests | Body returns to the queue Loading state, then to Queue populated, Empty queue, or (on repeated failure) the queue Load-error state again | "Loading support requests..." while fetching; on repeated failure the same "Support requests could not be loaded right now." text with "Retry" remains |
| Row "Retry" button (Error state, session cannot open) | Tap | Re-runs the "Open Session" action for that same request: authorization check (FEAT-31.SPEC-005), then FEAT-31.SPEC-003 | Row returns to the Opening state | Same feedback as "Open Session": banner and mirrored view on success; the row's inline reason and "Retry" button again on repeated failure |
| Mirrored screen edit/send/approve/pay/download/export controls | Tap (any) | Blocked before reaching the underlying feature's own logic, per FEAT-31.SPEC-005 | No state change | "Not available in a support session." (or, for downloads/exports specifically, "Downloads are not available in a support session." / "Exports are not available in a support session.") |
| "Return to queue" link | Tap | Navigates back to the queue view; the session itself is not closed by this action | Queue view shown | Queue list appears; if the session is still open, its row (if still pending elsewhere) is replaced by a persistent notice: "Session open on {freelancer_account name} -- Resume" |
| "Resume" notice (session still open) | Tap | Returns to the mirrored view at its last screen | Mirrored view with banner reappears | Same banner and mirrored screen as before navigating away |

### Accessibility Notes

- **Focus order:** Queue rows in submitted order (oldest first), each with its "Open Session" control; inside a session, the banner text is announced first, followed by the mirrored screen's own focus order, followed by "Return to queue."
- **Dynamic announcements:** The read-only banner is announced to assistive technology the instant a session opens (not just visually shown), and again if Dana resumes an open session from the queue. Every blocked-control message ("Not available in a support session.") is announced at the moment of the attempt, not only shown visually -- consistent with ASMP-27's requirement that the read-only state never relies on colour alone.
- **Keyboard alternatives:** Every action on this screen, including "Open Session," "Return to queue," and "Resume," is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Queue loading | "Loading support requests..." text in the body; no rows and no "Open Session" controls shown | Dana opens the console, returns to the queue, or taps the queue "Retry" button | The queue fetch succeeds (Empty queue or Queue populated) or fails (Queue load error) |
| Queue load error | "Support requests could not be loaded right now." text with a "Retry" button in the body; no rows shown; nothing about any request changes | The queue fetch fails | Dana taps "Retry" (returns to Queue loading) |
| Empty queue | "No open support requests." message in the body | No pending requests exist | A new request is submitted (FEAT-31.SPEC-001) |
| Queue populated | List of pending requests as described in Layout and Content | One or more pending requests exist | Dana opens one |
| Opening | Loading indicator over the queue row Dana selected | Dana taps "Open Session" and authorization passes | FEAT-31.SPEC-003 completes (success or failure) |
| Session open (mirrored view) | Permanent read-only banner plus the mirrored screen for whichever feature Dana is viewing; loads like Nadia's own screens (Non-Functional Notes, Responsiveness) | FEAT-31.SPEC-003 completes successfully | Session closes (FEAT-31.SPEC-004) |
| Queue (session open elsewhere) | Queue list shown with a persistent "Session open on {freelancer_account name} -- Resume" notice in place of that request's row | Dana taps "Return to queue" while a session remains open | Dana taps "Resume," or the session closes automatically (FEAT-31.SPEC-004) |
| Error (session cannot open) | Inline message on the queue row naming the specific reason (e.g., "This account's data could not be loaded right now.") with a "Retry" button on that row; nothing about the account changes | FEAT-31.SPEC-003 reports a load failure | Dana taps the row's "Retry" button (returns to Opening) or taps "Open Session" on a different request |
| Offline/Degraded | N/A -- product-features.md's States field marks this feature's Offline-degraded state "N/A -- support sessions are an operator-side, connectivity-required action" | -- | -- |

## Validation Rules

**Option A -- Reference Logic/Rule spec:**
Authorization for opening a session, and every read-only enforcement rule applied to the mirrored view, is governed by FEAT-31.SPEC-005 (Support Access Authorization & Read-Only Rules). This screen checks authorization the instant "Open Session" is tapped, and applies the read-only enforcement continuously for the session's duration.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Open Session (success) | The freelancer's own default landing screen (mirrored, read-only) | FEAT-01 through FEAT-25 (whichever the freelancer would land on) |
| Navigating within a session | Whichever mirrored screen Dana selects | FEAT-01 through FEAT-25 |
| Return to queue | This screen (queue view) | -- |
| Session auto-closes | This screen (queue view), with a notice | -- |

## Data Model

**Creates:** None (FEAT-31.SPEC-001 creates the Support Access Session record).
**Reads:** Support Access Session -- list of records with `operator` unset (the queue), and the single selected record's `freelancer_account` and `request_text` once opened. Freelancer Account and its dependent data across FEAT-01 through FEAT-25 -- read-only, exactly as Nadia would see it, for the duration of an open session.
**Updates:** None directly -- FEAT-31.SPEC-003 sets `operator` and `opened_at`, and FEAT-31.SPEC-004 sets `closed_at`; this screen triggers and displays those changes but does not write them itself.
**Deletes:** None.

## Business Rules

- Only one Support Access Session may be open at a time for Dana (one-account-at-a-time), per FEAT-31.SPEC-005 and XBR-29.
- Every control an edit, send, approve, pay, file-download, or export action would use is unavailable for the entire duration of an open session, with no exception, per FEAT-31.SPEC-003 and FEAT-31.SPEC-005.
- This screen provides no manual "End Session" control -- the only path that closes an open session is the automatic inactivity close (FEAT-31.SPEC-004), per the Brief's Non-Goals ("A manual or on-demand close control for Dana"). Dana ends her diagnosis simply by stepping away; the session record remains technically open, enforcing read-only, until inactivity closes it automatically.
- The queue lists requests across every freelancer account but never merges more than one account's data into a single mirrored view -- each open session is scoped to exactly one Freelancer Account (dependency map, Read (list) note).
- XBR-29: sessions are read-only in every feature, cover one account at a time, end after inactivity, exclude file downloads and data/accounting exports, are always announced to the freelancer by email (FEAT-31.SPEC-007), and are always listed in her trail (FEAT-13).

## Edge Cases

- **A session cannot open (the account's data cannot be loaded read-only)** -- Nothing about the account changes; Dana sees the specific reason inline on the queue row, per product-features.md's Error state definition.
- **Dana taps "Open Session" twice rapidly** -- Second tap is ignored while the first attempt is in progress (row shows loading state).
- **Dana attempts to open a second session while one is already open** -- Refused before FEAT-31.SPEC-003 is triggered, with "Your support session on {freelancer_account name} is still open and ends automatically after inactivity. Tap Resume to return to it."; nothing about either account changes. Dana can Resume the open session or wait for it to close automatically; no manual close exists.
- **The queue fetch fails or is slow** -- The body shows the queue Loading state, then on failure the queue Load-error state with "Retry"; an open session (if any) is unaffected and its "Resume" notice remains available.
- **A session Dana had open closes automatically (inactivity) while she is mid-navigation on a mirrored screen** -- She is returned to the queue view immediately, with a notice naming the freelancer and stating the session closed after inactivity; because the session is unconditionally read-only, no in-progress work is ever lost.
- **The freelancer account tied to a queued request is deleted (FEAT-24) before Dana opens it** -- The request disappears from the queue as part of that account's full removal; nothing is shown to Dana beyond the row no longer being present.
- **Concurrent-edit conflict** -- Not applicable to this screen directly: the dependency map's Contention note for Support Access Session states "None -- only Dana opens and closes a session, one account at a time, and the record is never edited after it closes; Nadia only reads it." The mirrored view's own underlying entities are read-only here (Dana never writes), so no load-then-save race exists on this screen for Dana to encounter; any contention among Nadia's own concurrent sessions is each underlying feature's own concern, unaffected by Dana's read-only presence.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-31.SPEC-001 (Contact Support Screen) | Navigation (inbound, indirect) | A submitted request populates this screen's queue |
| FEAT-31.SPEC-003 (Support Session Open & Read-Only Enforcement) | Triggers (outbound) | "Open Session" fires this automation |
| FEAT-31.SPEC-004 (Support Session Auto-Close on Inactivity) | Triggered by (inbound) | Auto-close returns Dana to the queue with a notice |
| FEAT-31.SPEC-005 (Support Access Authorization & Read-Only Rules) | References (inbound) | Authorization and every read-only enforcement rule applied here |
| FEAT-32 (Payment Account Connection) | Navigation (outbound) | Inside a session, Dana sees only the payment connection status, never the processor account reference or credentials (feature-dependency-map.md, Cross-Feature Touchpoints) |
| FEAT-01 through FEAT-25 (various features) | Navigation (outbound) | The mirrored read-only view reads each feature's own screens and data exactly as Nadia would see them, with every write control disabled |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| support_session_open_attempted | outcome (opened / denied / load_failed) | Dana taps "Open Session" | N/A -- no metric in success-metrics.md is connected to Operator Support Access or names this behavior; retained per product-features.md's Signals field so support activity stays observable |
| support_session_resumed | -- | Dana taps "Resume" on an open session from the queue | N/A -- same reason as above |

## Acceptance Criteria

**FEAT-31.SPEC-002-AC-01:** Given Dana opens the Support Session Console with two pending requests, when the queue loads, then both appear oldest first, each with the freelancer's account name, a preview of the request text, and the submission time.

**FEAT-31.SPEC-002-AC-02:** Given Dana has no session open, when she taps "Open Session" on a queued request, then FEAT-31.SPEC-003 opens the session and the mirrored view appears under the "Read-only support session -- {freelancer_account name}" banner.

**FEAT-31.SPEC-002-AC-03:** Given Dana is inside an open session, when she looks at any edit, send, approve, or pay control on a mirrored screen, then it is disabled or not shown, and a direct attempt shows "Not available in a support session."

**FEAT-31.SPEC-002-AC-04:** Given Dana is inside an open session, when she attempts to download a deliverable file, then the control is not offered and a direct attempt shows "Downloads are not available in a support session."

**FEAT-31.SPEC-002-AC-05:** Given Dana already has a session open on one freelancer account, when she taps "Open Session" on a different queued request, then the attempt is refused with "Your support session on {freelancer_account name} is still open and ends automatically after inactivity. Tap Resume to return to it.", the "Resume" notice remains shown, and no second session opens.

**FEAT-31.SPEC-002-AC-06:** Given Dana is inside an open session, when she taps "Return to queue," then the queue view appears with a persistent "Session open on {freelancer_account name} -- Resume" notice, and the session itself remains open.

**FEAT-31.SPEC-002-AC-07:** Given Dana's open session has just closed automatically after inactivity (FEAT-31.SPEC-004) while she was viewing a mirrored screen, then she is returned to the queue view immediately with a notice naming the freelancer and stating the session closed after inactivity.

**FEAT-31.SPEC-002-AC-08:** Given a queued request's account data cannot be loaded read-only, when Dana taps "Open Session" on it, then nothing about the account changes and she sees the specific reason inline on that row.

**FEAT-31.SPEC-002-AC-09:** Given Dana taps "Open Session" twice in rapid succession on the same request, when the first attempt is still in progress, then the second tap has no effect.

**FEAT-31.SPEC-002-AC-10:** Given no support requests are pending, when Dana opens the console, then it shows "No open support requests."

**FEAT-31.SPEC-002-AC-11:** Given Dana is inside a session viewing payment settings, when she looks for the payment connection detail, then she sees connection status only, never the processor account reference or credentials.

**FEAT-31.SPEC-002-AC-12:** Given Dana opens the console, when the queue fetch is in progress, then the body shows "Loading support requests..." with no rows and no "Open Session" controls.

**FEAT-31.SPEC-002-AC-13:** Given the queue fetch fails, when the console finishes loading, then the body shows "Support requests could not be loaded right now." with a "Retry" button; when Dana taps "Retry" and the fetch succeeds, then the queue (or "No open support requests.") appears.

**FEAT-31.SPEC-002-AC-14:** Given a session could not open and the row shows its inline reason with a "Retry" button, when Dana taps "Retry," then the row returns to its loading state and the open action runs again for that same request.

**FEAT-31.SPEC-002-AC-15:** Given Dana has a session open and has returned to the queue, when she taps "Open Session" on a different request, then she sees no manual close instruction, only the message naming the open account, stating it ends automatically after inactivity, and pointing to Resume, and tapping "Resume" returns her to the open session.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 7 | 7 |
| States | 9 (queue loading, queue load error, empty queue, populated, opening, session open, queue with open session, error, offline/degraded N/A) | 9 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |



# Automation Spec: Support Session Open & Read-Only Enforcement

## Overview

**Name:** Support Session Open & Read-Only Enforcement
**ID:** FEAT-31.SPEC-003
**Type:** Automation
**Purpose:** Opens the Support Access Session record the instant Dana starts a session, and enforces that every edit, send, approve, pay, file-download, and export-generation control is unavailable for the session's duration.
**Parent Feature:** FEAT-31 -- Operator Support Access

## Scope and Non-Goals

**In Scope:**
- Authorization check at the moment Dana selects a queued request (FEAT-31.SPEC-005: Dana-only, one-account-at-a-time)
- Setting `operator` and `opened_at` on the selected Support Access Session record
- Loading the named freelancer's account data read-only across FEAT-01 through FEAT-25
- Applying the read-only enforcement to every control on every mirrored screen for the session's duration
- Triggering the session-opened notice (FEAT-31.SPEC-007) and the activity trail entry (FEAT-13)

**Non-Goals:**
- Closing the session -- handled exclusively by FEAT-31.SPEC-004 (Support Session Auto-Close on Inactivity); this automation has no closing logic of its own, consistent with the Brief's Non-Goals excluding any manual close path.
- Defining the exact set of excluded actions and their denied messages -- owned by FEAT-31.SPEC-005 (Support Access Authorization & Read-Only Rules); this automation applies those rules but is not their source of truth.
- Rendering the mirrored screens' own content -- each of FEAT-01 through FEAT-25 owns its own screen; this automation only enforces the read-only constraint across whichever one Dana is viewing.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Dana selects a queued request and taps "Open Session" | FEAT-31.SPEC-002 (Operator Support Session Console) | Fires when authorization (FEAT-31.SPEC-005) passes -- Dana has no other session currently open | The selected Support Access Session's `freelancer_account` and `request_text`; Dana's operator identity |

## Processing Logic

1. Verify authorization per FEAT-31.SPEC-005: confirm Dana has no other Support Access Session currently open (one-account-at-a-time) and that she is the sole role permitted to open a session.
2. If authorization fails, stop -- no record is changed (see Outcome Definitions).
3. Load the named freelancer's account data read-only, spanning FEAT-01 through FEAT-25, exactly as the freelancer would see it herself.
4. If the account's data cannot be loaded read-only, stop -- `operator`/`opened_at` stay unset (see Outcome Definitions) and nothing about the account changes.
5. Set `operator` to Dana's identity and `opened_at` to the current timestamp on the selected Support Access Session record. The mirrored view is not yet shown to Dana; the row stays in its Opening state.
6. Trigger FEAT-13.SPEC-003 (Activity Entry Recording) to write the session-opened trail entry, with Dana as actor and `opened_at` as the timestamp. The trail entry is a precondition for the session becoming visible to Dana. If the write fails, go to step 9 (Trail failure).
7. Trigger FEAT-31.SPEC-007 (Support Session Opened Notice) to queue the email to Nadia. Accepting the notice into SPEC-007's delivery queue is a precondition for the session becoming visible to Dana (delivery retries after that point belong to FEAT-31.SPEC-007). If the trigger is not accepted, go to step 9 (Notice failure).
8. Only after steps 6 and 7 both succeed: apply the read-only enforcement continuously for the session's duration (every edit, send, approve, pay, file-download, and export-generation control disabled or not shown on every mirrored screen Dana views, per FEAT-31.SPEC-005's exact excluded-action list) and display the permanent "Read-only support session -- {freelancer_account name}" banner on every mirrored screen (FEAT-31.SPEC-002).
9. On a step 6 or step 7 failure, roll back: unset `operator` and `opened_at` together so the request returns to the pending state and stays in the queue, and never show the mirrored view. If a session-opened trail entry had already been written (notice failure), write a second FEAT-13.SPEC-003 entry recording "Support session did not start" with Dana as actor, so Nadia's trail never shows an open session that Dana never saw. Dana sees the partial-failure message and a Retry control (FEAT-31.SPEC-002).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Session opened | Authorization passes, the account loads read-only, the trail entry is written, and the notice trigger is accepted | `operator` and `opened_at` set on the Support Access Session record; session-opened trail entry written; notice queued | Mirrored view with the permanent banner appears | FEAT-31.SPEC-002 (renders it), FEAT-31.SPEC-007 (notice sent), FEAT-13.SPEC-003 (trail entry written) |
| Authorization denied | Dana already has another session open, or a role other than Dana attempts to trigger this automation | None | The exact denied message from FEAT-31.SPEC-005: "Your support session on {freelancer_account name} is still open and ends automatically after inactivity. Tap Resume to return to it." Dana can Resume the open session or wait for it to close automatically; no manual close exists | FEAT-31.SPEC-002 |
| Partial failure (trail entry) | The account loaded and `operator`/`opened_at` were set, but the FEAT-13.SPEC-003 session-opened trail entry could not be written | Rolled back: `operator` and `opened_at` unset together; no trail entry exists; no notice triggered | The mirrored view never appears. Dana sees inline on the queue row "The session could not be started right now. Nothing was opened." with a "Retry" button; the request stays in the queue. Retry re-runs the whole automation from step 1 | FEAT-31.SPEC-002, FEAT-13.SPEC-003 |
| Partial failure (notice) | The trail entry was written but FEAT-31.SPEC-007 did not accept the notice trigger | Rolled back: `operator` and `opened_at` unset together; the written trail entry is followed by a "Support session did not start" entry | Same message and "Retry" button as the trail-entry failure; the mirrored view never appears | FEAT-31.SPEC-002, FEAT-31.SPEC-007, FEAT-13.SPEC-003 |
| Load failure | The named freelancer's account data cannot be loaded read-only (e.g., the account is mid-deletion via FEAT-24) | None -- `operator`/`opened_at` remain unset | Dana sees the specific reason inline on the queue row (e.g., "This account's data could not be loaded right now."); nothing about the account changes | FEAT-31.SPEC-002 |

## Data Model

**Reads:** Support Access Session -- the selected pending record's `freelancer_account` and `request_text`. Freelancer Account and every entity it owns across FEAT-01 through FEAT-25 -- read-only, for the session's duration.
**Creates:** None (FEAT-31.SPEC-001 already created the Support Access Session record).
**Updates:** Support Access Session -- `operator` (Dana's identity) and `opened_at` (current timestamp).
**Deletes:** None.

## Business Rules

- XBR-29: sessions are read-only in every feature, cover one account at a time, end after inactivity, exclude file downloads and data/accounting exports, are always announced to the freelancer by email, and are always listed in her trail.
- Authorization and the exact excluded-action list are owned by FEAT-31.SPEC-005; this automation enforces them but does not restate or duplicate them.
- The read-only enforcement is unconditional for the entire session -- there is no partial-edit window, no grace period, and no escalation path to write access.
- XBR-29's "always announced" and "always listed in her trail" are guaranteed by ordering: a session becomes visible to Dana only after its trail entry is written and its notice trigger is accepted, so a session Dana can see is always both logged and announced; a session that cannot be logged or announced never opens.
- Only Dana may trigger this automation; no other role has a control that reaches it.

## Edge Cases

- **The freelancer account is deleted (FEAT-24) between Dana selecting the request and this automation loading it** -- Load failure outcome: nothing changes, Dana sees the reason. The request itself is removed from the queue as part of that account's full deletion.
- **Dana's authorization check passes but the account load then fails** -- No partial state: `operator` and `opened_at` are set together with a successful load, never independently of it; a load failure leaves both unset.
- **Concurrent trigger firing (Dana attempts to open two different requests from two browser tabs at effectively the same time)** -- Each triggering attempt is evaluated independently against the one-account-at-a-time rule at the moment it fires; whichever attempt's authorization check completes first opens successfully, and the second is refused with "Your support session on {freelancer_account name} is still open and ends automatically after inactivity. Tap Resume to return to it.", even though both taps were near-simultaneous.
- **The trail entry or notice trigger fails after `operator`/`opened_at` are set** -- Rolled back per Processing Logic step 9; the request returns to the pending queue, Dana retries, and no session is ever visible without both its trail entry and its accepted notice. Concurrent "Open Session" attempts during the rolled-back window are evaluated against the one-account-at-a-time rule only after the rollback completes.
- **Trigger fires while a previous run is in flight** -- The "Open Session" control on the queue is disabled for the row being opened while this automation is processing (FEAT-31.SPEC-002's loading state), so a second run for the same request cannot start until the first completes.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-31.SPEC-002 (Operator Support Session Console) | Triggered by (inbound) | "Open Session" fires this automation |
| FEAT-31.SPEC-005 (Support Access Authorization & Read-Only Rules) | References (inbound) | Authorization check and the excluded-action list this automation enforces |
| FEAT-31.SPEC-007 (Support Session Opened Notice) | Triggers (outbound) | Fires on successful open |
| FEAT-13.SPEC-003 (Activity Entry Recording) | Triggers (outbound) | Writes the session-opened trail entry |

## Analytics and Success Signals

- **support_session_opened** (no properties beyond the event itself) -- N/A -- no metric in success-metrics.md is connected to Operator Support Access or names this behavior; retained per product-features.md's Signals field so support activity stays observable
- **support_session_open_denied** (reason: already_open) -- N/A -- same reason as above
- **support_session_open_load_failed** (reason: account_load / trail_entry / notice_trigger) -- N/A -- same reason as above

## Acceptance Criteria

**FEAT-31.SPEC-003-AC-01:** Given Dana selects a queued request with no other session currently open, when she taps "Open Session," then `operator` and `opened_at` are set on the record, the mirrored read-only view appears with the permanent banner, the opened notice (FEAT-31.SPEC-007) is triggered, and the trail entry is written.

**FEAT-31.SPEC-003-AC-02:** Given Dana already has a session open on one freelancer account, when she attempts to open a second, then authorization fails, no record changes, and she sees "Your support session on {freelancer_account name} is still open and ends automatically after inactivity. Tap Resume to return to it."

**FEAT-31.SPEC-003-AC-03:** Given the selected request's freelancer account cannot be loaded read-only, when this automation attempts to open the session, then `operator` and `opened_at` remain unset and Dana sees the specific reason inline.

**FEAT-31.SPEC-003-AC-04:** Given a session has opened successfully, when Dana views any mirrored screen, then every edit, send, approve, pay, file-download, and export-generation control is disabled or not shown.

**FEAT-31.SPEC-003-AC-05:** Given Dana attempts to open two different requests from two browser tabs at effectively the same time, when both "Open Session" taps fire, then only the first to complete its authorization check succeeds and the second is refused with "Your support session on {freelancer_account name} is still open and ends automatically after inactivity. Tap Resume to return to it."

**FEAT-31.SPEC-003-AC-06:** Given this automation is still processing a prior "Open Session" tap for a request, when Dana taps the same request's "Open Session" control again, then no second run starts because the control is disabled while the first is in flight.

**FEAT-31.SPEC-003-AC-07:** Given the account loads read-only and `operator`/`opened_at` are set but the FEAT-13.SPEC-003 session-opened trail entry cannot be written, when this automation handles the failure, then `operator` and `opened_at` are unset together, no notice is triggered, the mirrored view never appears, and Dana sees "The session could not be started right now. Nothing was opened." with a "Retry" button while the request stays in the queue.

**FEAT-31.SPEC-003-AC-08:** Given the trail entry was written but FEAT-31.SPEC-007 does not accept the notice trigger, when this automation handles the failure, then `operator` and `opened_at` are unset together, a "Support session did not start" trail entry follows the earlier entry, the mirrored view never appears, and Dana sees the same message with "Retry."

**FEAT-31.SPEC-003-AC-09:** Given a prior attempt was rolled back by AC-07 or AC-08, when Dana taps "Retry" and the trail entry and notice trigger both succeed, then the session opens with the permanent banner and Nadia's trail shows the session-opened entry.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 5 (opened, denied, load failure, partial failure trail entry, partial failure notice) | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |



# Automation Spec: Support Session Auto-Close on Inactivity

## Overview

**Name:** Support Session Auto-Close on Inactivity
**ID:** FEAT-31.SPEC-004
**Type:** Automation
**Purpose:** Closes an open Support Access Session automatically after a period of inactivity, recording the close time and returning Dana to the queue.
**Parent Feature:** FEAT-31 -- Operator Support Access

## Scope and Non-Goals

**In Scope:**
- Tracking Dana's last action within an open session (any interaction on the mirrored view or console, per FEAT-31.SPEC-002 and FEAT-31.SPEC-003)
- Closing the session once inactivity reaches the defined threshold
- Recording `closed_at` and writing the trail entry
- Returning Dana to the queue with a notice

**Non-Goals:**
- Any manual or on-demand close action -- not established by any Stage 2 field; product-features.md's Validation & Limits states only that "a session ends automatically after a period of inactivity," so this is the sole closing path this feature defines (Non-Goals: "A manual or on-demand close control for Dana").
- Notifying Nadia by email when a session closes -- product-features.md's Communications field names only the request-confirmation (FEAT-31.SPEC-006) and session-opened (FEAT-31.SPEC-007) emails; the close time reaches her only through her activity trail (FEAT-13), not a third email.
- Deleting or archiving the closed Support Access Session -- excluded per the dependency map's lifecycle statement that the entity is "Never edited after closing" and "Deleted by FEAT-24" only.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Inactivity threshold reached on an open session | System (inactivity timer) | Fires when the elapsed time since Dana's last action within the open session reaches platform parameter: `support-session-inactivity-timeout-minutes` | The open Support Access Session's `operator`, `opened_at`, `freelancer_account`, and the timestamp of Dana's last recorded action |

## Processing Logic

1. Track the timestamp of Dana's last action within the currently open session (any interaction on the mirrored view, FEAT-31.SPEC-002, or the opening action itself, FEAT-31.SPEC-003).
2. Continuously evaluate the elapsed time since that last-action timestamp against platform parameter: `support-session-inactivity-timeout-minutes`.
3. If a new action is recorded before the threshold is reached, reset the last-action timestamp and continue the session uninterrupted.
4. When the elapsed time reaches the threshold with no intervening action, close the session: set `closed_at` to the current timestamp on the Support Access Session record.
4a. If the `closed_at` write fails, take no further step: `closed_at` stays unset, the session remains open and enforced read-only, and Dana is not moved (see Outcome Definitions, Close write failed).
5. Once `closed_at` is set, revoke the read-only mirrored-view access immediately -- the enforcement surface from FEAT-31.SPEC-003 stops applying because no session is open.
6. Return Dana to the Support Session Console queue (FEAT-31.SPEC-002) with a notice naming the freelancer whose session closed and stating the reason ("closed after inactivity").
7. Trigger FEAT-13.SPEC-003 (Activity Entry Recording) to write the session-closed trail entry, with Dana as actor, `closed_at` as the timestamp, and closure reason "inactivity."
7a. If the trail entry write fails, the close is not undone (the record is never edited after `closed_at` is set): the session stays closed and Dana stays in the queue, and the trail entry is re-attempted on each subsequent evaluation cycle until FEAT-13.SPEC-003 accepts it, writing exactly one session-closed entry per closure (see Outcome Definitions, Trail entry pending).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Session closed | Elapsed inactivity reaches the threshold with no intervening action | `closed_at` set on the Support Access Session record | Dana is returned to the queue with a notice naming the freelancer and stating the session closed after inactivity | FEAT-31.SPEC-002, FEAT-13.SPEC-003 |
| No action (already closed) | A second evaluation cycle reaches the same session after it was already closed by an earlier cycle | None | None -- silent no-op | -- |
| Close write failed | The inactivity threshold was reached but the `closed_at` write failed | None -- `closed_at` stays unset; no trail entry is written | None shown to Dana: the session stays open and enforced read-only, the banner remains, and she is not returned to the queue. The close is re-evaluated on the next evaluation cycle and re-attempted if the inactivity gap still stands; an action by Dana before then resets the clock (No action, activity reset) | FEAT-31.SPEC-002 |
| Trail entry pending | `closed_at` was set but the FEAT-13.SPEC-003 session-closed trail entry could not be written | `closed_at` remains set (final, never edited); the trail entry is re-attempted on each subsequent evaluation cycle until accepted, one entry per closure | Dana is returned to the queue with the normal notice; the session is not enforced or shown as open. Nadia's trail lacks the closed entry until the retry succeeds | FEAT-31.SPEC-002, FEAT-13.SPEC-003 |
| No action (activity reset the clock) | An action was recorded before the threshold was reached | None -- the last-action timestamp resets | None -- the session continues uninterrupted | FEAT-31.SPEC-002 |

## Data Model

**Reads:** Support Access Session -- `operator`, `opened_at`, `freelancer_account`, and the last-action timestamp of the currently open record; also the `closed_at` of a closed record whose session-closed trail entry is still pending.
**Creates:** None.
**Updates:** Support Access Session -- `closed_at` (current timestamp).
**Deletes:** None.

## Business Rules

- XBR-29: sessions end after inactivity, with no manual close path.
- The inactivity threshold is platform parameter: `support-session-inactivity-timeout-minutes`, applied uniformly to every session regardless of freelancer account or request content.
- Once `closed_at` is set, the Support Access Session record is never edited again (dependency map, lifecycle note); this automation's write is the record's final state change short of account deletion (FEAT-24).
- Only one session can be open at a time (FEAT-31.SPEC-005), so this automation never needs to choose among multiple simultaneously open sessions for the same operator.

## Edge Cases

- **Dana performs an action at the exact moment the inactivity threshold is reached** -- The action recorded before this automation's close-evaluation completes resets the last-action timestamp and cancels the close for that cycle; only a confirmed inactivity gap of the full threshold, with no action recorded inside it, results in a close.
- **A freelancer account tied to an open session is deleted (FEAT-24) while the session is still open** -- FEAT-24 removes the Support Access Session as part of the full account deletion; this automation's close path is superseded and takes no further action on a record that no longer exists.
- **Concurrent trigger firing (two inactivity-evaluation cycles reach the same session at effectively the same time)** -- The evaluation is idempotent: whichever cycle completes first sets `closed_at`; the second finds `closed_at` already set and takes no further action.
- **Trigger fires while a previous run is in flight** -- Because at most one session can be open per operator at any time (FEAT-31.SPEC-005), and this automation only ever evaluates the single currently open session, no second run for the same session can be in flight concurrently with a first; a run that completes finds no session left to close on its next cycle.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-31.SPEC-003 (Support Session Open & Read-Only Enforcement) | References (inbound) | This automation closes only a session that FEAT-31.SPEC-003 opened |
| FEAT-31.SPEC-002 (Operator Support Session Console) | Affects (outbound) | Returns Dana to the queue with a notice |
| FEAT-31.SPEC-005 (Support Access Authorization & Read-Only Rules) | References (inbound) | The one-account-at-a-time constraint this automation relies on |
| FEAT-13.SPEC-003 (Activity Entry Recording) | Triggers (outbound) | Writes the session-closed trail entry with closure reason "inactivity" |

## Analytics and Success Signals

- **support_session_closed** (closure_reason: inactivity) -- N/A -- no metric in success-metrics.md is connected to Operator Support Access or names this behavior; retained per product-features.md's Signals field so support activity stays observable

## Acceptance Criteria

**FEAT-31.SPEC-004-AC-01:** Given Dana's open session has had no action for platform parameter: `support-session-inactivity-timeout-minutes`, when this automation evaluates it, then `closed_at` is set, Dana is returned to the queue with a notice naming the freelancer and stating the session closed after inactivity, and the trail entry is written with closure reason "inactivity."

**FEAT-31.SPEC-004-AC-02:** Given Dana performs an action one second before the inactivity threshold would be reached, when this automation next evaluates the session, then the last-action timestamp has reset and the session remains open.

**FEAT-31.SPEC-004-AC-03:** Given a session has already been closed by an earlier evaluation cycle, when a second evaluation cycle reaches the same session, then no further change is made and no duplicate notice or trail entry is produced.

**FEAT-31.SPEC-004-AC-04:** Given two inactivity-evaluation cycles reach the same open session at effectively the same time, when both attempt to close it, then only the first sets `closed_at` and the second finds it already closed.

**FEAT-31.SPEC-004-AC-05:** Given the freelancer account behind an open session is deleted (FEAT-24) while the session is still open, when the deletion completes, then this automation takes no further action on that now-removed record.

**FEAT-31.SPEC-004-AC-06:** Given Dana has just closed her previous session automatically, when she returns to the queue, then no other session for her remains open, so no second close-in-flight scenario can occur for her.

**FEAT-31.SPEC-004-AC-07:** Given the inactivity threshold is reached and the `closed_at` write fails, when this automation handles the failure, then `closed_at` stays unset, no trail entry is written, the session remains open and read-only with the banner still shown, and Dana is not returned to the queue; on the next evaluation cycle the close is re-attempted if she has still taken no action.

**FEAT-31.SPEC-004-AC-08:** Given `closed_at` has been set but the FEAT-13.SPEC-003 session-closed trail entry cannot be written, when this automation handles the failure, then the session stays closed, Dana is returned to the queue with the normal notice, and the trail entry is re-attempted on each later evaluation cycle until accepted, producing exactly one session-closed entry with closure reason "inactivity."

**FEAT-31.SPEC-004-AC-09:** Given a `closed_at` write failed and Dana performs an action before the next evaluation cycle, when this automation next evaluates the session, then the last-action timestamp has reset and the session stays open.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 5 (closed, already closed, activity reset, close write failed, trail entry pending) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 4 | 4 |



# Logic/Rule Spec: Support Access Authorization & Read-Only Rules

## Overview

**Name:** Support Access Authorization & Read-Only Rules
**ID:** FEAT-31.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs who may open, view, or never see a support session, the one-account-at-a-time and unconditional read-only constraints, the excluded actions (file downloads, data/accounting export generation, signing in as a client contact), and Nadia's standing right to see every session on her account.
**Parent Feature:** FEAT-31 -- Operator Support Access
**Governed Entity:** Support Access Session

## Scope and Non-Goals

**In Scope:**
- Field validation for every Support Access Session field
- The Dana-only, one-account-at-a-time authorization rule for opening a session
- The unconditional read-only rule and its excluded actions (edit, send, approve, pay, file-download, data/accounting export generation, client-contact sign-in)
- Nadia's standing right to see every session on her own account
- Owen and Priya's total exclusion from any visibility of support sessions
- Defaults and derivations for `operator`, `opened_at`, and `closed_at`

**Non-Goals:**
- The queue display, the mirrored screen layout, and the permanent banner -- owned by FEAT-31.SPEC-002 (Operator Support Session Console), which references this spec for the rules it displays.
- The step-by-step processing of opening and closing a session -- owned by FEAT-31.SPEC-003 (Support Session Open & Read-Only Enforcement) and FEAT-31.SPEC-004 (Support Session Auto-Close on Inactivity), which enforce these rules but are not their source of truth.
- A manual or on-demand close rule -- not established by any Stage 2 field; product-features.md's Validation & Limits states only that "a session ends automatically after a period of inactivity," so no manual-close authorization rule exists here, per the Brief's Non-Goals.
- Rules governing the freelancer's own data inside the mirrored view (e.g., what fields a proposal or invoice has) -- each underlying feature's own Logic/Rule spec owns its own entity's rules; this spec governs only the Support Access Session entity and the read-only constraint it imposes on top of them.

## Governed Entity

**Entity:** Support Access Session
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| freelancer_account | text (reference) | The one Freelancer Account being viewed or requested about (required) |
| request_text | text | Nadia's description of her problem, captured when she submits the request (required) |
| operator | text (reference) | Dana's identity, set only once she opens the session (required once opened) |
| opened_at | date/time | The timestamp the session opened (required once opened) |
| closed_at | date/time | The timestamp the session closed (required once closed) |

The entity has no explicit `status` field -- its lifecycle state (requested / opened / closed) is derived from which of `operator`, `opened_at`, and `closed_at` are set, per the Feature Dependency Map.

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-31.SPEC-001 | Contact Support Screen | `request_text` field validation, on blur and on submit |
| FEAT-31.SPEC-002 | Operator Support Session Console | Authorization on "Open Session" tap; read-only enforcement rendered on every mirrored screen; screen entry restricted to Dana only |
| FEAT-31.SPEC-003 | Support Session Open & Read-Only Enforcement | Authorization check at open time; applies the excluded-action list for the session's duration |
| FEAT-31.SPEC-004 | Support Session Auto-Close on Inactivity | The single automatic close path; no manual-close rule exists to enforce |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| freelancer_account | Required; system-set to the account of the freelancer submitting the request, never user-entered | Always | On create | N/A -- system-set, never invalid | No |
| request_text | Required, non-empty, 1-2000 characters | Always | On blur and on submit | "Describe your problem before sending." / "Please shorten your message to 2000 characters or fewer." | Yes |
| operator | No validation beyond data type -- system-set only when Dana opens the session (FEAT-31.SPEC-003); never user-entered | Always | -- | -- | -- |
| opened_at | No validation beyond data type -- system-set timestamp on open (FEAT-31.SPEC-003) | Always | -- | -- | -- |
| closed_at | No validation beyond data type -- system-set timestamp on automatic close (FEAT-31.SPEC-004) | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Opening requires both fields together | operator, opened_at | `opened_at` is non-null if and only if `operator` is non-null -- both are written atomically by FEAT-31.SPEC-003 | Not applicable -- this is a system-write invariant with no user-facing validation path; a load or authorization failure leaves both fields unset together (FEAT-31.SPEC-003) |
| Closing requires opening first | opened_at, closed_at | `closed_at` can only be set on a record where `opened_at` is already set -- a session cannot close before it has opened | Not applicable -- system-write invariant; FEAT-31.SPEC-004 only ever evaluates records with `opened_at` set and `closed_at` unset |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Submit a support request | Nadia | Always, for her own account only | -- |
| Submit a support request | Owen, Priya, Dana | Never | The Contact Support screen (FEAT-31.SPEC-001) is not shown anywhere in their product view |
| View own support sessions in the activity trail | Nadia | Always -- every session on her account, per the Access Matrix's "View (sees every support session on her account)" | -- |
| View support sessions | Owen | Never | Support session entries never appear in his view of the activity trail; Access Matrix gives him "None" for Support Access |
| View support sessions | Priya | Never | Same as Owen -- "None" for Support Access |
| View the queue of pending requests | Dana | Always | -- |
| View the queue of pending requests | Nadia, Owen, Priya | Never | The Operator Support Session Console (FEAT-31.SPEC-002) does not exist anywhere in their product |
| Open a session | Dana | Always, provided she has no other session currently open (one-account-at-a-time) | If Dana already has an open session: blocking message "Your support session on {freelancer_account name} is still open and ends automatically after inactivity. Tap Resume to return to it." The open session is unaffected; Dana can Resume it or wait for the automatic close, after which she can open another |
| Open a session | Nadia, Owen, Priya | Never | No control to open a session exists anywhere for these roles |
| View the named freelancer's account data during an open session | Dana | Always, for the exact duration of her open session, read-only | -- |
| View the named freelancer's account data | Owen, Priya | Never | Not applicable -- these roles have no relationship to a support session and no such view is ever offered to them |
| Edit, send, approve, or pay any control during an open session | Dana | Never | Control is not shown as available (FEAT-31.SPEC-003); a direct attempt is refused: "Not available in a support session." |
| Download a deliverable file during an open session | Dana | Never | Download control is never offered; a direct attempt is refused: "Downloads are not available in a support session." |
| Generate a data or accounting export during an open session | Dana | Never | Export control is never offered; a direct attempt is refused: "Exports are not available in a support session." |
| Sign in as a client contact | Dana | Never | No such control exists anywhere in the product for this role |
| View Nadia's sign-in credentials | Dana | Never | Never displayed anywhere in the mirrored view |
| View the payment processor account reference | Dana | Never | Only connection status is ever shown; the reference itself is never displayed (feature-dependency-map.md, Payment Account Connection Data Sensitivity note) |
| Manually close an open session | Dana | Never -- no such action exists in the product | Not applicable -- no control exists to attempt; the sole close path is automatic on inactivity (FEAT-31.SPEC-004) |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| operator | Unset (null) at creation; derived to Dana's identity the instant a session opens (FEAT-31.SPEC-003) | On open only | No -- system-set only |
| opened_at | Unset (null) at creation; derived to the current timestamp the instant a session opens (FEAT-31.SPEC-003) | On open only | No -- system-set only |
| closed_at | Unset (null) until the automatic inactivity close fires (FEAT-31.SPEC-004); derived to the current timestamp at that instant | On automatic close only | No -- system-set only; no manual close exists to override it with |

## Business Rules

- One session may be open per operator (Dana) at a time (XBR-29): opening a second while one is open is refused, per Authorization Rules above.
- Sessions are unconditionally read-only for their entire duration, with no exception -- excluded actions are edit, send, approve, pay, file-download, and data/accounting export generation (product-features.md, Validation & Limits; scope-boundaries.md SC-04).
- Dana never signs in as a client contact, under any circumstance (product-features.md, Primary Flows & Alternates; scope-boundaries.md SC-04).
- Nadia has a standing, unconditional right to see every session on her own account, in her activity trail (Access Matrix).
- Owen and Priya have no access whatsoever to support sessions and are never notified about them (Access Matrix; product-features.md, Access field: "Owen and Priya have no access and never see support sessions").
- No manual or on-demand close path exists for Dana; the only closure is the automatic inactivity close (FEAT-31.SPEC-004), per the Brief's Non-Goals.
- While one session is open, Dana's only permitted paths are to Resume it (or keep working in it) or to wait for the automatic inactivity close; any denied second-open message must name the open account, state that the session ends automatically after inactivity, and point to Resume -- it never instructs her to close the session, because no close control exists.
- Card data is never visible in a session because the product never holds it at all, in a session or otherwise (BRIEF.md, Constraints; product-features.md, Validation & Limits).

## Edge Cases

- **Dana attempts to construct a direct link to a file or export, bypassing the rendered console UI** -- The download or export is refused with the same exact denied message ("Downloads are not available in a support session." / "Exports are not available in a support session.") regardless of how the request is made, since these rules are enforced at the authorization boundary, not only in the rendered controls.
- **Dana's authorization is checked at the moment she selects "Open Session," not retained from an earlier page load** -- If she already has another session open in a separate tab, the second open attempt is refused with "Your support session on {freelancer_account name} is still open and ends automatically after inactivity. Tap Resume to return to it.", even though nothing visibly changed on the screen she is looking at.
- **A support request exists for a freelancer account that has since begun deletion (FEAT-24)** -- Dana cannot open a session on it; the request is removed from the queue as part of the account's full deletion, per FEAT-31.SPEC-003's load-failure handling.
- **request_text at exactly 2000 characters** -- Passes validation. 2001 characters shows the length error.
- **request_text as whitespace only** -- Treated as empty; the "Describe your problem before sending." error applies.
- **Owen or Priya somehow reaches a stale reference to a support session** -- Treated as unauthorized like any out-of-scope access: the session and its content are never shown, consistent with their "None" Access Matrix entry.
- **A session's `opened_at` and `closed_at` are both set, and someone attempts to re-open it** -- Not possible: FEAT-31.SPEC-003 only ever operates on pending (unopened) requests from the queue; a closed session is never re-offered as an "Open Session" target.

## Acceptance Criteria

**FEAT-31.SPEC-005-AC-01:** Given Nadia is submitting a support request, when she leaves the problem description empty and attempts to send, then she sees "Describe your problem before sending." and the send does not proceed.

**FEAT-31.SPEC-005-AC-02:** Given Nadia enters a problem description of exactly 2000 characters, when she sends it, then it is accepted; at 2001 characters, she sees "Please shorten your message to 2000 characters or fewer."

**FEAT-31.SPEC-005-AC-03:** Given a session is opened by FEAT-31.SPEC-003, when the record is written, then `operator` and `opened_at` are both set together -- neither is ever set without the other.

**FEAT-31.SPEC-005-AC-04:** Given a session's `opened_at` is unset, then FEAT-31.SPEC-004's auto-close evaluation never considers it, since `closed_at` can only be set on a record where `opened_at` is already set.

**FEAT-31.SPEC-005-AC-05:** Given Nadia is signed in to her own account, when she opens her activity trail, then she sees every support session on her account, past and present.

**FEAT-31.SPEC-005-AC-06:** Given Owen or Priya is signed in to their client portal, when they look anywhere in their product view, then no support session content or entry is ever shown to them.

**FEAT-31.SPEC-005-AC-07:** Given Dana is signed in as the operator, when she opens the Support Session Console, then she sees the queue of pending requests.

**FEAT-31.SPEC-005-AC-08:** Given Nadia, Owen, or Priya attempts to reach the Support Session Console, then it does not exist anywhere in their product.

**FEAT-31.SPEC-005-AC-09:** Given Dana has no session currently open, when she opens one on a queued request, then the open succeeds.

**FEAT-31.SPEC-005-AC-10:** Given Dana already has a session open, when she attempts to open a second, then she sees "Your support session on {freelancer_account name} is still open and ends automatically after inactivity. Tap Resume to return to it." with no instruction to close anything, the open session stays open, and the second session does not open.

**FEAT-31.SPEC-005-AC-11:** Given Dana is inside an open session, when she views the freelancer's account data, then she sees it exactly as the freelancer would, for the duration of the session only.

**FEAT-31.SPEC-005-AC-12:** Given Dana is inside an open session, when she attempts any edit, send, approve, or pay action, then the control is not shown as available, and a direct attempt shows "Not available in a support session."

**FEAT-31.SPEC-005-AC-13:** Given Dana is inside an open session, when she attempts to download a deliverable file, then she sees "Downloads are not available in a support session."

**FEAT-31.SPEC-005-AC-14:** Given Dana is inside an open session, when she attempts to generate a data or accounting export, then she sees "Exports are not available in a support session."

**FEAT-31.SPEC-005-AC-15:** Given Dana is diagnosing a client-side sign-in problem, when she looks for a way to sign in as the client contact, then no such control exists anywhere in the product.

**FEAT-31.SPEC-005-AC-16:** Given Dana is inside an open session viewing account settings, when she looks for Nadia's sign-in credentials, then they are never displayed.

**FEAT-31.SPEC-005-AC-17:** Given Dana is inside an open session viewing payment settings, when she looks for the processor account reference, then only the connection status is shown, never the reference itself.

**FEAT-31.SPEC-005-AC-18:** Given Dana is inside an open session, when she looks for a way to end it manually, then no such control exists anywhere -- the session remains open until it closes automatically on inactivity.

**FEAT-31.SPEC-005-AC-19:** Given Dana attempts to construct a direct link to a file bypassing the rendered console, when the request reaches the authorization boundary, then it is refused with the same exact denied message as the rendered control.

**FEAT-31.SPEC-005-AC-20:** Given a queued request's freelancer account begins deletion (FEAT-24) before Dana opens it, when she attempts to open it, then the request has already been removed from the queue as part of that deletion.

**FEAT-31.SPEC-005-AC-21:** Given Nadia enters only whitespace in the problem description, when she attempts to send, then she sees "Describe your problem before sending." exactly as if the field were empty.

**FEAT-31.SPEC-005-AC-22:** Given no card data of any kind exists in the product, when Dana views any screen inside a session, then no card data is ever shown, because none is ever held.

**FEAT-31.SPEC-005-AC-23:** Given Dana has one session open in one browser tab, when she attempts to open a different request in a second tab, then the second attempt is refused with "Your support session on {freelancer_account name} is still open and ends automatically after inactivity. Tap Resume to return to it." even though the first tab's screen shows no visible change.

**FEAT-31.SPEC-005-AC-24:** Given a session has already closed (`closed_at` set), when anyone looks for a way to re-open it from the queue, then it is not offered there -- it was already removed from the pending queue the moment it opened, and closing does not return it.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 18 | 18 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 8 | 8 |
| Edge Cases | 7 | 7 |



# Notification Spec: Support Request Confirmation

## Overview

**Name:** Support Request Confirmation
**ID:** FEAT-31.SPEC-006
**Type:** Notification
**Purpose:** Sends Nadia a confirmation email the moment her support request is received, so she knows it reached Dana even after she leaves the Contact Support screen.
**Parent Feature:** FEAT-31 -- Operator Support Access

## Scope and Non-Goals

**In Scope:**
- The emailed confirmation sent the instant a support request is created
- Its content, delivery rules, and edge cases

**Non-Goals:**
- The on-screen confirmation shown immediately after Nadia sends her request -- that inline feedback is owned by FEAT-31.SPEC-001 (Contact Support Screen); this spec covers only the separate emailed confirmation product-features.md's Communications field names.
- Dana's diagnostic reply -- product-features.md's Primary Flows & Alternates states Dana "replies by email," sent by her personally, outside the product; this is not a system-templated notification this feature owns.
- Any notice about the session opening -- a distinct communication with its own trigger and content, owned by FEAT-31.SPEC-007 (Support Session Opened Notice).

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, the instant a support request is created | Nadia may have already left the Contact Support screen; email reaches her wherever she is, and the Access Matrix and Communications field both name only an emailed confirmation for this moment -- the in-app confirmation is already handled inline by the triggering screen (FEAT-31.SPEC-001) |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A support request is created | FEAT-31.SPEC-001 (Contact Support Screen) | Fires the instant the Support Access Session record is created, on every successful send | Nadia's account name and email, the submitted `request_text` |

## Audience and Preferences

**Recipients:** Nadia only -- the freelancer who submitted the request. No other role receives this email: Owen and Priya have "None" for Support Access in the Access Matrix, and Dana receives the request in her own queue (FEAT-31.SPEC-002), not by email.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Support request confirmation | Always on (transactional) | On | N/A -- this is a transactional record email; per XBR-30, transactional emails core to the record always send and cannot be disabled |

**Quiet Hours:** N/A -- the product definition (FEAT-14, Notifications (Email)) establishes no quiet-hours mechanism for any notification; this email sends immediately regardless of time of day, consistent with its purpose as an immediate receipt.

## Content Definition

**Email:**
- **Subject:** We received your support request
- **Body:**
  Hi {freelancer_first_name},

  We received your support request:

  "{request_text}"

  Dana will look into it and reply to you by email as soon as possible.

  -- Clientroom Support
- **CTA:** None -- this is a receipt, not an action prompt. Nadia's next step is to wait for Dana's reply, which is sent by email outside the product (Non-Goals); there is nothing in-product for her to do until then.

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {freelancer_first_name} | Freelancer Account -- name | Nadia | Never empty -- the freelancer's name is required at sign-up (FEAT-20, dependency map: Freelancer Account, name required); the greeting is never rendered without it |
| {request_text} | Support Access Session -- request_text | "I can't get my payment account to finish connecting." | Never empty -- request_text is required at submission (FEAT-31.SPEC-005); this email cannot be triggered by a record that lacks it |

## Delivery Rules

**Batching:** None -- each support request produces exactly one confirmation, delivered individually. Multiple requests from the same freelancer in the same day each get their own email, never combined, so each confirmation matches the specific message she sent.
**Deduplication:** Exactly one confirmation per Support Access Session record. The creating automation in FEAT-31.SPEC-001 fires this notification once, on successful creation only; there is no retry path that re-creates the same record, so no duplicate trigger exists.
**Retry on failure:** Retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`. After the final failure, the failure is surfaced to Nadia as a delivery warning on her account (FEAT-14's general delivery-failure pattern, XBR-30); her request itself is unaffected -- Dana still sees it in her queue (FEAT-31.SPEC-002) regardless of this email's delivery outcome.
**Expiry:** An unsent confirmation is not attempted further after the final retry (the end of platform parameter: `transactional-email-retry-window`). The record of her submitted request remains visible to her in-product through the Contact Support screen's own on-screen confirmation (FEAT-31.SPEC-001), which is not affected by this email's fate.

## Edge Cases

- **The request cannot be edited or withdrawn once submitted** -- No such capability exists in this feature (FEAT-31.SPEC-001, Non-Goals), so the confirmation always describes exactly what was sent; there is no scenario where the content changes between trigger and delivery.
- **Dana opens a session on Nadia's account before this confirmation is delivered** -- The confirmation's content is unaffected; it always describes only the original request. The session opening produces its own, separate notice (FEAT-31.SPEC-007).
- **Nadia's sign-in email changes (FEAT-21) between submitting the request and this email being sent** -- FEAT-21 requires re-verification before an email change takes effect; this notification is sent to whichever address is current and verified at the moment of the send attempt, since it is a one-time transactional email with no preference-evaluation window to wait for.
- **The freelancer account is deleted (FEAT-24) between request creation and this email being sent** -- FEAT-24's account deletion cascades immediately, including this Support Access Session and any pending notification; if the deletion completes before the send attempt, the confirmation is not sent.
- **Quiet hours colliding with expiry** -- Not applicable: this product defines no quiet-hours mechanism (see Audience and Preferences), so no such collision can occur for this or any other notification.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-31.SPEC-001 (Contact Support Screen) | Triggered by (inbound) | The submitted request creates the Support Access Session record that fires this notification |

## Analytics and Success Signals

- **support_request_confirmation_delivered** (channel: email) -- N/A -- no metric in success-metrics.md is connected to Operator Support Access or names this behavior; retained so confirmation delivery stays observable
- **support_request_confirmation_failed** (reason: retries_exhausted) -- N/A -- same reason as above

## Acceptance Criteria

**FEAT-31.SPEC-006-AC-01:** Given Nadia submits a support request describing a payment connection problem, when the Support Access Session record is created, then she receives an email with the subject "We received your support request" quoting her exact description back to her.

**FEAT-31.SPEC-006-AC-02:** Given Nadia submits two separate support requests on the same day, when both are created, then she receives two separate confirmation emails, each quoting its own request, never combined into one.

**FEAT-31.SPEC-006-AC-03:** Given this email fails to deliver on its first attempt, when the retry window runs, then it is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window` before being surfaced as a delivery warning on Nadia's account.

**FEAT-31.SPEC-006-AC-04:** Given this email exhausts all retries without delivering, when the final attempt fails, then no further attempt is made and Dana still sees the request in her queue regardless.

**FEAT-31.SPEC-006-AC-05:** Given Nadia's sign-in email is mid-change (pending re-verification) when this notification is triggered, then it is sent to whichever address is current and verified at the moment of the send attempt.

**FEAT-31.SPEC-006-AC-06:** Given Nadia's account is deleted before this email's send attempt completes, when the deletion finishes first, then this confirmation is never sent.

**FEAT-31.SPEC-006-AC-07:** Given Dana opens a session on Nadia's account moments after the request is submitted, when this confirmation email is delivered, then its content still describes only the original request, not the session opening.

**FEAT-31.SPEC-006-AC-08:** Given Nadia has no notification preferences that could disable this email, when a request is submitted, then the confirmation always sends -- there is no control anywhere that turns it off.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |



# Notification Spec: Support Session Opened Notice

## Overview

**Name:** Support Session Opened Notice
**ID:** FEAT-31.SPEC-007
**Type:** Notification
**Purpose:** Sends Nadia an email notice whenever a support session opens on her account, so the operator's access is always visibly announced to her, never silent.
**Parent Feature:** FEAT-31 -- Operator Support Access

## Scope and Non-Goals

**In Scope:**
- The emailed notice sent the instant a support session opens
- Its content, delivery rules, and edge cases

**Non-Goals:**
- A notice when the session closes -- product-features.md's Communications field names only the request-confirmation and session-opened emails; the close time reaches Nadia through her activity trail (FEAT-13), not a third email, per FEAT-31.SPEC-004's Non-Goals.
- The activity trail entry itself -- owned by Immutable Activity & Audit Trail (FEAT-13), written independently by FEAT-31.SPEC-003 at the same moment this notice is triggered; this spec covers only the email.
- Any content about what Dana is diagnosing or intends to change -- a session grants read-only visibility only, and this notice does not describe session activity beyond the fact that it opened, consistent with the unconditional read-only rule (FEAT-31.SPEC-005).

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, the instant a session opens | Nadia may not be inside the product at the moment Dana opens a session; ASMP-23 requires operator access to always be announced to the freelancer, and email is the only channel that reaches her outside an active session, matching product-features.md's Communications field |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A support session opens | FEAT-31.SPEC-003 (Support Session Open & Read-Only Enforcement) | Fires the instant `operator` and `opened_at` are set on the Support Access Session record | Nadia's account name and email, `opened_at` |

## Audience and Preferences

**Recipients:** Nadia only -- the freelancer whose account the session was opened on. No other role receives this email: Owen and Priya have "None" for Support Access, and Dana is the actor who caused the event, not its recipient.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Support session opened notice | Always on (transactional) | On | N/A -- per ASMP-23 and XBR-29, operator access is always announced to the freelancer; this is a transactional record email that cannot be disabled, consistent with XBR-30 |

**Quiet Hours:** N/A -- the product definition (FEAT-14, Notifications (Email)) establishes no quiet-hours mechanism for any notification; this email sends immediately regardless of time of day. A support session is itself a privacy-relevant event (ASMP-23), so an immediate, unheld notice is the only correct behavior even if a quiet-hours mechanism existed elsewhere in the product.

## Content Definition

**Email:**
- **Subject:** A support session was opened on your account
- **Body:**
  Hi {freelancer_first_name},

  Dana from Clientroom support opened a read-only session on your account at {opened_at_time} to help with your recent request.

  Everything Dana sees is read-only -- nothing can be changed, sent, approved, or paid during this session. You can see exactly when it started and ended in your activity trail.

  -- Clientroom Support
- **CTA:** View activity trail -- deep-links to FEAT-13.SPEC-001 (Activity Trail) for this account's session entries

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {freelancer_first_name} | Freelancer Account -- name | Nadia | Never empty -- the freelancer's name is required at sign-up (dependency map: Freelancer Account, name required); the greeting is never rendered without it |
| {opened_at_time} | Support Access Session -- opened_at, shown in the freelancer's own time zone (FEAT-15) | 2:14 PM | Never empty -- this notification is triggered only after FEAT-31.SPEC-003 sets `opened_at`, so it always has a value |

## Delivery Rules

**Batching:** None -- each session opening is its own event, delivered individually. Because at most one session may be open per operator at a time (FEAT-31.SPEC-005), consecutive sessions on the same account cannot overlap in time; each still produces its own notice so Nadia's timeline of who looked, and when, stays exact.
**Deduplication:** Exactly one notice per `opened_at` timestamp. FEAT-31.SPEC-003 sets `operator` and `opened_at` exactly once per session and triggers this notification exactly once on that write; a session, once opened, is never re-opened (FEAT-31.SPEC-005, Edge Cases).
**Retry on failure:** Retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, consistent with FEAT-31.SPEC-006. After the final failure, it is surfaced as a delivery warning on Nadia's account (FEAT-14's general delivery-failure pattern, XBR-30).
**Expiry:** After the final retry (the end of platform parameter: `transactional-email-retry-window`), no further attempt is made. Nadia's activity trail entry (written independently by FEAT-13.SPEC-003 at the moment the session opened) remains the permanent, always-visible record of the session regardless of this email's delivery outcome -- satisfying ASMP-23's "always announced" requirement even if the email itself is never delivered.

## Edge Cases

- **The session Dana opened closes almost immediately after opening (auto-close on inactivity, FEAT-31.SPEC-004)** -- This notice still sends normally; FEAT-31.SPEC-004 does not cancel or suppress it, since the event this notice announces (the session opening) already occurred.
- **Nadia's account is deleted (FEAT-24) between the session opening and this email's send attempt** -- FEAT-24's cascade removes the Support Access Session and any pending notification; if the deletion completes first, delivery is cancelled with the account.
- **Multiple support requests are pending but only one session opens** -- This notice describes only the session that actually opened, at its own `opened_at` time; it never lists or references the freelancer's other pending, unopened requests.
- **This email fails on every retry** -- Nadia may not learn about the session by email, but the activity trail entry (FEAT-13) remains the authoritative record; ASMP-23's guarantee is satisfied by the trail even when the email itself never arrives.
- **Quiet hours colliding with expiry** -- Not applicable: this product defines no quiet-hours mechanism (see Audience and Preferences), so no such collision can occur.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-31.SPEC-003 (Support Session Open & Read-Only Enforcement) | Triggered by (inbound) | The session-opening write fires this notification |
| FEAT-13.SPEC-001 (Activity Trail) | Navigation (outbound) | The CTA deep-links to Nadia's activity trail |
| FEAT-13.SPEC-003 (Activity Entry Recording) | References (inbound) | The same open event also produces the trail entry independently of this email |

## Analytics and Success Signals

- **support_session_opened_notice_delivered** (channel: email) -- N/A -- no metric in success-metrics.md is connected to Operator Support Access or names this behavior; retained so notice delivery stays observable
- **support_session_opened_notice_failed** (reason: retries_exhausted) -- N/A -- same reason as above

## Acceptance Criteria

**FEAT-31.SPEC-007-AC-01:** Given Dana opens a session on Nadia's account at 2:14 PM (her local time), when FEAT-31.SPEC-003 sets `operator` and `opened_at`, then Nadia receives an email with the subject "A support session was opened on your account" naming that exact time.

**FEAT-31.SPEC-007-AC-02:** Given this notice is delivered, when Nadia taps "View activity trail," then she lands on her Activity Trail (FEAT-13.SPEC-001) and sees the session entry.

**FEAT-31.SPEC-007-AC-03:** Given the session Dana opened closes automatically within a minute of opening, when this notice's delivery attempt runs, then it still sends normally.

**FEAT-31.SPEC-007-AC-04:** Given Nadia has three pending, unopened support requests and Dana opens a session on only one, when this notice is delivered, then it describes only the session that opened, not the other pending requests.

**FEAT-31.SPEC-007-AC-05:** Given this email fails to deliver on its first attempt, when the retry window runs, then it is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window` before being surfaced as a delivery warning on Nadia's account.

**FEAT-31.SPEC-007-AC-06:** Given this email exhausts all retries without delivering, when the final attempt fails, then Nadia's activity trail entry for the session remains visible regardless.

**FEAT-31.SPEC-007-AC-07:** Given Nadia's account is deleted before this email's send attempt completes, when the deletion finishes first, then this notice is never sent.

**FEAT-31.SPEC-007-AC-08:** Given Nadia has no notification preferences that could disable this email, when a session opens on her account, then the notice always sends -- there is no control anywhere that turns it off.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |

# FEAT-05 — Client Portal Access (Magic-Link Login)

This chapter covers Client Portal Access (Magic-Link Login), a Core-tier feature. It contains the feature breakdown brief followed by every specification in full: 9 specifications carrying 99 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-05.SPEC-001 | Request Sign-In Link | screen | 11 |
| FEAT-05.SPEC-002 | Link Verification Landing | screen | 12 |
| FEAT-05.SPEC-003 | Portal Home | screen | 13 |
| FEAT-05.SPEC-004 | Magic Link Issuance | automation | 9 |
| FEAT-05.SPEC-005 | Magic Link Verification | automation | 10 |
| FEAT-05.SPEC-006 | Link Validity & Recognition Rules | logic-rule | 13 |
| FEAT-05.SPEC-007 | Portal Access & Isolation Rules | logic-rule | 12 |
| FEAT-05.SPEC-008 | Magic Link Sign-In Email | notification | 10 |
| FEAT-05.SPEC-009 | Portal Record First-View Capture | automation | 9 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Client Portal Access (Magic-Link Login)

## Summary

**Feature:** Client Portal Access (Magic-Link Login)
**ID:** FEAT-05
**Description:** A client contact signs in passwordless, by requesting a one-time email link, and lands in a view scoped strictly to their own company's projects — no account-creation step, no password to remember.
**Priority:** Core
**Phase:** MVP
**Type:** Platform
**Rationale:** BRIEF.md, Target Users & Roles: contacts "sign in passwordless with a magic link by email and never hit an account-creation wall." Without this, no client can reach any client-facing feature at all. MVP phase: every client-facing feature depends on it. [RESEARCH-INFORMED: a single client-facing portal that consolidates files, invoices, and approvals is the most consistently praised feature in the category (G2 and Capterra reviews across HoneyBook and SuiteDash, HIGH); portal quality — branding, mobile experience, reliability — is where products differ] [INFERRED: carried from Visionary draft]

**Key Capabilities:**
- Request a sign-in link — contact enters their email and receives a one-time link
- Enter the portal — clicking the link opens a view scoped to their own company only
- Re-request access — a fresh link can be requested any time, invalidating unused prior ones
- See where things stand — the portal home lists the contact's projects with each one's current stage, milestones, and what is waiting on them (accept, review, approve, pay — limited to what their role allows)

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-05.SPEC-001 | Request Sign-In Link | Screen | Owen, Priya | Contact enters their email and requests a one-time sign-in link |
| FEAT-05.SPEC-002 | Link Verification Landing | Screen | Owen, Priya | Contact sees link verification in progress, then either enters the portal or sees an expired/invalid explanation with a one-tap re-request |
| FEAT-05.SPEC-003 | Portal Home | Screen | Owen, Priya | Contact sees their company's projects, each one's stage, milestones, and what is waiting on them, scoped to their role |
| FEAT-05.SPEC-004 | Magic Link Issuance | Automation | Owen, Priya | Generates a single-use, time-limited sign-in link for a recognized contact and invalidates any prior unused link for that contact |
| FEAT-05.SPEC-005 | Magic Link Verification | Automation | Owen, Priya | Validates a clicked link, creates the scoped session, and records the sign-in |
| FEAT-05.SPEC-006 | Link Validity & Recognition Rules | Logic/Rule | Owen, Priya | Governs single-use/time-limited link enforcement, invalidation on re-request, and that only recognized contacts can obtain or use a link |
| FEAT-05.SPEC-007 | Portal Access & Isolation Rules | Logic/Rule | Owen, Priya, Nadia, Dana | Governs client isolation, role-scoped portal display, and multi-freelancer separation for a contact who serves several freelancers |
| FEAT-05.SPEC-008 | Magic Link Sign-In Email | Notification | Owen, Priya | Emails the one-time sign-in link to the requesting contact |
| FEAT-05.SPEC-009 | Portal Record First-View Capture | Automation | Owen, Priya | Detects a client contact's first view of a proposal, deliverable, or invoice reached through the portal and hands the timestamped event to FEAT-13's audit trail |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Request a sign-in link | FEAT-05.SPEC-001, FEAT-05.SPEC-004, FEAT-05.SPEC-008 | Request screen collects the email; Issuance automation validates recognition and generates the link; the Notification delivers it | Phase 2 (Explicit) |
| Enter the portal | FEAT-05.SPEC-002, FEAT-05.SPEC-005, FEAT-05.SPEC-003 | Verification landing shows progress; Verification automation validates the token, creates the session, and scopes it; the contact lands on Portal Home | Phase 2 (Explicit) |
| Re-request access | FEAT-05.SPEC-002, FEAT-05.SPEC-004, FEAT-05.SPEC-006 | The expired/invalid landing exposes a one-tap re-request that re-runs Issuance under the invalidate-prior-link rule | Phase 2 (Explicit) |
| See where things stand | FEAT-05.SPEC-003 | Portal Home lists projects, stage, milestones, and what is waiting, scoped to the contact's role | Phase 2 (Explicit) |

**Analyst-Discovered Specs** — Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-05.SPEC-006 | Link Validity & Recognition Rules | Phase 5 (Rule Discovery) | The Validation & Limits field (single-use, time-limited, invalidate prior unused links on re-request) combined with XBR-28's recognized-contact-only rule apply across the request, issuance, and verification specs — past the threshold for a shared standalone Logic/Rule spec |
| FEAT-05.SPEC-007 | Portal Access & Isolation Rules | Phase 5 (Rule Discovery) / Phase 6 (Failure Analysis) | Client isolation (XBR-09), role-scoped display (XBR-08), and the multi-freelancer separation case (Primary Flows & Alternates) govern all three screens and both automations; the persona journey's requirement that a role's scope be shown plainly rather than as a broken control is a rule, not an accident |
| FEAT-05.SPEC-008 | Magic Link Sign-In Email | Phase 4 (Notification surfacing) | The Communications field names an email sent on every login request, with its own audience (the requesting contact) and delivery behavior — not a same-screen toast |
| FEAT-05.SPEC-009 | Portal Record First-View Capture | Phase 4 (Trigger-Response) / Journey Step Coverage | XBR-05 lists "a contact's first view of a proposal/deliverable/invoice" as a trail event with this feature among the affected features, and the "Pointing to the Record in a Scope Dispute" journey (Edge/Recovery) relies on that first-view timestamp to settle a dispute; no record-owning feature (FEAT-03, FEAT-06, FEAT-09, FEAT-10) captures this event today, and it is inherently a property of the contact's portal session (FEAT-05 owns portal access), so this feature owns the detection and the hand-off to FEAT-13 |

## Entity-Lifecycle Coverage Matrix

**Entity: Client Contact** *(this feature authenticates against it and records login activity; creation, role assignment, and removal belong to FEAT-18)*

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A | Owned by FEAT-18 (contact added by Nadia, or invited by a Primary contact). This feature only authenticates an already-recognized contact. | — |
| Read (single) | FEAT-05.SPEC-004, FEAT-05.SPEC-005 | Issuance looks up the contact by the submitted email to confirm recognition; Verification looks up the contact bound to the clicked token | — |
| Read (list) | N/A | This feature never lists contacts; contact rosters are owned by FEAT-18 | — |
| Update | FEAT-05.SPEC-005 | Verification writes the `last_sign_in` timestamp on a successful sign-in | Role changes and status changes remain FEAT-18's concern |
| Delete/Archive | N/A | Access is removed by FEAT-18 (status set to Removed) and the contact record is deleted only by FEAT-24; this feature has no delete/archive action of its own, and no retention or purge decision belongs here — recorded as an explicit non-goal | — |
| State Transition | N/A | Contact status (Invited, Active, Removed) transitions are owned by FEAT-18; this feature only checks the current status to admit or deny sign-in | — |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Project | FEAT-05.SPEC-003 | Portal Home reads the contact's client company's projects and their derived stage to show where things stand |
| Branding Profile | FEAT-05.SPEC-001, FEAT-05.SPEC-002, FEAT-05.SPEC-003, FEAT-05.SPEC-008 | Every portal page and the sign-in email apply the owning freelancer's logo and brand colour, or the neutral default (XBR-31) |
| Custom Domain Record | FEAT-05.SPEC-002, FEAT-05.SPEC-003 (Later phase) | With a verified custom domain, portal pages and links are served at it (XBR-35); until then, and always as a fallback, the shared default address is used. Not active in MVP — FEAT-27 is Later phase (ASMP-32) |
| Activity Log Entry | Written by FEAT-13 on this feature's behalf | FEAT-05.SPEC-005 triggers a sign-in trail entry, and FEAT-05.SPEC-009 triggers a first-view trail entry, both through FEAT-13; this feature never reads or manages the log itself |
| Notification | Written by FEAT-14 on this feature's behalf | FEAT-05.SPEC-008 triggers the email through FEAT-14's delivery capability; this feature never reads or manages Notification records itself |
| Proposal, Deliverable, Invoice (each record's first-view marker, e.g. Deliverable's `first_client_view_at`) | FEAT-05.SPEC-009 (event only — the records themselves are owned by FEAT-03, FEAT-06, and FEAT-09/FEAT-10) | SPEC-009 detects the first portal view of a record on behalf of whichever feature's screen displays it, and hands the timestamped event to FEAT-13; the owning feature's record carries the resulting first-view field, which this feature never writes directly |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Contact submits the sign-in request | Validate the email against recognized contacts (FEAT-18) and issue a single-use, time-limited link, invalidating any prior unused link for that contact | Standalone Automation | FEAT-05.SPEC-004 |
| Link is issued | Send the link to the contact by email | Standalone Notification | FEAT-05.SPEC-008 |
| Contact submits the sign-in request | Show a same-screen "check your email" confirmation, whether or not the email matched a recognized contact | Inline in triggering screen | FEAT-05.SPEC-001 |
| Contact clicks the emailed link | Validate the token is unused and unexpired, mark it used, and create the scoped session | Standalone Automation | FEAT-05.SPEC-005 |
| Link verification succeeds | Update the contact's `last_sign_in` timestamp | Standalone Automation | FEAT-05.SPEC-005 |
| Link verification succeeds | Write an append-only Activity Log Entry for the sign-in | Cross-feature — logged in touchpoints | FEAT-05.SPEC-005 → FEAT-13 (XBR-05) |
| Link verification succeeds | Route the contact into the portal home scoped to that specific Client Contact's freelancer and client company | Standalone Logic/Rule (isolation) | FEAT-05.SPEC-007 |
| Link is expired, already used, or otherwise invalid | Show a clear explanation and a one-tap way to request a fresh link | Standalone Logic/Rule, surfaced by the landing screen | FEAT-05.SPEC-006, shown by FEAT-05.SPEC-002 |
| Contact requests a new link while a prior one is unused | Invalidate the prior link before issuing the new one | Standalone Logic/Rule | FEAT-05.SPEC-006 |
| A link is opened outside the contact's own scope (wrong company or wrong freelancer) | Show the same plain explanation as an expired link — never another company's data | Standalone Logic/Rule (isolation) | FEAT-05.SPEC-007 |
| A recognized contact for several freelancers signs in | Each freelancer's portal is entered separately under that freelancer's own branding; one sign-in never reveals another freelancer's clients | Standalone Logic/Rule (isolation) | FEAT-05.SPEC-007 |
| Portal Home loads | Read the contact's client company's projects and derive each project's stage, milestones, and what is waiting on the contact | Inline in triggering screen | FEAT-05.SPEC-003 |
| Portal Home loads for a contact with no active projects | Show a plain "nothing here yet" message, not an error | Inline in triggering screen (Empty state) | FEAT-05.SPEC-003 |
| Priya (Reviewer) views Portal Home | Show only the actions her role allows (view, comment); never a disabled or confusing Approve control | Standalone Logic/Rule, surfaced inline | FEAT-05.SPEC-007, shown by FEAT-05.SPEC-003 |
| A sign-in link cannot be verified because the contact has no connectivity | Show a connectivity notice rather than a false error or a blank page | Inline in triggering screen (Offline/Degraded) | FEAT-05.SPEC-002 |
| Contact requests, uses, or lets expire a link, or views the portal home | Emit the feature's signals (`magic_link_requested`, `magic_link_used`, `magic_link_expired`, `portal_home_viewed`) | Inline in the triggering screen or automation | FEAT-05.SPEC-001 / FEAT-05.SPEC-004 / FEAT-05.SPEC-005 / FEAT-05.SPEC-003 |
| A contact views a proposal, deliverable, or invoice through the portal for the first time (whether the screen belongs to FEAT-03, FEAT-06, FEAT-07, FEAT-09, or FEAT-10) | Detect that this is the contact's first view of that specific record and capture the timestamp | Standalone Automation | FEAT-05.SPEC-009 |
| A contact's first view of a record is captured | Hand the timestamped event to FEAT-13, which writes the append-only trail entry the owning feature's record later exposes (e.g., Deliverable's `first_client_view_at`) | Cross-feature — logged in touchpoints | FEAT-05.SPEC-009 → FEAT-13 (XBR-05) |
| A contact re-views a record already marked as first-viewed | No further event is captured — the first view is recorded exactly once per record per contact | Standalone Automation (idempotency rule within SPEC-009) | FEAT-05.SPEC-009 |

## Shared Context

**Shared Entities:**
- Client Contact — read by FEAT-05.SPEC-004 and FEAT-05.SPEC-005 to confirm recognition and identity; updated (`last_sign_in` only) by FEAT-05.SPEC-005. Fields relevant to this feature: `email`, `status`, `last_sign_in`, the freelancer/client company it belongs to.
- Sign-in link/token (feature-internal, not a Domain Entity Inventory record) — created by FEAT-05.SPEC-004, consumed exactly once by FEAT-05.SPEC-005, governed end to end by FEAT-05.SPEC-006.
- Branding Profile — read by every screen in this feature and by the sign-in email so the portal and its email consistently carry the owning freelancer's branding (XBR-31).
- First-view event (feature-internal, not a Domain Entity Inventory record) — detected by FEAT-05.SPEC-009 wherever a portal session first opens a proposal, deliverable, or invoice screen (owned by FEAT-03, FEAT-06/FEAT-07, or FEAT-09/FEAT-10); consumed by FEAT-13, which turns it into the record's first-view trail entry.

**Shared UI Patterns:**
- Accessibility and phone-first layout — all three screens (FEAT-05.SPEC-001, FEAT-05.SPEC-002, FEAT-05.SPEC-003) are readable at phone sizes without zooming, usable with a screen reader, and keep text contrast readable whatever the freelancer's brand colour (States field, Accessibility; ASMP-27). Spec Writers for all three should describe this consistently rather than re-deriving it per screen.
- Plain, non-alarming error and empty messaging — the expired/invalid state on FEAT-05.SPEC-002 and the no-projects state on FEAT-05.SPEC-003 both use the same calm, explanatory tone with a single clear next action (request a new link; nothing to browse yet), never a technical error page.
- One-tap re-request — the same re-request control (submit email again, or a single tap when the email is already known from context) appears on FEAT-05.SPEC-002's error state and is the entry point back into FEAT-05.SPEC-001 / FEAT-05.SPEC-004.

**Shared Validation:**
- FEAT-05.SPEC-006 is the single source of truth for link validity (single-use, time-limited, invalidated on re-request) and for recognized-contact enforcement; FEAT-05.SPEC-001, FEAT-05.SPEC-004, and FEAT-05.SPEC-005 all defer to it rather than duplicating these checks.
- FEAT-05.SPEC-007 is the single source of truth for client isolation, role-scoped display, and multi-freelancer separation; FEAT-05.SPEC-002, FEAT-05.SPEC-003, and FEAT-05.SPEC-005 all enforce it rather than each defining their own scoping logic.

## Internal Dependency Map

```
FEAT-05.SPEC-001 (Request Sign-In Link) -> [contact submits email] -> FEAT-05.SPEC-006 (Link Validity & Recognition Rules) -> [request valid] -> FEAT-05.SPEC-004 (Magic Link Issuance)
FEAT-05.SPEC-004 (Magic Link Issuance) -> [link generated] -> FEAT-05.SPEC-008 (Magic Link Sign-In Email)
FEAT-05.SPEC-001 (Request Sign-In Link) -> [request submitted] -> FEAT-05.SPEC-001 (same-screen confirmation shown)
[contact clicks the emailed link] -> FEAT-05.SPEC-002 (Link Verification Landing) -> [verifying] -> FEAT-05.SPEC-005 (Magic Link Verification)
FEAT-05.SPEC-005 (Magic Link Verification) -> [token valid] -> FEAT-05.SPEC-006 (Link Validity & Recognition Rules) -> [confirmed] -> FEAT-05.SPEC-007 (Portal Access & Isolation Rules) -> [scoped] -> FEAT-05.SPEC-003 (Portal Home)
FEAT-05.SPEC-005 (Magic Link Verification) -> [token invalid, expired, or reused] -> FEAT-05.SPEC-002 (Link Verification Landing) [error state]
FEAT-05.SPEC-002 (Link Verification Landing) -> [contact taps "Request a new link"] -> FEAT-05.SPEC-001 (Request Sign-In Link)
FEAT-05.SPEC-005 (Magic Link Verification) -> [sign-in recorded] -> FEAT-13 (Immutable Activity & Audit Trail) [cross-feature]
FEAT-05.SPEC-003 (Portal Home) -> [role-scoped display] -> FEAT-05.SPEC-007 (Portal Access & Isolation Rules)
FEAT-03 (Proposal Acceptance) / FEAT-06 (Deliverable Upload & Sharing) / FEAT-07 (Deliverable Review & Feedback) / FEAT-09 (Invoice Generation & Sending) / FEAT-10 (Invoice Payment Processing) -> [contact opens the record's screen within the portal session] -> FEAT-05.SPEC-009 (Portal Record First-View Capture) [cross-feature]
FEAT-05.SPEC-009 (Portal Record First-View Capture) -> [first view detected] -> FEAT-13 (Immutable Activity & Audit Trail) [cross-feature]
```

**Default Entry:** FEAT-05.SPEC-001 (Request Sign-In Link) — the screen shown when a contact arrives with no active session, whether from the public product page (FEAT-33) or directly. A contact who still holds a valid session from a prior sign-in and returns via FEAT-33's "returns to the portal in one step" connection lands directly on FEAT-05.SPEC-003 (Portal Home) instead, without a new link request; once that session lapses, the contact is returned to FEAT-05.SPEC-001.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-05.SPEC-002 | Inbound | FEAT-02 (Proposal Creation & Sending) | Owen opens the proposal-sent email's link and signs in | Contact clicks the emailed link |
| FEAT-05.SPEC-002 | Inbound | FEAT-06 (Deliverable Upload & Sharing) | A contact opens a deliverable-ready email's link and signs in | Contact clicks the emailed link |
| FEAT-05.SPEC-002 | Inbound | FEAT-18 (Client Contact Management & Roles) | A newly added or invited contact signs in for the first time from the invitation email | Contact clicks the invitation link |
| FEAT-05.SPEC-001 | Inbound | FEAT-33 (Portal Referral Attribution) | A returning contact reaches the sign-in entry point (or, with a live session, Portal Home directly) from the public product page | Contact returns to the portal |
| FEAT-05.SPEC-003 | Outbound | FEAT-03 (Proposal Acceptance) | Owen opens the proposal waiting on him from Portal Home | Owen taps the waiting proposal |
| FEAT-05.SPEC-003 | Outbound | FEAT-07 (Deliverable Review & Feedback) | Priya or Owen opens a deliverable waiting for review from Portal Home | Contact taps the waiting deliverable |
| FEAT-05.SPEC-003 | Outbound | FEAT-08 (Milestone Approval) | Owen opens a milestone waiting for his approval from Portal Home | Owen taps the waiting milestone |
| FEAT-05.SPEC-003 | Outbound | FEAT-10 (Invoice Payment Processing) | Owen opens an invoice waiting on him from Portal Home | Owen taps the waiting invoice |
| FEAT-05.SPEC-003 | Outbound | FEAT-18 (Client Contact Management & Roles) | Owen invites a Reviewer colleague from his Portal Home view | Owen taps invite from Portal Home |
| FEAT-05.SPEC-004, FEAT-05.SPEC-005, FEAT-05.SPEC-006 | Inbound | FEAT-18 (Client Contact Management & Roles) | Only contacts FEAT-18 has added and not removed can request or use a magic link (XBR-28) | Contact requests or uses a link |
| FEAT-05.SPEC-003, FEAT-05.SPEC-007 | Inbound | FEAT-18 (Client Contact Management & Roles) | Role (Primary vs. Reviewer) drives Portal Home's display and action scope (XBR-08) | Contact role assigned or changed |
| FEAT-05.SPEC-005 | Outbound | FEAT-13 (Immutable Activity & Audit Trail) | Successful sign-in writes an append-only trail entry with actor and timestamp (XBR-05) | Link verified successfully |
| FEAT-05.SPEC-009 | Inbound | FEAT-03 (Proposal Acceptance) | A contact's first view of a proposal, within the portal session this feature owns, is detected for the audit trail | Contact opens the Proposal Review & Accept screen |
| FEAT-05.SPEC-009 | Inbound | FEAT-06 (Deliverable Upload & Sharing) / FEAT-07 (Deliverable Review & Feedback) | A contact's first view of a deliverable, within the portal session this feature owns, is detected for the audit trail and feeds the Deliverable's `first_client_view_at` field | Contact opens a deliverable review screen |
| FEAT-05.SPEC-009 | Inbound | FEAT-09 (Invoice Generation & Sending) / FEAT-10 (Invoice Payment Processing) | A contact's first view of an invoice, within the portal session this feature owns, is detected for the audit trail | Contact opens the invoice list or pay-invoice screen |
| FEAT-05.SPEC-009 | Outbound | FEAT-13 (Immutable Activity & Audit Trail) | A captured first-view event writes an append-only trail entry with actor, timestamp, and affected record (XBR-05); this is the evidence the "Pointing to the Record in a Scope Dispute" journey relies on | First view of a proposal, deliverable, or invoice detected |
| FEAT-05.SPEC-008 | Outbound | FEAT-14 (Notifications & Delivery) | The sign-in email relies on FEAT-14's transactional email delivery capability, including bounce/delivery status (ASMP-29) | Link issued |
| FEAT-05 (feature-level) | Outbound | FEAT-19 (Freelancer Branding) | Every portal page and the sign-in email carry the owning freelancer's Branding Profile, or the neutral default (XBR-31) | Contact reaches any portal page or receives the email |
| FEAT-05 (feature-level) | Outbound | FEAT-33 (Portal Referral Attribution) | Portal pages carry the discreet referral mark alongside branding, without exposing client/project/freelancer data (XBR-32) | Contact reaches any portal page |
| FEAT-05 (feature-level) | Inbound | FEAT-27 (Custom Domain per Freelancer) | Later phase: with a verified custom domain, portal links are served at it instead of the shared default address; the shared address always remains available as a fallback (XBR-35, ASMP-32) | Domain verified (Later phase) |
| FEAT-05.SPEC-003 | Inbound | FEAT-31 (Operator Support Access) | Dana never signs in as a client contact and has no portal access (SC-04); the Access Matrix marks Support Access "None" for this feature | N/A — permanently excluded |

## Non-Functional Notes

**Data volumes / growth:** Contact and login-event volume is bounded by SC-21's expected scale (a few thousand freelancers in year one, each with 3–15 active clients and a handful of contacts per client); each sign-in only overwrites a single `last_sign_in` field on an existing Client Contact record, so this feature introduces no unbounded growth of its own.

**Responsiveness:** Client-facing pages become interactive within roughly 2 seconds on a typical mobile connection (ASMP-21); the Client Portal Login Success metric additionally targets at least 95% of magic-link sign-in attempts succeeding on the first try, with a failed or expired attempt recoverable through one additional request (success-metrics.md, Client Portal Login Success). The Client Portal Mobile Responsiveness metric applies this same 2-second target to the client-facing pages Portal Home routes into.

**Data sensitivity / privacy:** Contact email and login timestamps are personal data, GDPR-class (ASMP-24; dependency map, Client Contact Data Sensitivity). Strict client isolation applies throughout this feature (ASMP-23; XBR-09): a contact reaches only their own company's projects under one freelancer, a contact for several freelancers sees each portal separately, and an out-of-scope or expired link never reveals another company's data.

**Compliance flags:** GDPR-class handling applies to the contact's email and sign-in history (ASMP-24); if that contact is later erased (FEAT-18), their name remains on evidence records such as prior acceptances or approvals, but this feature holds no evidentiary record of its own beyond the sign-in trail entry it triggers in FEAT-13 (ASMP-20). No payment or card data is ever handled by this feature (ASMP-24).

## Non-Goals

- **Password-based sign-in or an account-creation step** — Excluded by this feature's own description and BRIEF.md's Target Users & Roles: contacts "never hit an account-creation wall." Magic-link email is the only sign-in path for a client contact.
- **Public or anonymous portal access** — Excluded per scope-boundaries.md (SC-03): only contacts explicitly added through FEAT-18 can ever reach a client's portal view; the referral mark (FEAT-33) leads only to a public product page and never exposes portal content.
- **Client-side roles beyond Primary and Reviewer** — Excluded per scope-boundaries.md (SC-02): this feature scopes and displays the portal for exactly the two client-contact roles the persona set establishes; no further client-side tier (e.g., a client-side "admin") is modeled.
- **Native mobile app entry point** — Excluded per scope-boundaries.md (SC-06): the portal is a web experience made excellent on mobile browsers, not a native app; there is no app-based sign-in flow.
- **Operator sign-in as a client contact** — Excluded per scope-boundaries.md (SC-04): Dana (Support Operator) never signs in as a client contact and has no portal access; her read-only support sessions are owned entirely by FEAT-31.
- **Custom-domain serving at MVP** — Deferred per scope-boundaries.md's Deferral note and ASMP-32: the shared default portal address is the only serving address in MVP; domain verification and serving at a freelancer's own domain is FEAT-27's Later-phase capability, referenced here only as a future entry point (XBR-35).
- **Retention/purge policy for `last_sign_in` and issued links** — Not applicable to this feature: `last_sign_in` is retained for the life of the account and removed only through FEAT-24 account deletion; issued sign-in tokens are transient (single-use, time-limited, superseded on re-request) and carry no retention decision of their own beyond FEAT-05.SPEC-006's invalidation rule.
- **A standalone Integration spec for email delivery in this feature** — The transactional email capability FEAT-05.SPEC-008 relies on is owned by FEAT-14 (External Touchpoints table); recorded here as a cross-feature touchpoint rather than duplicated as an Integration spec, consistent with how FEAT-02 and FEAT-03 use the same capability.



# Screen Spec: Request Sign-In Link

## Overview

**Name:** Request Sign-In Link
**ID:** FEAT-05.SPEC-001
**Type:** Screen
**Purpose:** A client contact enters their email and requests a one-time sign-in link, without ever seeing an account-creation step.
**Parent Feature:** FEAT-05 -- Client Portal Access (Magic-Link Login)

## Scope and Non-Goals

**In Scope:**
- Collecting the contact's email address
- Submitting the request and showing a same-screen confirmation
- Re-entry point for a fresh request after an expired or invalid link (FEAT-05.SPEC-002)
- Neutral confirmation messaging regardless of whether the email matched a recognized contact, so an unrecognized email is never distinguishable from a recognized one

**Non-Goals:**
- Account creation, password entry, or any credential set-up -- excluded per BRIEF.md, Target Users & Roles: contacts "never hit an account-creation wall"; magic-link email is this feature's only sign-in path.
- Determining whether the submitted email is recognized, generating the link, and enforcing single-use/time-limit rules -- owned by FEAT-05.SPEC-004 (Magic Link Issuance) and FEAT-05.SPEC-006 (Link Validity & Recognition Rules); this screen only collects the email and displays the confirmation.
- Client-side roles beyond Primary and Reviewer -- excluded per scope-boundaries.md SC-02: the product models exactly Owen (Primary) and Priya (Reviewer); no further client-side tier exists to select on this screen.
- Native mobile app entry -- excluded per scope-boundaries.md SC-06: this is a web screen made excellent on mobile browsers, not an app-based sign-in flow.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| External (FEAT-33, public product page) | A returning contact with no live session reaches the portal from the "Made with Clientroom" referral mark or the public product page | None -- form starts empty |
| Direct arrival (no active session) | Contact opens the shared portal address directly, or a session has lapsed | None -- form starts empty |
| FEAT-05.SPEC-002 (Link Verification Landing) | Contact taps "Request a new link" on an expired, invalid, or out-of-scope link | Email pre-filled when it was known from the expired link's context; otherwise empty |

**Default Entry:** This is the feature's Default Entry screen -- the screen shown to any contact who arrives with no active session, per the Feature Breakdown Brief's Internal Dependency Map.

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Owen (Client Primary Contact) | Full screen | Submit a sign-in request for their own email | -- |
| Priya (Client Reviewer Contact) | Full screen | Submit a sign-in request for their own email | -- |
| Nadia (Freelancer) | Full screen (this is a public-facing entry point, not a freelancer surface) | No portal-specific action -- Nadia signs in through her own freelancer account, not through this screen | This screen never authenticates Nadia as a freelancer; submitting her own email here is treated the same as any email (an unrecognized-as-client-contact address), and she sees the same neutral confirmation with no link ever issued |
| Dana (Support Operator) | Full screen (public entry point) | None -- Dana never signs in as a client contact (SC-04) | Same neutral confirmation as any submission; no link is ever issued to Dana because she holds no Client Contact record |
| Unauthenticated | Yes -- this screen requires no authentication by definition | Yes -- submitting the request is the unauthenticated action this screen exists for | -- |
| Expired session | Yes -- a contact whose session lapsed is routed here | Yes | The contact is returned here automatically from any portal page once their session lapses (per the Default Entry note in the Feature Breakdown Brief); no error dialog appears, since arriving here is the expected recovery path |

## Layout and Content

**Header:** The owning freelancer's Branding Profile logo (or the neutral default when unset) at the top, centered, per XBR-31. Below it, the screen title "Sign in to your client portal."

**Body:** A single-column form containing:
- Email (text input, required, email-format keyboard on mobile)
- "Send sign-in link" (primary action button, full width)
- A single line of explanatory text below the title: "We'll email you a link. No password needed."

**Footer:** The discreet "Made with Clientroom" referral mark (FEAT-33), positioned below the form, never overriding the freelancer's branding (XBR-31, XBR-32).

### Responsive Behavior

- **Compact breakpoint (phone width):** Single-column form, full width with the platform's standard side gutter; button spans the form width; logo scales to a size that keeps the header short enough that the email field is visible without scrolling.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Email input | Type | Captures the entered email | Field shows entered text | Standard input focus state |
| Email input | Blur (empty or malformed) | Validates the email format per FEAT-05.SPEC-006 | Field shows error state if invalid | "Enter a valid email address" below the field |
| "Send sign-in link" button | Tap | Submits the email to FEAT-05.SPEC-004 (Magic Link Issuance) | Button shows loading state during submission | On completion, the screen replaces the form with the confirmation state (see States) |
| "Send sign-in link" button (while loading) | Tap | No action -- debounced against double-submit | None | Button remains in loading state |
| Confirmation state's "Try a different email" link | Tap | Returns the form to its empty, editable state | Confirmation replaced by the empty form | Form reappears with focus on the email field |
| "Made with Clientroom" referral mark | Tap | Navigates externally to the public product page (FEAT-33); does not affect this screen's own state | This screen's state is unchanged | Standard external navigation |

### Accessibility Notes

- **Focus order:** Logo (skippable, not focusable) -> title -> email input -> "Send sign-in link" button -> referral mark link.
- **Validation announcements:** The email-format error is announced to assistive technology and programmatically associated with the field when it appears on blur.
- **Confirmation announcement:** When the confirmation state replaces the form, its heading is announced so a screen-reader user is told the request was received.
- **Keyboard alternatives:** Every action on this screen (submit, "Try a different email") is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Empty (default) | Empty email field, "Send sign-in link" enabled once the field has content | Screen first opens | User submits a request |
| Filling | Email field contains user input | User types in the field | User submits or navigates away |
| Submitting | Button shows loading state, field disabled | User taps "Send sign-in link" with a validly formatted email | Submission completes |
| Confirmation | Form replaced by: "Check your email. If {email} matches an account, we've sent a sign-in link to it." with a "Try a different email" link | Submission completes, regardless of whether the email matched a recognized contact | User taps "Try a different email," or navigates away |
| Validation Error | Email field shows error state and message | Blur or submit with an invalid email format | User corrects the field |
| Offline/Degraded | Banner "You're offline. Connect to the internet to request a sign-in link." above the form; the "Send sign-in link" button is disabled while offline | Connectivity is lost while the screen is open, or the screen loads without connectivity | Connectivity is restored -- banner clears and the button re-enables; no request is queued, since a stale request would defeat the single-use/time-limited link model (FEAT-05.SPEC-006) |

## Validation Rules

Validation governed by FEAT-05.SPEC-006 (Link Validity & Recognition Rules). See that spec for the email-format rule and for why recognition itself is never disclosed on this screen. This screen applies format validation on field blur and on form submit.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Contact clicks the emailed link (external, not a navigation from this screen) | FEAT-05.SPEC-002 (Link Verification Landing) | -- |

<!-- This screen has no in-screen navigation action -- the only path forward is through the emailed link, which the contact opens from their email client, not from a control on this screen. -->

## Data Model

**Creates:** None directly -- the submitted email is handed to FEAT-05.SPEC-004 (Magic Link Issuance), which performs the recognition lookup and any resulting write.
**Reads:** Branding Profile -- logo and brand_colour, to render the header per the owning freelancer (resolved from the portal address the contact arrived at).
**Updates:** None.
**Deletes:** None.

## Business Rules

- The confirmation message is identical whether or not the submitted email matches a recognized Client Contact (FEAT-05.SPEC-006) -- this screen never reveals which emails are recognized, so it cannot be used to enumerate a freelancer's clients.
- XBR-28: only contacts added through Client Contact Management & Roles (FEAT-18) are ever issued a link; this screen accepts any email as input but the issuance decision belongs entirely to FEAT-05.SPEC-004 and FEAT-05.SPEC-006.
- Double-submission is prevented while a request is in flight (button loading state); a second request for the same email after the first completes is a legitimate re-request and is not blocked by this screen (FEAT-05.SPEC-006 governs invalidation of the prior link).

## Edge Cases

- **User submits the same email twice in quick succession, each completing separately** -- Each submission is treated as an independent re-request; FEAT-05.SPEC-006's invalidate-prior-link rule ensures only the most recently issued link remains usable. The screen shows the same confirmation both times.
- **User navigates away during "Submitting" and returns** -- The screen reloads to its empty, default state; whether the in-flight request completed does not change what this screen shows, since it never discloses recognition status.
- **User pastes an email with leading/trailing whitespace** -- Whitespace is trimmed before format validation and submission.
- **Screen loaded from a shared or bookmarked link with no session** -- No pre-fill occurs; the form starts empty as it would for any direct arrival.
- **This screen never loads or updates a shared entity, so no concurrent-edit conflict applies here** -- N/A: the only write this feature performs (Client Contact's `last_sign_in`) happens in FEAT-05.SPEC-005 after verification, not on this screen.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-05.SPEC-004 (Magic Link Issuance) | Triggers (outbound) | Submitting the form triggers issuance processing for the entered email |
| FEAT-05.SPEC-006 (Link Validity & Recognition Rules) | References (inbound) | Email-format validation and the no-enumeration confirmation rule are governed there |
| FEAT-05.SPEC-002 (Link Verification Landing) | Navigation (inbound) | The "Request a new link" control on an expired/invalid link routes here |
| FEAT-33 (Portal Referral Attribution) | Navigation (inbound) | A returning contact with no live session reaches this screen from the public product page or referral mark |
| FEAT-19 (Freelancer Branding) | References (inbound) | Branding Profile supplies the logo and colour shown in the header |
| FEAT-33 (Portal Referral Attribution) | Navigation (outbound) | The footer referral mark links out to the public product page |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|------------------|
| magic_link_requested | request_type (initial / re-request, inferred from Entry Point) | The contact submits a validly formatted email and the request reaches FEAT-05.SPEC-004 | supports success-metrics.md: "Client Portal Login Success" |
| magic_link_request_validation_failed | reason (invalid_format) | Form submit is blocked by field-level validation | N/A -- no Stage 2 metric measures client-side input error frequency; retained so a rise in malformed submissions is visible to the freelancer's product team, not silently dropped |

## Acceptance Criteria

**FEAT-05.SPEC-001-AC-01:** Given Owen is on the Request Sign-In Link screen, when he enters his email and taps "Send sign-in link," then the button shows a loading state and, on completion, the screen shows "Check your email. If {his email} matches an account, we've sent a sign-in link to it."

**FEAT-05.SPEC-001-AC-02:** Given Priya is on the Request Sign-In Link screen, when she enters an email that has never been added as a client contact, then she sees the identical confirmation message as a recognized contact, and no link is ever issued to that address.

**FEAT-05.SPEC-001-AC-03:** Given Owen is on the Request Sign-In Link screen, when he leaves the email field blank or malformed and moves focus away, then the field shows the error "Enter a valid email address" and the button remains disabled for an empty field.

**FEAT-05.SPEC-001-AC-04:** Given Owen is on the confirmation state, when he taps "Try a different email," then the form returns to its empty, editable state with focus on the email field.

**FEAT-05.SPEC-001-AC-05:** Given Owen taps "Send sign-in link" and taps it again before the first request completes, then the second tap has no effect and the button remains in its loading state.

**FEAT-05.SPEC-001-AC-06:** Given Priya loses connectivity while viewing this screen, when the screen detects the loss, then the banner "You're offline. Connect to the internet to request a sign-in link." appears and the "Send sign-in link" button is disabled.

**FEAT-05.SPEC-001-AC-07:** Given Priya's connectivity is restored after the offline banner appeared, when connectivity returns, then the banner clears, the button re-enables, and no request was queued or auto-submitted while she was offline.

**FEAT-05.SPEC-001-AC-08:** Given a contact arrives at this screen from FEAT-05.SPEC-002's "Request a new link" control with a known email, when the screen loads, then the email field is pre-filled with that address.

**FEAT-05.SPEC-001-AC-09:** Given Owen reaches this screen through his freelancer's Branding Profile, when the screen renders, then the header shows that freelancer's logo and brand colour, or the neutral default if none is set.

**FEAT-05.SPEC-001-AC-10:** Given a contact's session lapses while viewing any portal page, when the lapse is detected, then the contact is returned to this screen with no error dialog, since this is the feature's expected recovery path.

**FEAT-05.SPEC-001-AC-11:** Given Priya is on the Request Sign-In Link screen, when she taps the "Made with Clientroom" referral mark, then she is navigated externally to the public product page (FEAT-33) and this screen's own state is unaffected.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 6 (empty, filling, submitting, confirmation, validation error, offline) | 6 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



# Screen Spec: Link Verification Landing

## Overview

**Name:** Link Verification Landing
**ID:** FEAT-05.SPEC-002
**Type:** Screen
**Purpose:** The contact sees their emailed link being verified, then either enters the portal or sees a plain explanation with a one-tap way to request a fresh link.
**Parent Feature:** FEAT-05 -- Client Portal Access (Magic-Link Login)

## Scope and Non-Goals

**In Scope:**
- The in-progress verification appearance while FEAT-05.SPEC-005 validates the clicked token
- The successful-verification transition into the portal
- The expired, already-used, and out-of-scope error explanations, each with a one-tap re-request
- The offline/degraded state when the link cannot be verified without connectivity

**Non-Goals:**
- Validating the token, creating the scoped session, and recording the sign-in -- owned by FEAT-05.SPEC-005 (Magic Link Verification); this screen only reflects that automation's outcome.
- Determining what "expired," "already used," and "out-of-scope" mean and how invalidation works -- owned by FEAT-05.SPEC-006 (Link Validity & Recognition Rules) and FEAT-05.SPEC-007 (Portal Access & Isolation Rules); this screen shows the same plain explanation for every disqualifying reason rather than distinguishing them.
- Collecting a new email to request a fresh link -- the one-tap re-request either resubmits the already-known email directly or routes to FEAT-05.SPEC-001 (Request Sign-In Link) when no email is known from context; either way, the email-entry work belongs to that screen.
- Custom-domain serving of this landing page -- deferred per scope-boundaries.md's Deferral note and ASMP-32: in MVP this screen is served only at the shared default address; FEAT-27 (Later phase) is the only source of a custom-domain path.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| External (email client) | Contact clicks the sign-in link in the Magic Link Sign-In Email (FEAT-05.SPEC-008) | The token embedded in the link |
| External (email client) | Contact clicks a proposal-sent, deliverable-ready, or invitation email's embedded link from FEAT-02 (FEAT-02.SPEC-011), FEAT-06 (FEAT-06.SPEC-006), or FEAT-18 (FEAT-18.SPEC-010, New Contact Invitation Email) | The token embedded in the link, plus the specific record the originating email pointed to |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Owen (Client Primary Contact) | Full screen, for a link addressed to Owen's own recognized contact record | Request a fresh link if the clicked one fails | -- |
| Priya (Client Reviewer Contact) | Full screen, for a link addressed to Priya's own recognized contact record | Request a fresh link if the clicked one fails | -- |
| Nadia (Freelancer) | Full screen if she opens a client-addressed link (this is a public entry point, not role-gated by URL alone) | None -- Nadia holds no Client Contact record, so the token addressed to a contact never verifies for her | Same plain expired/invalid explanation and re-request option shown to any contact whose token does not verify; never a distinct "not a client" message, which would leak that the address is freelancer-owned |
| Dana (Support Operator) | Full screen if she opens a client-addressed link | None -- Dana never signs in as a client contact (SC-04) | Same plain expired/invalid explanation as above |
| Unauthenticated | Yes -- verifying a link is itself the unauthenticated step | Yes -- the one-tap re-request is available without prior authentication | -- |
| Expired session | N/A -- this screen is reached only from a fresh link click, never from an existing session; a contact with an expired session lands on FEAT-05.SPEC-001 instead, per that spec's Access and Visibility table | N/A | N/A |

## Layout and Content

**Header:** The owning freelancer's Branding Profile logo (or neutral default), centered, per XBR-31.

**Body (Verifying state):** A centered progress indicator with the text "Signing you in..."

**Body (Error state):** A plain, non-alarming heading -- "This link isn't valid anymore" -- with one line of explanation ("Sign-in links are single-use and time-limited, and yours has expired, already been used, or was requested again since.") and a single "Send me a new link" button.

**Body (Offline/Degraded state):** A connectivity notice heading -- "You're offline" -- with the text "We can't verify your sign-in link without a connection. Reconnect and try the link again." and a "Retry" button.

**Footer:** The discreet "Made with Clientroom" referral mark (FEAT-33), consistent with FEAT-05.SPEC-001 and FEAT-05.SPEC-003.

### Responsive Behavior

- **Compact breakpoint (phone width):** Content vertically centered in the viewport, full width with the platform's standard side gutter; the "Send me a new link" / "Retry" button spans the content width.
- **Medium size class and above:** Content remains centered and capped at a consistent platform-wide width; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| (automatic) | Screen loads with a token | Triggers FEAT-05.SPEC-005 (Magic Link Verification) | Screen enters Verifying state | Progress indicator with "Signing you in..." |
| "Send me a new link" button | Tap | If the contact's email is known from the clicked link's context, resubmits it directly to FEAT-05.SPEC-004 (Magic Link Issuance); otherwise navigates to FEAT-05.SPEC-001 (Request Sign-In Link) | Button shows loading state, or screen navigates | On direct resubmission: transitions to the same "Check your email" confirmation described in FEAT-05.SPEC-001. On navigation: FEAT-05.SPEC-001 loads with the email field empty |
| "Retry" button (offline state) | Tap | Re-attempts verification of the same token | Screen returns to Verifying state | Progress indicator reappears |
| "Made with Clientroom" referral mark | Tap | Navigates externally to the public product page (FEAT-33); does not affect this screen's own state | This screen's state is unchanged | Standard external navigation |

### Accessibility Notes

- **Focus order:** Logo (skippable) -> heading -> body text -> primary action button (when present).
- **Verifying announcement:** The "Signing you in..." status is announced to assistive technology as the screen enters the Verifying state, so a screen-reader user is not left in silence during the wait.
- **Error/offline announcement:** The heading of the Error or Offline/Degraded state is announced immediately when that state is entered.
- **Keyboard alternatives:** "Send me a new link" and "Retry" are reachable and activatable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Verifying (default) | Progress indicator, "Signing you in..." | Screen first opens with a token | FEAT-05.SPEC-005 returns an outcome |
| Success (transitional) | Same as Verifying, momentarily, before navigation | FEAT-05.SPEC-005 confirms the token is valid and the session is created | Immediate navigation to FEAT-05.SPEC-003 (Portal Home) |
| Error (expired, used, or out-of-scope) | Plain explanation heading, body text, and "Send me a new link" button | FEAT-05.SPEC-005 reports the token as invalid, expired, already used, or out of the contact's scope (FEAT-05.SPEC-006, FEAT-05.SPEC-007) | Contact taps "Send me a new link" |
| Offline/Degraded | Connectivity notice with "Retry" button; no automatic queued retry | Verification cannot reach FEAT-05.SPEC-005 due to lost connectivity, at load or mid-verification | Connectivity is restored and the contact taps "Retry" -- verification is not attempted automatically, since a delayed automatic retry against a single-use, time-limited link could itself land on an inconsistent state |

## Validation Rules

Validation governed by FEAT-05.SPEC-006 (Link Validity & Recognition Rules) for token validity, and FEAT-05.SPEC-007 (Portal Access & Isolation Rules) for scope. See those specs for the full set of conditions that produce the Error state on this screen; this screen applies no rules of its own beyond routing the outcome to Success, Error, or Offline/Degraded.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Verification succeeds | FEAT-05.SPEC-003 (Portal Home) | -- |
| "Send me a new link" (email unknown from context) | FEAT-05.SPEC-001 (Request Sign-In Link) | -- |

## Data Model

**Creates:** None directly -- session creation and the resulting Activity Log Entry are written by FEAT-05.SPEC-005.
**Reads:** Branding Profile -- logo and brand_colour, resolved from the token's associated freelancer once verification identifies it (or the neutral default while the freelancer is not yet known, during the Verifying state).
**Updates:** None directly -- FEAT-05.SPEC-005 updates the Client Contact's `last_sign_in` on success.
**Deletes:** None.

## Business Rules

- The same plain explanation is shown for every disqualifying reason -- expired, already used, or out-of-scope (FEAT-05.SPEC-007) -- so this screen never distinguishes "wrong company" from "expired," per XBR-09's requirement that an out-of-scope link never reveal another company's data.
- Automatic retry is never attempted on this screen: a failed or offline verification always requires an explicit tap ("Retry" or "Send me a new link"), so a stale automatic re-check can never silently consume a link's single use.
- The one-tap re-request (FEAT-05.SPEC-006) issues a new link under the invalidate-prior-link rule -- tapping it does not attempt to revalidate the link that failed.

## Edge Cases

- **Contact double-taps "Send me a new link"** -- The second tap is ignored while the first request is in flight (button loading state), mirroring FEAT-05.SPEC-001's double-submit prevention.
- **Token is well-formed but was never issued (tampered or guessed link)** -- FEAT-05.SPEC-005 reports it as invalid; this screen shows the same Error state as an expired link, with no distinct message that would hint the token format was recognized.
- **Contact clicks an old email's link after already signing in via a newer one** -- The older token verifies as already-used per FEAT-05.SPEC-006's invalidate-on-re-request rule; the Error state and re-request option are shown exactly as for any used link.
- **Connectivity drops mid-verification (after tap, before an outcome returns)** -- The screen transitions from Verifying to Offline/Degraded rather than hanging indefinitely; tapping "Retry" re-attempts verification of the same token, which remains valid to retry since a network failure never consumes the link's single use.
- **This screen never loads or updates a shared entity, so no concurrent-edit conflict applies here** -- N/A: verification's write to Client Contact's `last_sign_in` happens inside FEAT-05.SPEC-005, and that write is a single-field overwrite by a single automation, not a screen-mediated edit.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-05.SPEC-005 (Magic Link Verification) | Triggers (outbound) | Screen load triggers token verification; the automation's outcome drives this screen's state |
| FEAT-05.SPEC-006 (Link Validity & Recognition Rules) | References (inbound) | Defines what makes a token expired, used, or otherwise invalid |
| FEAT-05.SPEC-007 (Portal Access & Isolation Rules) | References (inbound) | Defines the out-of-scope condition shown identically to an expired link |
| FEAT-05.SPEC-004 (Magic Link Issuance) | Triggers (outbound) | The re-request control resubmits the known email to issuance |
| FEAT-05.SPEC-001 (Request Sign-In Link) | Navigation (outbound) | The re-request control routes here when no email is known from context |
| FEAT-05.SPEC-003 (Portal Home) | Navigation (outbound) | Successful verification navigates here |
| FEAT-02 (Proposal Creation & Sending), FEAT-06 (Deliverable Upload & Sharing), FEAT-18 (Client Contact Management & Roles) | Navigation (inbound) | Their emailed links are the entry points that land a contact here |
| FEAT-33 (Portal Referral Attribution) | Navigation (outbound) | The footer referral mark links out to the public product page |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|------------------|
| magic_link_verification_error_shown | reason (invalid_expired_or_used / out_of_scope, without further detail per the no-distinction rule above) | The screen enters the Error state | supports success-metrics.md: "Client Portal Login Success" |
| magic_link_offline_shown | -- | The screen enters the Offline/Degraded state | N/A -- no Stage 2 metric measures connectivity-caused verification interruptions; retained so this failure mode is distinguishable from an expired-link failure when reviewing login success |

## Acceptance Criteria

**FEAT-05.SPEC-002-AC-01:** Given Owen has just clicked his sign-in link, when the screen loads, then it shows the Verifying state with "Signing you in..." while FEAT-05.SPEC-005 validates the token.

**FEAT-05.SPEC-002-AC-02:** Given Owen's token verifies successfully, when verification completes, then he is navigated to FEAT-05.SPEC-003 (Portal Home) without further action.

**FEAT-05.SPEC-002-AC-03:** Given Priya clicks a link that has already been used, when verification completes, then she sees "This link isn't valid anymore" with the explanation text and a "Send me a new link" button.

**FEAT-05.SPEC-002-AC-04:** Given Owen clicks a link addressed to a different freelancer's client company than his own, when verification completes, then he sees the identical "This link isn't valid anymore" explanation as an expired link -- never another company's data.

**FEAT-05.SPEC-002-AC-05:** Given Priya is shown the Error state and her email is known from the clicked link's context, when she taps "Send me a new link," then a new link request is submitted directly and she sees the "Check your email" confirmation without re-entering her address.

**FEAT-05.SPEC-002-AC-06:** Given Owen is shown the Error state and no email is known from context, when he taps "Send me a new link," then he is navigated to FEAT-05.SPEC-001 (Request Sign-In Link) with an empty email field.

**FEAT-05.SPEC-002-AC-07:** Given Priya opens her sign-in link with no connectivity, when the screen attempts verification, then she sees "You're offline" with a "Retry" button and no verification attempt is made until she taps it.

**FEAT-05.SPEC-002-AC-08:** Given Owen taps "Retry" on the Offline/Degraded state after connectivity is restored, when the retry fires, then the screen returns to the Verifying state and re-attempts validation of the same token.

**FEAT-05.SPEC-002-AC-09:** Given Priya taps "Send me a new link" and taps it again before the first request completes, then the second tap has no effect and the button remains in its loading state.

**FEAT-05.SPEC-002-AC-10:** Given Owen clicks a well-formed but never-issued token, when verification completes, then he sees the same "This link isn't valid anymore" Error state as any other invalid link.

**FEAT-05.SPEC-002-AC-11:** Given the screen is showing any state, when it renders its header, then it shows the owning freelancer's Branding Profile logo and colour, or the neutral default when the freelancer is not yet resolved.

**FEAT-05.SPEC-002-AC-12:** Given Owen is on this screen in any state, when he taps the "Made with Clientroom" referral mark, then he is navigated externally to the public product page (FEAT-33) and this screen's own state is unaffected.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 4 | 4 |
| States | 4 (verifying, success, error, offline) | 4 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



# Screen Spec: Portal Home

## Overview

**Name:** Portal Home
**ID:** FEAT-05.SPEC-003
**Type:** Screen
**Purpose:** The contact sees their company's projects, each project's current stage and milestones, and exactly what is waiting on them, scoped to what their role allows.
**Parent Feature:** FEAT-05 -- Client Portal Access (Magic-Link Login)

## Scope and Non-Goals

**In Scope:**
- Listing the contact's client company's projects with each project's derived stage
- Showing each project's milestones and what is currently waiting on the contact (accept, review, approve, pay), limited to what the contact's role allows
- The empty state for a contact with no active projects
- Navigation into the waiting item's own feature (accept a proposal, review a deliverable, approve a milestone, pay an invoice, or invite a Reviewer colleague)

**Non-Goals:**
- Accepting proposals, reviewing deliverables, approving milestones, or paying invoices -- each is owned by its own feature (FEAT-03, FEAT-07, FEAT-08, FEAT-10); this screen only surfaces that something is waiting and links to where the action happens.
- Determining which role sees which action -- owned by FEAT-05.SPEC-007 (Portal Access & Isolation Rules), the single source of truth for role-scoped display; this screen enforces its output rather than defining it.
- Managing client contacts or roles -- excluded from this screen's scope per the Entity-Lifecycle Coverage Matrix: contact creation, role assignment, and invitation belong to FEAT-18, even though Owen can launch an invite from this screen.
- A general activity feed or notification center -- excluded per scope-boundaries.md's Feature Scope Exclusions and the Later-phase status of In-App Notification Center (FEAT-29); this screen shows current project state, not a historical feed.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-05.SPEC-002 (Link Verification Landing) | Verification succeeds | The verified Client Contact's identity, scoped freelancer, and client company |
| FEAT-33 (Portal Referral Attribution) | A contact with a live session returns via the public product page | Existing session -- no new link request |
| Any portal page (session lapse recovery) | N/A -- a lapsed session returns the contact to FEAT-05.SPEC-001, not here | N/A |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Owen (Client Primary Contact) | Full project list for his own client company: stage, milestones, and everything waiting on him | Tap through to accept, approve, pay, or review; invite a Reviewer colleague | -- |
| Priya (Client Reviewer Contact) | Full project list for her own client company: stage and milestones, and review items waiting on her | Tap through to review and comment only; no Approve, Accept, or Pay control is shown | If Priya reaches a waiting item that is Owen-only (a proposal or invoice), no such item ever appears in her waiting list, so there is no disabled control to encounter (FEAT-05.SPEC-007) |
| Nadia (Freelancer) | None -- this is a client-facing screen; Nadia never signs in as a client contact | None | Nadia has no route to this screen at all; she has her own freelancer-side project view (FEAT-01) |
| Dana (Support Operator) | None -- Dana never signs in as a client contact and has no portal access (SC-04, Access Matrix: Client Portal Access -- None) | None | Dana has no route to this screen; her read-only support session (FEAT-31) views a freelancer's account, never a client's portal |
| Unauthenticated | No | No | Redirected to FEAT-05.SPEC-001 (Request Sign-In Link) |
| Expired session | No | No | Redirected to FEAT-05.SPEC-001; in-progress reading state on this screen is simply lost, since Portal Home holds no user-entered data to preserve |

## Layout and Content

**Header:** The owning freelancer's Branding Profile logo (or neutral default), left-aligned, per XBR-31. For Owen only, an "Invite a colleague" action (right-aligned) that opens FEAT-18's invite flow.

**Body:** A vertical list of project cards, one per project in the contact's client company. Each card shows:
- Project name
- Stage label (Draft, In Progress, Complete, Cancelled, Archived -- the Project entity's derived `stage` field)
- A "Waiting on you" region, present only when at least one item needs the contact's attention, listing each waiting item (a proposal to accept, a deliverable to review, a milestone to approve, or an invoice to pay) with its type and a tap target
- Milestone summary: a compact list of the project's milestones with each one's status (Defined, Deliverable Uploaded, Approved, Reopened)

Cards are ordered with the most recent "Waiting on you" activity first, then by project recency.

**Footer:** The discreet "Made with Clientroom" referral mark (FEAT-33).

### Responsive Behavior

- **Compact breakpoint (phone width):** Project cards stack in a single column, full width; the milestone summary within a card collapses to a scrollable horizontal strip of milestone chips; the "Invite a colleague" header action collapses to an icon-only button for Owen.
- **Medium size class and above:** Project cards remain single-column but cap at a consistent platform-wide content width and center horizontally; the milestone summary within a card shows as a wrapped list instead of a horizontal strip.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| "Invite a colleague" (Owen only) | Tap | Navigates to FEAT-18's invite-a-Reviewer flow | Screen navigates away | Standard navigation transition |
| Project card | Tap | Expands or navigates to that project's underlying detail within whichever waiting-action feature applies (see Navigation Out) | Depends on destination | Standard navigation transition |
| "Waiting on you" item | Tap | Navigates directly to the specific action screen (accept, review, approve, or pay) for that item | Screen navigates away | Standard navigation transition |
| Milestone chip | Tap | Navigates to FEAT-07 (Deliverable Review & Feedback) for that milestone's deliverables | Screen navigates away | Standard navigation transition |
| "Made with Clientroom" referral mark | Tap | Navigates externally to the public product page (FEAT-33); does not affect this screen's own state | This screen's state is unchanged | Standard external navigation |

### Accessibility Notes

- **Focus order:** Header logo (skippable) -> "Invite a colleague" (Owen only) -> project cards in list order, each card's "Waiting on you" items before its milestone summary.
- **Dynamic content announcement:** When Portal Home loads, the presence of any "Waiting on you" items is announced as a summary (e.g., "2 items waiting on you") so a screen-reader user does not have to traverse every card to learn whether action is needed.
- **Keyboard alternatives:** Every card, waiting item, and milestone chip is reachable and activatable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Loaded (default) | Project list as described above | Screen loads with at least one project | User navigates away |
| Loading | Skeleton placeholders in place of project cards | Screen first opens, before project data returns | Data load completes |
| Empty | Plain message "Nothing here yet -- {freelancer name} hasn't sent you anything to review." (no error styling) | Contact has no active projects (before any proposal has been sent) | A project reaches a state the contact can see |
| Error | Error banner "We couldn't load your projects. Try again." with a Retry button | Project data fails to load | User taps Retry and load succeeds |
| Offline/Degraded | Banner "You're offline -- showing the last loaded view." above the project list; the list shown is the last successfully loaded snapshot, read-only (tapping a waiting item that requires a live connection shows the same offline notice rather than navigating) | Connectivity lost while viewing a previously loaded Portal Home | Connectivity restored -- banner clears and the screen refreshes automatically |

## Validation Rules

Validation governed by FEAT-05.SPEC-007 (Portal Access & Isolation Rules) for what a given role's session may display. This screen has no user-entered fields, so no field-level validation applies.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Owen taps a waiting proposal | FEAT-03 (Proposal Acceptance) | FEAT-03 |
| Owen or Priya taps a waiting deliverable | FEAT-07 (Deliverable Review & Feedback) | FEAT-07 |
| Owen taps a waiting milestone | FEAT-08 (Milestone Approval) | FEAT-08 |
| Owen taps a waiting invoice | FEAT-10 (Invoice Payment Processing) | FEAT-10 |
| Owen taps "Invite a colleague" | FEAT-18 (Client Contact Management & Roles) | FEAT-18 |

## Data Model

**Creates:** None.
**Reads:** Project -- `project_name`, `stage`, `currency` (for display formatting) for every project belonging to the contact's client company; Milestone -- `name`, `order`, `status` for each project's milestones; Branding Profile -- `logo`, `brand_colour`. Proposal, Invoice, and Milestone waiting-state fields are read indirectly through each owning feature's own "waiting on" determination (Proposal `status`, Invoice `status`, Milestone `status`) to compose the "Waiting on you" region.
**Updates:** None -- this screen is read-only.
**Deletes:** None.

## Business Rules

- XBR-08 and FEAT-05.SPEC-007: Priya's "Waiting on you" region never lists a proposal or invoice item, since Reviewer contacts cannot accept, approve, or pay; only Owen's region can include those item types.
- XBR-09 and FEAT-05.SPEC-007: the project list shown is scoped to exactly the contact's own client company under exactly the freelancer whose portal they signed into; a contact who is also a contact for a different freelancer sees that freelancer's projects only after a separate sign-in to that freelancer's portal (FEAT-05.SPEC-007).
- A project's `stage` label shown here is the same derived value defined in the dependency map's Project entity -- this screen never computes its own stage logic.

## Edge Cases

- **Contact has projects but none currently have anything waiting on them** -- Each project card shows its stage and milestones with no "Waiting on you" region; the screen is not treated as empty, since projects exist.
- **A waiting item is resolved by someone else while Portal Home is open (e.g., Owen approves a milestone Priya was also viewing)** -- Portal Home is a snapshot, not live-updating; the resolved item still shows as waiting until the contact reloads or navigates back to this screen, at which point the refreshed data no longer lists it. No error occurs if the contact taps the now-resolved item -- the destination screen (e.g., FEAT-08) shows its own current state.
- **Project count is large enough to require scrolling** -- The list scrolls normally; no pagination or truncation is applied, since the dependency map's Non-Functional Notes bound this feature's data volume to a few thousand freelancers with a handful of client contacts each, well within a single scrollable list.
- **Contact reaches Portal Home for a client company that was archived after the link was issued but before verification completed** -- Treated the same as "no active projects": the Empty state's message is shown rather than an error, since archiving does not delete the underlying record.
- **This screen reads shared entities (Project, Milestone) but never writes them, so no concurrent-edit conflict applies here** -- N/A: Portal Home is read-only; a change made elsewhere (by Nadia, or by the contact's own action on another screen) is reflected only on the next load, per the snapshot behavior above, not as an in-place conflict.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-05.SPEC-002 (Link Verification Landing) | Navigation (inbound) | Successful verification lands the contact here |
| FEAT-05.SPEC-007 (Portal Access & Isolation Rules) | References (inbound) | Governs role-scoped display and client isolation for everything shown here |
| FEAT-03 (Proposal Acceptance) | Navigation (outbound) | Waiting-proposal item deep-links here |
| FEAT-07 (Deliverable Review & Feedback) | Navigation (outbound) | Waiting-deliverable item and milestone chips deep-link here |
| FEAT-08 (Milestone Approval) | Navigation (outbound) | Waiting-milestone item deep-links here |
| FEAT-10 (Invoice Payment Processing) | Navigation (outbound) | Waiting-invoice item deep-links here |
| FEAT-18 (Client Contact Management & Roles) | Navigation (outbound) | Owen's "Invite a colleague" action deep-links here |
| FEAT-05.SPEC-009 (Portal Record First-View Capture) | Triggers (outbound) | Navigating from a waiting item into a proposal, deliverable, or invoice screen is the moment SPEC-009 detects a first view |
| FEAT-33 (Portal Referral Attribution) | Navigation (outbound) | The footer referral mark links out to the public product page |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|------------------|
| portal_home_viewed | role (Owen / Priya), project_count, waiting_item_count, load_time_ms | Portal Home successfully loads | supports success-metrics.md: "Client Portal Mobile Responsiveness" |
| portal_home_waiting_item_opened | item_type (proposal / deliverable / milestone / invoice), role | Contact taps a "Waiting on you" item | supports success-metrics.md: "Client Portal Login Success" (a completed sign-in that leads into the waiting action is the flow this metric ultimately protects) |

## Acceptance Criteria

**FEAT-05.SPEC-003-AC-01:** Given Owen signs in and lands on Portal Home, when the screen loads, then he sees his client company's projects, each with its stage, milestone summary, and any items waiting on him.

**FEAT-05.SPEC-003-AC-02:** Given Priya signs in and lands on Portal Home, when she views her "Waiting on you" region, then it lists only deliverables waiting for her review and never a proposal or invoice item.

**FEAT-05.SPEC-003-AC-03:** Given Owen has a project with a proposal waiting on him, when he taps that waiting item, then he is navigated to FEAT-03 (Proposal Acceptance) for that specific proposal.

**FEAT-05.SPEC-003-AC-04:** Given Owen is on Portal Home, when he taps "Invite a colleague," then he is navigated into FEAT-18's invite-a-Reviewer flow.

**FEAT-05.SPEC-003-AC-05:** Given Priya is on Portal Home, when she looks for an "Invite a colleague" action, then it is not shown, since only Owen (Primary contact) can invite.

**FEAT-05.SPEC-003-AC-06:** Given a contact with no active projects lands on Portal Home, when the screen loads, then it shows "Nothing here yet -- {freelancer name} hasn't sent you anything to review." rather than an error.

**FEAT-05.SPEC-003-AC-07:** Given Owen's project data fails to load, when the load fails, then he sees "We couldn't load your projects. Try again." with a Retry button.

**FEAT-05.SPEC-003-AC-08:** Given Priya loses connectivity while viewing a previously loaded Portal Home, when connectivity drops, then the banner "You're offline -- showing the last loaded view." appears and the last-loaded project list remains visible.

**FEAT-05.SPEC-003-AC-09:** Given Priya's connectivity returns after the offline banner appeared, when connectivity is restored, then the banner clears and the screen refreshes automatically.

**FEAT-05.SPEC-003-AC-10:** Given Owen is a client contact for two different freelancers, when he signs into each freelancer's portal separately, then each Portal Home shows only that freelancer's projects under that freelancer's own branding, never both together.

**FEAT-05.SPEC-003-AC-11:** Given Owen taps a project card with no items currently waiting on him, when the card is shown, then it displays the project's stage and milestone summary with no "Waiting on you" region.

**FEAT-05.SPEC-003-AC-12:** Given Portal Home renders for either Owen or Priya, when the page becomes interactive, then it emits `portal_home_viewed` with the load time, supporting the mobile responsiveness target.

**FEAT-05.SPEC-003-AC-13:** Given Owen is on Portal Home, when he taps the "Made with Clientroom" referral mark, then he is navigated externally to the public product page (FEAT-33) and this screen's own state is unaffected.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 5 (loaded, loading, empty, error, offline) | 5 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



# Automation Spec: Magic Link Issuance

## Overview

**Name:** Magic Link Issuance
**ID:** FEAT-05.SPEC-004
**Type:** Automation
**Purpose:** Generates a single-use, time-limited sign-in link for a recognized contact and invalidates any prior unused link for that contact.
**Parent Feature:** FEAT-05 -- Client Portal Access (Magic-Link Login)

## Scope and Non-Goals

**In Scope:**
- Looking up the submitted email against recognized Client Contacts (FEAT-18)
- Generating a single-use, time-limited token when the email matches a recognized, active contact
- Invalidating any prior unused token for that contact before issuing the new one
- Handing the issued link to FEAT-05.SPEC-008 (Magic Link Sign-In Email) for delivery
- Returning the same neutral outcome to the triggering screen regardless of whether the email matched

**Non-Goals:**
- Verifying a clicked link, creating the session, or recording the sign-in -- owned by FEAT-05.SPEC-005 (Magic Link Verification), a distinct automation with its own trigger and outcomes.
- Defining what "single-use," "time-limited," and "invalidate prior unused links" mean in detail -- owned by FEAT-05.SPEC-006 (Link Validity & Recognition Rules), the single source of truth this automation enforces rather than re-derives.
- Sending the email itself -- owned by FEAT-05.SPEC-008 (Magic Link Sign-In Email); this automation only produces the link and triggers that notification.
- Creating, inviting, or removing Client Contacts -- owned entirely by FEAT-18 per the Entity-Lifecycle Coverage Matrix; this automation only reads the existing contact roster to check recognition.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Contact submits a sign-in request | FEAT-05.SPEC-001 (Request Sign-In Link) | Fires on every validly formatted email submission, regardless of recognition outcome | Submitted email |
| Contact requests a fresh link after an expired/invalid one | FEAT-05.SPEC-002 (Link Verification Landing) | Fires when the contact taps "Send me a new link" and an email is known from the failed link's context | The email known from the expired link's context |

## Processing Logic

1. Receive the submitted email from the triggering screen.
2. Look up an active Client Contact record (status: Active) whose `email` field matches the submitted email, scoped to any freelancer's client roster (a person may be a contact for more than one freelancer, each under a separate Client Contact record per the dependency map's Relationships note).
3. If no active, recognized contact matches: take no further action beyond returning the neutral outcome to the triggering screen (Outcome: No Match). No token is generated and no email is sent.
4. If one or more active, recognized contact records match (the same email held by more than one freelancer's roster): issue a separate token per matching Client Contact record, one per freelancer, each following the remaining steps independently.
5. For each matching Client Contact: check whether an existing unused, unexpired token already exists for that contact (per FEAT-05.SPEC-006). If one exists, mark it invalidated before proceeding.
6. Generate a new single-use token bound to that Client Contact record, with an expiry set to the current time plus platform parameter: `magic-link-expiry-window` (FEAT-05.SPEC-006).
7. Hand the issued token and its associated Client Contact and Branding Profile (for the owning freelancer) to FEAT-05.SPEC-008 (Magic Link Sign-In Email) for delivery.
8. Return the neutral "request received" outcome to the triggering screen, identical to the No Match outcome in step 3.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Link issued (one match) | Submitted email matches exactly one active Client Contact | A new token is created; any prior unused token for that contact is invalidated | Same neutral confirmation as No Match -- no distinguishable feedback | FEAT-05.SPEC-001 or FEAT-05.SPEC-002 (triggering screen); FEAT-05.SPEC-008 (email delivery) |
| Link issued (multiple matches, one contact is a client of several freelancers) | Submitted email matches active Client Contact records under more than one freelancer | A new token is created and any prior unused token invalidated for each matching contact record independently | Same neutral confirmation; each freelancer's email arrives as its own message (FEAT-05.SPEC-008), each usable only for that freelancer's portal | FEAT-05.SPEC-001 or FEAT-05.SPEC-002; FEAT-05.SPEC-008 (once per matching contact) |
| No match | Submitted email matches no active Client Contact | None | Same neutral confirmation as a successful issuance -- the contact cannot tell recognition failed | FEAT-05.SPEC-001 or FEAT-05.SPEC-002 |
| Match found but contact is Removed | Submitted email matches a Client Contact whose `status` is Removed | None -- treated identically to No Match | Same neutral confirmation | FEAT-05.SPEC-001 or FEAT-05.SPEC-002 |
| Automation failure | Processing error while looking up recognition or generating the token | None persisted -- the operation is atomic per contact match | Same neutral confirmation is still shown (the triggering screen never surfaces an issuance failure, since doing so would itself leak whether the email matched); the request is treated as failed to deliver and no email attempt is made | FEAT-05.SPEC-001 or FEAT-05.SPEC-002 |

## Data Model

**Reads:** Client Contact -- `email`, `status`, the freelancer and client company it belongs to; Sign-in link/token (feature-internal) -- any existing unused, unexpired token for the matched contact.
**Creates:** Sign-in link/token (feature-internal, not a Domain Entity Inventory record) -- one per matched, active Client Contact, per FEAT-05.SPEC-006.
**Updates:** Sign-in link/token -- marks any prior unused token for the matched contact as invalidated.
**Deletes:** None.

## Business Rules

- XBR-28: only Client Contacts added through FEAT-18 and currently Active are ever issued a link; a Removed contact's email is treated identically to an unrecognized one.
- FEAT-05.SPEC-006 governs single-use, time-limit, and invalidate-on-re-request behavior end to end; this automation enforces those rules rather than defining its own token lifetime or reuse logic.
- The neutral outcome (step 8) is returned identically whether or not a match was found, whether one or several freelancers matched, and even on an internal processing failure -- this automation never produces a triggering-screen-visible signal that would let a submitted email be tested for recognition.
- A person who is a Client Contact for several freelancers receives one issued link per freelancer, each scoped and invalidated independently -- issuing or invalidating one freelancer's link never affects another's (FEAT-05.SPEC-007).

## Edge Cases

- **Submitted email has mixed case or surrounding whitespace** -- Comparison against Client Contact `email` is case-insensitive and whitespace-trimmed, consistent with `email` being unique within a client company per the dependency map.
- **Contact is re-added (Removed, then re-invited) between two requests** -- The current `status` at the moment of this request governs; a request made while Removed yields No Match, and a later request made after re-activation issues a link normally.
- **Contact requests a new link while an unused token from the previous request is still valid** -- The prior token is invalidated as part of step 5 before the new one is generated, per FEAT-05.SPEC-006; the previous email's link, if since opened, shows the "not valid anymore" explanation (FEAT-05.SPEC-002).
- **Concurrent trigger firing (the same contact submits two requests within moments, e.g., a double-click across two browser tabs)** -- Each request runs independently; whichever completes last wins the invalidation race, since it invalidates whatever token existed when it read state, and its own newly generated token remains the valid one. At most one token per contact survives the pair.
- **Trigger fires while a previous run for the same contact is still in flight** -- The dependency-invalidate step reads the token state at the moment it runs, so a second request that starts before the first finishes generating its token will (once it reaches step 5) invalidate whichever token exists at that instant, including one generated moments earlier by the still-finishing first run; the end state always has at most one valid token per contact, consistent with the single-valid-link business rule above.
- **Delivery capability (FEAT-14) is unavailable when handing off to FEAT-05.SPEC-008** -- The token is still created; delivery failure and retry behavior belong to FEAT-05.SPEC-008 and the underlying Transactional Email Delivery capability (FEAT-14.SPEC-001), not to this automation, which completes once the hand-off is made.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-05.SPEC-001 (Request Sign-In Link) | Triggered by (inbound) | Every submitted request fires this automation |
| FEAT-05.SPEC-002 (Link Verification Landing) | Triggered by (inbound) | The one-tap re-request (when the email is known from context) fires this automation |
| FEAT-05.SPEC-006 (Link Validity & Recognition Rules) | References (inbound) | Defines token lifetime, single-use, and invalidation rules enforced here |
| FEAT-05.SPEC-008 (Magic Link Sign-In Email) | Affects (outbound) | The issued token and contact/branding context are handed off for delivery |
| FEAT-18 (Client Contact Management & Roles) | References (inbound) | Recognition is checked against the Client Contact roster FEAT-18 maintains |

## Analytics and Success Signals

- **magic_link_issued** (matched_freelancer_count: 1 or more) -- supports success-metrics.md: "Client Portal Login Success"
- **magic_link_issuance_no_match** () -- N/A -- no Stage 2 metric measures unrecognized-email attempts directly, and this event is never surfaced to the requester (the neutral outcome rule above); retained internally so recognition failures are distinguishable from delivery failures when diagnosing login success shortfalls.

## Acceptance Criteria

**FEAT-05.SPEC-004-AC-01:** Given Owen submits his own recognized, Active email on FEAT-05.SPEC-001, when the automation runs, then a new single-use token is generated and handed to FEAT-05.SPEC-008 for delivery.

**FEAT-05.SPEC-004-AC-02:** Given Priya submits an email that matches no Client Contact, when the automation runs, then no token is created and the triggering screen shows the identical confirmation as a successful issuance.

**FEAT-05.SPEC-004-AC-03:** Given Owen has an unused, unexpired token from a prior request, when he submits a new request, then the prior token is invalidated before the new token is generated.

**FEAT-05.SPEC-004-AC-04:** Given a person's email matches Active Client Contact records under two different freelancers, when the automation runs, then a separate token is issued for each freelancer, each independently scoped and each triggering its own FEAT-05.SPEC-008 email.

**FEAT-05.SPEC-004-AC-05:** Given a Client Contact whose `status` is Removed submits their email, when the automation runs, then the outcome is identical to No Match -- no token is issued.

**FEAT-05.SPEC-004-AC-06:** Given Priya submits her email with extra whitespace and different letter casing than stored, when the automation runs, then it still matches her Client Contact record and issues a token.

**FEAT-05.SPEC-004-AC-07:** Given Owen submits two requests within moments of each other from two open tabs, when both complete, then exactly one valid token remains for Owen's contact record.

**FEAT-05.SPEC-004-AC-08:** Given the automation encounters an internal processing error while checking recognition, when it fails, then the triggering screen still shows the same neutral confirmation as a successful request, and no email is sent.

**FEAT-05.SPEC-004-AC-09:** Given Priya taps "Send me a new link" on FEAT-05.SPEC-002 with her email known from context, when the automation runs, then it processes identically to a fresh submission from FEAT-05.SPEC-001, including invalidating her prior token.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 | 2 |
| Outcome Paths | 5 | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Automation Spec: Magic Link Verification

## Overview

**Name:** Magic Link Verification
**ID:** FEAT-05.SPEC-005
**Type:** Automation
**Purpose:** Validates a clicked link, creates the scoped session, and records the sign-in.
**Parent Feature:** FEAT-05 -- Client Portal Access (Magic-Link Login)

## Scope and Non-Goals

**In Scope:**
- Validating the clicked token against FEAT-05.SPEC-006's single-use and time-limit rules
- Checking the token resolves to a contact and scope the requesting browser is entitled to see (FEAT-05.SPEC-007)
- Marking the token used and creating the scoped portal session on success
- Updating the Client Contact's `last_sign_in` timestamp on success
- Triggering the Activity Log Entry for the sign-in (FEAT-13)

**Non-Goals:**
- Generating the token in the first place, or invalidating a prior one on re-request -- owned by FEAT-05.SPEC-004 (Magic Link Issuance); this automation only consumes a token FEAT-05.SPEC-004 already produced.
- Defining single-use, time-limit, and recognized-contact rules -- owned by FEAT-05.SPEC-006, enforced here rather than re-derived.
- Defining client isolation and role-scoped display -- owned by FEAT-05.SPEC-007; this automation applies that spec's scoping when creating the session, but does not define the scoping logic itself.
- Rendering the verification progress, success transition, or error explanation -- owned by FEAT-05.SPEC-002 (Link Verification Landing), the sole screen that triggers and displays the outcome of this automation.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Contact opens the emailed link | FEAT-05.SPEC-002 (Link Verification Landing) | Fires automatically when the screen loads with a token | The token embedded in the clicked link |

## Processing Logic

1. Receive the token from the triggering screen.
2. Look up the token among issued Sign-in link/tokens. If no such token exists, proceed to the Invalid outcome.
3. If the token exists, check its current state per FEAT-05.SPEC-006: already used, expired (past platform parameter: `magic-link-expiry-window` from issuance), or invalidated by a later re-request. If any of these apply, proceed to the corresponding outcome (each rendered identically by FEAT-05.SPEC-002, per that spec's Business Rules).
4. If the token is valid and unused, resolve the Client Contact it is bound to, and confirm that contact's `status` is still Active (a contact could be removed between issuance and click). If not Active, proceed to the Invalid outcome.
5. Apply FEAT-05.SPEC-007's isolation rules to determine the scope (the specific freelancer and client company) the resulting session is bound to.
6. Mark the token used (this is the single consuming read -- no other verification attempt against the same token can succeed afterward).
7. Create the scoped portal session, bound to the resolved Client Contact, freelancer, and client company.
8. Update the Client Contact's `last_sign_in` field to the current timestamp.
9. Trigger an Activity Log Entry for the sign-in, handed to FEAT-13 (XBR-05): event_type "client portal sign-in," actor the Client Contact, occurred_at now, affected_record the Client Contact, project none (a sign-in is account-level, not project-level).
10. Return the Success outcome, carrying the new session, to the triggering screen.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Success | Token exists, is unused, unexpired, not invalidated, and its Client Contact is Active | Token marked used; scoped session created; Client Contact `last_sign_in` updated; Activity Log Entry written | Screen transitions from Verifying directly to Portal Home | FEAT-05.SPEC-002 (transition); FEAT-05.SPEC-003 (destination); FEAT-13 (trail entry) |
| Invalid -- unknown token | No matching token found | None | FEAT-05.SPEC-002 shows the plain "not valid anymore" explanation | FEAT-05.SPEC-002 |
| Invalid -- already used | Token exists but was already marked used by an earlier verification | None | Same plain explanation as unknown token | FEAT-05.SPEC-002 |
| Invalid -- expired | Token exists, unused, but past its expiry | None | Same plain explanation | FEAT-05.SPEC-002 |
| Invalid -- invalidated by re-request | Token exists, unused, unexpired, but superseded by a later issuance for the same contact (FEAT-05.SPEC-006) | None | Same plain explanation | FEAT-05.SPEC-002 |
| Invalid -- contact no longer Active | Token is otherwise valid, but the bound Client Contact's `status` is no longer Active | None | Same plain explanation | FEAT-05.SPEC-002 |
| Failure (processing error) | An internal error occurs during lookup, session creation, or the `last_sign_in` write | No partial state -- if the session cannot be created, the token is not marked used, so the contact retains the ability to retry the same click | FEAT-05.SPEC-002 shows its Offline/Degraded state when the failure is connectivity-related, or otherwise the same "not valid anymore" explanation as any invalid token, with the link remaining usable to retry since it was never consumed | FEAT-05.SPEC-002 |

## Data Model

**Reads:** Sign-in link/token -- state (issued, used, expired, invalidated), bound Client Contact reference; Client Contact -- `status`, freelancer and client company it belongs to.
**Creates:** Portal session (feature-internal) -- scoped to the resolved Client Contact, freelancer, and client company; Activity Log Entry -- via FEAT-13, on successful sign-in.
**Updates:** Sign-in link/token -- marked used; Client Contact -- `last_sign_in` set to the current timestamp.
**Deletes:** None.

## Business Rules

- XBR-05: every successful sign-in writes an append-only Activity Log Entry with actor and timestamp; this is the only record this automation contributes to the trail.
- FEAT-05.SPEC-006 governs every condition that makes a token invalid (unknown, used, expired, invalidated); this automation checks those conditions but does not redefine them.
- FEAT-05.SPEC-007 governs the scope the resulting session is bound to; a session is never created with a broader scope than the one Client Contact record it is bound to, even when that person holds contact records for other freelancers.
- Marking a token used (step 6) happens only once verification has otherwise fully succeeded up to that point; a token is never marked used and then subsequently fail this automation for a different reason, which would strand the contact with no usable link and no way to know why.

## Edge Cases

- **Token is opened twice in rapid succession (e.g., an email client pre-fetches the link, then the contact taps it)** -- Whichever verification reaches step 6 first marks the token used; the second verification finds the token already used and returns the Invalid -- already used outcome. To avoid stranding a legitimate contact behind an email pre-fetch, FEAT-05.SPEC-008's link is constructed so that automated pre-fetching by mail clients does not itself consume the token (the link requires the contact's own tap-through action, not merely being fetched as a preview resource).
- **Concurrent trigger firing (the same token opened on two devices at once)** -- Only one verification can mark the token used; per the business rule above, the losing attempt sees Invalid -- already used, never a partial or inconsistent session.
- **Trigger fires while a previous run for the same token is still in flight** -- The token's used-state check and used-state write are treated as a single atomic step (6); a second verification arriving before the first completes waits for that step to resolve and then evaluates the token's now-current state, guaranteeing at most one Success outcome per token.
- **Client Contact is removed (FEAT-18) after the link was issued but before it is clicked** -- Verification proceeds through steps 1-4 and returns Invalid -- contact no longer Active at step 4, before any session is created.
- **The token's bound client company is archived between issuance and click** -- The session is still created if the contact and token otherwise verify (archiving a client does not revoke an already-issued, unused, unexpired link); FEAT-05.SPEC-003 (Portal Home) then reflects that project's archived state normally rather than this automation blocking sign-in.
- **Verification succeeds but the Activity Log Entry hand-off to FEAT-13 fails** -- The session is still created and the contact still reaches Portal Home; the trail write is retried by FEAT-13's own delivery guarantee, since blocking a successful sign-in on a downstream logging failure would contradict this automation's own Success outcome already returned to the contact.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-05.SPEC-002 (Link Verification Landing) | Triggered by (inbound) | Screen load with a token fires this automation; the outcome drives the screen's state |
| FEAT-05.SPEC-006 (Link Validity & Recognition Rules) | References (inbound) | Defines every condition checked in steps 2-4 |
| FEAT-05.SPEC-007 (Portal Access & Isolation Rules) | References (inbound) | Defines the scope applied when creating the session in step 5 |
| FEAT-05.SPEC-003 (Portal Home) | Affects (outbound) | Destination the contact reaches on Success |
| FEAT-13 (Immutable Activity & Audit Trail) | Affects (outbound) | Receives the sign-in trail entry (XBR-05) |
| FEAT-18 (Client Contact Management & Roles) | References (inbound) | Supplies the Client Contact's current `status` checked in step 4 |

## Analytics and Success Signals

- **magic_link_used** (outcome: success) -- supports success-metrics.md: "Client Portal Login Success"
- **magic_link_expired** (reason: expired / already_used / invalidated_by_reissue / contact_inactive / unknown_token) -- supports success-metrics.md: "Client Portal Login Success" (a failed-or-expired attempt recoverable through one additional request is exactly what this metric's target measures)

## Acceptance Criteria

**FEAT-05.SPEC-005-AC-01:** Given Owen clicks a valid, unused, unexpired link addressed to his Active Client Contact record, when verification runs, then the token is marked used, his `last_sign_in` is updated, a sign-in Activity Log Entry is written, and he is delivered to FEAT-05.SPEC-003 (Portal Home).

**FEAT-05.SPEC-005-AC-02:** Given Priya clicks a link that has already been used, when verification runs, then the outcome is Invalid -- already used, and FEAT-05.SPEC-002 shows the plain explanation.

**FEAT-05.SPEC-005-AC-03:** Given Owen clicks a link after platform parameter: `magic-link-expiry-window` has elapsed since issuance, when verification runs, then the outcome is Invalid -- expired.

**FEAT-05.SPEC-005-AC-04:** Given Owen requests a new link while an old one is still unused, then clicks the old link, when verification runs, then the outcome is Invalid -- invalidated by re-request.

**FEAT-05.SPEC-005-AC-05:** Given Priya's Client Contact record was removed by FEAT-18 after her link was issued, when she clicks the link, then the outcome is Invalid -- contact no longer Active, and no session is created.

**FEAT-05.SPEC-005-AC-06:** Given a fabricated or guessed token that was never issued, when verification runs, then the outcome is Invalid -- unknown token, shown identically to any other invalid outcome.

**FEAT-05.SPEC-005-AC-07:** Given a valid token is opened on two devices at effectively the same moment, when both verifications run, then exactly one succeeds and the other returns Invalid -- already used.

**FEAT-05.SPEC-005-AC-08:** Given an email client pre-fetches Owen's sign-in link as a preview without his own tap, when the pre-fetch occurs, then the token is not consumed, and Owen's own subsequent tap still verifies successfully.

**FEAT-05.SPEC-005-AC-09:** Given verification succeeds but the Activity Log Entry hand-off to FEAT-13 fails, when this occurs, then Owen still reaches Portal Home normally, and the trail entry is retried by FEAT-13 rather than blocking his sign-in.

**FEAT-05.SPEC-005-AC-10:** Given a processing error occurs before the token is marked used, when the error occurs, then the token remains valid and unused, and the contact can retry the same link.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 7 | 7 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Link Validity & Recognition Rules

## Overview

**Name:** Link Validity & Recognition Rules
**ID:** FEAT-05.SPEC-006
**Type:** Logic/Rule
**Purpose:** Governs single-use and time-limited link enforcement, invalidation on re-request, and that only recognized, Active contacts can obtain or use a link.
**Parent Feature:** FEAT-05 -- Client Portal Access (Magic-Link Login)
**Governed Entity:** Sign-in link/token (feature-internal, not a Domain Entity Inventory record)

## Scope and Non-Goals

**In Scope:**
- Field-level rules for the sign-in token: how it is generated, when it expires, and what makes it valid to consume
- The invalidate-prior-link rule triggered by a re-request
- Recognized-contact enforcement: which submitted emails result in a token at all
- Authorization for who may request and who may use a link
- The no-enumeration rule: identical outward behavior whether or not an email is recognized

**Non-Goals:**
- Client isolation and role-scoped portal display once a session exists -- owned by FEAT-05.SPEC-007 (Portal Access & Isolation Rules); this spec governs the token's own lifecycle, not what a resulting session may see.
- The screens that collect an email or show a verification outcome -- owned by FEAT-05.SPEC-001 and FEAT-05.SPEC-002, which enforce these rules but do not define them.
- The processing steps that generate, consume, and act on a token -- owned by FEAT-05.SPEC-004 (Magic Link Issuance) and FEAT-05.SPEC-005 (Magic Link Verification); this spec defines the rules those automations enforce.
- Client Contact creation, role assignment, or removal -- owned by FEAT-18 per the Entity-Lifecycle Coverage Matrix; this spec only reads a contact's current `status` to decide recognition.

## Governed Entity

**Entity:** Sign-in link/token
**Source:** Feature Breakdown Brief, Shared Context ("Sign-in link/token (feature-internal, not a Domain Entity Inventory record) -- created by FEAT-05.SPEC-004, consumed exactly once by FEAT-05.SPEC-005, governed end to end by FEAT-05.SPEC-006")

| Field | Data Type | Description |
|-------|-----------|-------------|
| token_value | text | The single-use credential embedded in the emailed link |
| client_contact | text (reference) | The Client Contact record this token authenticates |
| issued_at | date (timestamp) | When the token was generated |
| expires_at | derived | `issued_at` + platform parameter: `magic-link-expiry-window` |
| status | enum | Issued (unused, unexpired), Used, Expired, Invalidated (superseded by a later re-request) |
| used_at | date (timestamp) | When the token was marked Used, if applicable |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-05.SPEC-001 | Request Sign-In Link | Email-format validation on blur and submit; no-enumeration confirmation on every submit |
| FEAT-05.SPEC-002 | Link Verification Landing | Displays the outcome of validity checks identically regardless of which rule failed |
| FEAT-05.SPEC-004 | Magic Link Issuance | Recognition check, prior-token invalidation, and new-token generation during processing |
| FEAT-05.SPEC-005 | Magic Link Verification | Token-state checks (unused, unexpired, not invalidated) and contact-Active check during processing |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| Submitted email (FEAT-05.SPEC-001 input) | Must be a validly formatted email address | Always | On blur, on submit | "Enter a valid email address" | Yes |
| token_value | Generated as a single, unguessable value per issuance -- no validation rule accepts a caller-supplied value | Always (system-generated only) | At generation | N/A -- never user input | N/A |
| client_contact | No validation beyond data type -- set internally to the recognized, Active Client Contact resolved by FEAT-05.SPEC-004; never user-editable | Always | At generation | N/A -- never user input | N/A |
| issued_at | No validation beyond data type -- system-set to the current timestamp at generation | Always | At generation | N/A -- never user input | N/A |
| expires_at | Always exactly `issued_at` + platform parameter: `magic-link-expiry-window`; not user-configurable | Always | At generation | N/A -- never user input | N/A |
| status | Must transition only Issued -> Used, Issued -> Expired, or Issued -> Invalidated; never backward | Always | At every state check | N/A -- internal state, no user-facing error | Yes (enforced by the automations, not surfaced as a field error) |
| used_at | No validation beyond data type -- system-set by FEAT-05.SPEC-005 at the moment a token is marked Used; remains empty for a token never used | On use | At verification (FEAT-05.SPEC-005) | N/A -- never user input | N/A |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Single valid token per contact | client_contact, status | At most one token with status Issued may exist for a given client_contact at any time; issuing a new one first invalidates any existing Issued token for that contact | N/A -- enforced silently as part of issuance; no error is shown to the requester (the neutral confirmation applies regardless) |
| Expiry precedes use | expires_at, used_at, status | A token whose current time has passed expires_at cannot transition to Used, even if otherwise unused; the check evaluates status as Expired instead | N/A -- surfaced to the contact only as the shared "not valid anymore" explanation on FEAT-05.SPEC-002 |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Request a sign-in link (submit an email) | Owen, Priya (any recognized, Active Client Contact) | Always -- the request screen accepts any email as input | N/A -- there is no denial state on request; an unrecognized or inactive email is accepted by the form and simply produces no token (No Match outcome, FEAT-05.SPEC-004), shown as the identical "Check your email" confirmation, never a distinct denial |
| Request a sign-in link | Nadia (Freelancer) | Never as a recognized client contact -- Nadia holds no Client Contact record of her own | Nadia's own email submission is processed exactly like any unrecognized email: the same neutral confirmation is shown and no link is issued |
| Request a sign-in link | Dana (Support Operator) | Never -- Dana holds no Client Contact record (SC-04) | Same neutral confirmation as any unrecognized email; no link is issued |
| Use (click) a valid, unused, unexpired token | Owen, Priya (the specific recognized, Active Client Contact the token is bound to) | Token must be Issued (not Used, Expired, or Invalidated) and its bound contact must be Active | The plain "This link isn't valid anymore" explanation on FEAT-05.SPEC-002, with a one-tap re-request; identical for every disqualifying reason |
| Use a token bound to a different contact than the one clicking it | Nobody | Never -- a token authenticates exactly the Client Contact it was generated for; there is no "use on behalf of" path | Same "not valid anymore" explanation; the token simply does not resolve to a usable session for anyone other than its bound contact, since possession of the link (not a separate identity check) is the only claim a browser can make, and an already-used or expired token fails identically for any holder |
| Re-request a link while a prior unused token exists | Owen, Priya (the contact who holds the prior token) | Always, per the recognized-contact rule above | N/A -- re-requesting is never denied; it always invalidates the prior token per the Cross-Field Rule above |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|---------------------|
| token_value | System-generated, single, unguessable value | On create (issuance) | No |
| issued_at | Current timestamp | On create (issuance) | No |
| expires_at | issued_at + platform parameter: `magic-link-expiry-window` | On create (issuance), fixed thereafter | No |
| status | Issued | On create (issuance) | No -- transitions only through the automations (Used by FEAT-05.SPEC-005, Expired by time passing, Invalidated by FEAT-05.SPEC-004 on re-request) |

## Business Rules

- XBR-28: magic links are single-use and time-limited; requesting a new link invalidates earlier unused ones; only contacts added through Client Contact Management & Roles (FEAT-18) are recognized. This spec is the rule's owning authority per the dependency map.
- The recognized-contact check and the no-enumeration confirmation together mean this feature never reveals, through timing, wording, or any other observable difference, whether a submitted email belongs to a client contact of any freelancer on the platform.
- A token's validity is evaluated fresh at the moment of use (FEAT-05.SPEC-005), never cached from issuance time -- a token that was valid when emailed but has since expired, been used, or been invalidated by re-request fails at click time, not before.
- Invalidating a prior token (on re-request) is a state transition (Issued -> Invalidated), never a deletion -- the record persists so a subsequent click against it resolves to the same shared "not valid anymore" explanation rather than an unknown-token error, keeping the contact's experience identical either way.

## Edge Cases

- **Contact requests a link, lets it expire, then requests again** -- The expired token remains status Expired (it was never re-requested against, so no separate Invalidated transition applies); the new token is Issued fresh with its own full expiry window.
- **Contact requests a link twice within the same instant that a still-forming first token has not yet reached Issued status** -- Per FEAT-05.SPEC-004's concurrency handling, whichever request's invalidation step runs last determines the single surviving Issued token; no scenario leaves two simultaneously Issued tokens for one contact.
- **Token is at the exact expiry boundary (used at precisely expires_at)** -- Treated as expired; the boundary itself is not valid, since "time-limited" means strictly before expires_at.
- **A recognized contact's status changes from Active to Removed between issuance and click** -- The click-time Authorization Rules check re-evaluates `status` at use, not at issuance, so a Removed contact's still-unused token fails per the "Use a valid token" row's Active-contact condition.
- **A person holds Client Contact records under two different freelancers with the same email** -- Each freelancer's token is entirely independent under this spec's rules: invalidating or using one contact's token has no effect on the other's, per the Cross-Field Rule's scoping to a single client_contact.
- **Submitted email matches a contact record exactly except for case or whitespace** -- Recognition matching normalizes case and trims whitespace before comparison (consistent with FEAT-05.SPEC-004's processing logic), so the rule set treats these as the same email.

## Acceptance Criteria

**FEAT-05.SPEC-006-AC-01:** Given Owen submits a validly formatted, recognized, Active email, when the request is processed, then a new token is issued with status Issued and an expiry of platform parameter: `magic-link-expiry-window` from now.

**FEAT-05.SPEC-006-AC-02:** Given Priya submits an email matching no Client Contact, when the request is processed, then no token is created and she sees the identical confirmation as a recognized submission.

**FEAT-05.SPEC-006-AC-03:** Given Dana (Support Operator) submits her own email, when the request is processed, then no token is issued, since she holds no Client Contact record, and she sees the same neutral confirmation.

**FEAT-05.SPEC-006-AC-04:** Given Owen has an Issued, unused token, when he requests a new link, then the prior token transitions to Invalidated before the new token is issued.

**FEAT-05.SPEC-006-AC-05:** Given Owen's token's expires_at has passed and it was never used, when he clicks it, then it is treated as Expired and he sees "This link isn't valid anymore."

**FEAT-05.SPEC-006-AC-06:** Given Priya clicks a token exactly at its expiry instant, when verification checks it, then it is treated as expired (the boundary itself does not verify).

**FEAT-05.SPEC-006-AC-07:** Given Owen's Client Contact status changes to Removed after his token was issued but before he clicks it, when he clicks the still-unused, unexpired token, then it fails per the Active-contact condition and he sees the plain explanation.

**FEAT-05.SPEC-006-AC-08:** Given a person is a Client Contact for two different freelancers with the same email, when they request a link, then each freelancer issues and governs its own token independently.

**FEAT-05.SPEC-006-AC-09:** Given Owen submits his email with different letter casing and surrounding whitespace than stored, when the request is processed, then it still matches his Client Contact record.

**FEAT-05.SPEC-006-AC-10:** Given Priya's token has already transitioned to Used, when she clicks the same link again, then it is treated as invalid and she sees the plain explanation, never a distinct "already signed in" message that would confirm the token had once been valid.

**FEAT-05.SPEC-006-AC-11:** Given Owen requests a link and lets it expire without ever re-requesting, when he later requests a fresh link, then the fresh token is Issued with a full new expiry window, independent of the expired one.

**FEAT-05.SPEC-006-AC-12:** Given Priya's token is Invalidated by a later re-request, when she clicks the invalidated (not the new) link, then she sees the same plain explanation as an expired or used link.

**FEAT-05.SPEC-006-AC-13:** Given Nadia (Freelancer) submits her own sign-in email on the client portal's request screen, when the request is processed, then no token is issued to her, since she holds no Client Contact record, and she sees the same neutral confirmation as any other submission.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 7 | 7 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 6 | 6 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



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



# Notification Spec: Magic Link Sign-In Email

## Overview

**Name:** Magic Link Sign-In Email
**ID:** FEAT-05.SPEC-008
**Type:** Notification
**Purpose:** Emails the one-time sign-in link to the requesting contact whenever a link is issued, so they can enter their scoped portal without a password.
**Parent Feature:** FEAT-05 -- Client Portal Access (Magic-Link Login)

## Scope and Non-Goals

**In Scope:**
- The single-recipient email delivered on every successful magic-link issuance
- Content and branding for the email, on the one channel this communication uses
- Retry and expiry behavior tied to the underlying token's own time limit

**Non-Goals:**
- Deciding whether an email is recognized and generating the token -- owned by FEAT-05.SPEC-004 (Magic Link Issuance); this spec begins once issuance hands off an already-generated token.
- A standalone Integration spec for the underlying email-sending capability -- the transactional email capability this notification relies on is owned by FEAT-14 (External Touchpoints table) and recorded here only as a cross-feature touchpoint, consistent with how FEAT-02 and FEAT-03 use the same capability.
- In-app or push delivery of the sign-in link -- excluded because a contact with no active session, by definition, has no in-app surface to receive a push or in-app notification on; email is the only channel that can reach someone who is not currently signed in.
- Notifying anyone other than the requesting contact -- excluded per BRIEF.md's Constraints on strict client isolation: this is a single-recipient, single-purpose credential email, never CC'd or forwarded to another contact or to the freelancer.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, on every successful issuance (FEAT-05.SPEC-004) | The recipient has no active session and no in-app surface to reach; email is the only channel that can deliver a credential to someone who is, by definition, signed out (BRIEF.md, Target Users & Roles: contacts "sign in passwordless with a magic link by email") |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Link issued | FEAT-05.SPEC-004 (Magic Link Issuance) | Fires once per matched Client Contact record when a token is successfully generated | Token (link), Client Contact's `email`, the owning freelancer's Branding Profile |

## Audience and Preferences

**Recipients:** The single Client Contact (Owen or Priya, per the Access Matrix) whose email was recognized and matched by FEAT-05.SPEC-004. When a person is a Client Contact for more than one freelancer, each freelancer's issuance sends its own separate email to that same address, each carrying only that freelancer's own link and branding.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|----------------------|
| None | N/A | N/A | N/A |

This email carries a single-use sign-in credential the contact themselves just requested; per XBR-30, transactional emails core to the record always send and are not subject to an optional-notification preference. There is no preference screen for a client contact to manage, consistent with the Access Matrix (client contacts have no Settings surface of their own).

**Quiet Hours:** N/A -- this email is a direct, immediate response to the contact's own just-completed action (requesting a link) and carries a time-limited credential; holding it for a quiet-hours window would shrink the usable portion of the token's own time limit and contradict the immediacy the request implies.

## Content Definition

**Email:**
- **Subject:** Sign in to your {freelancer_business_name} client portal
- **Body:**
  Hi {contact_name},

  Click the button below to sign in. This link is single-use and expires soon, so use it right away.

  If you didn't request this, you can ignore this email -- no one can sign in without clicking the link themselves.
- **CTA (button):** Sign in -- deep-links to FEAT-05.SPEC-002 (Link Verification Landing) with the issued token, which then routes to FEAT-05.SPEC-003 (Portal Home) on success

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|--------------------------|----------------|------------------------|
| {freelancer_business_name} | Freelancer Account -- business_name (or, before that is set, the freelancer's own name) | Nadia Ross Design | Renders as "your client portal" (subject becomes "Sign in to your client portal") -- required before the first invoice per the dependency map, but this email can fire earlier, so the fallback covers that window |
| {contact_name} | Client Contact -- name | Owen Carter | Greeting renders as "Hi there," |

The freelancer's Branding Profile `logo` and `brand_colour` (or the neutral default) apply to the email's visual header per XBR-31; the discreet "Made with Clientroom" referral mark appears per XBR-32, alongside the branding without overriding it.

## Delivery Rules

**Batching:** None -- each issuance produces exactly one email for exactly one token; a contact requesting a fresh link before a prior one is used still receives a new, separate email for the new token (the prior token is invalidated per FEAT-05.SPEC-006, so there is never more than one valid credential to batch).
**Deduplication:** At most one email per issued token -- FEAT-05.SPEC-004 triggers this notification exactly once per successful issuance; a token is never re-issued, so no duplicate email for the same token can occur.
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001). Because the underlying token is single-use and time-limited (FEAT-05.SPEC-006), a retry that succeeds after part of the token's validity window has already elapsed still delivers a usable link as long as it arrives before platform parameter: `magic-link-expiry-window` has fully elapsed from issuance; the retry window is chosen to fit inside the token's own expiry so a delayed-but-successful delivery is not itself worthless.
**Expiry:** If delivery has not succeeded after the final retry, no further attempt is made and the delivery failure is surfaced to the freelancer as a warning (XBR-30), since email is the only channel reaching the contact and a silently lost sign-in email would leave the contact unable to reach the portal at all, with no indication anything was wrong.

## Edge Cases

- **Contact's email bounces (invalid or full mailbox)** -- Treated as a delivery failure per the Retry on failure rule; after the final retry, the bounce is surfaced to the freelancer as a delivery warning on the client's contact record (FEAT-14, XBR-30), since the freelancer -- not the contact, who never confirms receipt -- is the only party positioned to correct the contact's email.
- **Contact requests a new link before this email for the prior link has finished sending** -- The prior token is invalidated (FEAT-05.SPEC-006) independently of this email's delivery state; if the first email later succeeds in delivering, its link shows the shared "not valid anymore" explanation when clicked, since the token itself -- not the email -- carries validity.
- **The token expires before the email is delivered (extreme delivery delay)** -- The delivered email's link will show "not valid anymore" when clicked; this is treated as an issuance/delivery-timing rarity rather than a notification defect, since platform parameter: `magic-link-expiry-window` is set wide enough that a normal delivery, including one retry cycle, comfortably completes within it.
- **A person is a Client Contact for two freelancers and requests a link from a page that only shows one context** -- Each freelancer's matched issuance triggers its own separate email, sent independently and simultaneously; the two emails carry different branding and different tokens, and neither references the other.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-05.SPEC-004 (Magic Link Issuance) | Triggered by (inbound) | Every successful issuance fires this notification |
| FEAT-05.SPEC-006 (Link Validity & Recognition Rules) | References (inbound) | Governs the token's expiry window this email's retry rule is bounded by |
| FEAT-05.SPEC-002 (Link Verification Landing) | Navigation (outbound) | The "Sign in" CTA deep-links here with the issued token |
| FEAT-19 (Freelancer Branding) | References (inbound) | Supplies the logo and brand colour applied to this email |
| FEAT-33 (Portal Referral Attribution) | References (inbound) | Supplies the referral mark shown alongside branding |
| FEAT-14 (Notifications (Email)) | References (inbound) | Owns the underlying Transactional Email Delivery capability (FEAT-14.SPEC-001) this notification is sent through |

## Analytics and Success Signals

- **magic_link_email_sent** (matched_freelancer_count context inherited from the triggering issuance) -- supports success-metrics.md: "Client Portal Login Success"
- **magic_link_email_delivery_failed** (retry_count_exhausted: true) -- supports success-metrics.md: "Client Portal Login Success" (a lost sign-in email is the clearest cause of a failed first-try sign-in this metric tracks)

## Acceptance Criteria

**FEAT-05.SPEC-008-AC-01:** Given Owen's email is recognized and a token is issued, when FEAT-05.SPEC-004 completes, then Owen receives an email with subject "Sign in to your {his freelancer's business name} client portal" and a "Sign in" button linking to his issued token.

**FEAT-05.SPEC-008-AC-02:** Given Priya's freelancer's `business_name` is not yet set, when her sign-in email is composed, then the subject renders as "Sign in to your client portal" and the greeting still uses her name.

**FEAT-05.SPEC-008-AC-03:** Given Owen taps the "Sign in" button in the email, when the link opens, then he lands on FEAT-05.SPEC-002 (Link Verification Landing) with his token.

**FEAT-05.SPEC-008-AC-04:** Given Owen is a Client Contact for two freelancers and requests a link once, when both issuances complete, then he receives two separate emails, each with that freelancer's own branding and its own token.

**FEAT-05.SPEC-008-AC-05:** Given delivery of Priya's sign-in email fails on the first attempt, when the delivery capability retries, then it retries up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window` before giving up.

**FEAT-05.SPEC-008-AC-06:** Given all retries for Owen's sign-in email are exhausted, when the final attempt fails, then Nadia sees a delivery warning on the client's contact record, since Owen has no other way to be reached.

**FEAT-05.SPEC-008-AC-07:** Given Priya requests a fresh link while a prior email is still in transit, when the prior email eventually delivers, then clicking its link shows "This link isn't valid anymore," since FEAT-05.SPEC-006 invalidated the prior token independently of this email's delivery.

**FEAT-05.SPEC-008-AC-08:** Given this notification has no preference control for the recipient, when a Client Contact requests a link, then the email always sends -- there is no opt-out surface to check.

**FEAT-05.SPEC-008-AC-09:** Given a link is requested at any hour, when the email is triggered, then it sends immediately with no quiet-hours hold, since the underlying credential is time-limited.

**FEAT-05.SPEC-008-AC-10:** Given the freelancer's Branding Profile has a logo and brand colour set, when this email renders, then it displays that branding alongside the "Made with Clientroom" referral mark, with the mark never overriding the branding.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (no preference -- always sends) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 4 | 4 |



# Automation Spec: Portal Record First-View Capture

## Overview

**Name:** Portal Record First-View Capture
**ID:** FEAT-05.SPEC-009
**Type:** Automation
**Purpose:** Detects a client contact's first view of a proposal, deliverable, or invoice reached through the portal and hands the timestamped event to FEAT-13's audit trail.
**Parent Feature:** FEAT-05 -- Client Portal Access (Magic-Link Login)

## Scope and Non-Goals

**In Scope:**
- Detecting the first time a portal session opens a proposal, deliverable, or invoice screen, whichever feature owns that screen
- Capturing the timestamp of that first view exactly once per record per contact
- Handing the timestamped event to FEAT-13, which writes the append-only trail entry
- Idempotency: a re-view of an already-first-viewed record produces no further event

**Non-Goals:**
- Writing the Activity Log Entry itself -- owned by FEAT-13; this automation only detects the moment and hands off the event.
- Displaying the resulting `first_client_view_at` field (e.g., on the Deliverable) -- owned by the record's own feature (FEAT-03, FEAT-06/FEAT-07, FEAT-09/FEAT-10), which exposes the field this automation's event ultimately feeds.
- Detecting views that happen outside a portal session (e.g., the freelancer's own view of her sent proposal) -- this automation is scoped to the portal session FEAT-05 owns; a freelancer's own screens have no client "first view" to detect.
- Any retention or purge decision for first-view data -- excluded per this feature's own Non-Goals: `last_sign_in` and issued-link data carry no retention decision of this feature's own, and first-view events are FEAT-13's evidentiary record, governed by FEAT-13's own retention (life of the account, per the dependency map).

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Contact opens the Proposal Review & Accept screen | FEAT-03 (Proposal Acceptance) | Fires when a portal session opens that screen for a specific Proposal | Client Contact reference, Proposal reference, current timestamp |
| Contact opens a deliverable review screen | FEAT-06 (Deliverable Upload & Sharing) / FEAT-07 (Deliverable Review & Feedback) | Fires when a portal session opens that screen for a specific Deliverable | Client Contact reference, Deliverable reference, current timestamp |
| Contact opens the invoice list or pay-invoice screen | FEAT-09 (Invoice Generation & Sending) / FEAT-10 (Invoice Payment Processing) | Fires when a portal session opens that screen for a specific Invoice | Client Contact reference, Invoice reference, current timestamp |

## Processing Logic

1. Receive the opening event from the triggering screen: which portal session opened it, which record (Proposal, Deliverable, or Invoice) it opened, and the current timestamp.
2. Check whether a first-view event already exists for that specific record and that specific Client Contact.
3. If a first-view event already exists, take no further action (No-Action outcome).
4. If no first-view event exists for that record/contact pair, capture the current timestamp as the first-view moment.
5. Hand the timestamped event to FEAT-13 (XBR-05): event_type "first client view," actor the Client Contact, occurred_at the captured timestamp, affected_record the specific Proposal, Deliverable, or Invoice, project the record's owning project.
6. The record's own owning feature (FEAT-03, FEAT-06/FEAT-07, or FEAT-09/FEAT-10) reads this event to populate its own first-view field (e.g., Deliverable's `first_client_view_at`) through its own data flow -- this automation's responsibility ends at the FEAT-13 hand-off.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| First view captured | No prior first-view event exists for this record/contact pair | A timestamped first-view event is created and handed to FEAT-13 | None visible to the contact -- the capture is silent and does not alter the screen they are viewing | FEAT-13 (trail entry); the owning record's feature (FEAT-03, FEAT-06/FEAT-07, FEAT-09/FEAT-10), which later exposes the resulting field |
| No action (already viewed) | A first-view event already exists for this record/contact pair | None | None visible -- identical to a normal view of an already-viewed record | None beyond the triggering screen, which renders normally either way |
| Failure (hand-off to FEAT-13 fails) | The event hand-off cannot be delivered | No first-view event is durably recorded | None visible to the contact -- the screen renders normally regardless of the trail write's outcome, since a client contact must never see the platform's internal audit mechanics | FEAT-13 (retried per its own delivery guarantee) |

## Data Model

**Reads:** Client Contact -- identity of the portal session's contact; Proposal, Deliverable, or Invoice -- the reference to the record being viewed and its `first_client_view_at`-equivalent state (to check whether a first view already exists).
**Creates:** First-view event (feature-internal, not a Domain Entity Inventory record) -- one per record per contact, handed to FEAT-13.
**Updates:** None directly by this automation -- the owning record's `first_client_view_at`-equivalent field (e.g., Deliverable's `first_client_view_at`) is written by that record's own feature (FEAT-06) once it consumes this automation's event, per the Entity-Lifecycle Coverage Matrix's note that this feature "never writes directly" to that field.
**Deletes:** None.

## Business Rules

- XBR-05: this first-view event is one of the record-worthy events that always writes an append-only trail entry with actor and timestamp; it is owned by FEAT-13, this automation is the source that detects and hands it off.
- Idempotency: at most one first-view event exists per record per contact, ever -- a contact re-viewing the same record any number of times after the first produces no further event, no matter how the feature's own screen re-renders on subsequent visits.
- This automation runs on behalf of whichever feature's screen the client contact opened (FEAT-03, FEAT-06, FEAT-07, FEAT-09, or FEAT-10); it is owned by FEAT-05 because first view is inherently a property of the portal session FEAT-05 owns, not of the record's own feature, per the Feature Breakdown Brief's Discovery Rationale.
- The evidence this event produces is what the "Pointing to the Record in a Scope Dispute" journey relies on to settle a dispute; the timestamp captured is therefore the moment the record's screen opened within the portal session, not any later moment (e.g., scrolling, dwelling, or closing the screen).

## Edge Cases

- **Contact opens the same record twice in the same session, moments apart** -- The second open finds the first-view event already exists (step 2) and takes No-Action; only the first open's timestamp is ever recorded.
- **Concurrent trigger firing (the same contact opens the same record from two devices or tabs at effectively the same time)** -- Whichever open reaches step 2 first captures the first-view event; the second finds it already exists and takes No-Action. At most one first-view event per record per contact is ever created, regardless of how many near-simultaneous opens occur.
- **Trigger fires while a previous run for the same record/contact pair is still in flight** -- The existence check (step 2) and the event creation (steps 4-5) are treated as a single atomic step for a given record/contact pair, so a second trigger arriving before the first completes waits for that step to resolve and then correctly finds the event already captured.
- **Two different contacts (Owen and Priya) open the same Deliverable** -- Each contact's first view is tracked independently, since the first-view event is scoped to a record/contact pair, not to the record alone; both Owen's and Priya's first views are captured separately.
- **The underlying record is removed or voided between a prior first view and a later re-view attempt** -- The idempotency check in step 2 still finds the existing first-view event for that record/contact pair (the event is immutable and never deleted alongside the record, consistent with FEAT-13's Activity Log Entry never being edited or deleted except by FEAT-24 account deletion); no new event is captured, since a first view, once recorded, is a historical fact independent of the record's current state.
- **Hand-off to FEAT-13 fails after the timestamp is captured** -- The screen the contact is viewing renders normally regardless; FEAT-13's own delivery guarantee retries the hand-off, and until it succeeds, a subsequent open of the same record is treated as a new candidate for first-view capture only if FEAT-13 confirms no entry was ultimately recorded, preventing a permanently lost first-view record.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-03 (Proposal Acceptance) | Triggered by (inbound) | Opening the Proposal Review & Accept screen fires this automation |
| FEAT-06 (Deliverable Upload & Sharing) / FEAT-07 (Deliverable Review & Feedback) | Triggered by (inbound) | Opening a deliverable review screen fires this automation |
| FEAT-09 (Invoice Generation & Sending) / FEAT-10 (Invoice Payment Processing) | Triggered by (inbound) | Opening the invoice list or pay-invoice screen fires this automation |
| FEAT-13 (Immutable Activity & Audit Trail) | Affects (outbound) | Receives the timestamped first-view event and writes the append-only trail entry (XBR-05) |
| FEAT-05.SPEC-003 (Portal Home) | Triggered by (inbound) | Navigating from a Portal Home waiting item into a proposal, deliverable, or invoice screen is the entry path that leads to this automation's trigger |

## Analytics and Success Signals

- **first_view_captured** (record_type: proposal / deliverable / invoice) -- N/A -- no Stage 2 metric in success-metrics.md is connected to this feature's first-view detection specifically; the two metrics connected to FEAT-05 (Client Portal Login Success, Client Portal Mobile Responsiveness) measure the sign-in flow and page responsiveness, not record-viewing behavior. This event is retained because it is the operational signal that the evidentiary hand-off to FEAT-13 (relied on by the scope-dispute journey) is firing as expected.

## Acceptance Criteria

**FEAT-05.SPEC-009-AC-01:** Given Owen opens a proposal waiting on him for the first time through his portal session, when the screen opens, then a first-view event is captured and handed to FEAT-13 with his identity, the timestamp, and the proposal reference.

**FEAT-05.SPEC-009-AC-02:** Given Priya has already had a first-view event captured for a specific deliverable, when she opens that same deliverable again, then no new event is captured.

**FEAT-05.SPEC-009-AC-03:** Given Owen opens the same invoice twice in quick succession from two open tabs, when both opens are processed, then exactly one first-view event exists for that invoice and Owen's contact record.

**FEAT-05.SPEC-009-AC-04:** Given both Owen and Priya open the same deliverable for the first time, each from their own portal session, when both views occur, then two separate first-view events are captured, one per contact.

**FEAT-05.SPEC-009-AC-05:** Given the hand-off to FEAT-13 fails after Owen's first view of an invoice is captured, when the failure occurs, then Owen's invoice screen still renders normally, and FEAT-13's own retry eventually completes the trail write.

**FEAT-05.SPEC-009-AC-06:** Given a deliverable is later superseded by a new version after Priya's first view of the original was captured, when she opens the superseded version again, then no new first-view event is captured, since her first-view event for that deliverable already exists.

**FEAT-05.SPEC-009-AC-07:** Given Owen opens a proposal through the portal for the first time, when the first-view event is captured, then the timestamp recorded is the moment the screen opened, not any later moment such as scrolling or closing the screen.

**FEAT-05.SPEC-009-AC-08:** Given this automation fires from FEAT-03, FEAT-06, FEAT-07, FEAT-09, or FEAT-10's own screens, when any of those triggers fire, then the same detection and idempotency logic applies uniformly regardless of which feature's screen triggered it.

**FEAT-05.SPEC-009-AC-09:** Given Owen's first-view event for a proposal is already recorded, when he later re-opens that proposal after it has been voided and re-sent as a new Proposal record (FEAT-02's void-and-resend), then the new Proposal record is a distinct record from the voided one, so his open of the new record is evaluated as its own first-view candidate, independent of the voided proposal's already-captured event.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 | 3 |
| Outcome Paths | 3 | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |

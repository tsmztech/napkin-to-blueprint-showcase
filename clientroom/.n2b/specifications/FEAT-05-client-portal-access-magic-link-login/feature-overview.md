---
document_type: feature-overview
feature_number: FEAT-05
feature_name: Client Portal Access (Magic-Link Login)
feature_slug: client-portal-access-magic-link-login
priority_tier: Core
feature_type: Platform
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 9
screen_count: 3
automation_count: 3
logic_rule_count: 2
integration_count: 0
notification_count: 1
---

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
